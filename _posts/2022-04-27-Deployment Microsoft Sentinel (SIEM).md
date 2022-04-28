---
title: Deployment of Microsoft Sentinel (SIEM)
date: 2022-04-27 20:20:00 -0500
categories: [Azure, security]
tags: [azure,sentinel,siem]     # TAG names should always be lowercase
---

# Sentinel SIEM

## Table of contents
1. [Create Azure Subscription](#subscription)
2. [Create a VM in Azure](#VM)
3. [Create a log repository (Log Analytics Workspace)](#workspace)
4. [Set up Azure Sentinel (SIEM)](#sentinel)

## Creating Azure Subscription <a name="subscription"></a>

We need to visit [Azure web site](https://azure.microsoft.com/en-us/free/). There,  you should click in "Start Free" and continue with the steps.

![Azure Homepage](/assets/img/media/azure-homepage.png "Azure homepage"){: width="700" height="400" }

Now, we can sign in portal.azure.com.

![Azure portal](/assets/img/media/azure-portal.png "Azure portal"){: width="700" height="400" }

## Creating VM in Azure <a name="VM"></a>

In order to create our VM, we click on "Virtual machines" option. Next, click in "Create" &rarr; "Azure virtual machine" with the following parameters:

* Resource group: create new &rarr; "SentinelLab"
* virtual machine name: server-vm
* Region: Central US
* user and pass

This server will serve as a vulnerable machine. Therefore, In the networking section, we are going to create a NSG (Network Security Group) which allows all traffic. The configurations is this:

* NIC network security group: advanced
* configure network security group: create new &rarr; 
* create inbound rule: from any to any by all protocols with priority 100

**NOTE**: You shouldn't do this in a real enrivonment.

![Inbound rule](/assets/img/media/inbound-rule.png "Inbound rule"){: width="700" height="400" }

Finally, click in "review+create"

![creating vm](/assets/img/media/vm-creating.png "creating vm"){: width="700" height="400" }

when the VM is deployed, we should test our RDP. To do that, we can run "mstsc" and test our connection.


![rdp vm](/assets/img/media/rdp-connection.png "rdp vm"){: width="700" height="400" }

## Creating a log analytics workspace <a name="workspace"></a>

Now the goal is to ingest logs from the virtual machine. For that, it needs to create a log analytics workspace. here is a description of what log analytic is:

*Log Analytics collects data from a variety of sources and uses a powerful query language to give you insights into the operation of your applications and resources. Use Azure Monitor to access the complete set of tools for monitoring all of your Azure resources.*

After, in the search bar we type ***log analytics workspaces*** &rarr; *create log analytics workspace*.

for this example, we select those parameters:

* resource group: the same of the VM
* name: LAW-Sentinel
* region: Central US

![workspace](/assets/img/media/create-law.png "workspace"){: width="700" height="400" }

Finally, Sentinel will connect to this workspace in order to display the data

### Enablig Microsoft Defender for Cloud

Now, it's time to enable gather logs from VM into log analytics workspace. So, you need to search it and go to *environment settings* &rarr; select the subscription &rarr; auto provisioning &rarr; Log Analytics agent for Azure VMs to on &rarr; connect Azure VMs to a differente workspace: LAW-Sentinel

![settings](/assets/img/media/settings.png "settings"){: width="700" height="400" }

![enable-law-auto](/assets/img/media/enable-law-auto.png "enable-law-auto"){: width="700" height="400" }

Afterward, we should back to the log analytics workspace to connect the VM

![law-vm](/assets/img/media/law-vm.png "law-vm"){: width="700" height="400" }

![law-vm-connect](/assets/img/media/law-vm-connect.png "law-vm-connect"){: width="700" height="400" }

**Note**: the VM must be running to connect with the log analytics workspace 

## Set up Sentinel <a name="sentinel"></a>

type *Microsoft Sentinel* in the search bar &rarr; create Microsoft Sentinel &rarr; select the workspace

![sentinel](/assets/img/media/sentinel.png "sentinel"){: width="700" height="400" }

we have a 30-day trial:

*Microsoft Sentinel free trial activated:
The free trial is active on this workspace from 26/4/2022 to 27/5/2022 at 23:59:59 UTC.
During the trial, up to 10 GB/day are free for both Microsoft Sentinel and Log Analytics. Data beyond the 10 GB/day included quantity will be billed*

### Monitoring failed login attempts

first, we need to understand how an failed login attempt looks in the vm:


![logon-failure](/assets/img/media/logon-failure.png "logon-failure"){: width="700" height="400" }

* event ID: 4625
* workstation name: name of source machine
* Source Network Address: source IP


