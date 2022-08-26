---
Title: FreeIPA Introduction
tags:
  - FreeIPA
date: 2016-12-18 12:17:32
---


# FreeIPA Introduction

## About
In order to make my infrastructure more easy to manage, I decide to install a FreeIPA server.
One of my former colleague and now friend talk me a lot about this product times ago.
So I decided to try it.


NB. This article is not finish yet, so some typos can still be included or paragraph not yet formatted.

### Objectives
Install a functional FreeIPA server to:

- manage users, groups, automount
- centralise ssh user public keys


This is the first step, I will add new modules like

 - FreeOTP


### Requirements
 - a CentOS machine (10G HD System, 1 VCPU, 2G RAM )
 - a local domain name like myplace.local

### References
This article is base on:

 - [FreeIPA](http://www.freeipa.org/page/Quick_Start_Guide "FreeIPA Quick Start Guide")
 - [UnixMen](https://www.unixmen.com/configure-freeipa-server-centos-7/ "FreeIPA on CentOS 7")

## Core Installation

### Prepare the machine
Add your machine ip and name in your /etc/hosts file

    echo "$(ip -o -4 addr show  | grep -v " lo" | head -1 | awk {'print $4'} | cut -d'/' -f1) $(hostname) $(hostname -s)" >> /etc/hosts

That would add something like :

    10.0.0.1 ipaserver.myplace.local ipaserver


### Install the software

Install the requirements to set up everything :

    yum install ipa-server bind-dyndb-ldap ipa-server-dns

Prepare the following infos before executing the installation process:

 - domain name (myplace.local) will be determined based on machine hostname, you just have to confirm it
 - Directory Manager password
 - IPA admin password
 - A DNS Forwarder

Then execute the following command and **answer the questions**

    ipa-server-install --setup-dns


At the end the following message will appear :
> Be sure to back up the CA certificates stored in /root/cacert.p12
> These files are required to create replicas. The password for these 
> files is the Directory Manager password

**Backup this file !! **

#### Firewall
if you want to use the local firewall on the centos, allow incoming traffic:

    firewall-cmd --permanent --add-service=ntp
    firewall-cmd --permanent --add-service=http
    firewall-cmd --permanent --add-service=https
    firewall-cmd --permanent --add-service=ldap
    firewall-cmd --permanent --add-service=ldaps
    firewall-cmd --permanent --add-service=kerberos
    firewall-cmd --permanent --add-service=kpasswd

If you want to use your FreeIPA to handle also the PKI part, add the RootCA, located in /etc/ipa/ca.crt, in your firefox or system's CA.


## Configure FreeIPA

Everything can be done through a web interface or commands lines.
I will use command line.
The web interface is reachable through https://ipaserver.myplace.local/


### User Management

A NFS Server will be used in order to have a unique point of storage for homedir.
This will avoid multiple time the same data over multiple machine.

The NFS Server will be a freebsd server with ZFS Storage.

### NFS Mount configuration for homedir

On FreeIPA Server execute the following command:

    ipa automountmap-add default auto.home

    Added automount map "auto.home"
    
      Map: auto.home


    ipaserver# ipa automountkey-add default --key "/exports/home" --info auto.home auto.master

    Added automount key "/exports/home"
    
      Key: /exports/home
      Mount information: auto.home


    ipaserver# ipa automountkey-add default --key "*" --info "-fstype=nfs4,rw,sec=krb5,soft,rsize=8192,wsize=8192 nfssrv.myplace.local:/exports/home/&" auto.home
    
    Added automount key "*"
    
      Key: *
      Mount information: -fstype=nfs4,rw,sec=krb5,soft,rsize=8192,wsize=8192 nfssrv.myplace.local:/exports/home/&

On Linux machines ensure that home dir will be created automatically if it doesn't exist.

    authconfig --enablemkhomedir --update


### Linux
#### Linux NFS Server

Source :

 - [blog delouw](https://blog.delouw.ch/2015/03/14/using-ipa-to-provide-automount-maps-for-nfsv4-home-directories/)


Configure sssd + krb5.conf

    ipa service-add nfs/nfssrv.myplace.local
    ipa-getkeytab -s ipasrv.myplace.local -p nfs/nfssrv.myplace.local -k /etc/krb5.keytab



#### Linux IPA Client

    ipa-server# ipa host-add ipaclient2.myplace.local --ip-address=A.B.C.D
    ipa-server# ipa service-add nfs/ipaclient2.myplace.local
    
    
    ipa-client# yum install ipa-client
    ipa-client# mkdir -p /exports/home
    ipa-client# authconfig --enablemkhomedir --update
    ipa-client# generate /etc/krb5.conf
    ipa-client# kinit
    ipa-client# ipa-getkeytab -s ipasrv.myplace.local -p nfs/${HOSTNAME} -k /etc/krb5.keytab

Then :

    ipa-client# ipa-client-install

Or

    ipa-client# ipa-client-install --domain=myplace.local --server=ipaserver.myplace.local --realm=MYPLACE.LOCAL

To finish with:
    
    ipa-client# ipa-client-automount --location=default


Ensure that the automount is working execute the following command:

    mount | grep auto

The result should be like :

>auto.home on /exports/home type autofs (rw,relatime,fd=18,pgrp=19004,timeout=300,minproto=5,maxproto=5,indirect)

If not edit /etc/nsswitch.conf
find the automount line, sss may be missing it should be like :

automount:  files sss

Fix it, restart sssd and autofs.

### FreeBSD
#### FreeBSD NFS Server


#### FreeBSD IPA Client

Source :
 - https://blog.hostileadmin.com/2016/03/24/integrating-freebsd-w-freeipasssd/
 - https://forums.freebsd.org/threads/46526/

On the FreeBSD Client (not yet functional), compile sssd with smb support 
Edit *sssd/Makefile*  and add the option ***--with-krb5-conf=/etc/krb5.conf*** 

(thx https://community.riocities.com/freebsd_nfv4_krb.html)

    freebsd_srv# mkdir -p /usr/compat/linux/proc
    freebsd_srv# echo "linproc /usr/compat/linux/proc linprocfs rw 0 0" >> /etc/fstab
    mkdir /var/log/krb5

**edit /etc/pam.d/system**

TBF

On the FreeIPA Server execute the following commands

    ipaserver# ipa-host-add FREEBSD_FQDN
    ipaserver# ipa-getkeytab -s ${HOSTNAME} -p host/FREEBSD_FQDN -k <location to export the keytab>

Now, copy the newly created keytab file to */etc/krb5.keytab* on the FreeBSD Client

Add TLS_CACERT /etc/ipa/ca.crt in /usr/local/etc/openldap/ldap.conf


### Rules

#### Allow some users to connect throught ssh

Based on http://www.freeipa.org/page/Howto/HBAC_and_allow_all

Create a new rule name allow_ssh

    ipaserver# ipa hbacrule-add allow_ssh
    
    Added HBAC rule "allow_ssh"
    
      Rule name: allow_ssh
      Enabled: TRUE

Associate newly created HBAC rule

    ipaserver# ipa hbacrule-add-service allow_ssh --hbacsvcs=sshd
    
      Rule name: allow_ssh
      Enabled: TRUE
      Services: sshd
    
    Number of members added 1


Associate a User to this rule

    ipaserver# ipa hbacrule-add-user allow_ssh --user=username_allowed
    
      Rule name: allow_ssh
      Enabled: TRUE
      Users: username_allowed
    
    Number of members added 1

    ipaserver# ipa hbacrule-add-host allow_ssh --hosts=ipaclient.myplace.local
    
      Rule name: allow_ssh
      Enabled: TRUE
      Users: username_allowed
      Hosts: ipaclient.myplace.local
    
    Number of members added 1


#### Allow some users to excute commands with sudo
In the previous section about allowing ssh command, I've written the command output, but in the following one, I will not.

Before allowing user to execute specific command through sudo, we need to allow the user to access sudo.

    ipaserver# ipa hbacrule-add allow_sudo
    ipaserver# ipa hbacrule-add-service allow_sudo --hbacsvcs=sudo
    ipaserver# ipa hbacrule-add-user allow_sudo --user=username_allowed
    ipaserver# ipa hbacrule-add-host allow_sudo --hosts=ipaclient.myplace.local

Ok now let's allow the user execute a command like whoami.

Beware that sssd is a caching system, so I can take time to refresh the data (up to 6h)
 - Incrementally, meaning only changes to rule since the last full update (ldap_sudo_smart_refresh_interval, the time in seconds); the default is 15 minutes,
 - Fully, which dumps the entire caches and pulls all the current rules on the LDAP server(ldap_sudo_full_refresh_interval, the time in seconds); the default is six hours.

Source [sssd-ldap-sudo](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/sssd-ldap-sudo.html "sssd-ldap-sudo")

    ipaserver# ipa sudorule-add whoami
    ipaserver# ipa sudocmd-add /usr/bin/whoami
    ipaserver# ipa sudorule-add-allow-command whoami --sudocmds /usr/bin/whoami
    ipaserver# ipa sudorule-add-host whoami --hosts ipaclient.myplace.local
    ipaserver# ipa sudorule-add-user whoami --users username_allowed


Now if we want to allow executing sudo whoami without autenticate the user we add this:

    ipaserver# ipa sudorule-add-option whoami --sudooption '!authenticate'
