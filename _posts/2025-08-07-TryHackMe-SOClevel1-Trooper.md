---
layout: post
title: TryHackMe Room SOC Level 1 Trooper
---

![Trooper]({{site.baseurl}}/assets/images/Trooper/trooper.png)

#### Use Cyber Threat Intelligence knowledge and skills to identify a threat based on a report.

A multinational technology company has been the target of several cyber attacks in the past few months. The attackers have been successful in stealing sensitive intellectual property and causing disruptions to the company's operations. A threat advisory report about similar attacks has been shared, and as a CTI analyst, your task is to identify the Tactics, Techniques, and Procedures (TTPs) being used by the Threat group and gather as much information as possible about their identity and motive. For this task, you will utilise the OpenCTI platform as well as the MITRE ATT&CK navigator, linked to the details below.

Start the virtual machine and utilize OpenCTI and ATT&CK Navigator

#### APT X Report
![aptxreportpg1]({{site.baseurl}}/assets/images/Trooper/aptxreportpg1.png)
![aptxreportpg2]({{site.baseurl}}/assets/images/Trooper/aptxreportpg2.png)

#### Question 1: What kind of phishing campaign does APT X use as part of their TTPs?

In the report and in the ATT&CK Navigator spear-phishing emails are mentioned for this APT group <br>

Answer: spear-phishing emails

#### Question 2: What is the name of the malware used by APT X?

Most of the provided report is regarding the malware USBFerry

Answer: USBFerry

#### Question 3: What is the malware's STIX ID?

For this I looked up the malware in OpenCTI :
![usbferrystixid]({{site.baseurl}}/assets/images/Trooper/usbferrystixid.png)

Answer: malware--5d0ea014-1ce9-5d5c-bcc7-f625a07907d0

#### Question 4: With the use of a USB, what technique did APT X use for initial access?
Going back to ATT&CK Navigator under the “initial access” section USBFerry clearly falls under Replication Through Removable Media which is also highlighted for the APT group.

Answer: Replication Through Removable Media

#### Question 5: What is the identity of APT X?

This was quickly found through opening other articles hyperlinked in the report
![tropictrooperOpenCTI]({{site.baseurl}}/assets/images/Trooper/tropictrooperOpenCTI.png)

Answer: Tropic Trooper

#### Question 6: On OpenCTI, how many Attack Pattern techniques are associated with the APT?
Going to the Knowledge tab we can see the number of attack patterns.  To see them listed and to find out more about each one, you can click on attack patterns under Arsenal on the far right
![tropictrooperOpenCTI]({{site.baseurl}}/assets/images/Trooper/attackpatterns.png)

Answer: 39

#### Question 7: What is the name of the tool linked to the APT?

![tropictrooperOpenCTI]({{site.baseurl}}/assets/images/Trooper/bitsadmin.png)
In a summary of their recent campaign, this is how the BITSAdmin tool is used in the attack chain: a system configuration file (in.sys) will drop a backdoor installer (UserInstall.exe) then delete itself. The backdoor installer will drop a normal sidebar.exe file (a Windows Gadget tool, a feature already discontinued by Windows), a malicious loader (in "C:\ProgramData\Apple\Update\wab32res.dll"), and an encrypted configuration file. UserInstall.exe will abuse the BITSadmin command-line tool to create a job and launch sidebar.exe.

Answer: BITSAdmin

#### Question 8: Load up the Navigator. What is the sub-technique used by the APT under Valid Accounts?

In the ATT&CK Navigator dashboard, expanding Valid accounts:

![tropictrooperOpenCTI]({{site.baseurl}}/assets/images/Trooper/validaccounts.png)

Answer: Local Accounts

#### Question 9: Initial Access, Persistence,  Defense Evasion and Privilege Escalation
Valid and local accounts can be used across multiple stages of the attack kill chain.


Answer: Initial Access, Persistence,  Defense Evasion and Privilege Escalation

#### Question 10: What technique is the group known for using under the tactic Collection?

![tropictrooperOpenCTI]({{site.baseurl}}/assets/images/Trooper/automatedcollection.png)

Answer: Automated collection

We can pull up this attack pattern on OpenCTI to get more info and courses of action:

![autocollectionopencti]({{site.baseurl}}/assets/images/Trooper/autocollectionOpenCTI.png)

`Room completed`
