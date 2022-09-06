# LAB-02 Kubewarden - ClusterAdmissionPolicy  



Kubewarden is a policy engine for Kubernetes. Its mission is to simplify the adoption of policy-as-code . Since PodSecurityPolicy (PSP) is being deprecated in Kubernetes 1.21, you can use Kubewarden as a replacement to PSP policies . 



In this lab we will be performing following tasks , 

Task 1 : Install Kubewarden stack and deploy policy engine 

Task 2 : Enforce Admission control policy 



## Task 1 : Install Kubewarden stack and deploy policy engine 

In this exercise we will add helm chart and install kubewarden . The Kubewarden stack is made of the following components:

- An arbitrary number of `ClusterAdmissionPolicy` resources: this is how policies are defined inside Kubernetes
- An arbitrary number of `PolicyServer` resources: this component represents a Deployment of a Kubewarden `PolicyServer`. The policies defined by the administrators are loaded and evaluated by the Kubewarden `PolicyServer`
- A Deployment of `kubewarden-controller`: this is the controller that monitors the `ClusterAdmissionPolicy` resources and interacts with the Kubewarden `PolicyServer` components

In order to create Policies we will have to install kubewarden-crds , kubewarden-controller and kubewarden-defaults 

#### Step 1 ) Installation of Cert-manager

Kubewarden chart depends on cert-manager . Since it is a dependency we will have to first install cert-manager . 

To Install latest version of `cert-manager`, on Rancher server UI click on left most corner near Rancher logo ->Home -> rke2-cluster1 -> Kubectl icon  !

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

```
helm install --wait -n kubewarden --create-namespace kubewarden-crds kubewarden/kubewarden-crds

helm install --wait -n kubewarden kubewarden-controller kubewarden/kubewarden-controller

helm install --wait -n kubewarden kubewarden-defaults kubewarden/kubewarden-defaults


```

Wait until you see an output similar to below screen-shot , 

![](../images/pic4.png)

Now we have deployed Kubewarden stack . Next step is to deploy policy server .

#### Step 3 ) Deploy Policy server 

A Kubewarden `PolicyServer` is completely managed by the `kubewarden-controller`. Multiple `PolicyServers` can be deployed in the same Kubernetes cluster.

The `PolicyServer` is the component which executes the Kubewarden policies when requests arrive and validates them. To deploy PolicyServer , on Rancher server UI click on left most corner near Rancher logo  -> Home -> rke2-cluster1 -> Kubectl icon . 

Create a yaml file `policyserver.yaml` with below content and save it . 

```
apiVersion: policies.kubewarden.io/v1alpha2
kind: PolicyServer
metadata:
  name: reserved-instance-for-tenant-a
spec:
  image: ghcr.io/kubewarden/policy-server:v1.1.2
  replicas: 2
  serviceAccountName: ~
  env:
  - name: KUBEWARDEN_LOG_LEVEL
    value: debug
```

Now deploy `policyserver.yaml` file using below command , 

```
kubectl apply -f policyserver.yaml
```

You should see an output similar to below screen-shot ,

![](../images/pic5.png)

Now we have successfully deployed Policy server . 

**End of Task1 



## Task 2 : Enforce Admission Control Policy 

Once you have the Kubewarden instance running, it is time to deploy some policies to replace the `PodSecurityPolicy` object . The `ClusterAdmissionPolicy` resource is the core of the Kubewarden stack. This resource defines how policies evaluate requests.

Enforcing policies is the most common operation which a Kubernetes administrator  will perform. You can declare as many policies as you want, and each  policy will target one or more specific Kubernetes resources (i.e., `pods`, `Custom Resource`). You will also specify the type of operation(s) that will be applied for the targeted resource(s). The operations available are `CREATE`, `UPDATE`, `DELETE` and `CONNECT`.

