---
title: "Intro to Azure Recon with BloodHound"
date: 2024-09-28T06:53:07+01:00
description: "BloodHound together with an ingestor Azurehound can map Azure AD relationships, revealing roles, permissions, and attack paths, helping identify misconfigurations and privilege escalation opportunities in cloud environments."
categories: [Cloud]
tags: [Azure, Red Teaming, Entra-ID, Powershell, Bloodhound]
image: "https://i.pinimg.com/originals/86/41/a4/8641a4adec1d2b10746d02e664f6f9da.gif"
---

**Note : This article was inspired by content from [Pwned Labs](https://pwnedlabs.io/labs/intro-to-azure-recon-with-bloodhound). Special thanks to Pwned Labs for providing insightful resources and awesome learning experience for learners.**

> Overview => After discovering that a public company GitHub repository contained accidentally committed credentials, Mega Big Tech has requested us to investigate the extent of potential exposure. They want to determine if these credentials can be used to access their cloud environment and if any confidential data is at risk.


## **Setting Up**


- Download the latest `azurehound` binary from [here](https://github.com/bloodhoundad/azurehound/releases)
- Unzip and and move to `/bin` for faster access

```bash
❯ unzip azurehound-linux-amd64.zip 
❯ sudo mv azurehound /bin/
❯ sudo chmod 777 /bin/azurehound
```


Just as we need credential access when enumerating On-Prem, we also need credential access here too. Credentials have been given for this lab though :


```
IAM user : Jose.Rodriguez@megabigtech.com
Password : [REDACTED]
Login Portal : https://portal.azure.com/
Azure tenant ID : Find this on portal or by running 
	* az login
	* az account show

{
  --SNIP--
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  --SNIP--
}
```



To run `azurehound` we need the `tenantID`, Username and the Password. Just as shown above, you can get the `tenantID` by using `Azure-CLI`


```bash
azurehound -u "Jose.Rodriguez@megabigtech.com" -p '[REDACTED]' list --tenant "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" -o output.json 
```


![](https://i.imgur.com/VumnEvW.jpeg)



Start up both `neo4j` and `bloodhound`

```bash
❯ sudo neo4j console
❯ sudo bloodhound
```


Then go ahead and upload the `output.json` data created by `azurehound` 


![](https://i.imgur.com/ybktVk3.png)


Scrolling down under the **Database Info** tab you should see "Azure Objects" 


![](https://i.imgur.com/75QO2zg.png)



As user we start by enumerating the properties which our current user has


![](https://i.imgur.com/rliIHD5.png)


## **Nodes and Edges**



BloodHound consists of nodes and edges, which together form a graph that reveals potential attack paths and privilege relationships within the environment ;


- Nodes : represent **entities** or **objects** in Azure AD. Few examples include:
	- **Users**: Azure AD accounts.
	- **Groups**: Security or distribution groups in Azure AD.
	- **Applications**: Azure AD applications that use service principals.
	- **Service Principals**: Representations of applications within Azure.
	- **Subscriptions**: Azure subscription resources.
- Edges ; represent the **relationships** or **permissions** between nodes.
	 - **Membership**: A user or service principal being a member of a group.
    - **Ownership**: A user or group owning a service principal or application.
    - **Role Assignments**: Showing roles that provide permissions over resources like subscriptions or resource groups.
    - **Privileges**: Showing a user's privileged access over an entity (e.g., a user having `Owner` or `Contributor` role over a resource).


![](https://i.imgur.com/v61NxEV.png)


> Edges are far more important in Azure has they show us the permission resources have over each other, As attackers it reveals **attack paths** show ways one can navigate from a low-privilege node to a high-privilege node. Lot of `Az` edges have been discussed [here](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html)


Bloodhound tells us that our current user belongs to 5 `Az` admin roles with an assigned role named `UPDATE MANAGER` and four roles inherited from the `IT-Helpdesk` group.


![](https://i.imgur.com/wcYUmtv.png)


- **Update Manager**: Allows helpdesk to update manager roles when users change teams; low security impact.
- **Directory Readers**: Allows users to read basic directory info, excluding sensitive data.
- **Printer Technician**: Allows managing printers; low security interest.
- **Attribute Definition Reader**: Can read definitions of custom security attributes.
- **Attribute Assignment Reader**: Can read custom security attribute keys and values for Microsoft Entra objects.


## **Cypher Queries Overview**


Since generally there are no much queries like "**Find All Domain Admins**" in bloodhound for azure we can start by creating our own custom cypher query, This [cheatsheet](https://raw.githubusercontent.com/LuemmelSec/Custom-BloodHound-Queries/main/README.md) contains alot of cypher queries



For example, the below returns all Members of the Global Administrator role.

```cypher
MATCH p =(n)-[r:AZGlobalAdmin*1..]->(m) RETURN p
```


> Note : Azure Global Administrator role is equivalent to the Domain Admin privilege in Active Directory.



***Setting Up Query :***


![](https://i.imgur.com/ZyqSA60.png)


***Executing Query To List All Global Admins :***


![](https://i.imgur.com/lsWBOo2.png)


## **Hunting for custom security attributes**

- This feature was released in Azure on December of 2021
- This was introduced to  roll-out of Attribute Based Access Control (ABAC)
- Used to store sensitive data which is a potential target for attackers


Go ahead and install the `Microsoft.Graph` module as described by [Microsoft](https://learn.microsoft.com/en-us/entra/identity/users/users-custom-security-attributes?tabs=ms-graph#get-the-custom-security-attribute-assignments-for-a-user) on how to query for custom security attributes


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab>  Install-Module Microsoft.Graph
```

Then run the below command to perform authentication request as our current Azure AD user.

```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Connect-MgGraph

Welcome to Microsoft Graph!

Connected via delegated access using 14d82eec-204b-4c2f-b7e8-296a70dab67e
Readme: https://aka.ms/graph/sdk/powershell
SDK Docs: https://aka.ms/graph/sdk/powershell/docs
API Docs: https://aka.ms/graph/docs

NOTE: You can use the -NoWelcome parameter to suppress this message.
```


The below script then connects to Microsoft Graph, retrieves all users in Azure AD, and loops through each user to display their custom security attributes.


```powershell
# Connect to Microsoft Graph
Connect-MgGraph

# Retrieve all users
$allUsers = Get-MgUser -All

# Loop through all users and retrieve their custom security attributes
foreach ($user in $allUsers) {
    $userAttributes = Get-MgUser -UserId $user.Id -Property "customSecurityAttributes"
    
    # Display the additional properties of custom security attributes for each user
    Write-Host "User: $($user.UserPrincipalName)"
    $userAttributes.CustomSecurityAttributes.AdditionalProperties | Format-List
    Write-Host "---------------------------------------------"
}
```


As shown in the below screenshot the user `archive@megabigtech.com` has a custom security attribute named `Helpdesk` that seems to store the password of the user.

![](https://i.imgur.com/Olfp8E5.jpeg)


logging into to the [Azure Portal](https://portal.azure.com/) as user `jose....`, we can lookup the `archive@megabigtech.com` user in the console.


![](https://i.imgur.com/uQxUFCB.png)


We see that `Helpdesk` staff added the attribute to allow them to login and troubleshoot as this user.


![](https://i.imgur.com/tKM3MqK.png)


## **Manual Enumeration - Bloodhound misses**


BloodHound effectively identifies directory roles like this but currently misses Azure role assignments made at the subscription, management group, resource group, or resource level.

- Navigating to the Azure portal we can go ahead and select **"Microsoft Entra ID"**

![](https://i.imgur.com/8hH4Xia.png)


- Then click on **"Groups"**


![](https://i.imgur.com/ibuk4kK.png)


- Search for the helpdesk group and select the `IT-Helpdesk` to bring up its properties as shown in `bloodhound` previously.


![](https://i.imgur.com/zjXA8J7.png)

- Now click `Azure role assignments`.

![](https://i.imgur.com/awTdTBN.png)


As shown below the group, "`IT-Helpdesk`" is assigned the `Reader` role, with the scope set to the `SECURITY-PC` virtual machine.


![](https://i.imgur.com/DIlMhGd.png)



We could also have done the above using the PowerShell `Az` module and the `Get-AzRoleAssignment` cmdlet.



```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Connect-AzAccount   
Please select the account you want to login with.

Retrieving subscriptions for the selection...

Subscription name           Tenant
-----------------           ------
Microsoft Azure Sponsorship Default Directory


PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-AzRoleAssignment

RoleAssignmentName : 4b5ae432-6902-4ca2-bbed-815492eef631
RoleAssignmentId   : /subscriptions/ceff06cb-e29d-4486-a3ae-eaaec5689f94/resourcegroups/content-static-2/providers/Microsoft.Compute/virtualMachines/SECURITY-PC/providers/Microsoft.Authoriz
                     ation/roleAssignments/4b5ae432-6902-4ca2-bbed-815492eef631
Scope              : /subscriptions/ceff06cb-e29d-4486-a3ae-eaaec5689f94/resourcegroups/content-static-2/providers/Microsoft.Compute/virtualMachines/SECURITY-PC
DisplayName        : IT-HELPDESK
SignInName         : 
RoleDefinitionName : Reader
RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
ObjectId           : 8a517e87-6b05-45ae-b1ca-7436f1682602
ObjectType         : Group
CanDelegate        : False
Description        : 
ConditionVersion   : 
Condition          : 
```


![](https://i.imgur.com/L5Sodhg.png)



## **Extracting Credentials**


Since we have `Reader` access to a virtual machine, something that is worth checking is custom user data fields, Go ahead and click the `SECURITY-PC` resource


![](https://i.imgur.com/kZqoiYd.png)



Click on the `Operating system` menu under the `Settings` section, the `User data` field we see that an Azure CLI command has been added, including a comment with credentials!

![](https://i.imgur.com/mqUAOGb.png)


```
# Credentials: User: security-user | Password: Imp0sec0sT! az storage blob download --account-name securityconfigs --container-name security-pc --name config-latest.xml --auth-mode login
```


We can get the virtual machine user data from the command line instead of using the Az portal as we did previously, as shown below.

```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-AzVM -ResourceGroupName "content-static-2" -Name "SECURITY-PC" -UserData


ResourceGroupName : content-static-2
Id                : /subscriptions/ceff06cb-e29d-4486-a3ae-eaaec5689f94/resourceGroups/content-static-2/providers/Microsoft.Compute/virtualMachines/SECURITY-PC
VmId              : 648c8a08-c90a-4a95-8922-4cbf28375bcb
Name              : SECURITY-PC
Type              : Microsoft.Compute/virtualMachines
Location          : eastus
LicenseType       : Windows_Client
Tags              : {}
HardwareProfile   : {VmSize}
NetworkProfile    : {NetworkInterfaces}
SecurityProfile   : {UefiSettings, SecurityType}
OSProfile         : {ComputerName, AdminUsername, WindowsConfiguration, Secrets, AllowExtensionOperations, RequireGuestProvisionSignal}
ProvisioningState : Succeeded
StorageProfile    : {ImageReference, OsDisk, DataDisks, DiskControllerType}
Identity          : {PrincipalId, TenantId, Type}
Zones             : {1}
UserData          : IyBDcmVkZW50aWFsczogVXNlcjogc2VjdXJpdHktdXNlciB8IFBhc3N3b3JkOiBJbXAwc2VjMHNUIQpheiBzdG9yYWdlIGJsb2IgZG93bmxvYWQgLS1hY2NvdW50LW5hbWUgc2VjdXJpdHljb25maWdzIC0tY29udGFpbmVyL
W5hbWUgc2VjdXJpdHktcGMgLS1uYW1lIGNvbmZpZy1sYXRlc3QueG1sIC0tYXV0aC1tb2RlIGxvZ2luCg==
TimeCreated       : 10/31/2023 3:24:18PM
Etag              : "16"
```


The `UserData` variable is what we need which is in base64 format, we can go ahead and decode this using your linux OS

```bash
❯ echo "IyBDcmVkZW50aWFsczogVXNlcjogc2VjdXJpdHktdXNlciB8IFBhc3N3b3JkOiBJbXAwc2VjMHNUIQpheiBzdG9yYWdlIGJsb2IgZG93bmxvYWQgLS1hY2NvdW50LW5hbWUgc2VjdXJpdHljb25maWdzIC0tY29udGFpbmVyL
W5hbWUgc2VjdXJpdHktcGMgLS1uYW1lIGNvbmZpZy1sYXRlc3QueG1sIC0tYXV0aC1tb2RlIGxvZ2luCg==" | base64 -d

# Credentials: User: security-user | Password: Imp0sec0sT!
az storage blob download --account-name securityconfigs --container-name security-pc --name config-latest.xml --auth-mode login
```


Now trying to use the command shown after decoding the `b64` text we have the following error cos we are not yet logged in as the `security-user` who has the permission to do so.


![](https://i.imgur.com/jaPApg7.png)


Run the below command on `pwsh` to log in as the `security-user` user for privileged access


```powershell
az login --scope https://storage.azure.com/.default

# When prompted fill in the below
Username : security-user@megabigtech.com
Passowrd : *************
```


If we run the command again we have alot of config output and sensitive information including global administrator credentials 


![](https://i.imgur.com/zQCsW0e.jpeg)


We can also view this config files using the Az portal, Log in as the `security-user` user and search for the `securityconfigs` resource (storage account name)


![](https://i.imgur.com/DirawvP.png)



Under the "**Data Storage**" drop down select the `Containers` menu 


![](https://i.imgur.com/r75ffnW.png)

Then click on the `security-pc` container name

![](https://i.imgur.com/TFll4vS.png)


We can then see the config file we accessed earlier including the `flag.txt` file for this challenge lab

![](https://i.imgur.com/12YfIvo.png)

Go ahead and download the flag file using the `Azure-CLI` tool as used earlier

```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> az storage blob download --account-name securityconfigs --container-name security-pc --name flag.txt --auth-mode login         
Finished[#############################################################]  100.0000%
******************************
```

Have fun 𓆝 𓆟 𓆞 𓆝 𓆟

## **Resources** 

- **Lab Link :** [https://pwnedlabs.io/labs/intro-to-azure-recon-with-bloodhound](https://pwnedlabs.io/labs/intro-to-azure-recon-with-bloodhound)
- [https://learn.microsoft.com/en-us/entra/fundamentals/custom-security-attributes-overview](https://learn.microsoft.com/en-us/entra/fundamentals/custom-security-attributes-overview)
- [https://blog.netwrix.com/2022/02/10/what-are-azure-ad-custom-security-attributes/](https://blog.netwrix.com/2022/02/10/what-are-azure-ad-custom-security-attributes/)
- [https://posts.specterops.io/introducing-bloodhound-4-3-get-global-admin-more-often-5795cbf535b2](https://posts.specterops.io/introducing-bloodhound-4-3-get-global-admin-more-often-5795cbf535b2)
