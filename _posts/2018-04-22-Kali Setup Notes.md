---
title: "Basic Kali Setup Guide"
overview: "Kali is a staple of any red team analyst or pentester, with a huge range of tools that will see you through nearly any engagement. Here are a few of the basic setup steps I use when building a new image."
categories:
  - Red Team
tags:
  - kali 
  - red team
  - linux
  - pentest
classes: wide
---

![Banner](https://opalsec.github.io/assets/images/KaliSetupNotes/KaliBanner.png)

# Kali setup guide

## 1. Install vmware tools (Enable system time syncs, copy paste between guest & host, etc.)

#### We can achieve this by cloning and running the script from the below github repo:

```
$> cd ~/
$> apt-get install git gcc make linux-headers-$(uname -r)
$> git clone https://github.com/rasa/vmware-tools-patches.git
$> cd ~/vmware-tools-patches/
$> cp /media/cdrom0/VMwareTools-9.9.0-2304977.tar.gz downloads/
$> ./untar-and-patch-and-compile.sh
```

> #### **Also, make sure the "synchronise guest time with host" checkbox is ticked under** _VM>Settings>Options>VMWare Tools_

## 2. Creating an unprivileged user - necessary to run wireshark

```
$> Useradd -m <username> -G sudo -s /bin/bash
$> Passwd <username>     
Enter a password for the account.
```

**Note:** If no home directory is created:
```
$> mkdir /home/<username>
$> chown <username>:<username> /home/<username>
```

_(Once you've logged into the account)_

Edit the bashrc file in the background

`gedit ~/.bashrc &`

Append `PATH=$PATH:/sbin` to the end of the file - this adds sbin to the path and allows you to type ifconfig and other similar commands without having to sudo them.

## 3. Setting up wireshark to work with your new user

```
$> sudo apt-get install wireshark
$> sudo dpkg-reconfigure wireshark-common
$> sudo usermod -a -G wireshark $USER
```

Log out and in again - you should now be able to run it with your unprivileged user.

#### **Note:** If you're unable to capture on any interfaces, the below command sets the network privileges of the dumpcap exe:
`$> sudo setcap 'CAP_NET_RAW+eip CAP_NET_RAW+eip' /usr/bin/dumpcap`

## 4. Hardening SSH by creating new keys and changing the configured port:

Move your default keys to a new folder:

```
$> cd /etc/ssh/
$> mkdir default_kali_keys
$> mv ssh_host_* default_kali_keys/
```

#### Regenerate the keys:

`dpkg-reconfigure openssh-server`

#### Verify the new keys are different:

`md5sum ssh_host_*` 

#### Compare new key hashes to the hashes below:

```
cd ../default_kali_keys/
md5sum *
```

#### Change the default ssh port:

`$> gedit /etc/ssh/ssh_config &`

Change the number after the Port entry to whatever port you want SSH to be served up on

## 5. If Kali can't resolve the repos:

#### Indicated by it running through the apt-get upgrade process with the "unable to resolve http.kali.org" error.

```
$> cd /etc/apt
$> ls -lah | grep resolv
```
Elevate to root and add write permissions

`$> sudo chmod +w resolv.conf`

Open the file for editing as root so you have write permissions, add nameserver 8.8.8.8 to the end of the file; save it and close it

`$> sudo gedit resolv.conf &`
