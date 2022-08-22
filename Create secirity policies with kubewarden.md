Kubewarden is a Kubernetes Dynamic Admission Controller that validates incoming requests using policies written in WebAssembly. 





`kubectl apply -f https://github.com/jetstack/cert-manager/releases/latest/download/cert-manager.yaml`

![WhatsApp Image 2019-09-06 at 3.52.42 PM](images/WhatsApp Image 2019-09-06 at 3.52.42 PM.jpeg)





```
apiVersion: policies.kubewarden.io/v1alpha2
kind: PolicyServer
metadata:
  name: reserved-instance-for-tenant-a
spec:
  image: ghcr.io/kubewarden/policy-server:v0.3.0
  replicas: 2
  serviceAccountName: ~
  env:
  - name: KUBEWARDEN_LOG_LEVEL
    value: debug
```



![image002](images/image002.jpg)



![Screenshot from 2021-12-09 11-34-14](images/Screenshot from 2021-12-09 11-34-14.png)

![hpc-reference-architecture](images/hpc-reference-architecture.png)





