---
title: Deploying Wazuh in a home lab
date: 2022-09-04 12:30:00 -0500
categories: [siem, security]
tags: [wazuh,sentinel,siem]     # TAG names should always be lowercase
---

In this post we're going to deploy Wazuh. According to its website, Wazuh is  is a free and open source security platform that unifies XDR and SIEM capabilities. It protects workloads across on-premises, virtualized, containerized, and cloud-based environments.

Wazuh helps organizations and individuals to protect their data assets against security threats. It is widely used by thousands of organizations worldwide, from small businesses to large enterprises.

# Downloading Wazuh image

[In this link](https://documentation.wazuh.com/current/deployment-options/virtual-machine/virtual-machine.html) we can download the OVA to deploy Wazuh in Vmware or VirtualBox. After download the OVA, we only have to import it and start it.

![vmware](https://i.imgur.com/qyvCSxH.png)

the credentials to log in are:
* user: wazuh-user
* pass: wazuh

Then, we should check the ip address to access in a browser with:

```
ip a
```

In this case is: **192.168.197.139**

the Wazuh dashboard can be accessed from the web interface by using the following credentials:

* URL: https://<wazuh_server_ip>
* user: admin
* pass: admin

# Deploying agents

in the modules section, we confirm there are no agents enrolled in our Wazuh server and we select **add agent** to enroll our clients.

![no-age](https://i.imgur.com/znizZrf.png)

Next, we should select the options that match with our client where it will be installed the agent. In this scenario, Windows 10 and Ubuntu will be enrolled to Wazuh server

For Windows 10, those are the options:

* operating system: Windows
* Wazuh server address: 192.168.197.139 (ip address of Wazuh server)
* agent group: default
* copy the command to execute in Powershell in the client:

```
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.3.7-1.msi -OutFile ${env:tmp}\wazuh-agent-4.3.7.msi; msiexec.exe /i ${env:tmp}\wazuh-agent-4.3.7.msi /q WAZUH_MANAGER='192.168.197.139' WAZUH_REGISTRATION_SERVER='192.168.197.139' WAZUH_AGENT_GROUP='default'
NET START WazuhSvc
```

* start wazuh service in windows client with NET START WazuhSvc

![windows-agent](https://i.imgur.com/pjPCaax.png)


In ubuntu, the steps are similar, we only have to change the operating system to **Debian / Ubuntu** and the command is:

```
curl -so wazuh-agent-4.3.7.deb https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.3.7-1_amd64.deb && sudo WAZUH_MANAGER='192.168.197.139' WAZUH_AGENT_GROUP='default' dpkg -i ./wazuh-agent-4.3.7.deb
```

![ubuntu-agent](https://i.imgur.com/FNh5p45.png)

Finally, we start and confirm the wazuh agent is working corretly with:

```
systemctl daemon-reload
systemctl enable wazuh-agent --now
systemctl status wazuh-agent
```

# Exploring Dashboards

Accessing from the web interface, we can confirm the agents enrrolled:

![agents-enrolled](https://i.imgur.com/RDSz6DH.png)

Finally, we can see the information we cna obtain in Wazuh like security events and results of compliance:

![exploring-dash](https://i.imgur.com/jYzj3zZ.png)

![exploring-dash2](https://i.imgur.com/FbvEtDU.png)

![exploring-dash3](https://i.imgur.com/dd6MdEY.png)

![CIS](https://i.imgur.com/g1XXa8n.png)