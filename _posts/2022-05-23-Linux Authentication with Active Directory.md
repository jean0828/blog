---
title: Linux Authentication with Active Directory
date: 2022-05-23 19:20:00 -0500
categories: [Linux, Active Directory]
tags: [windows,active directory,elastic]     # TAG names should always be lowercase
---

This is a tutorial to access with an Active Directory user in a linux server.

For this tutorial we have the following devices:
* Active Directory Server: IP 10.0.0.4
* Linux Server: IP 10.0.0.5

## Instalation of Active Directory
1. we need a Windows server and adding the role of Active Directory.

![AD server installation](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/ADserver1.jpg)
![AD server installation part 2](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/ADserver2.png)
![AD server installation part 3](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/ADserver3.png)
![AD server installation part 4](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/ADserver4.png)
![AD server installation part 5](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/ADserver5.png)

2. Configure the domain

In this case the domain name is: *lab.local*

![AD server configuration](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/adconfiguration.png)
![AD server configuration part 2](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/adconfiguration2.png)
![AD server configuration part 3](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/adconfiguration3.png)

then next > install. Afterward, the server will reboot.


![AD server configuration part 4](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/adconfiguration4.png)

## Check Connectivity

3. Ensure the Linux server (in this case Centos7) responds to the domain.

![Linux connectivity](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/linuxserver.png)
![Linux connectivity part 2](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/linuxserver2.png)

## Install requirements in Linux

4. Install the following requirements in order to ensure a proper integration.

`yum install sssd realmd oddjob oddjob-mkhomedir adcli samba-common samba-common-tools krb5-workstation openldap-clients policycoreutils-python`

 ![requirements](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/requirements.png)
 
 ## Join the server to the Active Directory
 
 5. with an anccount of Active Directory execute this command:
 `realm join --user=[userAD] [domain name]`
 
  ![join](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/joindomain.png)
 
 6. check that the server was joined
 
 * type those commands: `realm list`, `id [userAD]@[domain name]`


 ![check](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/check.png)
 ![check part 2](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/check3.png)
 
 * see the server in AD

![check part 3](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/check2.png)

7. modify sssd configuration

modify the file /etc/sssd/sssd.conf from this:

![sssd](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/sssd.png)

to this:

![new sssd](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/sssd2.png)

This is used to allow auth without the domain name. 

* restart the service

![restart](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/restart.png)

* check that you can request info without write `user@domain` with this command: `id [userAD]`

![id user](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/sssd3.png)

## Auth with an ADUser in the Linux Server

We can do this by SSH. However, for this example I used only `su` command.

![auth](https://github.com/jean0828/AD-integration-Linux/blob/main/ADtutorial/userauth.png)


# Security Consideration

At this point all users from the Active Directory domain can log in the Linux server. However, they do not have sudo privileges. From a security view, we need to ensure that only allowed users can access to the server and not all. To solve that, we need to do the following steps:

* deny access for all users with this command: `realm deny --all`
* permit specific users or groups of Active Directory with those commands: `realm permit [user]@domain_name`, `realm permit -g [groupname]@domain_name`
* restart sssd service



