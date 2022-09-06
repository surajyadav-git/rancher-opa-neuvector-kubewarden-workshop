# LAB-03A - Install Kubewarden and configure policy server 



In this exercise we will add helm chart and install kubewarden . The Kubewarden stack is made of the following components:

- An arbitrary number of `ClusterAdmissionPolicy` resources: this is how policies are defined inside Kubernetes
- An arbitrary number of `PolicyServer` resources: this component represents a Deployment of a Kubewarden `PolicyServer`. The policies defined by the administrators are loaded and evaluated by the Kubewarden `PolicyServer`
- A Deployment of `kubewarden-controller`: this is the controller that monitors the `ClusterAdmissionPolicy` resources and interacts with the Kubewarden `PolicyServer` components

In order to create Policies we will have to install kubewarden-crds , kubewarden-controller and kubewarden-defaults 

### Step 3A.1 ) Installation of Cert-manager

Kubewarden chart depends on cert-manager . Since it is a dependency we will have to first install cert-manager . 

To Install latest version of `cert-manager`, on Rancher server UI click on left most corner near Rancher logo ->Home -> rke2-cluster1 -> Kubectl icon  !

![](images/pic1.png)

Run below commands in Kubectl shell  :

```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/latest/download/cert-manager.yaml
```

You should see an output similar to below screen-shot , 

![](images/pic2.png)



```
kubectl wait --for=condition=Available deployment --timeout=2m -n cert-manager --all
```

You should see an output similar to below screen-shot , 

![](images/pic3.png)

Now we have successfully deployed Certmanager in our cluster . The next step would be to install kubewarden stack .  

### Step 3A.2 ) Deploy Kubewarden stack 

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



![](images/pic4.png)

Now we have deployed Kubewarden stack . Next step is to deploy policy server .



### Step 3A.3 ) Deploy Policy server 

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

![](images/pic5.png)

Now we have successfully deployed Policy server . 



**End of Exercise 1A** 

Continue to: 

[Exercise-03B-EnforceAdmissionPolicy](https://github.com/dsohk/rancher-opa-neuvector-kubewarden-workshop/blob/main/Exercise-03B-EnforceAdmissionPolicy.md)



