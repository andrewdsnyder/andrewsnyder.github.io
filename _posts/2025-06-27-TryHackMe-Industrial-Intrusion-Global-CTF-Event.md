---
layout: post
title: TryHackMe Industrial Intrusion Global CTF Event
---

In this team-based CTF competition, there were multiple challenges that involved exploiting vulnerabilities to gain control of the provided infrastructure. I joined a team with 4 other people from all over the world.

We coordinated our plan of attack and progress via Discord, Trello, and a shared Google Docs page.


## OSINT 1

“Hexline, we need your help investigating the phishing attack from 3 months ago. We believe the threat actor managed to hijack our domain virielia-water.it.com and used it to host some of their infrastructure at the time. Use your OSINT skills to find information about the infrastructure they used during their campaign.”

After doing a Google search for virielia-water.it.com I found the domain contained a typo and the company's legitimate domain is virelia-water.it.com. I also noticed that it was hosted on Github so I searched the correct domain in the GitHub Code search

![OSINT1search]({{site.baseurl}}/assets/images/Industrial-Intrusion/OSINT1search.png)

The hexidecimal in the first part of the domain was converted to a string using CyberChef to reveal the flag
![OSINT1flag]({{site.baseurl}}/assets/images/Industrial-Intrusion/OSINT1-Cybercherf.png)

## OSINT 2

“Great work on uncovering that suspicious subdomain, Hexline. However, your work here isn’t done yet, we believe there is more.”

A fallback DNS was found
![fallbackDNS]({{site.baseurl}}/assets/images/Industrial-Intrusion/osint2img3.png)

and I ran a dig command to query the TXT records
![osint2dig]({{site.baseurl}}/assets/images/Industrial-Intrusion/osint2dig.png)

this revealed a base64 string which ended up being the flag
![osint2dig]({{site.baseurl}}/assets/images/Industrial-Intrusion/OSINT2Cyberchef.png)

The dig command (domain information groper) is used to entorragate DNS name servers and TXT records are used to store text-based information about a domain.

## OSINT 3

"After the initial breach, a single OT-Alert appeared in Virelia’s monthly digest—an otherwise unremarkable maintenance notice, mysteriously signed with PGP. Corporate auditors quietly removed the report days later, fearing it might be malicious. Your mission is to uncover more information about this mysterious signed PGP maintenance message."

I searched Github code based on the Virelia footer message and found a PGP-signed message.

![osint3img1]({{site.baseurl}}/assets/images/Industrial-Intrusion/osint3img1.png)

We were able to verify the authenticity and pull the matching key which contained the flag.

![osint3img1]({{site.baseurl}}/assets/images/Industrial-Intrusion/osint3.png)

A key server (in this case keyserver.ubuntu.com) contains the regular index of matching keys. GPG (or GNU Privacy Guard) is an open-source software suite that allows users to encrypt files, digitally sign documents, and verify the authenticity of messages.  When a message is signed with a PGP like this, it also provides non-repudiation.  With PGP (pretty good privacy), a session key is created and encrypted using the sender's public key.  The sender then sends the encrypted PGP session key to a recipient and they can decrypt it using their private key and the session key.

## Rogue Poller

"An intruder has breached the internal OT network and systematically probed industrial devices for sensitive data. Network captures reveal unusual traffic from a suspicious host scanning PLC memory over TCP port 502.
Analyse the provided PCAP and uncover what data the attacker retrieved during their register scans."

A PCAP was provided and I opened it in Wireshark. Modbus data is not encrypted and in this case using port 502. I applied a filter in Wireshark to display all packets with TCP port 502
![roguepoller1]({{site.baseurl}}/assets/images/Industrial-Intrusion/rogue-poller-portfilterwireshark.png)
 right clicked a packet > follow…> TCP Stream which revealed the flag: THM{1nDu5tr14L_r3g1st3rs}
![roguepoller2]({{site.baseurl}}/assets/images/Industrial-Intrusion/rogue-poller-tcpstream.png)
