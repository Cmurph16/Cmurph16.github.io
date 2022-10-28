---
title: "Try Hack Me - Git Happens"
date: "2022-05-12"
categories: 
  - "try-hack-me"
tags: 
  - "ctf"
  - "starter"
  - "thm"
---

## Intro

Git Happens is one of the challenges in [TryHackMe's](https://tryhackme.com) starter series. This is a group of easy CTF challenges that, as the name suggests, you should start with when trying challenges on TryHackMe. Git happens has one task, which is to find the "Super Secret Password".

> Boss wanted me to create a prototype, so here it is! We even used something called "version control" that made deploying this really easy!

## Enumeration

`nmap` was the first step I took on this challenge:

root@ip-10-10-118-90:~# nmap -T4 10.10.180.251

Starting Nmap 7.60 ( https://nmap.org ) at 2022-05-12 17:40 BST
Nmap scan report for ip-10-10-180-251.eu-west-1.compute.internal (10.10.180.251)
Host is up (0.0012s latency).
Not shown: 999 closed ports
PORT STATE SERVICE
80/tcp open http
MAC Address: 02:EE:D2:E7:C5:1B (Unknown)
Nmap done: 1 IP address (1 host up) scanned in 1.60 seconds
root@ip-10-10-118-90:~#
root@ip-10-10-118-90:~# nmap -p80 -A 10.10.180.251
Starting Nmap 7.60 ( https://nmap.org ) at 2022-05-12 17:41 BST
Stats: 0:00:00 elapsed; 0 hosts completed (0 up), 0 undergoing Script Pre-Scan
NSE Timing: About 0.00% done
Stats: 0:00:11 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 0.00% done
Nmap scan report for ip-10-10-180-251.eu-west-1.compute.internal (10.10.180.251)
Host is up (0.00043s latency).

PORT STATE SERVICE VERSION
80/tcp open http nginx 1.14.0 (Ubuntu)
| http-git:
| 10.10.180.251:80/.git/
| Git repository found!
|\_ Repository description: Unnamed repository; edit this file 'description' to name the...
|\_http-server-header: nginx/1.14.0 (Ubuntu)
|\_http-title: Super Awesome Site!
MAC Address: 02:EE:D2:E7:C5:1B (Unknown)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.8 (95%), Linux 3.1 (94%), Linux 3.2 (94%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.2 - 4.8 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux\_kernel

TRACEROUTE
HOP RTT ADDRESS
1 0.43 ms ip-10-10-180-251.eu-west-1.compute.internal (10.10.180.251)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.56 seconds
root@ip-10-10-118-90:~#

This shows there is a webserver on port 80 running `nginx 1.14.0 (Ubuntu)`. From the default scripts run because of the `-a` flag in the second `nmap` command, it is shown that the server is exposing a .git folder. If the name didn't give away what the challenge was, this also steers you to investigating git. ## Git To get the files and dig more into git, I downloaded the directory with:

wget -r http://10.10.180.251/.git/

From here, I used [this guide](https://medium.com/swlh/hacking-git-directories-e0e60fa79a36) to extract the password. Basically, you look through the commits to find one that looks off. To start, the most recent commit hash can be found like this:

$ cat .git/HEAD
ref: refs/heads/master
$ cat .git/refs/heads/master
0a66452433322af3d319a377415a890c70bbd263

Moving backwards from here involves using the git cat-file command. By running:

git cat-file -p OBJECT-HASH

you get different results based off of what the object is. If it's a tree, you will see the files present in the commit and the commit message. If it's a blob, you see the code. I kept running this with the parent hash until I found a commit message that said:

> Obfuscated the source code.  
> Hopefully security will be happy!

As you can guess, the commit before this was one that had unobfuscated code in it. I followed the same steps again with moving back one more commit, then investigated the source of the website and was able to find the password in the code:

... snipped ...
if (
username === "admin" &&
password === "T---redacted-for-rules---!"
) {
document.cookie = "login=1";
... snipped ...
