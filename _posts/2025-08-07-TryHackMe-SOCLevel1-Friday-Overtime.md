---
layout: post
title: TryHackMe Room SOC Level 1 Friday Overtime
---

#### Step into the shoes of a Cyber Threat Intelligence Analyst and put your investigation skills to the test.

Disclaimer
Please note: The artefacts used in this scenario were retrieved from a real-world cyber-attack. Hence, it is advised that interaction with the artefacts be done only inside the attached VM, as it is an isolated environment.

![fridayovertime]({{site.baseurl}}/assets/images/FridayOvertime/fridayovertime.png)

#### Hello Busy Weekend. . .
It's a Friday evening at PandaProbe Intelligence when a notification appears on your CTI platform. While most are already looking forward to the weekend, you realise you must pull overtime because SwiftSpend Finance has opened a new ticket, raising concerns about potential malware threats. The finance company, known for its meticulous security measures, stumbled upon something suspicious and wanted immediate expert analysis.
As the only remaining CTI Analyst on shift at PandaProbe Intelligence, you quickly took charge of the situation, realising the gravity of a potential breach at a financial institution. The ticket contained multiple file attachments, presumed to be malware samples.
With a deep breath, a focused mind, and the longing desire to go home, you began the process of: <br>
1. Downloading the malware samples provided in the ticket, ensuring they were contained in a secure environment.
2. Running the samples through preliminary automated malware analysis tools to get a quick overview.
3. Deep diving into a manual analysis, understanding the malware's behaviour, and identifying its communication patterns.
4. Correlating findings with global threat intelligence databases to identify known signatures or behaviours.
5. Compiling a comprehensive report with mitigation and recovery steps, ensuring SwiftSpend Finance could swiftly address potential threats.

First lets launch the VM and login to the DocIntel platform VM with the provided credentials. <br>

![pandaProbeLogin]({{site.baseurl}}/assets/images/FridayOvertime/pandaprobeLogin.png)

Here is the email from the analyst giving the rundown and .zip of the malware files:

![OliverBennettEmail]({{site.baseurl}}/assets/images/FridayOvertime/olverBennettEmail.png)

#### Question 1: Who shared the malware samples?
Answer: Oliver Bennett

#### Question 2: What is the SHA1 hash of the file "pRsm.dll" inside samples.zip?
To get this, I extracted the zip file using the password provided from Oliver, opened the terminal and typed `sha1sum pRsm.dll`<br>
Answer: 9d1ecbbe8637fed0d89fca1af35ea821277ad2e8

#### Question 3: Which malware framework utilizes these DLLs as add-on modules?
After a quick Google search I found the files I just unzipped were mentioned in this report: [welivesecurity](https://www.welivesecurity.com/2023/04/26/evasive-panda-apt-group-malware-updates-popular-chinese-software/){:target="_blank"} analyzing a backdoor MgBot malware used by APT group Evasive Panda. These additional modules are added after legitimate updates likely from supply-chain compromise and/or adversary-in-the-middle attacks.  The specific function of the pRsm.dll is to capture input and output audio streams.

Answer: MGBot

#### Question 4:  Which MITRE ATT&CK Technique is linked to using pRsm.dll in this malware framework?
Answer: T1123 Audio Capture found here: [attack.mitre.org](https://attack.mitre.org/versions/v12/techniques/T1123/){:target="_blank"}

#### Question 5: What is the CyberChef defanged URL of the malicious download location first seen on 2020-11-02?
This question kind of threw me off for a minute, I poked around  MITRE ATT&CK Audio Capture references, MGBot, and wasn’t seeing that date anywhere.  I then went back to the report that I originally found and saw the URL in the technical analysis “Malicious download locations according to ESET telemetry”: http://update.browser.qq[.]com/qmbs/QQ/QQUrlMgr_QQ88_4296.exe I then went to CyberChef, removed the parenthesis and copied in the URL, dragged defang URL over and hit bake: <br>
![defangURLCyberCHef1]({{site.baseurl}}/assets/images/FridayOvertime/defangURLCyberCHef1.png)

Answer: hxxp[://]update[.]browser[.]qq[.]com/qmbs/QQ/QQUrlMgr_QQ88_4296[.]exe

#### Question 6: What is the CyberChef defanged IP address of the C&C server first detected on 2020-09-14 using these modules?

On this same article I found The IP mentioned: “122.10.90[.]12 AS55933 Cloudie Limited 2020-09-14 MgBot C&C server.“ <br>
I just put that IP in CyberChef without the parenthesis, drag and drop defang IP, and hit bake:
![cyberchefDefang]({{site.baseurl}}/assets/images/FridayOvertime/cyberchefDefang.png)

answer: 122[.]10[.]90[.]12

#### Question 7: What is the md5 hash of the spyagent family spyware hosted on the same IP targeting Android devices in Jun 2025?

I searched the IP 122.10.90.12 in VirusTotal, went to the Relations section. In July 2025, under Communicating Files, there was a type of "Android". <br>
![virustotalsearchRelations]({{site.baseurl}}/assets/images/FridayOvertime/virustotalsearchRelations.png)
 I clicked on the name of that file "951F41930489A8BFE963FCED5D8DFD79" hyperlink and saw it was labeled as "trojan.spyagent/andr". Over on the details page we can see the different hashes of this file <br>
Answer: 951f41930489a8bfe963fced5d8dfd79

`room completed`
