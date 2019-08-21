---
description: Установка Asterisk
---

# Глава 3

> I long to accomplish great and noble tasks, but it is my chief duty to accomplish humble tasks as though they were great and noble. The world is moved along, not only by the mighty shoves of its heroes, but also by the aggregate of the tiny pushes of each honest worker.
>
> Helen Keller

In this chapter we’re going to walk through the installation of Asterisk from the source code. Many people shy away from this method, claiming that it is too difficult and time-consuming. Our goal here is to demonstrate that installing Asterisk from source is not actually that difficult to do. More importantly, we want to provide you with the best Asterisk platform on which to learn.

In this book we will be helping you build a functioning Asterisk system from scratch. Toward that goal, in this chapter we will build a base platform for your Asterisk system. Since we are installing from source, there is potentially a lot of variation in how you can do this. Our goal here is to deliver a standard sort of platform, suitable for explorations in many areas. It is possible to strip Asterisk down to the very basics and run a very lean machine; however, that exercise is left up to the reader. The process we discuss here is designed to get you up and running quickly and simply, without short-changing you on access to interesting features.

Most of the commands you see are going to be best handled with a series of copy-paste operations \(in fact, we strongly recommend you have an electronic version of this book handy for that very purpose\).[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409180936) While it looks like a lot of typing, the commands we take you through can get you from start to finish in less than 30 minutes, so it’s really not as complex as it might appear. We run some prerequisites, some compilation, and some post-install config, and Asterisk is ready to go.

For the sake of brevity, these steps will be performed on a CentOS 7 system. This is functionally equivalent to RHEL, and similar enough to Fedora that the steps should be quite similar. For other platforms such as Debian/Ubuntu and so forth, the instructions will also be similar, but you will need to adjust as needed for your platform.[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409178472)

The first part of the installation instructions will not deal with Asterisk as such, but rather some of the dependencies that either Asterisk requires or are necessary for some of the more useful features \(such as database integration\). We’ll try to keep the instructions general enough that they should be useful on any distribution of your choice.

These instructions assume that you are an experienced Linux administrator.[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409176504) A fully working Asterisk system will consist of enough discrete parts that you will find it challenging to deal with all of it if you have little or no Linux background. We’d still encourage you to dive right in, but please allow for the fact that there will be a steep learning curve if you don’t already have solid Linux command-line experience.

**Note**

