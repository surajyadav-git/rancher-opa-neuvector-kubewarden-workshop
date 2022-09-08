# Lab 1 - Open Policy Agent Gatekeeper

In this lab, we'll experience ease of installing Open Policy Agent (OPA) from Rancher cluster tools and quickly setup the constaint to restrict containers to runAs root. 

###### Usecase: Generally docker containers run with root privileges which is useful for the developers because of unrestricted container management e.g. installing packages, changing configurations, associating privileged ports etc. But running containers as root in QA and Production isn't a good security practice and can cause malicious process to gain non-federated access which in turn be catastrophic. Using OPA, we can create such policy which restrict cluster to run container as root user.     



## Task 1: Install Open Policy Agent (OPA) 

This action will involve 2 steps from 2B.1 to 2B.2

**Step 1)** Ensure that you are logged in to the Rancher (Refer to the Step 1 of section **Before We Begin**). Click on `Hamburger Menu icon (1)`. It will show list of clusters. Alternatively click on `Cluster Management (2)` .

![Screenshot-2022-07-24-at-6.19.44-PM](../images/Screenshot-2022-07-24-at-6.19.44-PM.png)

**Step 2)** Click on `Explore` button on downstream cluster `rke2-cluster1` from the main screen.

![Screenshot-2022-07-24-at-6.21.26-PM](../images/Screenshot-2022-07-24-at-6.21.26-PM.png)

**Step 3)** Now click on `Cluster Tools` on the botton of the left hand side static menu.  

![Screenshot-2022-07-24-at-6.22.46-PM](../images/Screenshot-2022-07-24-at-6.22.46-PM.png)



**Step 4)** Identify **OPA Gatekeeper** card and click on `Install` button. 

![Screenshot-2022-07-24-at-6.23.37-PM](../images/Screenshot-2022-07-24-at-6.23.37-PM.png)



**Step 5)** Choose `Default` value from **Install into Project** and click on `Next` button. 

![Screenshot-2022-07-24-at-6.24.51-PM](../images/Screenshot-2022-07-24-at-6.24.51-PM.png)



**Step 6)** Now click on `Install` button. 

![Screenshot-2022-07-24-at-6.26.02-PM](../images/Screenshot-2022-07-24-at-6.26.02-PM.png)



**Step 7)** Verify that installation is successfully completed and SUCCESS message is displayed. Close the logging screen by clicking on "x" button.  

![Screenshot-2022-07-24-at-6.27.26-PM](../images/Screenshot-2022-07-24-at-6.27.26-PM.png)



**End of Task 1**



## Task 2: Define OPA Template

This action will involve 4 steps from 1 to 4

**Step 1)** Go to `OPA Gatekeeper` menu option and select `Template`. Now click on `Create from YAML` at the right upper screen. 

![Screenshot-2022-09-05-at-9.11.20-PM](../images/Screenshot-2022-09-05-at-9.11.20-PM.png)

**Step 2)** Remove pre-filled text from the template editor. 

![Screenshot-2022-09-05-at-9.18.34-PM](../images/Screenshot-2022-09-05-at-9.18.34-PM.png)

**Step 3)** Copy [template.yaml](/template.yaml) file content into the OPA Template editor (Please open link in new tab and make sure to navigate back to the previous tab once content is copied). 

`Follow the below instruction to copy & paste template.yaml content`

Navigate to the [template.yaml](/template.yaml) url and click on `copy raw content` as shown in below screenshot. 

![Screenshot-2022-09-07-at-12.32.13-PM](../images/Screenshot-2022-09-07-at-12.32.13-PM.png)

Once the raw content is copied to the clipboard, this icon will turn into the green tick mark as shown in below screenshot. 

![Screenshot-2022-09-07-at-12.46.13-PM](../images/Screenshot-2022-09-07-at-12.46.13-PM.png)

Now, navigate back to the Rancher portal and paste the copied content into the OPA Template editor and click `Create`.  

![Screenshot-2022-09-05-at-9.25.25-PM](../images/Screenshot-2022-09-05-at-9.25.25-PM.png)

**Step 4)** Verify that template `k8spspallowedusers` is successfully create. 

![Screenshot-2022-09-05-at-9.26.42-PM](../images/Screenshot-2022-09-05-at-9.26.42-PM.png)

**End of Task 3**

## Task 4: Setup OPA Constraint

This action will involve 6 steps from 1 to 6

**Step 1)** Expand **OPA Gatekeeper** menu and click on `Constratints` submenu. Click on `Create` to proceed. 

![Screenshot-2022-07-24-at-6.29.03-PM](../images/Screenshot-2022-07-24-at-6.29.03-PM.png)



**Step 2)** Click on  `k8spspallowedusers` template on the screen. 

![Screenshot-2022-09-05-at-9.29.17-PM](../images/Screenshot-2022-09-05-at-9.29.17-PM.png)

**Step 3)** Fill the `Name` field of the Constraint and leave the field `Enforcement Action` to default vale `deny-deny` 

`Name : user's choice e.g. allowed-users-100-200`

`Enforcement Action : deny-deny admission request with any violation (Default)`

On the constraint creation screen, we have three tabs namely `Namespaces`, `Rules` and `Parameters`. Provide following details under the `Namespaces` tab.

`Scope : Cluster & Namespaced (From the dropdown)`

`Excluded Namespaces : kube-system, cattle-* (Manually choose all "cattle-" namespces)`

Now navigate to the `Rules` tab.

![Screenshot-2022-09-05-at-9.30.24-PM](../images/Screenshot-2022-09-05-at-9.30.24-PM.png)

