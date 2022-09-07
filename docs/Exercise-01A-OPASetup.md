# Lab 1 A - Installing OPA, defining Template and setting up OPA constraint

This exercise is independent. We'll experience ease of installing Open Policy Agent (OPA) from Rancher cluster tools and quickly setup the constaint to allow run only non-root containers. 

###### Usecase: Generally docker containers run with root privileges which is useful for the developers because of unrestricted container management e.g. installing packages, changing configurations, associating privileged ports etc. But running containers as root in QA and Production isn't a good security practice and can cause malicious process to gain non-federated access which in turn be catastrophic. Using OPA, we can create such policy which restrict cluster to run container as root user.     



## Task 1: Install Open Policy Agent (OPA) 

This action will involve 2 steps from 2B.1 to 2B.2

**Step 1A.1)** Ensure that you are logged in to the Rancher (Refer to the Step 1 of section **Before We Begin**). Click on `Hamburger Menu icon (1)`. It will show list of clusters. Alternatively click on `Cluster Management (2)` .

![Screenshot-2022-07-24-at-6.19.44-PM](../images/Screenshot-2022-07-24-at-6.19.44-PM.png)

**Step 1A.2)** Click on `Explore` button on downstream cluster `rke2-cluster1` from the main screen.

![Screenshot-2022-07-24-at-6.21.26-PM](../images/Screenshot-2022-07-24-at-6.21.26-PM.png)

**Step 1A.3)** Now click on `Cluster Tools` on the botton of the left hand side static menu.  

![Screenshot-2022-07-24-at-6.22.46-PM](../images/Screenshot-2022-07-24-at-6.22.46-PM.png)



**Step 1A.4)** Identify **OPA Gatekeeper** card and click on `Install` button. 

![Screenshot-2022-07-24-at-6.23.37-PM](../images/Screenshot-2022-07-24-at-6.23.37-PM.png)



**Step 1A.5)** Choose `Default` value from **Install into Project** and click on `Next` button. 

![Screenshot-2022-07-24-at-6.24.51-PM](../images/Screenshot-2022-07-24-at-6.24.51-PM.png)



**Step 1A.6)** Now click on `Install` button. 

![Screenshot-2022-07-24-at-6.26.02-PM](../images/Screenshot-2022-07-24-at-6.26.02-PM.png)



**Step 1A.7)** Verify that installation is successfully completed and SUCCESS message is displayed. Close the logging screen by clicking on "x" button.  

![Screenshot-2022-07-24-at-6.27.26-PM](../images/Screenshot-2022-07-24-at-6.27.26-PM.png)



**End of Task 1**



## Task 2: Define OPA Template

This action will involve 4 steps from 1A.8 to 1A.11

**Step 1A.8)** Go to `OPA Gatekeeper` menu option and select `Template`. Now click on `Create from YAML` at the right upper screen. 

![Screenshot-2022-09-05-at-9.11.20-PM](../images/Screenshot-2022-09-05-at-9.11.20-PM.png)

**Step 1A.9)** Remove pre-filled text from the template editor. 

![Screenshot-2022-09-05-at-9.18.34-PM](../images/Screenshot-2022-09-05-at-9.18.34-PM.png)

**Step 1A.10)** Specify template.yaml file content in the editor. Please refer below git link (Please open link in new tab)

[OPA-User-Template](/template.yaml)

![Screenshot-2022-09-05-at-9.25.25-PM](../images/Screenshot-2022-09-05-at-9.25.25-PM.png)

**Step 1A.11)** Verify that template `k8spspallowedusers` is successfully create. 

![Screenshot-2022-09-05-at-9.26.42-PM](../images/Screenshot-2022-09-05-at-9.26.42-PM.png)

**End of Action 3**

## Action 3: Setup OPA Constraint

This action will involve 6 steps from 1A.12 to 1A.17

