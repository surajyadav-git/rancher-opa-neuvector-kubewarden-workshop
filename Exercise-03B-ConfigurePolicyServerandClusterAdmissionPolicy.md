# LAB-03B - Configure Policy server and Cluster admission Policy 

### 03B - 1) configure Policy server 

A Kubewarden `PolicyServer` is completely managed by the `kubewarden-controller`. Multiple `PolicyServers` can be deployed in the same Kubernetes cluster.

The `PolicyServer` is the component which executes the Kubewarden policies when requests arrive and validates them. To deploy PolicyServer , on Rancher server UI click on left most corner near Rancher logo  -> Home -> rke2-cluster1 -> Kubectl icon . ![](images/Rancher(3).png)

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

### 03B - 2) Configure Cluster Admission Policy 

The `ClusterAdmissionPolicy` resource is the core of the Kubewarden stack. This resource defines how policies evaluate requests.

Enforcing policies is the most common operation which a Kubernetes administrator  will perform. You can declare as many policies as you want, and each  policy will target one or more specific Kubernetes resources (i.e., `pods`, `Custom Resource`). You will also specify the type of operation(s) that will be applied for the targeted resource(s). The operations available are `CREATE`, `UPDATE`, `DELETE` and `CONNECT`.

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
    - NET_ADMIN
```

Once deployed you should see an output similar to below screen-shot , 

![](images/pic5-166212546820724.png)

In this exercise we have deployed Policyserver and default ClusterAdmissionPolicy  .  

**End of Exercise 03B** 

Continue to: 
