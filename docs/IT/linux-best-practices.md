title: IT/Linux-Bests-Practices
date: 2016-04-20 08:50:04


## Overall
All those recommendations should be applied on servers and all environments (test, pre-production, production).
It can be also used on dev environments.

This document cover RHEL / Centos System and can be used for other unix systems

If you ask why I wrote this document that's because still in this year of 2016, some people or companies doesn't (**want to know|know**) how to manage their IT !!

## Requirements

### Time

Book some time to allow your IT team to do some labs, research keep inform about new technologies.
They will take it as a deep breath, change their daily and give you some feedback about new product that can make your company :
- more efficient
- attractive
- competitive
- may be cheaper
- avoid losing IT guys

### Hardware

Having hardware for testing is mandatory, you should try to have
a testing server for each model you have in production.
Maybe you can use your spare and test it in the same time.
It can sound like crazy but if you have multiple models in
production you can't bet on the total compatibility or using
another model as spare.
Event if linux is stable, sometime some chipsets can react in
different ways with the same OS baseline or with a different
distribution, kernel flavor.

Try to limit the HW model with two models a small one (1-2 CPUs)
and a bigger one (1-4 CPUs).
If you have any requirement about storage, think that it can be
done on dedicated boxes (nexenta, freebsd zfs filer, emc2, netapp)
or split over shards ([RedHat's ceph](https://www.redhat.com/fr/technologies/storage/ceph))

### Software

#### Support

If you use Linux in production with commercial software and if you don't have any linux guru in your team, some companies provide support for their OS like :
 - RedHat
 - Suse
 - Oracle

Other Linux distribution are supported by companies but not the editor.

Support provide you contact with the editor for some debugging & security issues.
They also provide in a short delay bug and security fixes on the version you use.


### Environment

Make multiple environments !!

Dev, Test, Qual, Production doesn't have the same architecture details and so SLA !!


### Infrastructure

The following points are mostly important component for a complete infrastructure, following products are just examples and I'm or was used to work with.

- Managed gigabit switch (yeah monitoring bandwidth is helpful ...)
- Servers with a remote management console (iKVM, HP ILO, Dell iDrac)
- Monitoring on all the IT Infrastructure with monitoring software
- Reporting with [JasperSoft](http://www.jaspersoft.com/), a free community edition is available.
- Inventory aka CMDB with [GLPI](http://glpi-project.org/) & [OCS Inventory](http://www.ocsinventory-ng.org/fr/)
- Workflow Tools aka BPM [Bonitasoft](http://fr.bonitasoft.com/)
- Versioning system ([GIT](https://git-scm.com/), also a big thanks to [@cicatrice](http://www.twitter.com/cicatrice)
for introducing me to [GOGS](https://gogs.io/)  a lightweight central repository
- DNS (private and if required public)
- NTP
- LDAP
- Syslog concentrator system
- IP Address Management aka IPAM
- True Firewalls with true filtering rules and no allowing **from ANY to ANY** is not a valid rule
- Local mirror for OS, software, packages deployment to avoid using your internet connectivity bandwith when you deploy and update servers
- Automation Tools :
  - OpenSource :
    - [The Foreman](http://www.theforeman.org/) and [Puppet](https://puppet.com/), also grouped under a unique project named [Katello](http://www.katello.org/)
    - [Ansible](https://www.ansible.com/)
  - Commercial : [RedHat Satellite](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux) based on katello

If you're asking why ansible and katello are both selected that's because they provide both services at different levels.

#### DNS infrastructure

Install a private DNS server to resolve internal and external zone (internet), and private NTP Server to sync your servers clock.
As you will allow in your firewall rules, that all your infrastructure to connect to these servers put them in a DMZ not in your lan.
If you can add a second servers it will increase the availability of the service.

???+ warning
Do not mix Public and Private DNS !!


## OS Installation

As we are talking about servers, there is no graphical gui installed on it, only ssh access for a good old terminal.
For those that are not familiar managing system with a terminal, this document can rude.

### Network

Machines must have a FQDN based a name template model defined by the company or the team.
Avoid to the maximum using public IP, Nat and Pat from public to private IP are working fine.
Don't forget to define reverse IP, it will help in debugging some situations.



### Storage

#### Split System & Data
The system and data must be separated on different disk for many reason.
 - IO performance, as system disk can be slower as the performance is focus on apps
 - Cost, high performance disks are expensives do not use them for the system
 - If you need to move the data disk to another system it will not embed system parts

#### System Partitions

Based on a Centos 7 minimal install (~292 Packages) will consume around 1.2 GB, but we need a disk of 15G for whole system.
This partition schema can be extended with LVM to fit requirements.

| Mount point | Size | Comment                                               |
| ----------- | ---- |-------------------------------------------------------|
| / | 2G | Root FS                                               |
| /boot | 500M | Boot partition                                        |
| /boot/efi | 200M | Uefi Partition (if machine configuration required it) |
| /var | 2G | variable data                                         |
| /var/log | 5G | Logs partition                                        |
| /var/tmp | 1G | Tmp file storage that survive to a reboot             |
| /tmp | 1G | Tmp file storage that is cleaned during boot          |
| swap | 2G | Swap partition                                        |

<br/>

[More information on Linux FS Hierarchy](http://www.tldp.org/LDP/Linux-Filesystem-Hierarchy/html/)

<br/>

???+ danger
This FS template is designed for a base system without any services.
If you want to install any software like MySQL or Apache, some modifications are required.

A home fs can be setup if you think that your users can store some data.

#### Application Partitions
You must use a or many dedicated disk(s) for your application.
Also partition schema should be defined to avoid any "surprise" like :
 - Someone activated the debug mode and logfiles increases faster that you can handle
 - you're under attack
 - you don't rotate your logs regularly
 - you don't compress old logs, and you keep them on the server
 - your backup is using a lot of space and make the disk full
 - and lot more...

You can use the following example.

| Mount point | Size | Comment |
| - | - | - |
| /srv | 100M | applications root folder |
| /srv/myapp | 100M | myapp root folder |
| /srv/myapp/app | XG | where binaries and libraries are stored to run myapp application (tomcat, mysql...) |
| /srv/myapp/data | XG | myapp data |
| /srv/myapp/logs | XG | myapp logs |
| /srv/myapp/tmp | XG | tmp folder for myapp |
| /srv/myapp/backups | XG | backup folder for myapp |


You can also add sub partition under data, backups and logs if you have multiple application component and if you think it's better.

Thank you [@cicatrice](https://twitter.com/cicatrice/) for showing me /srv partition

### Packages

Install only required packages nothing more !!
The more package you install on the machine, the more you will need to patch / upgrade.

First, you install the base system and in a second step a third party tool will deploy your required packages and configuration (Puppet, Chef, ...).
This method will allow you to modify your base applications' requirements without changing your installation process and all your active machines will inherit the new packages in the same time.

Also do not install any compiler on the server to avoid any security breach.

It's also recommended avoiding installing software from archive or third party tool (install shell).
Use the packaging system to deploy the software.

### Services

Disable useless services like wpa_supplicant, dhcpd, iptables, ip6tables, ipv6 if you don't need them.
They can be source of security breachs or false alerts.

## Users

First you have to clearly understand that you have two kind of users : 
 - physicals one aka human beings, coworkers, customers, providers
 - technicals one like apache, mysql...

Technicals accounts should not be used with password auth in most of the cases.
A sudo or ssh login should be enough.

Try to have a centralized tools that allow to have a clear overview of which application use which techical account.
In case of changing the password, you will not have a big surprise.

### Accounts

Avoid defining physical users on servers at the maximum, you can use a centralized infrastructure like LDAP or AD to manage your users.
Only technical or applicative users should be defined locally on the server like apache, mysql, ...
Even technical or applicative account can be defined in the LDAP system as SSSD subsystem will keep a local cache in case your LDAP infra is down.
If your User directory can be down for a time, create an account with sudo access to all commands, this account should be used only in case of emergency.

### Passwords
If the passwords are managed locally.
The users' password policy should be compliant with the company's policy.
PAM can be used to check the password compliance with the policy when the password is defined.

The major issue will be to distribute/synchronize the password to all machines.
If the password is the same, it can be defined with a configuration management software.

If you need to have a different password per server you will need to think it in a different way.
Some configuration management software may allow you to use override default value, so per server or group of server the password can be overrided.

Also, some software, mostly commercial, will allow you to manage from a central repository the password of your different account of the machine.
They will also allow you to change them on a defined frequency.

### UID / GID
In order to avoid any issues about software that have different uid/gid on different plateforme.
Ensure to prepare a mapping for all applications (internals or provided by third party software provider).
This will avoid issues about different uid/gid on machines

???+ warning
Beware of applicative account that will need credentials !!

Also don't forget that when some from the IT leave the department or the company.
Revoke immediately his access and change all passwords that he could have access!

## Manage the Server

### Connecting to the server

Use SSHv2 and avoid any weak protocols like ssh1, telnet, rlogin (yes still used in 2016 in some companies...)
Use standard account, direct root access should be used only in emergency.
You can do most of the check as a standard user, and it will avoid any mistake.

### Use SUDO !!!

You can use sudo to elevate your privileges and do some admin operations.
But don't use sudo with a shell as parameter or "sudo -s" it will not help to trace who make some errors or destroyed data.

### Use a centralized configuration management

Install a third party tool like puppet to allow you to :
 - ensure services are running or not
 - deploy configurations files
 - manage users
 - ensure systems inside a pool or a cluster are identical


### Manage your logs
Logs must be rotated on a daily basis, logrotate can help you doing it.
BTW logs must be also forwarded to a central point to avoid any deletion.

CF ElasticSearch Logstash Kibana stack (ELK).

### Crontab

Crontab should be created in the ** /etc/cron.d ** folder.
This method allow centralizing crontab in a single folder with explicits filenames, just avoid very long filename.

You should also deny access to the crontab for users, everything can be done in the ** /etc/cron.d ** folder.
Also, as users couldn't create their own crontab they will have to go through a process to submit new crontab and avoid some mistake or overlap during time period (tasks during backups).

Also, crontab must be redirected to a log file or to /dev/null, never in a local mailbox as they are never read, purged and can fill the associated FS.
If some information must be sent by mail, make it in the script.

### Job Scheduler / Task Management

Maybe using a Central Task Management system is a good idea instead of crons
https://www.sos-berlin.com/

### Release Management

Use a Release Management product like RedHat Satellite (spacewalk in community).
It will allow you to define multiple Channel Software Repository that are validated by IT Teams. Theses repository will be the source used on servers to install software from.

With this process, until new version are pushed in Channels, the version will not change and all machine subscribed to this channels will have the same software release.

You can manage as many channel as you want or need like :
 - Production RHEL 7
 - Test (Future Production) RHEL 7
 - Production HP Software
 - Your own software channel

Also, this Release Management Software will help you in making snapshot of the server before any updates to do a rollback if some issues occurs.


## OS Tuning

#### Network

[Linux Network Tuning 2013](http://www.nateware.com/linux-network-tuning-for-2013.html#.VxiJWj-rZ5Y)

#### IO Scheduler
By default the IO scheduler is CFQ.

 - CFQ
 CFQ places synchronous requests submitted by processes into a number of per-process queues and then allocates timeslices for each of the queues to access the disk. The length of the time slice and the number of requests a queue is allowed to submit depends on the I/O priority of the given process. Asynchronous requests for all processes are batched together in fewer queues, one per priority. While CFQ does not do explicit anticipatory I/O scheduling, it achieves the same effect of having good aggregate throughput for the system as a whole, by allowing a process queue to idle at the end of synchronous I/O thereby "anticipating" further close I/O from that process. It can be considered a natural extension of granting I/O time slices to a process.

???+ warning
source : [CFQ Scheduler](https://en.wikipedia.org/wiki/CFQ)


It can be change to the following :
 - noop
 recommended for SSD storage, and VM.
 The NOOP scheduler inserts all incoming I/O requests into a simple FIFO queue and implements request merging. This scheduler is useful when it has been determined that the host should not attempt to re-order requests based on the sector numbers contained therein. In other words, the scheduler assumes that the host is definitionally unaware of how to productively re-order requests.

???+ danger
  [Noop Scheduler](https://en.wikipedia.org/wiki/Noop_scheduler)

 - deadline
 recommanded for hypervisor and other machines
 The main goal of the Deadline scheduler is to guarantee a start service time for a request.[1] It does so by imposing a deadline on all I/O operations to prevent starvation of requests. It also maintains two deadline queues, in addition to the sorted queues (both read and write). Deadline queues are basically sorted by their deadline (the expiration time), while the sorted queues are sorted by the sector number.

???+ danger
  [Deadline Scheduler](https://en.wikipedia.org/wiki/Deadline_scheduler)


Beware that depending on the storage and the hardware the results can be different with huge delta, like with fc and multipath.
Another point to take care, is to test the infrastructure without any load in the same time to avoid any noise.


To change it permanently :
add "elevator=noop" to /boot/grub/menu.list in the corresponding kernel line like : ** kernel /vmlinuz-2.6.16.60-0.91.1-smp root=/dev/sysvg/root splash=silent splash=off showopts elevator=noop **

To change it dynamically :
On a specific disk :

``` shell
echo "scheduler_name" > /sys/block/<Disk_Name>/queue/scheduler
```

On all disks from emc:

``` shell
for disk in `ls -1 /sys/block |egrep '^emc|^sd'`;
do
  echo "deadline" > /sys/block/$disk/queue/scheduler;
done
```


## Security

### Auditd

With auditd you can track what happened to your server.
Which user with which program remove this file ...

But this tracking tool consume resources so use it with care (disk io and storage)

### Grub

If you have access to the machine console (physical, remote or virtual), you can reboot and use /bin/bash as init process.
With this trick you gain root access without any credentials.
You need to secure grub with a password.

### Packages / Programs

Do not install any compiler on the server.
If some compilers are installed, and you can't uninstall them, make them unavailable to everyone.
Raise alerts when packages are changes without any reason.

Avoid any setuid root program / scripts.

Any application that are executed on the server must be executed as another user than root except some product like apache that fork process directly with the good account.

Do not execute as root scripts that can be modified by a user, they could gain root access.

To keep tracks of changes on scripts deployed on your server use a versioning system (Git or SVN).
A configuration management can push the new version automatically.

### Users

Do not work as root or on the console when the machine is in maintenance mode !

Raise alerts when :
 - root is connected to the machine
 - same account is used from different locations
 - Failed login multiple time in a short time range and a long one.

### Filesystem

Make /tmp, /var/tmp and data FS structure in non-executable mode.

You can enhance with also
 - nodev – Do not allow character or special devices on this partition (prevents use of device files such as zero, sda etc).
 - nosuid - Do not set SUID/SGID access on this partition (prevent the setuid bit).

### Selinux

If you do know about Selinux, use it, it will increase the security for the system and of the applications.
If you don't know about it, do not activate it, it will block your applications.

### Password Policy

Avoid using the same password on every machine.

### SSH
SSH can be configured to use public key in a folder different from user's home.
With this method only admins can allow a public key to connect to an account, but private key must not be shared !!!

In the autorized_keys you can also set the ip source and the commands that are allowed to be executed.
ex : from="1.2.3.4",no-agent-forwarding,no-port-forwarding,no-X11-forwarding ssh-rsa ...

Also remove the ssh1 compatibility.

### Files and Directory permissions

Do not allow files and directory world writable !!
Grand access to file and directory with acl or group permissions.

???+ danger
You can add some crontab to list files and dirs with world writable & setuid files

### Ctrl-Alt-Delete combinaison

This key combination can be disabled in /etc/inittab to avoid any mistake on a KVM or in a console.


## Backups

Do not backup the OS !!
Backup only applications data and applications logs if they can't be forwarded to a central syslog.

The OS can be reinstalled in less than 10' for a vm and 30' for a physical machine with some tools like foreman / puppet.
All your configuration should be deployed by your configuration manager.

Also don't forget to use a different location for your backup to avoid any surprise (cryptlocker) or mistake (delete on the backup share).
Don't forget that backup is like schrodinger's cat, until your restore test you don't know if your backup are consistent.

???+ danger alert danger
Raid System is not a backup !!

## Monitoring

Never forget that your monitoring tool should be open enough to allow to be configured through API or command lines.
This will allow you to be flexible if you create machine or service on demand through an industrial mechanism.

### Capacity and availability
#### local agent

Here is short and not limited list of local tools that can help you :
 - ntop
 - htop
 - iotop
 - top
 - sar
 - tcpdump (remove it after debug to avoid security breach)

#### remote monitoring

 - SNMP (v2c or v3)
 - Librenms
 - Collectd
 - Grafana
 - Centreon
 - Shinken
 - Prometheus
 - Nagios
 - Cacti

### What should be monitored on a system

You must monitor what is running on your machine.

#### Availability

System Process like
 - sshd
 - rsyslog
 - crond

#### Capacity

 - CPU (User, System, Free, IOWait)
 - Memory (Total, Free, Cached, Shared, Swap)
 - Load (Beware that the alerting should be adapted depending of #cpu on your machine)
 - Storage (Threshold depends of your capacity 20% of free space on 1PB can be not as critical as on 1GB )
 - TCP Stack state

### What should be monitored on an application

#### Availability

Application process like
 - apache
 - mysql

#### Capacity

 - Storage (Threshold depends of your capacity 20% of free space on 1PB can be not as critical as on 1GB )
 - Application perf indicator (user connected, #queries, #transaction per second, jmx metrics)


### Batch Execution

If you schedule some batch (cronjob, at command ...) don't forget to monitor their execution and results.
IE: in centreon monitoring system you can create passive check that are feeded by nsca deamon.
This nsca daemon is contacted by send_nsca command that give some status about a command linked to the previous check.
With a check freshness define on the passive check, like 4h, if you don't have any feeback from the batch every 4h an alarm will raise.

### Log Management

 - {% link "Elastic Stack" https://www.elastic.co/products%} formerly ElasticSearch, Logstash, Kibana. OpenSource product, Commercial Support available.
 - Splunk (Commercial but Free for 500MB)
 - HP Arcsight (commercial)


## Documentation

Write any documentation related to your infrastructure.
Create some generic documentation and procedure for the OS and core application (DB, tomcat ...) about :
 - managing the process and service
 - how standard are defined (password policy, partitioning, security... )
 - network diagram to explain how your network / infrastructure are designed, this can help you investigate on problem and also give an overview for new comers.

Specific documentation will explain the workflow of application about :
 - how data are pushed, pulled or received
 - how users connect to application
 - how application are related to others applications

Don't forget that in most case a diagram is a better way to explain to someone instead of reading 100 pages of documentations.

For creating diagram, Visio on MS platform is your friend, for Apple user use Omnigraffle.
Both products are commercials but they worst it.

## Tools

 - [tmux](https://tmux.github.io/) or [screen](https://www.gnu.org/software/screen/)
 - [graphviz](http://www.graphviz.org/) (quick and simple diagram)
 - [Grafana](http://grafana.org/)

## 3rd Party Required Tools

 - Whiteboard !!!


