---
title: LDAP Monitoring with AMA and Microsoft Sentinel
date: 2023-08-23 08:00:00 -0500
categories: [siem, active directory]
tags: [active directory,ldap,siem,sentinel,ama]     # TAG names should always be lowercase
---
According to Microsoft. LDAP is “an application protocol for working with various directory services. Directory services, such as Active Directory, store user and account information, and security information like passwords. The service then allows the information to be shared with other devices on the network. Enterprise applications such as email, customer relationship managers (CRMs), and Human Resources (HR) software can use LDAP to authenticate, access, and find information.”. 

This Protocol provides clients the ability to open a simple bind to an Active Directory domain controller in order to establish an authenticated session.  However, the major problem with this simple bind is that the initial credentials from the client to the domain controller are sent in clear text and that could allow a man-in-the-middle attacker to successfully forward an authentication request to a Windows LADP server that hasn’t been configured to required signing or sealing on incoming connections.

Therefore, it’s important to monitor those events in our SIEM.

## Enabling LDAP events in Domain Controllers

To begin with, LDAP logging can be set on domain controllers to help you identify where insecure LDAP bind attempts are coming from. we can do it by Powershell or Regedit.

* Using Powershell:

```
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\NTDS\Diagnostics' -Name "16 LDAP Interface Events" -Value 2 -PropertyType DWORD -Force
```

* Using Regedit

Going to **HKLM\\SYSTEM\\CurrentControlSet\\Services\\NTDS\\Diagnostics** and setting the value of the registry called **16 LDAP Interface Events** to 2.

![regedit](https://i.imgur.com/ri9HFRV.png)

**NOTE**: In case, you need a verbose logging to review a security event, it should set '*15 Field Engineering* ' to '5'.

### Events of Interest

* **Event ID 2886**: it indicates  that **LDAP signing is not being enforced by your domain controller**.
* **Event ID 2887**: This event is logged each time a client computer attempts an unsigned LDAP bind.
* **Event ID 2889**: The event is logged whenever a client makes an LDAP bind that this directory Server is not configured to reject.

## Sending Logs To Sentinel

Now, it’s turn to install and selecting the events to ingest in Microsoft Sentinel. To do this task, it’s needed the following steps:

1. Install AMA (Azure Monitor Agent) in the DC and select the events of interest to ingest.

First, we have to go to Azure Monitor → Data Collection Rule → Create Data Collection Rule

![DCR](https://i.imgur.com/MdN4RXA.png)

In Resources, we add the DC to monitor and the log analytics workspace where the logs will be ingested

![dcrr](https://i.imgur.com/tkSxFf6.png)

![law](https://i.imgur.com/8hManS2.png)

2. Create the Xpath  query to ingest events of interest.

In collect and deliver tab, we have to go to custom and then add the specific [XPath](https://go.microsoft.com/fwlink/?linkid=2159994)  query to ingest LDAP events in the Log Analytics Workspace. For that reason, it’s mandatory to use the event viewer to know how the Xpath query should be.

![xpath](https://i.imgur.com/2lOm0mO.png)

Finally, the XPath query should look similar to this:

```
'Directory Service'!*[System[(EventID=2889 or EventID=2887 or EventID=2886)]]
```

![xpath](https://i.imgur.com/IdYeyN7.png)

3. Testing the events were being sent to the log analytics workspace

First, we have to do a simple LDAP bind. For this case, the bind connection was made from the same controller using [ldp](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771022(v=ws.11)) tool.

After some minutes, it’s possible to see those logs in the Log analytics Workspace.

![log](https://i.imgur.com/DzhJDn0.png)

As we can see, LDAP monitoring enabled. Now, it’s possible to use the Workbook called Insecure Protocols to visualize insecure LDAP connections.

![workbook1](https://i.imgur.com/bi4kwgv.png)




