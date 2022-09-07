# Lab-02 - Kubewarden - ClusterAdmissionPolicy  



Kubewarden is a policy engine for Kubernetes. Its mission is to simplify the adoption of policy-as-code . Since PodSecurityPolicy (PSP) is being deprecated in Kubernetes 1.21, you can use Kubewarden as a replacement to PSP policies . 



In this lab we will be performing following tasks, 

Task 1 : Install Kubewarden

Task 2 : Deploy a sample pod and check NET_RAW capabilities is inherited by default

Task 3 : Enforce Admission control policy to disallow NET_RAW capability for any pods in the default namespace

Task 4 : Redeploy the same sample pod and check NET_RAW capabilities is not available anymore in that pod.



## Task 1 : Install Kubewarden

In this exercise we will add helm chart and install kubewarden. The Kubewarden stack is made of the following components:

- An arbitrary number of `ClusterAdmissionPolicy` resources: this is how policies are defined inside Kubernetes
- An arbitrary number of `PolicyServer` resources: this component represents a Deployment of a Kubewarden `PolicyServer`. The policies defined by the administrators are loaded and evaluated by the Kubewarden `PolicyServer`
- A Deployment of `kubewarden-controller`: this is the controller that monitors the `ClusterAdmissionPolicy` resources and interacts with the Kubewarden `PolicyServer` components

In order to create Policies we will have to install kubewarden-crds , kubewarden-controller and kubewarden-defaults 

#### Step 1 ) Installation of Cert-manager

Kubewarden chart depends on cert-manager . Since it is a dependency we will have to first install cert-manager . 

To Install latest version of `cert-manager`, 

1. Login to your Rancher Server instance with your given credential. 
2. From the left-side menu, click the downstream cluster `rke2-cluster` to get into the **Cluster Explorer** page.
3. At this **Cluster Explorer** page, at the top icon menu bar. click the "> _" icon to open the web-based terminal shell.



![](../images/pic1.png)

Run below commands in Kubectl shell  :

```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/latest/download/cert-manager.yaml
```

You should see an output similar to below screen-shot , 

![](../images/pic2.png)



```
kubectl wait --for=condition=Available deployment --timeout=2m -n cert-manager --all
```

You should see an output similar to below screen-shot , 

![](../images/pic3.png)

Now we have successfully deployed Certmanager in our cluster . The next step would be to install kubewarden stack .  

#### Step 2 ) Deploy Kubewarden stack 

The following charts should be installed inside the `kubewarden` namespace in your Kubernetes cluster:

- `kubewarden-crds`, which will register the `ClusterAdmissionPolicy` and `PolicyServer` Custom Resource Definitions
- `kubewarden-controller`, which will install the Kubewarden controller
- `kubewarden-defaults`, which will create a `PolicyServer` resource named `default`. It can also installs a set of recommended policies to secure your cluster by enforcing some well known best practices

Open Kubectl shell . Add kubewarden helm chart using below command ,

```console
helm repo add kubewarden https://charts.kubewarden.io
```

Kubewarden stack can be deployed from above helm chart . Copy paste below commands in kubectl shell ,

```bash
helm install --wait -n kubewarden --create-namespace kubewarden-crds kubewarden/kubewarden-crds

helm install --wait -n kubewarden kubewarden-controller kubewarden/kubewarden-controller

helm install --wait -n kubewarden kubewarden-defaults kubewarden/kubewarden-defaults

```

Wait until you see an output similar to below screen-shot , 

![](../images/pic4.png)

Now we have deployed Kubewarden stack . Next step is to deploy policy server .



## Task 2: Deploy a sample pod and check NET_RAW capabilities is inherited by default



1. apply the sample pod manifest:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubewarden-test
  labels:
    app: kubewarden-test
spec:
  replicas: 1
  strategy: 
    type: RollingUpdate
  selector:
    matchLabels:
      app: kubewarden-test
  template:
    metadata:
      labels:
        app: kubewarden-test
    spec:
      containers:
      - name: kubewarden-test
        image: registry.suse.com/suse/sle15:latest
        imagePullPolicy: IfNotPresent
        command: ['sh', '-c', 'echo Container 1 is Running ; sleep 3600']
