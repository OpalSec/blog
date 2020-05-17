---
title: "Linux DFIR Cheat Sheet"
categories:
  - DFIR
tags:
  - linux
  - dfir
  - logging
  - forensics
classes: wide
---

# Overview
Linux is often used to provide Dev environments and backend servers, and need to be accounted for along with Windows workstations and servers and network appliances when building a network.

With the right logging and understanding of the artefacts they provide, you'll have a fighting chance of detecting and responding to intrusions against these assets.

# Directories of Interest

### /etc (equivalent of %SystemRoot%/System32/config)
 – The primary system configuration directory with separate configuration files/dirs for each app

### /var/log (equivalent of Windows event logs)
 – Security logs, application logs, etc. Logs normally kept for about 4-5 weeks

### /home/$USER (equivalent of %USERPROFILE%)
 – Contains user data and user configuration information

# Main Logs of Interest

All of these logs focus on tracking authentication and elevation events on the system. Use these to determine who was logged in during the time of the incident, if they were able to sudo, and if the HTTP service played a part in the breach.

- **/var/log/auth.log** - All authentication related events in Debian and Ubuntu server are logged here
- **/var/log/secure** - RedHat and CentOS based systems use this log file instead of /var/log/auth.log (It also tracks sudo logins, SSH logins and other errors logged by system security services daemon)
- **/var/log/audit/audit.log** - I don't run any systems that have this on file, but it looks like it's only used in RedHat and CentOS. This would contain SELinux message as well.
- **/var/log/faillog** - This file contains information on failed login attempts
- **/var/log/sudo.log** - Log of sudo activities, needs to be enabled - instructions below
- **/var/log/sudo-io** - All of the commands run during a particular sudo session.
- **/var/log/httpd/access_log** - All access requests received over HTTP are stored here
- **/var/log/wtmp** - Shows user, source, time, and duration of login but can only be viewed with the "last" command on a Linux system. This is covered later.

## Enabling sudo.log

Add this to your sudoers file
```
Defaults log_host, log_year
Defaults log_input, log_output, logfile="/var/log/sudo.log"
```

For Example:
```	
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults	env_reset
Defaults	mail_badpass
Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

#Logging Settings
Defaults log_host, log_year
Defaults log_input, log_output, logfile="/var/log/sudo.log"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root	ALL=(ALL:ALL) ALL

# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d
```

This will create both a /var/log/sudo.log file, but also the directory /var/log/sudo-io where you'll find all of the commands run during a particular sudo session.
Some of the files in that directory structure are gzip compressed, so you will need zcat to read them.

It will give you logs like this:
```
Sep 25 20:45:15 2019 : montana : HOST=Cortana : 1 incorrect password attempt ;
    TTY=pts/5 ; PWD=/home/montana ; USER=root ; COMMAND=/sbin/ifconfig
Sep 25 20:47:18 2019 : montana : HOST=Cortana : TTY=pts/6 ; PWD=/home/montana ;
    USER=root ; TSID=000003 ; COMMAND=/sbin/ifconfig
Sep 25 20:47:21 2019 : montana : HOST=Cortana : TTY=pts/6 ; PWD=/home/montana ;
    USER=root ; TSID=000004 ; COMMAND=/bin/su
```

# Users

Use these to establish what accounts are active and have sudo permissions; ensure there are no hashes in /etc/passwd (old unix convention and crackable)

- **/etc/passwd** - UID 0 accounts have admin privs, look for multiple UID 0 accounts
- **/etc/shadow** - look for "application" accounts with active passwords
- **/etc/sudoers** - who can run as root and what they can do
- **/etc/group** - group memberships, which is important when assessing if sensitive files have the SGID bit set

## User Artefacts

These can indicate what hosts were connected to/from via SSH; the browsing history; and what files an attacker looked at and may have interacted with if they used a graphical file explorer

### SSH:
- **$HOME/.ssh/known_hosts** – hosts user connected to from here
- **$HOME/.ssh/authorized_keys** – public keys used for logins to here
- **$HOME/.ssh/id_rsa** – private keys used to log in elsewhere
- **grep "PermitRootLogin " /etc/ssh/sshd_config 2>/dev/null \| grep -v "#" \| awk '{print  $2}'** - is root permitted to log in via ssh

