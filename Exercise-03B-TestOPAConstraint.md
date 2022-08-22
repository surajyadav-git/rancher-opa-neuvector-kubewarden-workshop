# Lab 3 B - Test registry access as per the OPA constraint setup

This exercise will continue from the previous Exercise 3A. We'll deploy a sample application rancher's hello-world from Harbor registry and ACR respectively to validate OPA Constraint.  



###### Usecase: It's now time to test and ensure that policies are working as we are expecting! 



## Action 1: Validate deployment - pulling container image from Harbor 

This action will involve 3 steps from 3B.1 to 3B.3

**Step 3B.1)** Make sure that you are at the Deployment screen in Rancher. Click on `Create`. 

![Screenshot-2022-07-24-at-6.41.15-PM](../images/Screenshot-2022-07-24-at-6.41.15-PM.png)



**Step 3B.2)** Fill in following details and click on `Create`

`Namespace : default`

`Name : user's choice e.g. harbor-hello-world`

`Container Image : harbor.yy.yy.yy.yy.sslip.io/library/hello-world:v0.1.2`

`Pull Seceret : as specified in Step 2A.5 (From dropdown) e.g. harbor-registry `

`Service Type : Node Port (From the dropdown)`

`Private Container Port : 80`

![Screenshot-2022-07-24-at-6.42.36-PM](../images/Screenshot-2022-07-24-at-6.42.36-PM.png)



**Step 3B.3)** Verify that deployment is failed due to the OPA constraint rules. 

![Screenshot-2022-07-24-at-6.45.17-PM](../images/Screenshot-2022-07-24-at-6.45.17-PM.png)



## Action 2: Validate deployment - pulling container image from ACR 

This action will involve 6 steps from 3B.4 to 3B.9

**Step 3B.4)** Stay on the same page and click `Create` on the top right side of the screen. 

![Screenshot-2022-07-24-at-6.49.19-PM](../images/Screenshot-2022-07-24-at-6.49.19-PM.png)



**Step 3B.5)** Fill in following details and click on `Create`

`Namespace : default`

`Name : user's choice e.g. acr-hello-world`

`Container Image : attendeexx.azurecr.io/library/hello-world:v0.1.2`

`Pull Seceret : as specified in Step 2A.8 (From dropdown) e.g. azure-cr `

`Service Type : Node Port (From the dropdown)`

`Private Container Port : 80`

![Screenshot-2022-07-24-at-6.51.58-PM](../images/Screenshot-2022-07-24-at-6.51.58-PM.png)

**Step 3B.6)** Verify that deployment is successfully created and click on the hyperlink of the deployment under **Name** column. 

![Screenshot-2022-07-24-at-6.53.25-PM](../images/Screenshot-2022-07-24-at-6.53.25-PM.png)

**Step 3B.7)** Verify that 1 Pod is in Running state. 

![Screenshot-2022-07-24-at-6.54.01-PM](../images/Screenshot-2022-07-24-at-6.54.01-PM.png)

**Step 3B.8)** Click on `Service Discovery (1)` option from the left hand static menu and then click on `Services (2)`  submenu under the Service Discovery. Now click on the link `3nnnn/TCP (3)` under **Target** column.

![Screenshot-2022-07-24-at-6.54.36-PM](../images/Screenshot-2022-07-24-at-6.54.36-PM.png)

**Step 3B.9)** Verify that Rancher's Hello world application is accessible on designated Node Port. 

![Screenshot-2022-07-24-at-6.56.09-PM](../images/Screenshot-2022-07-24-at-6.56.09-PM.png)

**End of Action 2**

**End of Exercise 3B**