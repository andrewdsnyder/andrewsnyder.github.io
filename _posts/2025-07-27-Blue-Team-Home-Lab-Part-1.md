---
layout: post
title: Blue Team Home Lab Part 1
---

## Overview

Previously, I have been doing one-offs of a single LAN with a couple VMs to get my hands dirty with different configurations and practice with tools. For this project, I want to create a whole home network where I can plug and play as well as integrate more as I go.  After multiple videos/walkthroughs, a bunch of documentation reading, building up, changing my mind and tearing down, I came up with a diagram and a gameplan. This lab is being created in VMware on an upgraded T480 Thinkpad running Debian 12 as the host OS. Much of the inspiration and learning for this has come from videos/courses by Grant Collins (projectsecurity.io), Tech with Gerard, facyber, and tryhackme SOC1 learning path.

<br>
<br>
## Topology (v1)

![topologyv1]({{site.baseurl}}/assets/images/Blue-Team-Home-Lab-Part-1/topologyv1.png)
<br>
<br>
## VMware Network Setup

One of the main reasons for this lab is to be able to isolate these instances and tools from the internet and play with them in a safe environment.  In order to do that I need to setup some configurations in the Network Configuration Editor:

![networkConfig]({{site.baseurl}}/assets/images/Blue-Team-Home-Lab-Part-1/networkConfig.png)

Some of these can be ignored for now. The focus at the moment is vmnet2 and vmnet10
<br>
<br>
## Setting up pfSense Firewall

To start: I downloaded the netgate installer for pfSense from [pfsense.org/download](https://www.pfsense.org/download/)

Next, in VMware hit create VM and select the netgate-installer-amd64.iso.<br>
VM name: pfSense<br>
Allocation: 32 GB of disk space, 1 processor, and 2GB of RAM for now (this can be changed later if needed) <br>
To start we will have it connected to a NAT network adapter.  The install is straightforward and I used the defaults.  Once it reboots, it will bring you to the main console view in the CLI.  The network config took quite a bit of breaking and reconfiguring, but I found out it was the least fussy when I added the network adapters and assigned them one at a time.  To do this, go over to the VMware library list > right click pfSense > hit settings > + add button > custom > select vmnet2 from the dropdown.  em1 connecting messages will start to pop up in the console. In the console hit 1 for “assign interfaces”, enter “n” to skip setting up VLANs, enter “em0” for WAN, “em1” for LAN. Next hit 2 for “Set interface addresses”, we want WAN on DHCP.  10.10.1.254 for the LAN IP and 24 for the subnet, skip the IPv6 stuff and for the DHCP on the LAN I set the range from 10.10.1.50-10.10.1.100.  Next I go back to the VM settings and added another network adapter: vmnet10 and assigned, set the address in the same way with the 10.10.10.50-100 DHCP range.

![pfSenseConfig]({{site.baseurl}}/assets/images/Blue-Team-Home-Lab-Part-1/pfSenseConfig.png)

This is a good preliminary setup and we can add more later as we go along. I am going to create another VM within VLAN1 that will allow me to see the pfSense web GUI to set up VLANs, rules, etc.
<br>
<br>
## Kali Linux VM Setup in VLAN1

I downloaded the Pre-built Vmware VM from [kali.org](https://www.kali.org/get-kali/#kali-virtual-machines)<br>
VM name: kali linux<br>
Allocation:  80 GB disk space, 4 processors, and 3GB of RAM<br>
Once installed, go to settings and select Network Adapter. I believe this defaults to NAT but I changed this to custom vmnet2 and this will be the only network adapter on this machine.  Open Firefox and go to 10.10.1.254 to get to the pfSense GUI default login is admin/pfsense. Here we will change the name of OPT1 to VLAN10 under Interfaces, and make sure both LAN and VLAN10 interfaces are enabled and the IPs are correct in the “static IPv4 Configuration section:

![lanConfig]({{site.baseurl}}/assets/images/Blue-Team-Home-Lab-Part-1/vlanconfig1.png)

next we’ll go to  services > DHCP Server and  enable, set the range and DNS:

![lanConfig2]({{site.baseurl}}/assets/images/Blue-Team-Home-Lab-Part-1/vlanconfig2.png)

After that we can go to  Firewall > rules and copy the LAN rule and paste it to VLAN10 and edit it to VLAN10 subnets like so:

![lanrules]({{site.baseurl}}/assets/images/Blue-Team-Home-Lab-Part-1/lanrules1.png)<br>

we will then edit the new rule on VLAN10 and change the source to VLAN10 subnets:<br>

![lanrules2]({{site.baseurl}}/assets/images/Blue-Team-Home-Lab-Part-1/lanrules2.png)<br>
I will be adding more VLANs later on in this homelab build, but for now VLAN1 and VLAN10 will be what I focus on.

<br>
<br>
## Wazuh install on VLAN1

I installed Ubuntu Server 24.04.2 from here: [ubuntu.com/download/server](https://ubuntu.com/download/server)

I allocated 60GB of disk space, 4 processors, and 8 GB of RAM for this VM for now.

I wont be going through setting up Ubuntu or Windows servers for the most part because its pretty straight forward.  I will be selecting install SSH on setup for everything though which is not a default option. When setting the IPv4 configuration on setup, I set subnet to 10.10.1.0/24, the address to 10.10.1.51, used the gateway for VLAN1: 10.10.1.254, name servers 10.10.1.254 (firewall) and 8.8.8.8 (Google)

Once the VM is setup I will right click it in VMware > Settings and change the network adapter configuration to vmnet2.  

I followed the steps here to install Wazuh: [documentation.wazuh.com](https://documentation.wazuh.com/current/quickstart.html) and once installation is complete the console will provide admin credentials.

On the Kali machine in Firefox, the Wazuh dashboard can be accessed from the new Ubuntu VM address: 10.10.1.51 and login with the creds that were just provided in console.

Note: this is elementary but I’ll just throw it in anyways: If at any point I was wondering if the IP was correctly configured I did a ip a or ip -c addr show and to check connection between machines I pinged them.

<br>
<br>

## Part 1 summary

The pfSense firewall has been setup, I have a Kali box and Wazuh on VLAN1, and VLAN10 is configured and ready to go for VM/container creation
