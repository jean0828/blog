---
title: Monitoring Powershell commands with Elastic Stack
date: 2022-05-22 20:20:00 -0500
categories: [Windows, security]
tags: [windows,siem,elastic]     # TAG names should always be lowercase
---

In some cases it's important to monitor all the powershell commands executed in a windows server because it can help us to alert possible attacks and lateral movements. For that reason, in this post it will show how to active powershell commands in the event viewer of Windows and then using an agent to send those logs for visualization.

# Enabling Powershell commands in event viewer

We can enable those settings by using an Active Directory environment or locally in the server. In this example it's made by AD using the GPO editor:  ```Administrative Templates``` &rarr; ```Windows Components``` &rarr; ```Windows PowerShell```:
* *Turn on Modle Logging*
* *Turn on Powershell Script Block Logging*

![Policy-powershell](https://i.imgur.com/iv8blvu.png){: width="700" height="400" }

# Instaling agent for monitoring Windows event with Elastic Stack

In this case, elastic agent was installed in the windows server. [For details about installation click here](https://www.elastic.co/guide/en/fleet/7.17/elastic-agent-installation.html). Then in kibana you have to create a policy to monitor those Windows channels: *Microsoft-Windows-Powershell/Operational channel*  y *Windows Powershell channel*

![elastic-policy](https://i.imgur.com/FXKRHKz.png){: width="700" height="400" }

# Testing that Powershell commands are sent to Elastic Stack

![pw-command](https://i.imgur.com/4X6gYn7.png){: width="700" height="400" }

Use this filter to see Powershell commands in Kibana:

* *event.provider*: PowerShell
* *Powershell.command.value*: exist

![kibana-discover](https://i.imgur.com/KF9shVo.png){: width="700" height="400" }

Furthermore, we can use the dashboard created by Elastic called **[Windows powershell] Overview** with useful information:

![kibana-dashboard](https://i.imgur.com/niJgPiE.png){: width="700" height="400" }

