# cdp-azure-quickstart

#### Step 1. Verifying access to CDP console
![CDP Landing Page](https://github.com/cpv0310/cdp-azure-tools/blob/master/screenshots/screenshot6.png?raw=true)

If you've reached the above landing page for the first time, you've come to the right place! In this quickstart, we're going to walkthrough step by step how to connect CDP to your Azure subscription so that you can begin to provision clusters and workloads. 

In order to complete this quickstart, you'll need access to two things.  

  1. The CDP console (if you've reached the above screen, you're good to go there)
  2. The Azure console
  3. Azure Cloud shell

#### Step 2.  How to create Azure AD App

Login to Azure portal and open "cloud shell" 

![Azure Cloud shell](https://github.com/cpv0310/cdp-azure-tools/blob/master/screenshots/azure-shell.png?raw=true)

Get subscription ID and Tenant ID by running the command below.

```az account list|jq '.[]|{"name": .name, "subscriptionId": .id, "tenantId": .tenantId, "state": .state}'```

The output of this command is as below:

![SubscriptionID and TenantID](https://github.com/cpv0310/cdp-azure-tools/blob/master/screenshots/sub-tenant-ID.png?raw=true)

Please note down SubscriptionID and TenantID -> You will need these values later.

Create an app in Azure AD and assign 'Contributor' role at subscription level

```az ad sp create-for-rbac --name http://cloudbreak-app --role Contributor --scopes /subscriptions/{subscriptionId}```

Note: Replace subscriptionId with the subscriptionId from #1 and replace the cloudbreak-app with something recognizable by you

The output of this command is as below:
![Output after app create](https://github.com/cpv0310/cdp-azure-tools/blob/master/screenshots/app-output.png?raw=true)
Save this output above.  You will use it to create your credential in Step 6

#### Step 3: Azure quickstart template

ARM template that deploys essential Azure resources for Cloudera CDP environment.

Click ' Deploy to Azure' (#3) and login to your subscription to create essential resources for CDP deployment in your subscription. These resources include VNet, ADLS Gen2, 4 User Managed Identities. Provide envName on the screen. (refer the screenshot below).  The Environment name that you provide is referred as <envName> going forward.  Environment Name will be used to name the Azure Storage Account and must be unique.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fcegganesh84%2Fcdp-azure-tools%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png" />
</a>

<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Fcegganesh84%2Fcdp-azure-tools%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="http://armviz.io/visualizebutton.png"/>
</a>

![Deploy To Azure](https://github.com/cpv0310/cdp-azure-tools/blob/master/screenshots/deployment.png?raw=true)





After you click the "Purchase" button, it will take a couple minutes and you will get a Resource Group that looks like this

![Deployed Resource Group](https://github.com/cpv0310/cdp-azure-tools/blob/master/screenshots/cdp-resourcegroup.png?raw=true)


---

#### Step 4: Fine grained logger/dataAccess/ranger identity role assignment
**Azure RM templates does not support role assignments at a scope other than resource group. So the
following role assignments need to be performed via CLI or UI.**

- Copy the script azure_msi_role_assign.sh from this repo
![Role Assignment](https://github.com/cpv0310/cdp-azure-tools/blob/master/screenshots/role-assignment-script.png?raw=true)

- In the script, replace the subscription id with your subscription id and the resource group with the Resource Group name you create in the last step

- Run the script on Azure shell.  Personally, I like to just paste the script into the shell but you can also create a shell script file and execute that from the prompt. 


#### Step 5. Create SSH public key OR locate one if you already have
You can find more details on SSH key requirement 
[here](https://docs.cloudera.com/management-console/cloud/environments-azure/topics/mc-azure-env-ssh-key.html) 
You can create one using PuttyGen (Windows) or by running ***ssh-keygen -t rsa*** (Linux/Mac)

If you complete step5, that means you have already created all required Azure resources for this quickstart.

#### Step 6. Creating a CDP Credential

 In the CDP Console, the first thing we're going to do is create our CDP Credential.  The CDP credential is the mechanism that allows CDP to create resources inside your Cloud Account.  
    1. From the CDP Home Screen, click the **Management Console** icon. 
    2. On the left side navigation plane, go to **Environments**
    3. From there, in the top left choose **Shared Resources**, then **Credentials**
    4. Click on the **Create Credential** button on the top right.

![CDP Credential Page](https://github.com/cpv0310/cdp-azure-tools/blob/master/screenshots/credential.png?raw=true)

- Provide the different values catured for subscriptionID, TenantID, AppID, Password in the steps above and click Create.

![CDP Credential Page](https://github.com/cpv0310/cdp-azure-tools/blob/master/screenshots/create-app1.png?raw=true)

  - The link explains how the credential is going to be used. [here](https://docs.cloudera.com/management-console/cloud/credentials-azure/topics/mc-credential.html).  
---

#### Step 7. Registering CDP Environment.

    1. Head back to CDP Management Console and Navigate to **Environments**
    2. Click **Register Environment**
    3. Provide an environment name and description.  The name can be any valid name. 
    4. Choose *azure* as the Cloud Provider
    5. Under *Microsoft Azure Credentials*, chose the credential we created earlier. 
    6. Click **Next**

![Chose credential](https://github.com/cpv0310/cdp-azure-tools/blob/master/screenshots/choose-cred.png?raw=true)


    7. Under *Data Lake Settings*, give your new Data Lake a name.  The name can be any valid name. Choose the latest Data Lake Version
    8. Choose *Light Duty* for Data Lake scale. 
    9. Click **Next**
    
  ![Chose credential](https://github.com/cpv0310/cdp-azure-tools/blob/master/screenshots/dl.png?raw=true)
  
    10. Choose your desired **region**, this should be the same region you used to deploy resources in Step3.
    11. Under *select network* choose **<vnet-name>** that was created in step3.
    12. Under *Security Access Settings* choose **Create New Security Groups**
    
   ![Network](https://github.com/cpv0310/cdp-azure-tools/blob/master/screenshots/network.png?raw=true). 
        
    13. Under *SSH Settings*, choose *New SSH Public Key* and provide the key that you created in Step5.
    
    14. Under *Logs - Storage and Audit*, choose the <rg-name -envName-LoggerIdentity> under *Logger Identity*, for logs location base choose **logs@<sa-name>**, and for *Ranger Audit Role* choose **<rg-name -envName-RangerIdentity>**
    
  ![logs](https://github.com/cpv0310/cdp-azure-tools/blob/master/screenshots/logs.png?raw=true).
	
    15.  Under *Data Access*, choose the <rg-name -envName-AssumerIdentity> under *Assumer Identity*, for storage location base choose **data@<sa-name>**, and for *Data Access Identity* choose **<rg-name -envName-DataAccessIdentity>**
    
  ![data](https://github.com/cpv0310/cdp-azure-tools/blob/master/screenshots/data.png?raw=true).
	
    16. (optional) Provide any tags you'd like these resources to be tagged with. 

    17. Click **Register Environment**

# Changelog

1.1st cut instructions [04/14/2020]
2. Update instructions [04/15/2020]