Kubernetes by default **connects** all the **containers running in the same node** (even if they belong to different namespaces) down to **Layer 2** (ethernet). This allows a malicious containers to perform an [**ARP spoofing attack**](https://github.com/carlospolop/hacktricks/blob/master/generic-methodologies-and-resources/pentesting-network/#arp-spoofing) to the containers on the same node and capture their traffic. 

In order to avoid such ARP spoofing attack it is important , not to allow `NET_RAW` capability .  The Kubewarden Policy `psp-capabilities` controls Container Capabilities . In below example you can see `NET_RAW` capability under `required_drop_capabilities` section . These are capabilities which must be dropped from containers and are removed from the default set  . 

Create a yaml file `clusteradmissionpolicy.yaml` with below content and save it . 

```
apiVersion: policies.kubewarden.io/v1alpha2
kind: ClusterAdmissionPolicy
metadata:
  name: psp-capabilities
spec:
  policyServer: reserved-instance-for-tenant-a
  module: registry://ghcr.io/kubewarden/policies/psp-capabilities:v0.1.7
  rules:
  - apiGroups: [""]
    apiVersions: ["v1"]
    resources: ["pods"]
    operations:
    - CREATE
    - UPDATE
  mutating: true
  settings:
    allowed_capabilities:
    - CHOWN
    required_drop_capabilities:
    - NET_RAW
    - NET_ADMIN
```

Once deployed you should see an output similar to below screen-shot , 

![](../images/pic5-166212546820724.png)

The following Pod would be rejected by the policy `psp-capabilities` : 

```
apiVersion: v1
kind: Pod
metadata:
  name: hello
spec:
  containers:
  - name: hello
    image: busybox
    command: [ "sh", "-c", "echo 'Hello!' && sleep 1h" ]
    securityContext:
      capabilities:
        add:
        - NET_RAW
```

Now let us go ahead and create our first policy . 

#### Step 1 ) Example : Blocking pods running as root

For this first example, we will use the  [user-group-psp policy](https://github.com/kubewarden/user-group-psp-policy)  .  Our goal will be to prevent the pods running as root on our Kubernetes cluster by enforcing this policy.  Let's define a `ClusterAdmissionPolicy` for that:

Deploy below ClusterAdmissionPolicy , by copy paste kubectl shell ,

```
kubectl apply -f - <<EOF
apiVersion: policies.kubewarden.io/v1alpha2
kind: ClusterAdmissionPolicy
metadata:
  name: psp-usergroup
spec:
  module: registry://ghcr.io/kubewarden/policies/user-group-psp:latest
  rules:
    - apiGroups:
        - ""
      apiVersions:
        - v1
      resources:
        - pods
      operations:
        - CREATE
        - UPDATE
  mutating: true
  settings:
    run_as_user:
      rule: MustRunAsNonRoot
    supplemental_groups:
      rule: MustRunAs
      ranges:
        - min: 1000
          max: 65535
EOF

```

Notice the policy is configured as `mutation: true`. This is required because the policy will add [supplementalGroups](https://kubernetes.io/docs/concepts/security/pod-security-policy/#users-and-groups) when the user does not define them.

This will produce the following output:

![](../images/pic7.png)

When a  `ClusterAdmissionPolicy` is defined, the status is set to `pending`, and it will force a rollout of the targeted `PolicyServer`. You can monitor the rollout by running the following command:

```
kubectl get clusteradmissionpolicy.policies.kubewarden.io/psp-usergroup
```

![](../images/pic8.png)

The `ClusterAdmissionPolicy` status will be set to active once the deployment is done for every `PolicyServer` instance. 

You can test the policy now . Copy paste below section on kubectl shell . Users should not be able to deploy pods running as root:

```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    securityContext:
      runAsNonRoot: false
      runAsUser: 0
EOF

```

```
Error from server: error when creating "STDIN": admission webhook "clusterwide-psp-usergroup-fb836.kubewarden.admission" denied the request: RunAsNonRoot should be set to true
```

You can see the Error states the pods cannot be deployed since the Kubewarden policy denied the request to run as root user . Let us see another example where the  parameter is set`runAsNonRoot: true`  but still denied  because the user ID is still pointing to root ie , UID 0 

```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    securityContext:
      runAsNonRoot: true
      runAsUser: 0
EOF
```

```
Error from server: error when creating "STDIN": admission webhook "clusterwide-psp-usergroup-fb836.kubewarden.admission" denied the request: Invalid user ID: cannot run container with root ID (0)
```

------

#### Step 2 ) Example : Allowing pod to use the port 443 only (Optional)

To replace the PSP configuration that blocks privileged containers, it's necessary to deploy the [Host Namespaces PSP](https://github.com/kubewarden/host-namespaces-psp-policy). This policy does not require any settings. Once running, it will block users to use ports other than 443 and ports between 5000 - 6000 . 



```
kubectl apply -f - <<EOF
apiVersion: policies.kubewarden.io/v1alpha2
kind: ClusterAdmissionPolicy
metadata:
  name: psp-hostnamespaces
spec:
  module: registry://ghcr.io/kubewarden/policies/host-namespaces-psp:v0.1.2
  rules:
    - apiGroups:
        - ""
      apiVersions:
        - v1
      resources:
        - pods
      operations:
        - CREATE
        - UPDATE
  mutating: false
  settings:
    allow_host_ipc: false
    allow_host_pid: false
    allow_host_ports:
      - min: 5000
        max: 6000
      - min: 443
        max: 443
    allow_host_network: false
EOF
```

Now the policy is created and enforced . The pod should be only able to expose the ports between 5000 and 6000 and port 443 . It should throw an  error when other port numbers are configured against the hostPort  section.

```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
    ports:
      - containerPort: 80
        hostPort: 80
  - name: sleeping-sidecar
    image: alpine
    command: ["sleep", "1h"]
EOF

```

```
Error from server: error when creating "STDIN": admission webhook "clusterwide-psp-hostnamespaces.kubewarden.admission" denied the request: Pod is using unallowed host ports in containers
```

Above example should work if 443 or a port range between 5000 to 6000 is used  . 

Continue to: [Lab03-NeuVector-Admission Control](https://github.com/dsohk/rancher-opa-neuvector-kubewarden-workshop/blob/main/docs/Lab03-NeuVector-Admission-Control.md)



**End of Exercise 03B .** 