If you want to learn the Linux command line, one of the best books we’ve found is The Linux Command Line by William Shotts, which has been released under a Creative Commons license, and dives straight into all the knowledge you need to use the Linux shell effectively. It can be found at [linuxcommand.org](http://linuxcommand.org/). You could memorize the book from front to back, and pretty much everything you’d learned would be something any seasoned Linux administrator would agree was worth knowing.

Another fantastic book is of course the legendary UNIX and Linux System Administration Handbook by Dan Mackin, Ben Whaley, Trent R. Hein, Garth Snyder, and Evi Nemeth \(Prentice Hall\). Highly recommended.

#### Asterisk Packages

There are Asterisk packages that can be installed using package management systems such as yum or apt-get. You are encouraged to use them once you are familiar with Asterisk.

If you are using RHEL, Asterisk is available from the [EPEL repository](http://fedoraproject.org/wiki/EPEL) from the Fedora project. Asterisk packages are available in the Universe repository for Ubuntu.

You should also note that because of Asterisk’s history, it is able to integrate with a multitude of telephony technologies; however, these days, someone new to Asterisk is going to want to learn SIP integration before worrying about more complex, obsolete or peripheral channel types. Once you are comfortable with Asterisk in a pure SIP environment, it’ll be much easier to look at integrating other channel types.

#### Asterisk-Based Projects

Many projects use Asterisk as their underlying platform. Some of these, such as the FreePBX GUI, have become so popular that many people mistake them for the Asterisk product itself. In fact, the FreePBX GUI is so ubiquitous it is found in most of the well-known Asterisk-based projects. These projects take the base Asterisk product and add a web-based administration interface, a complex database, and external functions that are useful in a typical PBX \(such as set provisioning, a time server, and so forth\).

We have chosen not to cover these projects in this book, for several reasons:

* This book tries, as much as possible, to focus on Asterisk and only Asterisk.
* Books have already been written about many of these Asterisk-based projects.
* We believe that if you learn Asterisk in the way that we will teach you, the knowledge will serve you well regardless of whether or not you eventually choose to use one of these prepackaged versions of Asterisk.
* If you want to be able to make sense of what’s going on under the hood of a FreePBX-based system, this book will introduce you to some of the skills you will need.
* For us, the power of Asterisk is that it does not attempt to solve your problems for you. These projects are truly amazing examples of what can be built with Asterisk. However, if you are looking to build your own Asterisk application \(which is really what Asterisk is all about\), these projects might create needless obstacles, simply because they are focused on simplifying the process of building a business PBX, not on making it possible to access the full potential of the Asterisk platform.

Some of the most popular Asterisk-based projects include \(in no particular order\):

[AsteriskNOW](http://www.asterisk.org/asterisknow)

Managed by Digium. Uses FreePBX GUI.

[Issabel](https://www.issabel.org/)

A fork of the original open source releases of the Elastix product.[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409149976) Uses FreePBX GUI.

[The Official FreePBX Distro](http://www.freepbx.org/freepbx-distro)

The official distro of the FreePBX project. Managed by Sangoma.

[Asterisk for Raspberry Pi](http://www.raspberry-asterisk.org/)

A complete install of Asterisk and FreePBX for the Raspberry Pi.

[AstLinux](https://www.astlinux-project.org/)

The AstLinux project caters to a community that want to run Asterisk on small, low-power, solid-state devices. The install size of the entire solution is measured in megabytes \(AstLinux was originally designed to fit on CompactFlash cards\). If you are fascinated by small computers, and want to play with a PBX-in-a-box that fits in your pocket, AstLinux may be for you.

We recommend that you check them out.[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409144568)

## Linux Installation

Asterisk is developed using Linux, and unless you’re very comfortable with porting software between various platforms, that is what you’re going to want to use.

In this book, we’re going to use CentOS as the platform. If you would prefer a different Linux distro, it is expected that you have sufficient Linux skills to understand what some of the differences may be. These days, it’s so easy and cheap to fire up an instance of any common distribution that there’s no real harm in using CentOS to learn, and then migrate to whatever you prefer when you’re ready.

We recommend installing the Minimal version of CentOS, since the installation process we will be going through handles all the prerequisites. This also ensures you’re not installing anything you don’t need.

### Choosing Your Platform

OK, so strictly speaking we’ve already chosen your platform for you, but there are several different ways to get a CentOS server up and running \(see [Table 3-1](3.%20Installing%20Asterisk%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22table03linux)\).

Table 3-1. Comparing Linux platforms that are suitable for Asterisk

| Platform | Pros | Cons |
| :--- | :--- | :--- |
| OpenStack \(DigitalOcean, Linode, VULTR, etc.\) | Up and running in minutes. Inexpensive to operate. Doesn’t require any resources on your local system. Accessible from anywhere. Can be used in a production environment. Fantastic for quick prototyping projects. | You pay as long as it’s running. The IP address is only yours for as long as the system is running. Requires some DevOps skills if you want to deploy in production. No firewall in place by default. |
| VirtualBox \(or other PC-based platform\) | Free to use. No external exposure. Excellent for small lab projects. | Requires more horsepower on your system. Requires storage space on your local system. Not easy to deploy into a production environment. |
| AWS and/or Lightsail | Inexpensive to operate. Doesn’t require any resources on your local system. Accessible from anywhere. Can be used in a production environment. Scales to enormous sizes. | You pay as long as it’s running. Somewhat more skills required to gather all the resources you need. |
| Physical hardware | Dedicated platform. Can be shipped and installed anywhere. Complete control over all aspects of environment, hardware, network, and so forth. | Risk of component failure. Power consumption. Noise. Potential costs for hosting. No inherent redundancy. |
| Other \(really anything that’ll run CentOS 7 should be fine\) | You can use an environment that you’re familiar with. | You’re on your own. |
| Other Linux \(you don’t actually have to run CentOS\) | You can run the exact environment you want. | You need to have strong Linux admin skills. |

For the purposes of learning, we recommend one of two simple ways to get going:

* If you are running Windows as your desktop: Download VirtualBox, then download the CentOS 7 Minimal ISO, and install on your local machine.
* If you are comfortable working with SSH-based, keyed connections to remote systems: Create a hosted system \(for example, a DigitalOcean CentOS droplet\).

This book was developed and tested using both VirtualBox and DigitalOcean.

### VirtualBox Steps

Grab a copy of VirtualBox from the [platform’s website](https://www.virtualbox.org/wiki/Downloads) and install it.

Download the Minimal ISO from the [Centos](https://www.centos.org/download/) website.

Get yourself a copy of [PuTTY](http://bit.ly/2J0ftwK) if you’re using Windows.

Create a new virtual machine with the following characteristics:

* Type: Linux
* Version: Red Hat \(64-bit\)
* Memory size: 2048 MB
* Hard disk: Create a virtual hard disk now
* File location: Pick a good spot for storing your virtual machine images
* File size: 16 GB is fine for what we’re doing here, but something larger would be needed for production

Once the basic machine has been defined, you’ll need to tweak it as follows:

* Storage: Under Storage, Controller: IDE ...
  1. You should see the CD/DVD has a tiny disc icon labeled Empty.
  2. Click on it and to the right under Attributes, there’ll be another tiny disc icon.
  3. Click on that, and it’ll ask you to Choose Optical Virtual Disk File.
  4. Locate on your hard drive the Minimal ISO you downloaded from CentOS, and choose it.
  5. The Storage Tree should now show the CentOS ISO.
* Network: Adaptor 1

Attached to: Bridged Adapter

Start up the machine you’ve just created, and it should take you through a basic installation of CentOS. Here are a few items you’ll want to specify \(for anything else, the defaults should do\):

* Date and time: Adjust to your time zone if you wish.
* Network and host name: Ethernet—toggle from off to on \(it should immediately grab an IP address from your network; if not, set one manually\). Press the Done button.
* Installation destination: It may require you to confirm the target, but you shouldn’t need to change anything. Press the Done button.
* That’s it. Begin Installation.

While the installation is taking place, set the root password, and also create a user named astmin. Make the astmin user an administrator.

The installation will take a few minutes. Grab a coffee!

Once the install is done, the installer will ask you to Reboot. The reboot should only take 15 seconds or so.

Congratulations, your system is ready. Log in as root.

### Linux \(OpenStack\) Host

You’ll obviously need an account with a hosted Linux provider if you’re going to use this method \(we’ve found OpenStack-based offerings to be the cheapest, relative to the quality/performance/simplicity offered\). We’ve been using DigitalOcean for many years, but have also found Linode and VULTR to be strong providers in this space.[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409086616) Once you’ve got that sorted, you can log in and create a new system something like the following:

* CentOS 7 \(lastest version, 64-bit\)
* 4 GB 2vCPUs \(we don’t really need the 4 GB RAM, but it is good to have the 2xCPUs; you can probably get away with 2 GB 1vCPU, if you’re really cost-conscious\)
* Data center closest to you

Once that’s up and running, log in as the default user \(as of this writing, it’s centos\).

**Warning**

Note that DigitalOcean instances do not have a firewall by default. Instead, they provide a firewall as a part of their environment. The system you build will therefore not have any native firewall in place, and will be subject to external attacks shortly after you complete configuration \(you’ll see this on the Asterisk console\). Different providers will have different firewall policies. You are responsible for making sure your firewalling is working correctly. We’ll be discussing security and anti-fraud in more detail later on in this book.

## Dependencies

The system you’ve just built isn’t really much more than a basic bootstrapped system. In order to prepare it for an Asterisk installation, there are a few things we’ll need to install first.

The following commands can be typed from the command line, or added to a simple shell script and run that way.

sudo yum -y update &&

sudo yum -y install epel-release &&

sudo yum -y install python-pip &&

sudo yum -y install vim wget dnf&&

sudo pip install alembic ansible &&

sudo pip install --upgrade pip &&

sudo mkdir /etc/ansible &&

sudo chown astmin:astmin /etc/ansible &&

sudo echo "\[starfish\]" &gt;&gt; /etc/ansible/hosts &&

sudo echo "localhost ansible\_connection=local" &gt;&gt; /etc/ansible/hosts &&

mkdir -p ~/ansible/playbooks

We’ve installed Ansible simply because it’s a quick and easy way to get all the dependencies met. We’ve written a playbook to perform the following operations:

1. Install dnf, vim, wget, and MySQL-python.
2. Install the MySQL community-edition repository.
3. Install mysql-server.
4. Tweak some variables in the mysql-server installation.
5. Start the mysql-server daemon.
6. Modify some MySQL credentials \(create users, set passwords\).
7. Create a MySQL database/schema for Asterisk to use.
8. Apply some security best practices \(remove anonymous user, test database, etc.\).
9. Create asterisk user.
10. Create astmin user.
11. Install dependencies for ODBC.
12. Install some diagnostic tools.
13. Tweak the firewall to allow SIP and RTP traffic.
14. Edit some ODBC config files.

This can all be done manually, but it’s just a lot of typing, and Ansible is really good at streamlining this process.

Create an Ansible playbook in the file ~/ansible/playbooks/starfish.yml.

**Note**

The libmyodbc8a.so file is versioned, so, if you don’t have version 8 of UnixODBC:

1. Run the playbook the first time \(to install the UnixODBC library\).
2. Find out what file was installed at /usr/lib64/libmyodbc&lt;version&gt;a.so.
3. Edit the playbook as appropriate \(provide the correct filename\).
4. Save and rerun the playbook \(which will then update the configuration files to point to the correct library\).

Here’s the playbook:

---

- hosts: starfish

 become: yes

 vars:

\# Use these on the first run of this playbook

 current\_mysql\_root\_password: ""

 updated\_mysql\_root\_password: "YouNeedAReallyGoodPassword"

 current\_mysql\_asterisk\_password: ""

 updated\_mysql\_asterisk\_password: "YouNeedAReallyGoodPasswordHereToo"

\# Comment the above out after the first run

\# Uncomment these for subsequent runs

\# current\_mysql\_root\_password: "YouNeedAReallyGoodPassword"

\# updated\_mysql\_root\_password: "{{ current\_mysql\_root\_password }}"

\# current\_mysql\_asterisk\_password: "YouNeedAReallyGoodPasswordHereToo"

\# updated\_mysql\_asterisk\_password: "{{ current\_mysql\_asterisk\_password }}"

 tasks:

 - name: Install epel-release

 dnf:

 name: epel-release

 state: present

 - name: Install dependencies

 dnf:

 name: \['vim', 'wget', 'MySQL-python'\]

 state: present

 - name: Install the MySQL repo.

 dnf:

 name: http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm

 state: present

 - name: Install mysql-server

 dnf:

 name: mysql-server

 state: present

 - name: Override variables for MySQL \(RedHat\).

 set\_fact:

 mysql\_daemon: mysqld

 mysql\_packages: \['mysql-server'\]

 mysql\_log\_error: /var/log/mysqld.err

 mysql\_syslog\_tag: mysqld

 mysql\_pid\_file: /var/run/mysqld/mysqld.pid

 mysql\_socket: /var/lib/mysql/mysql.sock

 when: ansible\_os\_family == "RedHat"

 - name: Ensure MySQL server is running

 service:

 name: mysqld

 state: started

 enabled: yes

 - name: update mysql root pass for localhost root account from local servers

 mysql\_user:

 login\_user: root

 login\_password: "{{ current\_mysql\_root\_password }}"

 name: root

 host: "{{ item }}"

 password: "{{ updated\_mysql\_root\_password }}"

 with\_items:

 - localhost

 - name: update mysql root password for all other local root accounts

 mysql\_user:

 login\_user: root

 login\_password: "{{ updated\_mysql\_root\_password }}"

 name: root

 host: "{{ item }}"

 password: "{{ updated\_mysql\_root\_password }}"

 with\_items:

 - "{{ inventory\_hostname }}"

 - 127.0.0.1

 - ::1

 - localhost.localdomain

 - name: create asterisk database

 mysql\_db:

 login\_user: root

 login\_password: "{{ updated\_mysql\_root\_password }}"

 name: asterisk

 state: present

 - name: asterisk mysql user

 mysql\_user:

 login\_user: root

 login\_password: "{{ updated\_mysql\_root\_password }}"

 name: asterisk

 host: "{{ item }}"

 password: "{{ updated\_mysql\_asterisk\_password }}"

 priv: "asterisk.\*:ALL"

 with\_items:

 - "{{ inventory\_hostname }}"

 - 127.0.0.1

 - ::1

 - localhost

 - localhost.localdomain

 - name: remove anonymous user

 mysql\_user:

 login\_user: root

 login\_password: "{{ updated\_mysql\_root\_password }}"

 name: ""

 state: absent

 host: "{{ item }}"

 with\_items:

 - localhost

 - "{{ inventory\_hostname }}"

 - 127.0.0.1

 - ::1

 - localhost.localdomain

 - name: remove test database

 mysql\_db:

 login\_user: root

 login\_password: "{{ updated\_mysql\_root\_password }}"

 name: test

 state: absent

 - user:

 name: asterisk

 state: present

 createhome: yes

 - group:

 name: asterisk

 state: present

 - user:

 name: astmin

 groups: asterisk,wheel

 state: present

 - name: Install other dependencies

 dnf:

 name:

 - unixODBC

 - unixODBC-devel

 - mysql-connector-odbc

 - MySQL-python

 - tcpdump

 - ntp

 - ntpdate

 - jansson

 - bind-utils

 state: present

\# Tweak the firewall for UDP/SIP

 - firewalld:

 port: 5060/udp

 permanent: true

 state: enabled

\# Tweak firewall for UDP/RTP

 - firewalld:

 port: 10000-20000/udp

 permanent: true

 state: enabled

 - name: Ensure NTP is running

 service:

 name: ntpd

 state: started

 enabled: yes

\# The libmyodbc8a.so file is versioned, so if you don't have version 8, see what the

\# /usr/lib64/libmyodbc&lt;version&gt;a.so file is, and refer to that instead

\# on your 'Driver64' line, and then run the playbook again

 - name: update odbcinst.ini

 lineinfile:

 dest: /etc/odbcinst.ini

 regexp: "{{ item.regexp }}"

 line: "{{ item.line }}"

 state: present

 with\_items:

 - regexp: "^Driver64"

 line: "Driver64 = /usr/lib64/libmyodbc8a.so"

 - regexp: "^Setup64"

 line: "Setup64 = /usr/lib64/libodbcmyS.so"

 - name: create odbc.ini

 blockinfile:

 path: /etc/odbc.ini

 create: yes

 block: \|

 \[asterisk\]

 Driver = MySQL

 Description = MySQL connection to 'asterisk' database

 Server = localhost

 Port = 3306

 Database = asterisk

 UserName = asterisk

 Password = {{ updated\_mysql\_asterisk\_password }}

 \#Socket = /var/run/mysqld/mysqld.sock

 Socket = /var/lib/mysql/mysql.sock

...

Run the playbook with the following command:

$ ansible-playbook ~/ansible/playbooks/starfish.yml

Sit back and watch the magic happen.

Once Ansible has completed the assigned tasks, verify that ODBC can connect to the database using the asterisk user credentials.

$ echo "select 1" \| isql -v asterisk asterisk password

You should see a result something like this:

+---------------------------------------+

\| Connected! \|

\| sql-statement \|

\| help \[tablename\] \|

\| quit \|

+---------------------------------------+

SQL&gt; select 1

+---------------------+

\| 1 \|

+---------------------+

\| 1 \|

+---------------------+

SQLRowCount returns 1

1 rows fetched

If you do not see the Connected! message, you need to troubleshoot your database and ODBC installation. The first thing you should do is make sure you can log into the database from the command line using the asterisk user \(mysql -u asterisk -p\). Most ODBC problems tend to end up being credentials problems \(i.e., wrong password or username\), so work backward to ensure all the credentials work as they should, and double-check that you didn’t get any problem messages from Ansible.

As of this writing, the version of jansson installed from the EPEL repo is an older version than the one Asterisk requires, so we’ll have to install that manually.

The system is now prepared, and we’re ready to download and install Asterisk.

## Asterisk Installation

Asterisk is officially delivered in a tarball \(as source code\), and it must be downloaded, extracted, and compiled.[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409043048) This is not difficult to do, so long as you have all the dependencies correct. Between the writing of this book and your reading of it, there may have been some changes to the various dependencies, so your install process may have to be run slightly differently. It’s often difficult to know the difference between an error message that can safely be ignored, and one that is indicating a critical problem; however, in general, you should have identified and resolved any errors in the previous processes before arriving at this step. If your dependencies are sorted, the Asterisk install will tend to go smoothly.

### Download and Prerequisites

Log out of the system, and log back in as user astmin.[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409035896)

Type the following commands from the shell in order to download the Asterisk source code:

**Note**

When you see us write &lt;TAB&gt; in a filename, what we mean is that you should press the Tab key on your keyboard and allow autocomplete to fill in what it can. The rest of the typing then follows.

$ mkdir ~/src

$ cd ~/src

$ wget https://downloads.asterisk.org/pub/telephony/asterisk/asterisk-16-current.tar.gz

$ tar zxvf asterisk-16-current.tar.gz

$ cd asterisk-16.&lt;TAB&gt; \# tab should auto-complete \(unless it has more than one match\)

We can now run a few prerequisites that the Asterisk team has defined, and also have the environment checked:

$ cd contrib/scripts \(or cd ~/src/asterisk-16.&lt;TAB&gt;/contrib/scripts

$ sudo ./install\_prereq install \# asterisk has a few prerequisites that this simplifies

$ cd ../..

$ ./configure --with-jansson-bundled

Asterisk is now ready to compile and install, but there are a few tweaks worth making to the configuration before compilation.

### Compiling and Installing

$ make menuselect

You will see a menu that presents various options you can select for the compiler. Use the arrow and Tab keys to move around, and the Enter key to select/deselect. For the most part, the defaults should be fine, but we want to make a few tweaks to the sound files in order to ensure we have all the sounds we want, in the best format.

**Note**

At this point you can also select other languages you wish to have on your system. We recommend you select the WAV and G722 formats \(and G729 as well, if you need to support it\).

Under Codec Translators \(--- External ---\):

* Select \[\*\] codec\_opus
* Select \[\*\] codec\_silk
* Select \[\*\] codec\_siren7
* Select \[\*\] codec\_siren14
* Select \[\*\] codec\_g729a

Under Core Sound Packages:

* Deselect \[\*\] CORE-SOUNDS-EN-GSM
* Select \[\*\] CORE-SOUNDS-EN-WAV
* Select \[\*\] CORE-SOUNDS-EN-G722

Under Extras Sound Packages:

* Select \[\*\] EXTRA-SOUNDS-EN-WAV
* Select \[\*\] EXTRA-SOUNDS-EN-G722

Save and Exit.

Three more commands and Asterisk is installed:

$ make \# this will take several minutes to complete

 \# \(depending on the speed of your system\)

$ sudo make install \# you must run this with escalated privileges

$ sudo make config \# this too

**Warning**

When the make config command has completed, it will suggest some commands to install the sample configuration files. For the purposes of this book, you do not want to do this. We will be building the necessary files by hand, so the sample files will only serve to disrupt and confuse that process. Having said that, the sample files are useful, and we will mention them throughout this book, since they are excellent reference material.

Reboot the system.

Once the boot is complete, log back in as the astmin user, and temporarily set SELinux to Permissive \(it will revert to Enforcing after each boot, so until we’ve sorted out the SELinux portion of the install, this has to happen on every boot\):

$ sudo setenforce Permissive

$ sudo sestatus

This should show Current mode: permissive

Verify that Asterisk is running with the following command:

$ ps -ef \| grep asterisk

You want to see the /user/sbin/asterisk daemon running \(currently as user root, but we’ll fix that shortly\).

Asterisk is now installed and is running; however, there are a few configuration settings we’ll need to make before the system is in any way useful.

### Initial Configuration

Asterisk stores its configuration files in the /etc/asterisk folder by default. The Asterisk process itself doesn’t need any configuration files in order to run; however, it will not be usable yet, since none of the features it provides have been specified. We’re going to handle a few of the initial configuration tasks now.

**Note**

Asterisk configuration files use the semicolon \(;\) character for comments, primarily because the hash character \(\#\) is a valid character on a telephone number pad.

The modules.conf file gives you fine-grained control over what modules Asterisk will \(and will not\) load. It’s usually not necessary to explicitly define each module in this file, but you could if you wanted to. We’re going to create a very simple file like this:

$ sudo chown asterisk:asterisk /etc/asterisk ; sudo chmod 664 /etc/asterisk

$ sudo -u asterisk vim /etc/asterisk/modules.conf

\[modules\]

autoload=yes

preload=res\_odbc.so

preload=res\_config\_odbc.so

We’re using ODBC to load many of the configurations of other modules, and we need this connector available before Asterisk attempts to load anything else, so we’ll pre-load it.

Next up, we’re going to tweak the logger.conf file just a bit from the defaults.

$ sudo -u asterisk vim /etc/asterisk/logger.conf

\[general\]

exec\_after\_rotate=gzip -9 ${filename}.2;

\[logfiles\]

;debug =&gt; debug

;security =&gt; security

console =&gt; notice,warning,error,verbose

;console =&gt; notice,warning,error,debug

messages =&gt; notice,warning,error

full =&gt; notice,warning,error,debug,verbose,dtmf,fax

;full-json =&gt; \[json\]debug,verbose,notice,warning,error,dtmf,fax

;syslog keyword : This special keyword logs to syslog facility

;syslog.local0 =&gt; notice,warning,error

You will notice that many lines are commented out. They’re there as a reference, because you’ll find when debugging your system you may want to frequently tweak this file. We’ve found it’s easier to have a few handy lines prepared and commented out, rather than having to look up the syntax each time.

The next file, asterisk.conf, defines various folders needed for normal operation, as well as parameters needed to run as the asterisk user:

$ sudo -u asterisk vim /etc/asterisk/asterisk.conf

\[directories\]\(!\)

astetcdir =&gt; /etc/asterisk

astmoddir =&gt; /usr/lib/asterisk/modules

astvarlibdir =&gt; /var/lib/asterisk

astdbdir =&gt; /var/lib/asterisk

astkeydir =&gt; /var/lib/asterisk

astdatadir =&gt; /var/lib/asterisk

astagidir =&gt; /var/lib/asterisk/agi-bin

astspooldir =&gt; /var/spool/asterisk

\[options\]

astrundir =&gt; /var/run/asterisk

astlogdir =&gt; /var/log/asterisk

astsbindir =&gt; /usr/sbin

runuser = asterisk ; The user to run as. The default is root.

rungroup = asterisk ; The group to run as. The default is root

We’ll configure more files later on, but these are all we need for the time being.

Let’s update the ownership of the files so the asterisk user has proper access to them.

$ sudo chown -R asterisk:asterisk {/etc,/var/lib,/var/spool,/var/log,/var/run}/asterisk

We also may need to add a rule to the /etc/tmpfiles.d folder, to allow Asterisk to create a socket at runtime.

$ sudo vim /etc/tmpfiles.d/asterisk.conf

d /var/run/asterisk 0775 asterisk asterisk

\(See man tmpfiles.d for more information.\)

Next up, we’re going to initialize the database with the tables Asterisk needs for ODBC-based configuration.

The Asterisk source files include a contribution that the Digium folks maintain as part of Asterisk, in order to version-control the database tables needed. This greatly simplifies keeping the database correct through the upgrade process.

Navigate to the relevant directory and make a copy of the configuration file.

$ cd ~/src/asterisk-16.&lt;TAB&gt;/contrib/ast-db-manage

$ cp config.ini.sample config.ini

Now, we’re going to open the file and give it the credentials for our database \(which are defined in the Ansible playbook named starfish.yml, under the variable current\_mysql\_asterisk\_password, which we used at the beginning of this chapter\):

$ vim config.ini

Find the lines that look similar to this:

\#sqlalchemy.url = postgresql://user:pass@localhost/asterisk

sqlalchemy.url = mysql://user:pass@localhost/asterisk

\# Logging configuration

\[loggers\]

keys = root,sqlalchemy,alembic

Make a copy of it, comment it out, and edit it with the correct credentials:

\#sqlalchemy.url = postgresql://user:pass@localhost/asterisk

\#sqlalchemy.url = mysql://user:pass@localhost/asterisk

sqlalchemy.url = mysql://asterisk:YouNeedAReallyGoodPasswordHereToo@localhost/asterisk

\# Logging configuration

\[loggers\]

keys = root,sqlalchemy,alembic

Now, with that very simple bit of configuration, we can use Alembic to automagically configure the database perfectly for Asterisk \(this used to be somewhat painful to do in past versions of Asterisk\):

$ alembic -c ./config.ini upgrade head

**Note**

Alembic is not used by Asterisk, so the configuration you’ve just performed does not allow Asterisk to use the database. All it does is run a script that creates the schema and tables Asterisk will use \(you could do this manually as well, but trust us, you want Alembic to handle this\). It’s part of the install/upgrade process. It’s especially useful if you have live tables, with real data in them, and want to be able to upgrade and retain the relevant configuration.

Log into the database now, and review all the tables that have been created:

$ mysql -u asterisk -p

mysql&gt; use asterisk;

mysql&gt; show tables;

You should see a list similar to this:

\| alembic\_version\_config \|

\| extensions \|

\| iaxfriends \|

\| meetme \|

\| musiconhold \|

\| ps\_aors \|

\| ps\_asterisk\_publications \|

\| ps\_auths \|

\| ps\_contacts \|

\| ps\_domain\_aliases \|

\| ps\_endpoint\_id\_ips \|

\| ps\_endpoints \|

\| ps\_globals \|

\| ps\_inbound\_publications \|

\| ps\_outbound\_publishes \|

\| ps\_registrations \|

\| ps\_resource\_list \|

\| ps\_subscription\_persistence \|

\| ps\_systems \|

\| ps\_transports \|

\| queue\_members \|

\| queue\_rules \|

\| queues \|

\| sippeers \|

\| voicemail \|

We’re not going to configure anything in the database as of yet. We’ve got some more base configuration to do first.

Exit the database CLI.

Now that we’ve got the database structure to handle Asterisk config, we’re going to tell Asterisk how to connect to the database.

$ sudo -u asterisk vim /etc/asterisk/res\_odbc.conf

Once again, you’ll need the credentials you defined in your Ansible playbook.

\[asterisk\]

enabled =&gt; yes

dsn =&gt; asterisk

username =&gt; asterisk

password =&gt; YouNeedAReallyGoodPasswordHereToo

pre-connect =&gt; yes

### SELinux Tweaks

We’re going to install some SELinux tools, and make a few changes to the SELinux configuration so that the system will boot properly.

**Note**

A common approach is to simply edit /etc/selinux/config, and set enforcing=disabled. We do not recommend this, but doing so completely disables SELinux and renders the following steps unnecessary.

$ sudo dnf -y install setools setroubleshoot setroubleshoot-server

$ sudo vim /etc/selinux/config

You’re going to change the line SELINUX=enforcing to SELINUX=permissive. This will ensure the logfiles show potential SELinux errors, without actually blocking the relevant processes.

Next, we’re going to give Asterisk ownership of the /etc/odbc.ini file.

$ sudo chown asterisk:asterisk /etc/odbc.ini

$ sudo semanage fcontext -a -t asterisk\_etc\_t /etc/odbc.ini

$ sudo restorecon -v /etc/odbc.ini

$ sudo ls -Z /etc/odbc.ini

If all is well, you should see now that the file context for this file has been set to asterisk\_etc\_t:

-rw-r--r--. asterisk asterisk system\_u:object\_r:asterisk\_etc\_t:s0 /etc/odbc.ini

There are a few more SELinux errors we’ve seen here during the writing of the book. They may have been corrected by the time you read this, but there should be no harm in running them:

$ sudo /sbin/restorecon -vr /var/lib/asterisk/\*

$ sudo /sbin/restorecon -vr /etc/asterisk\*

Reboot the system, and we’re going to check the log for any nasty SELinux errors before we set it to enforcing.

$ sudo grep -i sealert /var/log/messages

There may be a few messages in there complaining about things Asterisk doesn’t need \(for example, a hidden file named .odbc.ini\), but so long as it’s not full of errors about all sorts of important Asterisk components, you should be good to go. One last thing you have to change is an SELinux Boolean to allow Asterisk to create a TTY.

$ sudo setsebool -P daemons\_use\_tty 1

Edit the /etc/selinux/config file again, this time setting SELINUX=enforcing. Save and reboot once more.

Verify that Asterisk is running \(as user asterisk\).

$ ps -ef \| grep asterisk

asterisk 3992 3985 0 16:38 ? 00:00:01 /usr/sbin/asterisk -f -vvvg -c

OK, we’re nearly done with the installation now.

### Firewall Tweaks

We’ll make a couple of firewall tweaks to prepare our system for SIP \(and SIP Secure\).

$ sudo firewall-cmd --zone=public --add-service=sip --permanent

$ sudo firewall-cmd --zone=public --add-service=sips --permanent

### Final Tweaks

Your Asterisk system is ready to roll.

Let’s put some initial data into the configuration files, so that in the next chapter we can begin to work with our new Asterisk system.

Since we’re going to use the PJSIP channel for all of our calling, we’re going to tell Asterisk to look for PJSIP configuration in the database:

$ sudo -u asterisk vim /etc/asterisk/sorcery.conf

\[res\_pjsip\] ; Realtime PJSIP configuration wizard

; eventually more modules will use sorcery to connect to the

; database, but currently only PJSIP uses this

endpoint=realtime,ps\_endpoints

auth=realtime,ps\_auths

aor=realtime,ps\_aors

domain\_alias=realtime,ps\_domain\_aliases

contact=realtime,ps\_contacts

\[res\_pjsip\_endpoint\_identifier\_ip\]

identify=realtime,ps\_endpoint\_id\_ips

$ sudo -u asterisk vim /etc/asterisk/extconfig.conf

\[settings\] ; older mechanism for connecting all other modules to the database

ps\_endpoints =&gt; odbc,asterisk

ps\_auths =&gt; odbc,asterisk

ps\_aors =&gt; odbc,asterisk

ps\_domain\_aliases =&gt; odbc,asterisk

ps\_endpoint\_id\_ips =&gt; odbc,asterisk

ps\_contacts =&gt; odbc,asterisk

$ sudo -u asterisk vim /etc/asterisk/modules.conf

\[modules\]

autoload=yes

preload=res\_odbc.so

preload=res\_config\_odbc.so

noload=chan\_sip.so ; deprecated SIP module from days gone by

We now have to place one bit of config into the pjsip.conf file, which defines the transport mechanism.

$ sudo -u asterisk vim /etc/asterisk/pjsip.conf

\[transport-udp\]

type=transport

protocol=udp

bind=0.0.0.0

Finally, let’s log into the database, and define some sample configurations for PJSIP:

$ mysql -D asterisk -u asterisk -p

mysql&gt;

insert into asterisk.ps\_aors \(id, max\_contacts\) values \('0000f30A0A01', 1\);

insert into asterisk.ps\_aors \(id, max\_contacts\) values \('0000f30B0B02', 1\);

insert

 into asterisk.ps\_auths

 \(id, auth\_type, password, username\)

 values

 \('0000f30A0A01', 'userpass', 'not very secure', '0000f30A0A01'\);

insert

 into asterisk.ps\_auths

 \(id, auth\_type, password, username\)

 values

 \('0000f30B0B02', 'userpass', 'hardly to be trusted', '0000f30B0B02'\);

insert

 into asterisk.ps\_endpoints

 \(id, transport, aors, auth, context, disallow, allow, direct\_media\)

 values

 \('0000f30A0A01', 'transport-udp', '0000f30A0A01', '0000f30A0A01',

 'sets', 'all', 'ulaw', 'no'\);

insert

 into asterisk.ps\_endpoints

 \(id, transport, aors, auth, context, disallow, allow, direct\_media\)

 values

 \('0000f30B0B02', 'transport-udp', '0000f30B0B02', '0000f30B0B02',

 'sets', 'all', 'ulaw', 'no'\);

exit

Let’s reboot, and then we’ll log into our new Asterisk system and have a look at what we’ve created.

## Validating Your New Asterisk System

We don’t need to dive too deeply into the system at this point, since all the chapters that follow will be doing exactly that.

So all we need to do is verify that we can log into the system and that the PJSIP endpoints we’ve created are there.

$ sudo asterisk -rvvvv

\*CLI&gt; pjsip show endpoints

You should see the two endpoints we created listed as follows:

 Endpoint: &lt;Endpoint/CID.....................................&gt; &lt;State.....&gt; &lt;Channels.&gt;

 I/OAuth: &lt;AuthId/UserName...........................................................&gt;

 Aor: &lt;Aor............................................&gt; &lt;MaxContact&gt;

 Contact: &lt;Aor/ContactUri..........................&gt; &lt;Hash....&gt; &lt;Status&gt; &lt;RTT\(ms\)..&gt;

 Transport: &lt;TransportId........&gt; &lt;Type&gt; &lt;cos&gt; &lt;tos&gt; &lt;BindAddress..................&gt;

 Identify: &lt;Identify/Endpoint.........................................................&gt;

 Match: &lt;criteria.........................&gt;

 Channel: &lt;ChannelId......................................&gt; &lt;State.....&gt; &lt;Time.....&gt;

 Exten: &lt;DialedExten...........&gt; CLCID: &lt;ConnectedLineCID.......&gt;

==========================================================================================

 Endpoint: 0000f30A0A01 Not in use 0 of inf

 InAuth: 1/0000f30A0A01

 Transport: transport-udp udp 0 0 0.0.0.0:5060

 Endpoint: 0000f30B0B02 Unavailable 0 of inf

 InAuth: 2/0000f30B0B02

 Transport: transport-udp udp 0 0 0.0.0.0:5060

Objects found: 2

If you don’t see the two endpoints listed, you’ve got a configuration issue. You’re going to have to work backward to ensure you don’t have any errors that prevent Asterisk from connecting to the database and instantiating these two endpoints.

## Common Installation Errors

The following conditions \(in no particular order\) cause the majority of installation errors:

Syntax errors

In some cases, substituting a tab for a space can be enough to break something. UnixODBC, for example, has proven to be sensitive to missing spaces between key = value definitions. The best advice we can give here is to use copy/paste whenever possible, as opposed to manual input.

Permissions problems

These can be annoying to resolve, but error messages will generally provide the clues you need. The /var/log/messages file is often a gold mine for useful clues.

Missing steps

A missed step might not have any noticeable effects until many steps later. Double-check everything, and verify functionality before moving on.

Credentials problems

Always verify that the users and passwords you create work manually, before using them in a configuration file.

It’s not possible nor necessary to dig into every warning and error message you might see, but if we’ve provided a test to run, and it doesn’t produce anything like we said it should, you should probably work through that step again until you’ve figured out what’s going on.

## Some Final Configuration Notes

Once installed, Asterisk will have created an environment for itself in your Linux machine. The following sections have some useful tidbits of information about how you can interact with your new Asterisk installation.

### Sample Configuration Files for Future Reference

Even though we warned you not to run the sudo make samples command during the installation \(because that will fill your /etc/asterisk directory with a bunch of stuff you don’t want\), the sample files are nevertheless a fantastic reference. In your Asterisk source directory, you will find the following two directories:

~/src/asterisk-16.&lt;TAB&gt;/configs/basic-pbx

~/src/asterisk-16.&lt;TAB&gt;/configs/samples

The files in those folders are worth reading through \(especially for any module you’re working with and want to research how to do something\).

Give them a read when you have a chance.

**Warning**

Running make samples on a system that already has configuration files will overwrite the existing files.

### The Asterisk Shell Command

Asterisk can be run either as a daemon or as an application. In general, you will want to run it as an application when you are building, testing, and troubleshooting, and as a daemon when you put it into production.

The command to start Asterisk is the same regardless of whether you’re running it as a daemon or an application:

asterisk

However, without any arguments, this command will assume certain defaults and start Asterisk as a background application. In other words, you never want to run the command asterisk on its own, but rather will want to pass some options to it to better define the behavior you are looking for. The following list provides some examples of common usages:

-h

This command displays a helpful list of the options you can use. For a complete list of all the options and their descriptions, run the command man asterisk.

-c

This option starts Asterisk as an application \(in the foreground\). This means that Asterisk is tied to your user session. In other words, if you close your user session by logging out or losing the connection, Asterisk dies. This is the option you will typically use when building, testing, and debugging, but you would not want to use it in production. If you started Asterisk in this manner, type core stop now at the CLI prompt to stop Asterisk and exit.

-v, -vv, -vvv, -vvvv, etc.

This option can be used with other options \(e.g., -cvvv\) in order to increase the verbosity of the console output. It does exactly the same thing as the CLI command core set verbose n where n is any integer between 0 and 5 \(any integer greater than 5 will work, but will not provide any more verbosity\). Sometimes it’s useful to not set the verbosity at all. For example, if you are looking to see only startup errors, notices, and warnings, leaving verbosity off will prevent all the other startup messages from being displayed.

-d, -dd, -ddd, -dddd, etc.

This option can be used in the same way as -v, but instead of normal output, this will specify the level of debug output \(which is primarily useful for developers who wish to troubleshoot problems with the code\). You will also need to enable output of debugging information in the logger.conf file \(which we will cover in more detail in [Chapter 21](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch21.html%22%20/l%20%22asterisk-Monitoring)\).

-r

This command is essential if you want to connect to the CLI of an Asterisk process running as a daemon. You will probably use this option more than any other for Asterisk systems that are in production. This option will only work if you have a daemonized instance of Asterisk already running. To exit the CLI when this option has been used, type exit.

-T

This option will add a timestamp to CLI output.

-x

This command allows you to pass a string to Asterisk that will be executed as if it had been typed at the CLI. As an example, to get a quick listing of all the channels in use without having to start the Asterisk console, simply type asterisk -rx 'core show channels' from the shell, and you’ll get the output you are looking for.

-g

This option instructs Asterisk to dump a core file if it crashes.

We recommend you try out a few combinations of these commands to see what they do.

### safe\_asterisk

When you install Asterisk using the make config directive, it will create a script called safe\_asterisk, which is run during the init process of Linux each time you boot.

The safe\_asterisk script provides the following benefits:

* Restarts Asterisk automatically after a crash
* Can be configured to email the administrator if a crash has occurred
* Defines where crash files are stored \(/tmp by default\)
* Executes a script if a crash has occurred

You don’t need to know too much about this script, other than to understand that it should normally be running. In most environments this script works fine in its default format.

## Conclusion

In this chapter we’ve provided a curated example of how an Asterisk installation should go. We’ve chosen the Linux distribution and MySQL server for you for the sake of brevity, but noted that Asterisk is in fact quite flexible in such matters. We now have a solid foundation on which to build our Asterisk system. In the following chapters we will explore how to connect devices to our Asterisk system in order to start placing calls internally, and work toward increasingly complex concepts in later chapters \(such as video conferencing and WebRTC\).

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409180936-marker) It’s been released under a Creative Commons license, so if you have purchased a hard copy \(and we thank you!\), you can also download a soft copy for searching and copying/pasting.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409178472-marker) Asterisk should run on pretty much any Linux platform, and if you are familiar with the basic process of installing software on a Linux machine, you should find Asterisk a fairly straightforward installation.

[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409176504-marker) By which we mostly mean that you are comfortable administering a system exclusively from the shell.

[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409149976-marker) Elastix is no longer an Asterisk-based or open source product.

[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409144568-marker) After you read our book, of course.

[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409086616-marker) Amazon’s new Lightsail service also promises to simplify the creation of hosted Linux machines.

[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409043048-marker) Note that members of the community will also produce packaged versions of Asterisk. The EPEL repository, for example, maintains a version that can be installed using dnf \(yum\). As of this writing, only the tarball version is officially maintained, and we recommend this method at this time, mostly due to the many different modules that come with Asterisk, and the usefulness in being able to build what you need from source.

[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22idm46178409035896-marker) On a DigitalOcean instance, you’ll need to ensure your SSH key is in the file /home/astmin/.ssh/authorized\_keys.