```

wait until the deployment completed.

then, execute shell into the pod

run this command to check the inherited linux capabilities:

```bash
zypper install -y libcap-progs iftop
capsh --decode=$( cat /proc/$$/status | grep CapEff | cut -d : -f 2 | xargs ) | GREP_COLOR='01;31' grep --color=auto net_raw
```

expected output:

```bash
kubewarden-test1-75bc67757b-lkv9g:/ # capsh --decode=$( cat /proc/$$/status | grep CapEff | cut -d : -f 2 | xargs ) | GREP_COLOR='01;31' grep --color=auto net_raw
0x00000000a80425fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
kubewarden-test1-75bc67757b-lkv9g:/ # 
```



We oberved the CAP_NET_RAW linux capabiliteis exists even if the pod manfiest does not do anything requesting it in the security context configuration of its deployment.

 Also, the network utility `iftop`, which requires NET_RAW capability, should be runnable inside this pod.

run this command: `iftop`

expected output

![image-20220907202039308](../images/kubewarden-iftop-output.png)

## Task 3 : Enforce Admission Control Policy to drop NET_RAW capabilities

Once you have the Kubewarden instance running, it is time to deploy some policies to replace the `PodSecurityPolicy` object . The `ClusterAdmissionPolicy` resource is the core of the Kubewarden stack. This resource defines how policies evaluate requests.

Enforcing policies is the most common operation which a Kubernetes administrator  will perform. You can declare as many policies as you want, and each  policy will target one or more specific Kubernetes resources (i.e., `pods`, `Custom Resource`). You will also specify the type of operation(s) that will be applied for the targeted resource(s). The operations available are `CREATE`, `UPDATE`, `DELETE` and `CONNECT`.

Kubernetes by default **connects** all the **containers running in the same node** (even if they belong to different namespaces) down to **Layer 2** (ethernet). This allows a malicious containers to perform an [**ARP spoofing attack**](https://github.com/carlospolop/hacktricks/blob/master/generic-methodologies-and-resources/pentesting-network/#arp-spoofing) to the containers on the same node and capture their traffic. 

In order to avoid such ARP spoofing attack it is important , not to allow `NET_RAW` capability .  The Kubewarden Policy `psp-capabilities` controls Container Capabilities . In below example you can see `NET_RAW` capability under `required_drop_capabilities` section . These are capabilities which must be dropped from containers and are removed from the default set  . 

Create a yaml file `clusteradmissionpolicy.yaml` with below content and save it . 

```yaml
apiVersion: policies.kubewarden.io/v1alpha2
kind: AdmissionPolicy
metadata:
  name: drop-cap-net-raw
  namespace: default
spec:
  policyServer: default
  module: registry://ghcr.io/kubewarden/policies/psp-capabilities:v0.1.7
  rules:
  - apiGroups: [""]
    apiVersions: ["v1"]
    resources:
    - pods
    - deployments
    operations:
    - CREATE
    - UPDATE
  mutating: true
  settings:
    required_drop_capabilities:
    - NET_RAW
```

Once deployed you should see an output similar to below screen-shot , 

![](../images/kubewarden-admission-policy-crd.png)





## Task 4 : Redeploy the same sample pod and check NET_RAW capabilities is not available anymore in that pod.



Redeploy the sample pod `kubewarden-test1`

![image-20220907200313670](../images/kubewarden-redeploy-testpod.png)

shell into the redeployed pod

run this command to check the inherited linux capabilities:

```bash
zypper install -y libcap-progs iftop
capsh --decode=$( cat /proc/$$/status | grep CapEff | cut -d : -f 2 | xargs ) | GREP_COLOR='01;31' grep --color=auto net_raw
```

expected output (NET_RAW capabilities is gone/dropped in the pod, because of the enforcement by the admission policy in Kubewarden)

```bash
kubewarden-test1-5b76ccf5c4-mbjkz:/ # capsh --decode=$( cat /proc/$$/status | grep CapEff | cut -d : -f 2 | xargs ) | GREP_COLOR='01;31' grep --color=auto net_raw
kubewarden-test1-5b76ccf5c4-mbjkz:/ # capsh --decode=$( cat /proc/$$/status | grep CapEff | cut -d : -f 2 | xargs )                                               
0x00000000a80405fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
kubewarden-test1-5b76ccf5c4-mbjkz:/ # 
```

Also, the `iftop` utility is no longer runnable in this pod because of the lack of NET_RAW capability.

```bash
kubewarden-test1-5b76ccf5c4-mbjkz:/ # iftop
interface: eth0
IP address is: 10.42.176.73
MAC address is: e6:c7:47:9a:b4:0e
pcap_open_live(eth0): eth0: You don't have permission to capture on that device (socket: Operation not permitted)
```



Continue to: [Lab03-NeuVector-Admission Control](https://github.com/dsohk/rancher-opa-neuvector-kubewarden-workshop/blob/main/docs/Lab03-NeuVector-Admission-Control.md)

