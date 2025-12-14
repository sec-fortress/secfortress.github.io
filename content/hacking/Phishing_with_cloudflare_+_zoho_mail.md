---
date: '2025-12-14T17:17:07+01:00'
title: 'Phishing with cloudflare + zoho mail'
---

## The Hook

There is only one reason why this blog is up; I tried running a phishing engagement requested by a client and "Gophish" + "SMTP" setup had me hit a few blockers than I thought, due to compliance policies and time constraints.

Even after reaching out to DO and Amazon for a valid reason like, "the client is aware and this is done in a well controlled and authorized environment"; to use their SES, it still didn’t sit well with their policies. And honestly, I completely understand. Some platforms simply won’t bend their rules, no matter how legitimate your use case is.

![Image](https://i.imgur.com/ABQQeBP.png)

> Blocker 1: DigitalOcean blocks outbound SMTP connections by default to prevent spam and abuse. This meant I couldn’t connect GoPhish to external SMTP servers at all, instantly halting the initial setup pathway.



![Image](https://i.imgur.com/qZc46ug.png)


![Image](https://i.imgur.com/SS6tQVE.png)

> Blocker 2: Amazon SES allows sending only to **verified email addresses and domains** when your account is in the _sandbox_ environment. To move out of this restricted state, you must submit a detailed request explaining your use case and proving it is safe, compliant, and legitimate. Even with proper justification, approvals are not guaranteed and timelines can be unpredictable.




## The Process

You can go ahead and buy a similar looking domain to the original one at [https://domains.cloudflare.com/](https://domains.cloudflare.com/)


![Image](https://i.imgur.com/8wAVpU2.png)

Upon successful purchase of the domain, navigating to **DNS** -> **Records**, you should have an empty dns record list like the below  

![Image](https://i.imgur.com/6t80IWV.png)

Navigate to [https://www.zoho.com/mail/signup.html](https://www.zoho.com/mail/signup.html) to create a mail account and fill in the requirements as splay


![Image](https://i.imgur.com/9OdcRX5.png)

You should get a 7 digit OTP code to verify your account in the Email provided


![Image](https://i.imgur.com/eVPFmQQ.png)

Fill in the OTP to continue

![Image](https://i.imgur.com/HAPYmfq.png)

We can skip the below page by starting the 15 days free trial (No credit card required)

![Image](https://i.imgur.com/B2FWhj6.png)

Next we can go ahead and select "**Add an existing domain**"

Then add up the domain we created on cloud flare before  and fill the organization name respectfully

![Image](https://i.imgur.com/S6hqDgK.png)

![Image](https://i.imgur.com/McXG6zr.png)


You can then choose to authenticate 

![Image](https://i.imgur.com/xnI5SUF.png)




Once this is done navigate to https://directory.zoho.com/directory/[ORGANIZATION]/adminhome#/users/new 

- Select "**USERS**"
- Fill up field respectively imitating target email pattern
- Then click "**Add**"

![Image](https://i.imgur.com/aOwg3qT.png)

> Below form imitates hr department at target organization

![Image](https://i.imgur.com/0IjupPq.png)

## Progress In Between Problems

There are two blockers I encountered and solutions to solving them:

Outlook/Gmail Deliverability Requirements:

1. you need to either find a way to boost the domain or work with the client’s IT team to allow list/approve your phishing domain.
2. Without allowlisting, emails may land in Spam due to Microsoft’s filtering rules.
3. Send test emails to confirm inbox delivery before launching the full campaign.
4. The above mostly doesn't affect gmail client.

Using Client EML/ZIP Samples for Template Accuracy:

1. It is possible to request sample EML files from HR/IT strategically.
2. Or even better get it via an ongoing red team engagement.
3. Then we can Import the EML files into Zoho Mail to fine-tune your simulation, ensuring the messages align with the organization’s communication style and improve overall engagement accuracy.


Login to zoho mail with the previous created account and click on the settings Icon at the top right


![Image](https://i.imgur.com/7Di7W0F.png)


You should see a pop up window, scroll down and select "**Import/Export Emails**" >> "**Browse for eml or zip file**"


![Image](https://i.imgur.com/QpJ73Cy.png)

- Once this is done, the email template should be imported to your **inbox**
- If an email template has an image email signature, download the image and add it manually to zoho


### Re-adding email signature


Navigate to settings as done previously


![Image](https://i.imgur.com/oXBPmh9.png)

Click on the drop down to add the downloaded image email signature


![Image](https://i.imgur.com/DKIQs5E.png)

You can set other preferences also to make it comfortable sending multiple emails

![Image](https://i.imgur.com/fBiH5Rd.png)

> Please ignore the dummy email signature as shown above 

### Exporting mails from outlook/gmail to import to zoho

Below are the following steps to export an email template from outlook (Ask client to do this)

![Image](https://i.imgur.com/2E5p22D.png)

Below are the following steps to export an email template from gmail

![Image](https://i.imgur.com/PrcVYPC.png)


## The Tough Question


Since this is a phishing campaign, a common question that often comes up is: How do we accurately measure success?
How do we determine the percentage of users who clicked a link, submitted credentials, attempted to log in, or interacted with the payload in any way?

While metrics are important, the more critical factor is the environment you’re testing. Every organization is different, and an effective phishing simulation requires a deep understanding of the client; how they communicate, what they’re accustomed to, and what they are most likely (or unlikely) to fall for.

Before execution, it’s essential to study the client thoroughly to understand these behavioral patterns. That insight ultimately determines the realism and effectiveness of the campaign.

To address this, I’ll walk you through a process I developed in collaboration with a highly skilled DevSecOps engineer. Together, we designed an approach that significantly improved the success of the phishing engagement. Below is a breakdown of the metrics we used to evaluate the campaign’s performance.

![Image](https://i.imgur.com/sP92gzs.png)

![Image](https://i.imgur.com/DdVXNro.png)

Unfortunately, this is where I’ll have to bring this blog to an end. There are still many details I’m unable to disclose for personal and confidentiality reasons. That said, I genuinely hope this write-up helps someone navigating similar challenges or planning a similar engagement.

Sometimes, _sharing just enough is enough to make a difference._