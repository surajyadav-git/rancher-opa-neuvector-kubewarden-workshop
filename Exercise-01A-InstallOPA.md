# Lab 3 A - Installing OPA and setting up OPA constraint on RKE2 cluster

This exercise is independent. We'll experience ease of installing Open Policy Agent (OPA) from Rancher cluster tools and quickly setup the constaint to allow only specific registries for deployment. 

###### Usecase: At a time development and operations may need seperate/multiple container registries and it imposes security challenges. The Open Policy Agent Gatekeeper project can be leveraged to help enforce policies and strengthen governance in your Kubernetes environment. Rancher provides OPA Gatekeeper straight from the Cluster Tools to install and setup policies within few clicks. 



## Action 1: Install Open Policy Agent (OPA) 

This action will involve 2 steps from 2B.1 to 2B.2

**Step 3A.1)** Ensure that you are logged in to the Rancher (Refer to the Step 1 of section **Before We Begin**). Click on `Hamburger Menu icon (1)`. It will show list of clusters. Alternatively click on `Cluster Management (2)` .

![Screenshot-2022-07-24-at-6.19.44-PM](../images/Screenshot-2022-07-24-at-6.19.44-PM.png)

**Step 3A.2)** Click on `Explore` button on downstream cluster `rke2-cluster1` from the main screen.

![Screenshot-2022-07-24-at-6.21.26-PM](../images/Screenshot-2022-07-24-at-6.21.26-PM.png)

**Step 3A.3)** Now click on `Cluster Tools` on the botton of the left hand side static menu.  

![Screenshot-2022-07-24-at-6.22.46-PM](../images/Screenshot-2022-07-24-at-6.22.46-PM.png)



**Step 3A.4)** Identify **OPA Gatekeeper** card and click on `Install` button. 

![Screenshot-2022-07-24-at-6.23.37-PM](../images/Screenshot-2022-07-24-at-6.23.37-PM.png)



**Step 3A.5)** Choose `Default` value from **Install into Project** and click on `Next` button. 

![Screenshot-2022-07-24-at-6.24.51-PM](../images/Screenshot-2022-07-24-at-6.24.51-PM.png)



**Step 3A.6)** Now click on `Install` button. 

![Screenshot-2022-07-24-at-6.26.02-PM](../images/Screenshot-2022-07-24-at-6.26.02-PM.png)



**Step 3A.7)** Verify that installation is successfully completed and SUCCESS message is displayed. Close the logging screen by clicking on "x" button.  

![Screenshot-2022-07-24-at-6.27.26-PM](../images/Screenshot-2022-07-24-at-6.27.26-PM.png)



**End of Action 1**



## Action 2: Setup OPA Constraint

This action will involve 6 steps from 3A.7 to 3A.11

**Step 3A.7)** A new **OPA Gatekeeper** menu will appear in left side static menu. Expand menu and click on `Constratint` submenu. Click on `Create` to proceed. 

![Screenshot-2022-07-24-at-6.29.03-PM](../images/Screenshot-2022-07-24-at-6.29.03-PM.png)



**Step 3A.8)** Select `k8sallowedrepos` from the screen. 

![Screenshot-2022-07-24-at-6.30.32-PM](../images/Screenshot-2022-07-24-at-6.30.32-PM.png)



**Step 3A.8)** Fill in the following details and click on `Rules` link. 

`Name : user's choice e.g. allowed-registries`

`Enforcement Action : deny-deny admission request with any violation (Default)`

`Scope : Namespaced (From the dropdown)`

`Excluded Namespaces : kube-system, cattle-* (Manually choose all "cattle-" namespces)`

![Screenshot-2022-07-24-at-6.31.38-PM](../images/Screenshot-2022-07-24-at-6.31.38-PM.png)



**Step 3A.9)** Click on Add `Kind` button twice to create two rows for Kinds values. Fill `Pod` and `Deployment`. Now click on `Parameters` link.  

![Screenshot-2022-07-24-at-6.33.57-PM](../images/Screenshot-2022-07-24-at-6.33.57-PM.png)



**Step 3A.10)** Specify following yaml values and click `Create`. Make sure that Harbor url is provided correctly as per your allocation e.g. `harbor.20.207.199.86.sslip.io`. In this scenario, Harbor is allowed while Docker Hub (Public Registry) doesn't exisit in allowed list and hence the deployment from docker hub will be blocked.  

```yaml
repos:
 - harbor.yy.yy.yy.yy.sslip.io
 - docker.io
 - registry.gitlab.com
```

![Screenshot-2022-07-25-at-9.14.25-PM](../images/Screenshot-2022-07-25-at-9.14.25-PM.png)

**Step 3A.11)** Verify that constraint is successfully created. Now proceed to validate if the defined constraint is working as expected i.e. deployment pulling container image from ACR will be successful while Harbor will be failed. 

Now click on `Workload (2)` option from the left hand static menu and select `Deployments` sub menu. 

![Screenshot-2022-07-24-at-6.39.17-PM](../images/Screenshot-2022-07-24-at-6.39.17-PM.png)



**End of Action 2**

**End of Exercise 3A**

Continue to: [Exercise 3B-Validate OPA Gatekeeper](https://github.com/dsohk/rancher-private-registry-workshop/blob/main/docs/Exercise-03B-TestOPAConstraintForHarborDocker.md)