**Step 1A.12)** A new **OPA Gatekeeper** menu will appear in left side static menu. Expand menu and click on `Constratint` submenu. Click on `Create` to proceed. 

![Screenshot-2022-07-24-at-6.29.03-PM](../images/Screenshot-2022-07-24-at-6.29.03-PM.png)



**Step 1A.13)** Select `k8spspallowedusers` from the screen. 

![Screenshot-2022-09-05-at-9.29.17-PM](../images/Screenshot-2022-09-05-at-9.29.17-PM.png)

**Step 14)** Fill in the following details and click on `Rules` link. 

`Name : user's choice e.g. allowed-users-100-200`

`Enforcement Action : deny-deny admission request with any violation (Default)`

`Scope : Namespaced (From the dropdown)`

`Excluded Namespaces : kube-system, cattle-* (Manually choose all "cattle-" namespces)`



![Screenshot-2022-09-05-at-9.30.24-PM](../images/Screenshot-2022-09-05-at-9.30.24-PM.png)

**Step 15)** Click on Add `Kind` button twice to create two rows for Kind values. Fill values `Pod` and `Deployment`. Now click on `Parameters` link.  

![Screenshot-2022-09-05-at-9.33.03-PM](../images/Screenshot-2022-09-05-at-9.33.03-PM.png)

**Step 16)** Specify following yaml values and click `Create`.  

```yaml
runAsUser:
      rule: MustRunAs # MustRunAsNonRoot # RunAsAny 
      ranges:
        - min: 100
          max: 200
```



![Screenshot-2022-09-05-at-9.34.05-PM](../images/Screenshot-2022-09-05-at-9.34.05-PM.png)

**Step 17)** Verify that constraint is successfully created. Now proceed to validate if the defined constraint is working as expected i.e. deployment pulling container image from ACR will be successful while Harbor will be failed. 

Now click on `Workload` option from the left hand static menu and select `Deployments` sub menu. 

![Screenshot-2022-09-05-at-9.37.38-PM](../images/Screenshot-2022-09-05-at-9.37.38-PM.png)

**End of Task 3**

## Task 4: Validate constraint - careating container runAs root user. 

This action will involve 5 steps from 1 to 5

**Step 1)** Make sure that you are at the Deployment screen in Rancher. Click on `Create`.

![Screenshot-2022-09-06-at-7.18.20-AM](/Users/surajyadav/OneDrive - SUSE Software Solutions/SUSE/workshops/azure/8-sep-2022/container-security/gitclone/rancher-opa-neuvector-kubewarden-workshop/images/Screenshot-2022-09-06-at-7.18.20-AM.png)



**Step 2)** Fill in following details

`Namespace : default`

`Name : user's choice e.g. opa-allowed-users-nok`



![Screenshot-2022-09-06-at-7.19.39-AM](/Users/surajyadav/OneDrive - SUSE Software Solutions/SUSE/workshops/azure/8-sep-2022/container-security/gitclone/rancher-opa-neuvector-kubewarden-workshop/images/Screenshot-2022-09-06-at-7.19.39-AM.png)

**Step 3)** Now navigate to the `Containers` tab. 

`Container Image : nginx:latest`

![Screenshot-2022-09-06-at-7.19.55-AM](/Users/surajyadav/OneDrive - SUSE Software Solutions/SUSE/workshops/azure/8-sep-2022/container-security/gitclone/rancher-opa-neuvector-kubewarden-workshop/images/Screenshot-2022-09-06-at-7.19.55-AM.png)

**Step 4)** Now scroll down and stop at `Security Context` to ensure the default value of `Run as Non-Root` to `No` means container will run as root user. 



![Screenshot-2022-09-06-at-7.20.06-AM](/Users/surajyadav/OneDrive - SUSE Software Solutions/SUSE/workshops/azure/8-sep-2022/container-security/gitclone/rancher-opa-neuvector-kubewarden-workshop/images/Screenshot-2022-09-06-at-7.20.06-AM.png)

