+++
title = "Am I a hecker yet 🤔? CVE-2024-44871 and CVE-2024-44872!"
description = "The big question....."
date = 2024-09-13T13:44:33+01:00
+++

## **WayBack**

Three months ago, smack in the middle of my first semester university exams, I found myself pondering a big question: **"What minor skill can I pick up to keep my hacking chops sharp?"** Something that would make me feel like a "hecker" and not just another stressed-out student.

And then, a light bulb moment! 💡 CVEs! I dove headfirst into the wonderful (and sometimes daunting) world of CVEs. I came across three fantastic resources that became my road map to success:

1. [8 CVEs in 2 Weeks is Crazy](https://youtu.be/2VB4Zd5C8N8?si=o7twn_Tuh3cXAK7X) – Spoiler: it really is.
2. [Tips on Getting Your First CVE](https://www.linkedin.com/posts/seunghwan-yoon-936a90229_cve-cve-activity-7236717980829523968-b9UB?utm_source=share&utm_medium=member_desktop) – Trust me, this was pure gold!
3. [Found a CVE? How to Report](https://medium.com/@dub-flow/found-a-vulnerability-3-easy-steps-to-submitting-a-cve-012148533650) – Because finding is one thing, but reporting is a whole different ball game.

![](https://i.imgur.com/7sr51WN.png#center)


## **The Hunt Begins**

Armed with new knowledge and a thirst for exploits, I decided to go big or go home. I started poking around open-source platforms like GitHub, using some good old-fashioned Google dorks like:


```
inurl:"/docs/" "powered by PHP" OR "PHP CMS"
```

I aimed my sights at CMS platforms, focusing primarily on the PHP, Apache2, and MySQL stack. And then, jackpot! 🎰 I hit on a CMS called [Mozilo](https://www.mozilo.de/), a popular German CMS. **Hecker Mode: Activated!**


## **The Workflow of a Hecker**

So, what did my journey look like from finding the vulnerabilities to getting them published? Let me break it down:

- [x] **Vulnerability Discovery**: After digging around in Mozilo, I found some vulnerabilities that made me raise an eyebrow (or two).
- [x] **Vendor Notification**: I immediately reported the vulnerability to the Mozilo team on **Aug 19, 2024, at 11:59 AM**.
- [x] **To MITRE We Go!**: With no vendor response, I submitted the vulnerabilities to MITRE on **Aug 19, 2:37 PM**, marking the vendor confirmation as “No.”
- [x] **The Waiting Game**: The status was set to "**Reserved**" on **Sep 5, 2:39 PM**.
- [x] **Let's Publish!**: I sent a publication request to MITRE on **Sep 6, 5:19 PM**. Patience is a virtue… but I was getting impatient!
- [x] **Success!**: Finally, on **Sep 10, 5:38 PM**, the CVEs were published! 🎉


![](https://raw.githubusercontent.com/sec-fortress/sec-fortress.github.io/main/images/alhwr_w_athr.gif#center)



## **So… Am I a Hecker Yet?** 🤔



I've been asking myself this question a lot lately. I mean, I’ve got two fresh CVEs to my name, and they’re not just any CVEs – we’re talking **Authenticated Remote Code Execution (CVE-2024-44871)** and **Authenticated Cross-Site Scripting (CVE-2024-44872)**. You can check them out here if you’re curious:

- [CVE-2024-44871](https://vulners.com/vulnrichment/VULNRICHMENT:CVE-2024-44871)
- [CVE-2024-44872](https://vulners.com/vulnrichment/VULNRICHMENT:CVE-2024-44872)

So, according to the cyber security community, I'm starting to make a name for myself. But if you ask my family? Oh, boy. 😅

You’d think two CVEs would be enough to prove my “hecker” credentials. Yet, my family still doesn't get it. My mom’s idea of my hacking career? She thinks it's all about recovering her friend’s hacked WhatsApp account.

```
Mom: "Olaoluwa, can you help recover my friend's WhatsApp? It's been hacked!"

Me: "Mom, I just found a remote code execution vulnerability in a popular CMS."

Mom: "So… can you get her WhatsApp back or not?"
```

Honestly, I don’t even know where to start with that. 😂 I try explaining that I find vulnerabilities in software, not people’s social media, but to her, all hackers are the same. In her eyes, I’m still a long way from being a "real hecker."


![](https://i.pinimg.com/originals/3b/90/32/3b90320474ef6c53000b4b7210d92d74.gif#center)


## **The Support (or Lack Thereof)**

Despite the hilarious misunderstanding, my family's support has been unwavering, even if they don't fully understand what I do. From asking me if I’m going to be the next James Bond to warning me about "getting caught by the FBI," they've been there for all my highs and lows. And, hey, at least they're interested in my “hecking,” even if they think it involves more cloak-and-dagger than code and dorks.

## **How Far We've Come**

These past few months have been a rollercoaster – from long nights hunting for vulnerabilities to navigating the CVE reporting process. And even though my family doesn’t quite see the gravity of my work, I’ve met some amazing people who do. The cybersecurity community has been incredibly welcoming.

New opportunities and connections are coming up all the time, and I’m getting to work on projects that genuinely excite me. Every day is a new adventure – even if it does involve explaining (for the hundredth time) that I can’t actually help anyone recover a hacked Facebook account.

But hey, maybe one day I’ll be able to say, “Yes, mom, I’m a _real_ hecker.” Until then, I’ll keep doing what I love, one vulnerability at a time.

And who knows? Maybe next time I’ll find something even bigger… or figure out how to recover that WhatsApp account. 🙃


![](https://i.pinimg.com/originals/18/e8/71/18e871f68c8cb3c183c4289fae9e313b.gif#center)


