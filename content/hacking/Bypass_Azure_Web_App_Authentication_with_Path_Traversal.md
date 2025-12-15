---
title: 'Bypass Azure Web App Authentication with Path Traversal'
tags: [Azure, Red Teaming, Entra-ID, Kudu]
date: 2024-11-20T02:03:30+01:00
description: "Learn how attackers exploit path traversal vulnerabilities to bypass authentication and gain unauthorized access to Azure Web Apps."
---

**Note : This article was inspired by content from [Pwned Labs](https://pwnedlabs.io/labs/bypass-azure-web-app-authentication-with-path-traversal). Special thanks to Pwned Labs for providing insightful resources and awesome learning experience for learners.**

**Lab Link :** [https://pwnedlabs.io/labs/bypass-azure-web-app-authentication-with-path-traversal](https://pwnedlabs.io/labs/bypass-azure-web-app-authentication-with-path-traversal)



> Overview -> On an engagement for our client Mega Big Tech, we used a custom phishlet and successfully performed an Evilginx man-in-the-middle attack to gain valid company credentials. You are tasked with demonstrating the impact of the breach and gaining access to business critical information.


![](https://i.pinimg.com/control2/736x/22/25/3b/22253bf858c4d62c5100892e1b217b3b.jpg#center)



For this lab we have been given credentials to start with ;


```
Password: **********
Username: annette.palmer@megabigtech.com
```

We can go ahead and use the below command to authenticate to Az Services for further enumeration


```powershell
Connect-AzAccount
```

We can check the resources our current user has access to by using the below command


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-AzResource  

## Output

Name              : megabigtech-dev
ResourceGroupName : megabigtech-dev_group
ResourceType      : Microsoft.Web/sites
Location          : eastus
ResourceId        : /subscriptions/ceff06cb-e29d-4486-a3ae-eaaec5689f94/resourceGroups/megabigtech-dev_group/providers/Microsoft.Web/sites/megabigtech-dev
Tags              : 

```

We have got access to the resource group named `megabigtech-dev_group`, which serves an Azure Web App. We can go ahead and get the enabled host names.


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> (Get-AzWebApp -ResourceGroupName "megabigtech-dev_group" -Name "megabigtech-dev").EnabledHostNames

megabigtech-dev.azurewebsites.net
megabigtech-dev.scm.azurewebsites.net
```



---


***According to pwnedlabs*** ;


> Any time an app is created, App Service creates a `Kudu` companion app for it that allows us to manage the app instance, including getting terminal access. The location of this app can vary depending on the configuration.

  

- `https://<app-name>.scm.azurewebsites.net` (if the app isn't in an isolation tier)
- `https://<app-name>.scm.<ase-name>.p.azurewebsites.net` (if the app is internet-facing and in an isolated tier)
- `https://<app-name>.scm.<ase-name>.appserviceenvironment.net` (if the app is internal and in an isolated tier)"

***Further Explanation*** ;

![](https://i.imgur.com/BrQHnyH.png)


---


Accessing the `kudu` Az App service we have an authorization error for our present authenticated user


![](https://i.imgur.com/5op62R2.png)


However accessing the other host name gotten from our previous enumeration we have a functioning website

![](https://i.imgur.com/6TZNOHb.png)


Clicking the "**Status**" session as shown in the above image we are referred to a URL with a parameter.


![](https://i.imgur.com/d4cGD9u.png)


It is worth checking for LFI/Path Traversal vulnerabilities, Go ahead and intercept this request with burpsuite and send the request to the "**Repeater**"  tab.


![](https://i.imgur.com/mpiBMof.png)


Firstly we can confirm if we truly have a path traversal vulnerability, by changing the value of the endpoint as shown below.


![](https://i.imgur.com/rNhao1I.png)


Instead of `../latest` we can use the URI location `../status.aspx` to see if this works and as shown below we have the following output confirm that this vulnerability exists


![](https://i.imgur.com/hvIE09o.png)


Since the path in the URI worked let go ahead and fuzz for other paths, we can do this using burp also by sending the request from "**Repeater**" to "**Intruder**" and highlighting the full path including the parameter to be fuzzed



![](https://i.imgur.com/NlQZwk8.png)



For the wordlist you can go ahead and use this as provided by  [PwnedLabs](https://raw.githubusercontent.com/emadshanab/WordLists-20111129/refs/heads/master/Directories_Common.wordlist), Also go over to the "**Options**" tab and enable both options as shown below to avoid future errors.


![](https://i.imgur.com/IjrCGAu.png)



Running the attack as shown below and filtering by "**Length**" we can see that we have two `403` status code prompt meaning the paths exist but we don't have permissions to view this paths.



![](https://i.imgur.com/39HaPZG.png)



> Since we  have a path traversal vulnerability and reading files via this way doesn't imply whether we have permissions to read the file or not, nevertheless we can still read the file, sweet!!


We can go ahead and fuzz for files in the `/admin` endpoint, since it is the most juicy, don't forget to append the `.aspx` extension as this server is an `ASP.NET` server as shown in the below screenshot


![](https://i.imgur.com/U9tBKL2.png)


Once implemented you can go ahead and start the attack, just to understand this more, the below image should help ;


![](https://i.imgur.com/CyxcWSs.png)



Filtering by "**Status Code**" we can see that we successfully have access to the `/admin/login.aspx` endpoint as shown in the below screenshot



![](https://i.imgur.com/UOlf7t6.png)



We can go back to the repeater tab and try to read the source code of this login file as shown below


![](https://i.imgur.com/xsdBMDw.png)


We are referred to `../admin/login.aspx.cs` so let go ahead and see what we have there, whereas sending the request we have hardcoded credentials ;


![](https://i.imgur.com/cZu7fCg.png)



We can navigate to the login endpoint and login with the credentials we have at hand 


![](https://i.imgur.com/IbrSE3p.png)



Be cool ⋆˚🐾˖°