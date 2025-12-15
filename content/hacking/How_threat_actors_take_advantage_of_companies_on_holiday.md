---
title: "How threat actors take advantage of companies on holiday"
date: 2024-11-30T13:41:27+01:00
---


## **Overview**


During holiday periods, alot of companies scale down operations, whereas employees take time off work and most of all security teams operate with minimal staff. **Threat Actors/Advanced Persistent Threats (APTs)** use this as an avenue to launch cyberattacks as there are reduction in vigilance making this a prime window to carry out attacks like ransomeware, DDOS, Phishing and many other sophisticated cyber attacks. Here are few common tactics used by adversaries during holidays and recommendations to mitigate the risks.


## **1. Supply Chain Attacks**



This attacks are also called third-party attacks which involves a threat actors using vulnerable third party tools or services to infiltrate systems or networks, leveraging these connections to access the target company. A real world example as shown in this [blog](https://techtarget.com/whatis/feature/SolarWinds-hack-explained-Everything-you-need-to-know), where the Attackers positioned malicious code into SolarWind's software updates. where as when companies installed these "trusted" updates, they unknowingly let the attackers into their systems via the inserted malicious code. Another real word example is the supply chain attack in Google’s Open-Source Project, "Flank", This occurred when attackers gained access to the project's GitHub repository through stolen credentials, allowing them to inject malicious code into a legitimate update, which was then unknowingly distributed to users, potentially compromising systems that integrated the tampered software, More information about this can be found in the following [blog](https://www.stepsecurity.io/case-studies/flank) also, including another X post from 'StepSecurity' showing a more precise way of how this was carried out. 


{{< x user="step_security" id="1779885444781736297" >}}


## **2. Phishing Campaigns**


This are popularly known as email scams which are used to steal Personally identifiable information (PII) from victims, Information such as credit card details, transfer funds, third party login details etc could be compromised, Most times the urgency or generosity of staffs/employees might be leveraged to trick them into clicking malicious links or revealing sensitive information. Below is an example of how a phishing email is crafted as shown in the image below :


![](https://www.terranovasecurity.com/sites/default/files/migration/scenario-3.png)
_19 Phishing email examples_. (n.d.). https://www.terranovasecurity.com/blog/top-examples-of-phishing-emails



## **3. Exploitation of Reduced Monitoring**

There are several attacks caused due to this like ransomeware and Distributed Denial Of Service (DDOS) attacks, Infact according to Semperis'"2024 Ransomware Holiday Risk Report", 86 percent of organizations surveyed in the U.S., UK, France and Germany, are targeted by ransomware on a holiday or weekend and the most reason is due to the fact that companies/organizations are under staffed during this period leading to fallout in systems due to data breaches. An example is the Los Angeles Unified School District ransomeware attack which occurred over Labor Day weekend which was launched by a Russian-speaking ransomware gang known as VIce Society. This was confimred according to a tweet made by the "*
Los Angeles Unified*" official account on **September 6, 2022**.


{{< x user="LASchools" id="1567019506299727872" >}}


## **4. Exploitation of Delayed Patch Management**

During holidays, IT teams may delay routine updates and patches, leaving systems vulnerable to available known exploits/Common Vulnerabilities and Exposures
 (CVEs). Threat actors might capitalize on this gap where as launching attacks on unpatched systems, often leveraging automated scanning tools like shodan, whcih is a search engine for Internet-connected devices, to find entry points.

Below is a shodan query to discover Apache HTTP servers targetting a specific version known to have a critical vulnerability for example CVE-2021-41773, which allows path traversal and remote code execution.


```
product:"Apache httpd" version:"2.4.49"
```


![](https://i.imgur.com/tpjiNVS.png)



## **Mitigation Strategies**


> **Haven't covered all of this, there are still alot of attacks threat actors use to compromise companies during holiday periods, as the list is extensive, Now let look into the mitigation strategies to avoid the ones we have talked about** :


**1. Proactive Monitoring** : With the use of automated threat detetion and response systems in place, attacks like the supply chain attack could be avoided to fill gaps during reduced staffing.


**2. Employee Training** : It is important for organizations to conduct holiday themed phishing simulations and trainings to raise awareness about seasonal scams.

**3. Incident Response Plans** : A proper inident response lifecycle with contigency plans like detection and analysis, containment, eradication, recovery and post-incident activities for rapid response should be in place during holiday periods including on call staff rotations.


**4. Zero-Trust Policies** : It is likewise important to limit access to sensitive systems and implement strict identity verification, especially for temporary or remote staff.