**Step 4)** Click on Add `Kind` button twice to create two rows for Kind values. Fill values `Pod` and `Deployment`. Now click on `Parameters` link.  

![Screenshot-2022-09-05-at-9.33.03-PM](../images/Screenshot-2022-09-05-at-9.33.03-PM.png)

**Step 5)** Specify following yaml values and click `Create`.  

```yaml
runAsUser:
      rule: MustRunAs # MustRunAsNonRoot # RunAsAny 
      ranges:
        - min: 100
          max: 200
```



![Screenshot-2022-09-05-at-9.34.05-PM](../images/Screenshot-2022-09-05-at-9.34.05-PM.png)

Now click on `Workload` option from the left hand static menu and select `Deployments` sub menu. 

![Screenshot-2022-09-05-at-9.37.38-PM](../images/Screenshot-2022-09-05-at-9.37.38-PM.png)

**End of Task 4**

## Task 5: Validate constraint - Create deployment to run container as root user and our policy shouldn't allow to admit the container. 

This action will involve 5 steps from 1 to 5

**Step 1)** Make sure that you are at the Deployment screen in Rancher. Click on `Create`.

![Screenshot-2022-09-06-at-7.18.20-AM](../images/Screenshot-2022-09-06-at-7.18.20-AM.png)

**Step 2)** Deployment screen has 3 tabs `Deployment`, `Pod` and `Containers`. Fill in following details on the Deployment screen. 

`Namespace` : `default`

`Name` : user's choice e.g. `opa-allowed-users-nok`

Now navigate to the `Containers` tab. 

![Screenshot-2022-09-06-at-7.19.39-AM](../images/Screenshot-2022-09-06-at-7.19.39-AM.png)

**Step 3)** Fill following details on `Containers` tab. 

`Container Image : nginx:latest`

![Screenshot-2022-09-06-at-7.19.55-AM](../images/Screenshot-2022-09-06-at-7.19.55-AM.png)

**Step 4)** Now scroll down and stop at `Security Context` to ensure the default value of `Run as Non-Root` to `No`. It means container will run as root user. 

![Screenshot-2022-09-06-at-7.20.06-AM](../images/Screenshot-2022-09-06-at-7.20.06-AM.png)

**Step 5)** Click `Create` and verify that OPA constraint has stopped creating the deployment. You'll notice that the error message is clearly stating that allowed user range is between 100-200 and our container is trying to run as user id 0. 

![Screenshot-2022-09-06-at-7.22.21-AM](../images/Screenshot-2022-09-06-at-7.22.21-AM.png)

**End of Task 5**

## Task 6: Validate deployment - Create deployment to run container as non-root user and our policy should admit the container. 

This action will involve 6 steps from 1 to 6

**Step 1)** That was Good! Now let us see if the container runs with non-root user e.g. 101 which is within the specified range. 

For this exercise, we have to use one such container which is build to run as non-root e.g. `nginxinc/nginx-unprivileged:latest`. 

![Screenshot-2022-09-06-at-7.25.12-AM](../images/Screenshot-2022-09-06-at-7.25.12-AM.png)

**Step 2)** Create a fresh deployment with following details. 

`Namespace : default`

`Name : user's choice e.g. opa-allowed-users-ok`

Now navigate to the `Containers` tab. 

`Container Image : nginxinc/nginx-unprivileged:latest`

![Screenshot-2022-09-06-at-7.26.15-AM](../images/Screenshot-2022-09-06-at-7.26.15-AM.png)

**Step 3)** Now scroll down and stop at `Security Context` to change the default value of `Run as Non-Root` to `Yes`. It means container will run as non-root user. Also specify the value of field `Run as UserID`  to `101`. 

![Screenshot-2022-09-06-at-7.27.47-AM](../images/Screenshot-2022-09-06-at-7.27.47-AM.png)

**Step 4)** Now click `Create` and verify that deployment is created successfully. 

![Screenshot-2022-09-06-at-7.30.30-AM](../images/Screenshot-2022-09-06-at-7.30.30-AM.png)

**Step 5)** Let's validate if the container is really running as non-root user i.e. 101. Click on the 3 vertical dots menu at the right end of the new created deployment and click on ` > Execute Shell`. It will open an inline window just below the Deployment section. 

![Screenshot-2022-09-06-at-7.31.41-AM](../images/Screenshot-2022-09-06-at-7.31.41-AM.png)

**Step 6)** Execute command `id` in the shell and verify that `uid`, `gid`, and `groups` values are 101 as specified in the container image. 

![Screenshot-2022-09-06 at-7.34.09-AM](../images/Screenshot-2022-09-06-at-7.34.09-AM.png)

Bingo! The OPA constraint is working as we were expecting. But wait... How much you are familiar with REGO language and how much effort it will require to create an OPA template if you are relatively newer in this space? Definitely, there are readymade templates available on git [gatekeepr-library](https://github.com/open-policy-agent/gatekeeper-library) which you may explore. However, there are good alternatives for advance policy control like `ARP Spoofing` etc. Let's conclude it here the and move on to the next exercise to explore Kubewarden. 

**End of Task 6**

******
Important!! Please delete the OPA Constraint after the Lab 1 
Navigate to Constraint menu under OPA Gatekeeper. Select constraint `allowed-users-100-200` and click on Delete.
Confirm the deletion. 
Verify that constraint is deleted.
******

**End of LAB 1**

Continue to: [Lab 2: Kubewarden](/docs/Lab02-Kubewarden.md)