### Browsing History:
- **$HOME/.mozilla/firefox/*.default** - Firefox browser history
- **$HOME/.config/chromium/Default** - Chrome browser history

### File System Enumeration:
- **$HOME/.thumbnails** - Nautilus (graphical file explorer) thumbnails
- **$HOME/.recently-used.xbel** - Nautilus (graphical file explorer) recent files

## User Activity History

These are crucial in correlating a timeframe for an incident with known logon events, and seeing what commands had been run

- **last 2>/dev/null** - shows historical logins (contents of /var/log/wtmp)
- **lastlog 2>/dev/null \| grep -v "Never" 2>/dev/null** - shows last logon time per user, filtering out accounts never logged in
- **find /home -name .bash_history -print -exec cat {} 2>/dev/null \;** - location and contents (if accessible) of .bash_history file(s)
- **ls -la /root/.*_history 2>/dev/null** - root's history file if accessible

# Suspicious processes

Look for non-standard processes; ones being run with a SGID bit set; standard binaries located in non-standard locations

- **ps aux 2>/dev/null \| awk '{print $11}'\|xargs -r ls -la 2>/dev/null \|awk '!x[$0]++' 2>/dev/null** - Look up the binary paths & permissions of running processes

# Network Artefacts

Useful to identify hosts that were interacted with or connected to; if a static nameserver was configured (DNS C2); abnormal services are listening as reverse shells; and if the computer has been connected to different networks

- **arp -a 2>/dev/null** \<OR\> **ip n 2>/dev/null** - ARP history
- **grep "nameserver" /etc/resolv.conf 2>/dev/null** \<OR\> **systemd-resolve --status 2>/dev/null** - Configured nameservers
- **netstat -ntpl 2>/dev/null** \<OR\> **ss -t -l -n 2>/dev/null** - Listening TCP services
- **netstat -nupl 2>/dev/null** \<OR\> **ss -u -l -n 2>/dev/null** - Listening UDP services
- **cat /var/lib/dhcp/dhclient*leases 2>/dev/null** - DHCP lease history for individual interfaces

# Persistence

Look at the creation timestamps for jobs and entries in init, cron, and inetd-related directories and files. inetd files can be altered to serve bash instead of their original service, or new ones added to provide reverse shells.

## Hidden Files
- **find / -name ".*" -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null** - look for hidden files (lots of false positives for system files)

## Cron Jobs
- **ls -la /etc/cron\* 2>/dev/null** - lists cron jobs
- **cat /etc/crontab 2>/dev/null** - lists configured cron jobs
- **cat /var/spool/cron/crontabs 2>/dev/null** - lists user-configured cron jobs
- **cat /etc/anacrontab 2>/dev/null** - anacron is an alternate to cron which ensures jobs run ASAP if the computer was powered down during the time it was scheduled to run
- **ls -la /var/spool/anacron 2>/dev/null** - when were jobs last run

## init files

- **find /etc/init.d/ \\! -uid 0 -type f 2>/dev/null \|xargs -r ls -la 2>/dev/null** - files in init.d not belonging to root (used by **traditional System V Init** service management package)

- **find /etc/init \\! -uid 0 -type f 2>/dev/null \|xargs -r ls -la 2>/dev/null** - files in init not belonging to root (used by **newer Upstart** service management package primarily used in Ubuntu)
- **find /etc/rc.d \\! -uid 0 -type f 2>/dev/null \|xargs -r ls -la 2>/dev/null** - rc.d files not belonging to root (dictate run levels, e.g. in RHEL rc3.d - run level 3 - is for a system without a desktop)
- **find /etc/inittab/ \\! -uid 0 -type f 2>/dev/null \|xargs -r ls -la 2>/dev/null** - files not belonging to root, inittab defines the run level at boot, rc.d contains symlinks to scripts in init.d which are run

## inetd & xinetd

The inetd daemon was created to manage many daemons or services by listening to multiple ports and only invoking requested services when needed. This includes telnet, File Transfer Protocol (FTP), and Simple Mail Transfer Protocol (SMTP) 

The extended Internet services daemon (xinetd) replaced inetd, performing the same function as inetd, but in a more secure manner by providing service-specific configuration options for access control, enhanced logging, binding, redirection, and resource utilization control. Typical xinetd services include Remote Shell (RSH), FTP, telnet, and Post Office Protocol 3 (POP3).

- **cat /etc/inetd.conf/\* 2>/dev/null** - contains a list of network services that tell inetd which ports to listen to and which server programs to run in response to service requests
- **cat /etc/xinetd.conf/\* 2>/dev/null** - sets the number of active servers, specifies logging settings, limits the number of incoming connections, and reads all files in /etc/xinetd.d
- **cat /etc/xinetd.d/\* 2>/dev/null** - contains additional settings, including a disable parameter which is set to yes by default. 

### Example inetd backdoor:
```
Pick an obscure service from /etc/services associated with a tcp port 1024 and above...for example laplink
	laplink         1547/tcp     # laplink

Add the following line to /etc/inetd.conf
	laplink    stream  tcp     nowait  /bin/bash bash -i

restart inetd.conf
	$> killall -HUP inetd

This creates a listener on port tcp/1547 that will shovel you a bash shell. 
```
