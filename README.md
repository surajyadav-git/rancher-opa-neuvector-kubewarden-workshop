# Securing Kubernetes with OPA, Kubewarden and NeuVector
A technical hands-on workshop focus on Kubernetes admission controller.

## Pre-requisites

1. This Lab will require only chrome browser with fast and reliable internet connection. 

2. Make sure you have access to following urls behind your corporate firewall/personal computers. 

   [Github](https://github.com/) 

   [Sslip](https://sslip.io) (, *.sslip.io)

3. Watch previous demo on [Rancher private registry workshop](https://github.com/dsohk/rancher-private-registry-workshop/)   


4. If you want to spin this lab with your own personal/corporate Microsoft Azure Cloud account, please follow below link:

   [Automation](https://github.com/dsohk/workshops/tree/main/scenarios/azure/rancher-neuvector)  

5. Optional - Access to [SUSE Partner Portal](https://myaccount.suse.com/)



## Open Source Technologies Used

| Software  | Version         | Remarks                                                      |
| --------  | --------------- | ------------------------------------------------------------ |
| Rancher   | v2.6.7           | Enterprise-grade Kubernetes Management Platform              |
| RKE2      | v1.21.4+rke2r2            | Rancher Kubernetes Engine 2 - with Kubernetes version 1.21.4 |
| OPA Gatekeeper      | v100.2.0+up3.8.1            | Open Policy Agent Gatekeeper: Policy and Governance for Kubernetes |
| Kubewarden| v1.0            | Kubewarden is a policy engine for Kubernetes. Its mission is to simplify the adoption of policy-as-code. |
| NeuVector | v5.0.2            | Full Lifecycle Container Security |



## Before We Begin

Before we start the lab, please make sure you have been provided with the following lab access credentials from the instructor. Each participant should have their own unique environment and credentials.

| Item                              | Value                                                      |
| --------------------------------- | ---------------------------------------------------------- |
| Rancher Server URL                | https://rancher.xx.xx.xx.xx.sslip.io                       |
| Rancher Server Username           | admin                                                      |
| Rancher Server Bootstrap Password | system assigned strong password                            |
| NeuVector URL                        | https://neuvector.yy.yy.yy.yy.sslip.io                        |
| NeuVector Username                   | admin                                                      |
| NeuVector Password                   | system assigned strong password                            |



### Step 1 - Access Rancher and NeuVector Portals

a) With a Google Chrome browser, access Rancher Server URL (https://rancher.xx.xx.xx.xx.sslip.io). If you are greeted by an invalid SSL certificate error message, please follow the Note*.

b) Enter the admin credentials to login i.e. `Username: admin`  and `Password: system assigned strong password`. 

c) When it's successful, you will be landing on the Rancher Home page. 

d) Do the same for NeuVector URL (https://neuvector.yy.yy.yy.yy.sslip.io). You will then be led to NeuVector login page. If you are greeted by an invalid SSL certificate error message, please follow the Note*.

e) Enter the admin credentials to login i.e. `Username: admin`  and `Password: system assigned strong password`. 

##### *Note: 

If you are greeted by an invalid SSL certificate error message, you can click on the **Advanced** button

Click the Proceed to https://rancher.xx.xx.xx.xx.sslip.io and/or https://neuvector.yy.yy.yy.yy.sslip.io hyperlink to continue. 

Now the login page will be displayed and proceed to authenticate yourself by entering username and password.

If link isn't shown, click on the empty space on browser and type the word `thisisunsafe` without any spaces to bypass the warning message. 

Now the login page will be displayed and proceed to authenticate yourself by entering username and password.



## Lab Exercises

This lab consists of 3 exercises. 

* ##### Lab-01:[Open Policy Agent Gatekeeper](docs/Lab01A-OPAGatekeeper.md)
  
* ##### Lab-02:[Kubewarden](docs/Lab02-Kubewarden.md)
  
* ##### Lab-03:[NeuVector - Admission Control](docs/Lab03-NeuVector-Admission-Control.md)



