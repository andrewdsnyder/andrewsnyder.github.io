---
layout: post
title: Blue Team Home Lab Part 2
---

## Setting up Ubuntu/Docker/Portainer on VLAN10

This Ubuntu Server will host the Docker containers of vulnerable web apps that I will be able to use my security tools on.  This will be set up pretty much exactly the same as the Ubuntu server from part 1.


### Ubuntu Server VM setup on VLAN10

vmname: UbuntuServer
Allocation:  80 GB hard Disk space, 4GB of RAM and 2 processors for now.  SSH installed on setup and IP set to 10.10.10.50/24. Network adapter set to vmnet10

I spent way too much time trying to troubleshoot the vmtools copy/paste issue with the Ubuntu VM and decided I will just be SSH’ing (ssh andrew@10.10.10.50) from the kali box to set up configurations.


### Docker install and Deployment

Just follow the steps here for Docker install on Ubuntu: [docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/){:target="_blank"}

Run the following command to uninstall all conflicting packages:<br>
`for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done`<br>
I installed using the apt repository:<br>
`# Add Docker's official GPG key:` <br>
`sudo apt-get update` <br>
`sudo apt-get install ca-certificates curl` <br>
`sudo install -m 0755 -d /etc/apt/keyrings` <br>
`sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc` <br>
`sudo chmod a+r /etc/apt/keyrings/docker.asc` <br>

`# Add the repository to Apt sources:` <br>
`echo \`<br>
`"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \ `<br>
`$(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \ `<br>
`sudo tee /etc/apt/sources.list.d/docker.list > /dev/null ` <br>
`sudo apt-get update`<br>

install docker packages:<br>
`sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

to test that the install was successful:<br>
`sudo docker run hello-world`

### Portainer Install and Deployment

Next, Portainer will be installed. Portainer has been amazing to work with as it provides a nice GUI for deploying, configuring, etc.  
I followed these instructions to get it setup on my VLAN10 Ubuntu server: [docs.portainer.io](https://docs.portainer.io/start/install-ce/server/docker/linux){:target="_blank"}
To deploy, we first create the volume that Portainer Server will use to store its database:<br>
`docker volume create portainer_data`

Then, download and install the Portainer Server container:<br>
`docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:lts`

You can also check on portainer deployment status with the command `docker ps`<br>
To get to the web interface, go to the docker host server and the 9443 port provided in the install instructions: `10.10.10.50:9443` On first access it will prompt to create Portainer login credentials.

Next I need to go to Networks > add a network<br>
The first network added is the config: VLAN10-config<br>
I assigned a subnet range to these containers. Each web app having its own IP address will be useful for vulnerability scanning, and other detection from the SIEM.

![portainerNetworkConfig]({{site.baseurl}}/assets/images/Blue-Team-Home-Lab-Part-2/portainerNetworkConfig.png)

The second network added is the implementation of that config: VLAN10<br>
I clicked on the Creation box instead of Configuration > selected VLAN10-config from the drop down > create<br>
The containers can be created in portainer as pictured below. The only other config I did was go down to Networks tab at the bottom of the creation screen and select VLAN10.

![webappCreate]({{site.baseurl}}/assets/images/Blue-Team-Home-Lab-Part-2/webappCreate.png)

For now I’m just setting up Damn Vulnerable Web App and OWASP WebGoat. I will be able to plug and play here with anything else I want to deploy.  

![webappContainers]({{site.baseurl}}/assets/images/Blue-Team-Home-Lab-Part-2/webappContainers.png)
<br>
<br>

## Summary

I was able to get some more infrastructure on the home lab and successfully communicate between the firewall, VLAN1, and VLAN 10.  The plan for future containers is to deploy other web apps from Vulnhub and web apps that I create.  I’m itching to get some simulated incidents and DFIR going with the kali box and Tsurugi as well as vulnerability scans with Nessus, but before that I want to connect these endpoints to Wazuh.