**Step 5)** Click `Create` and verify that OPA constraint is stopping you to create the deployment. 

![Screenshot-2022-09-06-at-7.22.21-AM](../images/Screenshot-2022-09-06-at-7.22.21-AM.png)

**End of Task 4**

## Task 5: Validate deployment - careating container runAs non-root user. 

This action will involve 6 steps from 1 to 6

**Step 1)** That's Good! Now let us see if the container runs with non-root user e.g. 101 which is within the specified range. We have to use one such container which is build to run as non-root e.g. `nginxinc/nginx-unprivileged:latest`. 

![Screenshot-2022-09-06-at-7.25.12-AM](/Users/surajyadav/OneDrive - SUSE Software Solutions/SUSE/workshops/azure/8-sep-2022/container-security/gitclone/rancher-opa-neuvector-kubewarden-workshop/images/Screenshot-2022-09-06-at-7.25.12-AM.png)

**Step 2)** Create a fresh deployment with following details. 

`Namespace : default`

`Name : user's choice e.g. opa-allowed-users-ok`

Now navigate to the `Containers` tab. 

`Container Image : nginxinc/nginx-unprivileged:latest`

![Screenshot-2022-09-06-at-7.26.15-AM](/Users/surajyadav/OneDrive - SUSE Software Solutions/SUSE/workshops/azure/8-sep-2022/container-security/gitclone/rancher-opa-neuvector-kubewarden-workshop/images/Screenshot-2022-09-06-at-7.26.15-AM.png)

**Step 3)** Now scroll down and stop at `Security Context` to change the default value of `Run as Non-Root` to `Yes` means container will run as non-root user. Also specify the value of field `Run as UserID`  to `101`. 

![Screenshot-2022-09-06-at-7.27.47-AM](/Users/surajyadav/OneDrive - SUSE Software Solutions/SUSE/workshops/azure/8-sep-2022/container-security/gitclone/rancher-opa-neuvector-kubewarden-workshop/images/Screenshot-2022-09-06-at-7.27.47-AM.png)

**Step 4)** Now click `Create` and verify that deployment is created successfully. 

![Screenshot-2022-09-06-at-7.30.30-AM](/Users/surajyadav/OneDrive - SUSE Software Solutions/SUSE/workshops/azure/8-sep-2022/container-security/gitclone/rancher-opa-neuvector-kubewarden-workshop/images/Screenshot-2022-09-06-at-7.30.30-AM.png)

**Step 5)** Let's validate if the container is really running as non-root user i.e. 101. Click on the 3 vertical dots menu at the right end of the new created deployment and click on ` > Execute Shell`. It will open an inline window just below the Deployment section. 

![Screenshot-2022-09-06-at-7.31.41-AM](/Users/surajyadav/OneDrive - SUSE Software Solutions/SUSE/workshops/azure/8-sep-2022/container-security/gitclone/rancher-opa-neuvector-kubewarden-workshop/images/Screenshot-2022-09-06-at-7.31.41-AM.png)

**Step 6)** Execute command `id` in the shell and verify that `uid`, `gid`, and `groups` values are 101 as specified in the container. 

![Screenshot-2022-09-06 at-7.34.09-AM](../images/Screenshot-2022-09-06-at-7.34.09-AM.png)

Bingo! The OPA constraint is working as we were expecting. But wait... How much you are familiar with REGO language and how much effort it will require to create an OPA template if you are relatively newer in this space? Definitely, there are readymade templates available on git [gatekeepr-library](https://github.com/open-policy-agent/gatekeeper-library) which you may explore. However, there are good alternatives for advance policy control like `ARP Spoofing` etc. Let's conclude it here the and move on to the next exercise to explore Kubewarden. 

**End of Task 5**

**End of LAB 1**

Continue to: [Lab 2: Admission Control - Kubewarden](/docs/Lab02-Kubewarden.md)



