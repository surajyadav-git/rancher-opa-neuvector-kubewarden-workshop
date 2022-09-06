# `LAB`-04 - NeuVector (OPA functionality) 

## Background 

In previous workshop on Private Registry with Rancher, we have shown you the capability of Open Policy Agent (OPA).

GitHub link for Workshop 'https://github.com/dsohk/rancher-private-registry-workshop'

------------------------------------------------------------------------------------------

optional

Example: "K8sallowedrepo" which allows you to whitelist Container Registry,  allowing you to define our trusted sources for obtaining container images. 

OPA's Gatekeeper will allow/deny operation based on policy you have created to meet your requirement. The policy is build using Rego Language.  To know more about Rego Language visit URL 'https://www.openpolicyagent.org/docs/latest/policy-language/#what-is-rego'

----------------------------------------------------------------

OPA provides Policy-based control for cloud native environments. It provides flexible, fine-grained control for administrators. OPA uses Rego language to build its policies. You need to have good knowledge and skill on using Rego while you want to take advantage of OPA.  

But what if you don't have skilled and experience on Rego.  What if you want UI to reduce your learning curve. SUSE NeuVector is an possible solution you can look at. 

SUSE NeuVector is your Zero Trust Container Security Platform. 

In this exercise, we will create Admission Control Policy using NeuVector. Our intention is to prevent 'NGINX' image with high Severity Score to prevent running within our cluster. 

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Pre-requisite. 

You will be provided with Rancher and NeuVector URL. Please open the file with your favourite editor. Copy and paste the NeuVector URL in your browser. 

You will be promoted for username & credentials. Supply the credentials & login. Upon successful login, you will be prompted to accept the agreement. Please accept & proceed. 

You will be presented with NeuVector Dashboard. 

Before we proceed with the exercise. 

Admin (Right Hand Corner) > My Profile > Edit my profile > Session Timeout > Change current value to 3600. (Present value would be 300). You may be prompted to re-login.

1. NeuVector Dashboard > Assets > Registries  > Add Registries. 

   Name = 'DockerHub'

   Registry URL = 'https://registry.hub.docker.com'

   Filter = 'nginx:latest'

   Optionally you can select Scan Layer & Periodic Scan based on your requirement. 
   
   ![AddingRegistry-pg1](../images/AddingRegistry-pg1-1662430916095-2.PNG)

2. Test Setting it will open a pop up window and the registry will be scanned for the image. ![AddingRegistry-pg2](C:\Deepak-Data\SUSE\Rancher-Workshop\rancher-opa-neuvector-kubewarden-workshop\images\AddingRegistry-pg2.PNG)

3. Above result shows success. You can click on back button & than click on 'Submit' 

   ![AddingRegistry-pg3](C:\Deepak-Data\SUSE\Rancher-Workshop\rancher-opa-neuvector-kubewarden-workshop\images\AddingRegistry-pg3.PNG)

   4. Start Scan to scan the image in the registry.
      ![AddingRegistry-pg4](C:\Deepak-Data\SUSE\Rancher-Workshop\rancher-opa-neuvector-kubewarden-workshop\images\AddingRegistry-pg4.PNG)

5. Click on the 'nging:latest' and it will open up the scan result as below. 

   ![AddingRegistry-pg5](C:\Deepak-Data\SUSE\Rancher-Workshop\rancher-opa-neuvector-kubewarden-workshop\images\AddingRegistry-pg5.PNG)

We now have the CVE which we wanted to use. 

CVE = 'CVE-2019-8457', Severity = 'High' and Score = '7.5/9.8' 

## Step 4A.1 ) NeuVector Admission Policy.  

1. NeuVector Dashboard > Policy > Admission Control.![admission-control-pg1](C:\Deepak-Data\SUSE\Rancher Hands On Workshop\Sept8-2022-workshop\rancher-opa-neuvector-kubewarden-workshop\images\admission-control-pg1-1662391820906-2.png)

2. Toggle the status bar. You will be presented with Mode - Monitor & Protect. 

   ![admission-control-pg1.1](C:\Deepak-Data\SUSE\Rancher Hands On Workshop\Sept8-2022-workshop\rancher-opa-neuvector-kubewarden-workshop\images\admission-control-pg1.1.png)

3. Click on Add to create a new Admission Control Policy. A form will be presented to you. Post entering details as below, click on '+' symbol

Criterion = 'CVE Names'

Operator = 'Container All off ' or 'Contain Any of'

Value = 'CVE-2019-8457'

![admission-control-pg14](C:\Deepak-Data\SUSE\Rancher Hands On Workshop\Sept8-2022-workshop\rancher-opa-neuvector-kubewarden-workshop\images\admission-control-pg14.PNG)

![admission-control-pg16.1](C:\Deepak-Data\SUSE\Rancher Hands On Workshop\Sept8-2022-workshop\rancher-opa-neuvector-kubewarden-workshop\images\admission-control-pg16.1.PNG)

4. Our new policy with ID '1000 ' is created. For the policy to work toggle the mode from Monitor to Protect. 

   ![admission-control-pg16.2](C:\Deepak-Data\SUSE\Rancher Hands On Workshop\Sept8-2022-workshop\rancher-opa-neuvector-kubewarden-workshop\images\admission-control-pg16.2.PNG)

   Navigate to Dashboard > Notification > Events.  

   ![admission-control-pg16.3](C:\Deepak-Data\SUSE\Rancher Hands On Workshop\Sept8-2022-workshop\rancher-opa-neuvector-kubewarden-workshop\images\admission-control-pg16.3-1662393780750-12.PNG)

Next step is to test our newly create policy. For this we will need to move to your browser window where you have the Rancher UI.   

## Step 4B.1 ) NeuVector Admission Policy.  

Rancher UI > Home > Explore Cluster > 'rke2-cluster1' > Workload > Deployments > Create 

Namespace = 'default'

Name = 'nginx'

Container Page > Image = 'nginx:latest' 

Click on 'Create'

![NGINX-with-CVE-restricted-by-NeuVector-Admission-control](C:\Deepak-Data\SUSE\Rancher Hands On Workshop\Sept8-2022-workshop\rancher-opa-neuvector-kubewarden-workshop\images\NGINX-with-CVE-restricted-by-NeuVector-Admission-control.PNG)

We are getting this error as our NeuVector Admission Control in Protect Mode is restricting the image with 'CVE-2019-8457'

If you toggle back to NeuVector page

NeuVector > Dashboard > Notification > Risk Reports > In the Search box on right hand top corner - Type 'Denied'  ![NGINX-with-CVE-restricted-by-NeuVector-Admission-control-pg1](C:\Deepak-Data\SUSE\Rancher Hands On Workshop\Sept8-2022-workshop\rancher-opa-neuvector-kubewarden-workshop\images\NGINX-with-CVE-restricted-by-NeuVector-Admission-control-pg1-1662397373557-22.png)

We can clearly our risk report providing the details of Admission Control Denying the execution to create the deployment due to the NeuVector Policy. 

So without any knowledge of programming language like Rego, Rust using NeuVector's intuitive UI, we have created a policy to restrict image with CVE not be execute in our cluster. 

**End of Exercise 04**

 

