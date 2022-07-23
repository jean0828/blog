---
title: How to Configure CIS Compliance Report in Nessus
date: 2022-07-22 23:50:00 -0500
categories: [security, vulnerability scan]
tags: [nessus,cis compliance,vulnerability scan]     # TAG names should always be lowercase
---

Vulnerability scanner is a system designed to assess computers, networks or applications for known weaknesses. This system is a key part in any security program because we can automate those scans and centralize those results for better analysis in remediation. For that reason, we're going to install and configure Nessus Manager to check CIS compliance against a Windows Server.

## Installation Nessus

1. Clic in *try* in the following web page: https://www.tenable.com/products/nessus to get an email for installation guide
2. Once you have the email, you'll active the account and in *My Trials* section
3. Download nessus manager package. In this case, it will be installed in Kali Linux OS

![download-nm](https://i.imgur.com/4rWTSDZ.png)

4. update Kali Linux

```
apt update
```

![update-kali](https://i.imgur.com/v3iskBb.png)

5. install Nessus package with the follwing command:

```
dpkg -i Nessus-<version number>-debian6_amd64.deb
```

![install-nessus](https://i.imgur.com/dlgFDZQ.png)

6. start and enable Nessus service with those commands:
```
systemctl start nessusd.service
systemctl enable nessusd.service
```

7. Configuring nessus.

After the Nessus service starts, use a web browser to navigate to the Nessus Web Interface at: https://localhost:8834/.

![web-nessus](https://i.imgur.com/xruyBSh.png)

copy and paste the code obtained in the step 2

![web-nessus2](https://i.imgur.com/mopGF2I.png)

Then, create an username and password. for this case the username is *nessus_adm* and the pass is *1QazxsW@*

![web-nessus3](https://i.imgur.com/FHPmoXS.png)

Next, Nessus will take some minutes to initialize its plugins.

![web-nessus4](https://i.imgur.com/9S2ICeC.png)

## Creating a scan

Now, that we have all installed, we can create a compliance scan to see how our good or bad is a Windows server according to CIS. For this example, we have another Windows server 2016 and it will be tested with CIS L1. To do that, we can do the following steps:

* create a compliance scan
* select target (IP Address) 
* provide credentials
* select the benchmark

![create](https://i.imgur.com/5fq00lD.png)
![target](https://i.imgur.com/CqKXRA2.png)
![credential](https://i.imgur.com/IScB8SL.png)
![cis](https://i.imgur.com/LZTML5T.png)

Now, we can save and launch the scan.

![results](https://i.imgur.com/1cY5hUx.png)

## Creating a custom baseline based on CIS

it's important to know, that our server doesn't have any baseline, it only has all configurations by default. Therefore, the server only has a 84 items passed.
However, we can create a custom baseline based on CIS benchmark because an organization can focus on specific items according to the risk that they want to take it. To ilustrate this situation, **imagine that our company only has as a baseline a password policy and only wants to see those configurations:**.

* Account lockout duration
* Enforce password history
* Interactive logon: Machine account lockout threshold
* Maximum password age
* Minimum password age
* Minimum password length
* Password must meet complexity requirement
* Reset lockout counter after
* Network security: Force logoff when logon hours expire

In this situation, we have to download the CIS bechmark for Windows server 2016 here: https://www.tenable.com/audits/

![cis-file](https://i.imgur.com/j8fGQHQ.png)

after download the file, we can edit with visual studio or any text editor. the CIS benchmark file looks like this:

![cis-file2](https://i.imgur.com/VYVja9f.png)

```
    <custom_item>
      type           : LOCKOUT_POLICY
      description    : "Account lockout duration"
      info           : "Account lockout duration

This security setting determines the number of minutes a locked-out account remains locked out before automatically becoming unlocked. The available range is from 0 minutes through 99,999 minutes. If you set the account lockout duration to 0, the account will be locked out until an administrator explicitly unlocks it.

If an account lockout threshold is defined, the account lockout duration must be greater than or equal to the reset time.

Default: None, because this policy setting only has meaning when an Account lockout threshold is specified."
      solution       : "Policy Path: Account Policies\Account Lockout Policy
Policy Name: Account lockout duration"
      reference      : "800-171|3.1.8,800-53|AC-7a.,CN-L3|8.1.4.1(b),CSCv6|16.7,GDPR|32.1.b,HIPAA|164.306(a)(1),ITSG-33|AC-7a.,NESA|T5.5.1,NIAv2|AM24,TBA-FIISB|45.1.2,TBA-FIISB|45.2.1,TBA-FIISB|45.2.2"
      see_also       : "https://blogs.technet.microsoft.com/secguide/2016/10/17/security-baseline-for-windows-10-v1607-anniversary-edition-and-windows-server-2016/"
      value_type     : TIME_MINUTE
      value_data     : [15..MAX]
      lockout_policy : LOCKOUT_DURATION
    </custom_item>
```
after modifying the file, we can edit the scan in Nessus and see the new results.

![custom-cis](https://i.imgur.com/8ewTaeA.png)

![results-custom](https://i.imgur.com/MfSXTAV.png)

![results-custom2](https://i.imgur.com/Fiz6Fot.png)

it can be seen that the compliance report is according to the items that we want. Furthermore, if those items are configured, it will show in the next scan as a PASSED status.

![remediate](https://i.imgur.com/3dOcZhh.png)

if we see the history we see that we have improved the compliance report, remediating some items.

* Before:
![before](https://i.imgur.com/s6ScRvj.png)

* After
![after](https://i.imgur.com/70DphYe.png)

Finally, if there is no budget to purchase a vulnerability tool like Nessus, we can explore open source tools like this: https://github.com/0x6d69636b/windows_hardening. Then, we can create a central repository to save all the outputs and finally trying to create a dashboard with the results.
