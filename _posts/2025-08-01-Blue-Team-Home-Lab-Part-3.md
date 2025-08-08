---
layout: post
title: Blue Team Home Lab Part 3
---

## Overview

At this point, I have the Firewall (pfSense), Ubuntu hosting my SIEM (Wazuh), and a Kali box on VLAN1. On VLAN10, I have Ubuntu hosting docker containers and Portainer deployed for managing them.  I’m going to install Wazuh agents to the Kali box, the firewall, and the Docker host so I can get some connection/logs generated in Wazuh.

### Deploying Wazuh Agents on the Linux Endpoints

I followed this doucumentation to install the agent on Ubuntu and Kali: [documentation.wazuh.com/installation-guide/](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html){:target="_blank"}

First, we install the GPG key add the Wazuh repository: <br>
`curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg` <br>
`echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list` <br>
update the package info: <br>
`apt-get update`

Next we assign the manager IP and install the Wazuh-agent: <br>
`WAZUH_MANAGER="10.10.1.51" apt-get install wazuh-agent` <br>

Lastly, we enable and start the Wazuh agent service: <br>
`systemctl daemon-reload` <br>
`systemctl enable wazuh-agent` <br>
`systemctl start wazuh-agent` <br>

NOTE: At the time of this writeup, 4.12 is the current release.  To ensure compatibility, check that the Wazuh manager and agent are both on the same version.

### Additional Config for Monitoring docker

Additional configuration is needed on the Ubuntu host in VLAN10 for container security and can be found here: [documentation.wazuh.com/current/user-manual/capabilities/container-security/monitoring-docker/](https://documentation.wazuh.com/current/user-manual/capabilities/container-security/monitoring-docker.html){:target="_blank"}

Python Docker library is needed for the Docker Engine API so we can run this to get it on the Ubuntu Server: <br>
`pip3 install docker==7.1.0 urllib3==1.26.20 requests==2.32.2`<br>

Add the following configuration to the Wazuh agent configuration file `/var/ossec/etc/ossec.conf` to enable the Docker listener: <br>

Restart the Wazuh agent to apply the changes: <br>
`systemctl restart wazuh-agent` <br>


Once these settings are configured, on the left side of the Wazuh GUI go to Server Management > Settings > Docker listener and it should be enabled.

### Deploying Wazuh Agent on pfSense

We will need to do some extra configuration in our pfSense VM as well. First, in the pfSense GUI go to System > Advanced > Admin Access and click the radio button to enable secure shell. Next, we need to enable FreeBSD package repositories to install the Wazuh agent (they are disabled by default for security purposes). <br>
We can hit `8` on the main pfSense console to access the Shell. ALternatively, we could SSH in from the kali box.

go to `/usr/local/etc/pkg/repos/` and in both `pfSense.conf` and `FreeBSD.conf` edit the code to reflect enabled: `FreeBSD: { enabled: yes }` <br>

We can now install the agent: <br>
`pkg update` <br>
can run a `pkg search` to ensure 4.12 is the version we're installing <br>
`pkg install wazuh-agent` <br>

We can disable the package repositories again now. <br>

To ensure the same timezone is used between the agent and the manager:
`cp /etc/localtime /var/ossec/etc/` <br>

Just like with the other endpoints, we will update the manager IP at `/var/ossec/etc/ossec.conf` <br>
`<server>` <br>
	`<address>10.10.1.51</address>` <br>
`</server>` <br>

Now we enable the agent with `sysrc wazuh_agent_enable="YES"` <br>
create a symlink to retain file integrity: <br>
`ln -s /usr/local/etc/rc.d/wazuh-agent /usr/local/etc/rc.d/wazuh-agent.sh` <br>
and start the agent: <br>
`service wazuh-agent start` <br>

these instructions and the manual are provided upon install:

![wazuh-agent-instructions]({{site.baseurl}}/assets/images/Blue-Team-Home-Lab-Part-3/wazuh-agent-instructions.png)

voila! we have some connected agents!

![wazuh-agent-instructions]({{site.baseurl}}/assets/images/Blue-Team-Home-Lab-Part-3/wazuh-dashconnected.png)

In the left side directory go to Agent Management > Groups and create a group named pfSense.  We will then click manage agents and add the pfSense agent to the group.<br>

If you go to edit pfsense group it will bring up agent.conf where we can add the following code to specify the log format and define the log file location: <br>
`<agent_config>` <br>
  `<localfile>` <br>
  `<log_format>syslog</log_format>` <br>
`</agentconfig>` <br>

If you go to Server Management > Rules and search pfsense, you'll see there are already 2 rules:

![wazuh-agent-instructions]({{site.baseurl}}/assets/images/Blue-Team-Home-Lab-Part-3/wazuhrules.png)

Based on these configurations, the firewall drop event does not log by default, so we can create a new rule to see these events in our SIEM. One way to do this is to take the firewall drop event rule code that contains the `<options>no_log</options>`, copy it, and remove the portion that says `<options>no_log</options>` into a new rule like so:<br>

![newrule]({{site.baseurl}}/assets/images/Blue-Team-Home-Lab-Part-3/newrule.png)

Going back to the Wazuh dashboard, I can see some logs coming in and I will leave at this baseline configuration for now.

## Overview

Wazuh agents have now been installed on the Kali box, the Ubuntu server hosting my containers (with Docker monitoring is configured), and on the pfSense firewall.  To save some of my limited hard drive space I could set up cron jobs so these logs aren't constantly generating. I want to get some more VMs installed and start simulating attacks and detections though, so I will figure out the cron job configs later. I plan to do some writeups specifically on Wazuh custom rules and decoder creations as I dive deeper into this project.  I've also started looks at what NID I want to add to this project (maybe Snort or Security Onion). Until next time!
