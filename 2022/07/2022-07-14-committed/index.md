---
title: "Try Hack Me - Committed"
date: "2022-07-14"
categories: 
  - "try-hack-me"
tags: 
  - "git"
  - "thm"
---

## Intro

This is a quick CTF from [TryHackMe](https://tryhackme.com) that is pretty similar to [Git Happens](https://tryhackme.com/room/githappens), another challenge from TryHackMe (My writeup for that is [here](https://connormurphysecurity.com/try-hack-me/try-hack-me-git-happens/)). It's another instance of a git repo that has some secrets committed to it accidentally. All that was needed was to find them.

## Initial review

Once the machine loads, the files were accessible at `/home/ubuntu/commited`. A quick check shows the git repo in question:

ubuntu@thm-comitted:~/commited/commited$ ls -a
.  ..  .git  Readme.md  main.py
ubuntu@thm-comitted:~/commited/commited$ 

## Finding the right commit

My line of thinking was as this is an easy challenge, a commit message will give away which commit fixed the problem. All I would need to do is look at the previous commit and see what the code added was. This was how I initially looked through the commits:

ubuntu@thm-comitted:~/commited/commited/.git$ git log --all
commit 28c36211be8187d4be04530e340206b856198a84 (HEAD -> master)
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:49:32 2022 -0800

    Finished

commit 4e16af9349ed8eaa4a29decd82a7f1f9886a32db (dbint)
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:48:08 2022 -0800

    Reminder Added.

commit c56c470a2a9dfb5cfbd54cd614a9fdb1644412b5
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:46:39 2022 -0800

    Oops

commit 3a8cc16f919b8ac43651d68dceacbb28ebb9b625
Author: fumenoid <fumenoid@gmail.com>
Date:   Sun Feb 13 00:45:14 2022 -0800

    DB check

It looks like commit `3a8cc16f919b8ac43651d68dceacbb28ebb9b625` is the one that has the secrets. Before we dig into that, I want to share a way I think is better to search through the commits. The better command is `git log --oneline --graph --decorate --all`. After I finished this challenge, I did some research because the other way seemed to lose some context to the commits. I missed the `--all` flag at first and as this commit was in a branch it was missed. You still need the `--all` in this new command, but it hides some of the info you don't need at first and draws the commits in a linear format:

ubuntu@thm-comitted:~/commited/commited/.git$ git log --oneline --graph --decorate --all
\* 28c3621 (HEAD -> master) Finished
| \* 4e16af9 (dbint) Reminder Added.
| \* c56c470 Oops
| \* 3a8cc16 DB check
| \* 6e1ea88 Note added
|/  
\* 9ecdc56 Database management features added.
\* 26bcf1a Create database logic added
\* b0eda7d Connecting to db logic added
\* 441daaa Initial Project.

Note the difference in the length of the commit ID returned (`3a8cc16` and `3a8cc16f919b8ac43651d68dceacbb28ebb9b625`), but either size will work.

## Digging into the commit in question

Now we have a commit in question, we can check it out with the git `cat-file` command. This lets us see which files and code were included in the commit. It is used as follows:

ubuntu@thm-comitted:~/commited/commited/.git$ git cat-file -p dc1ca4ca1d54e7a4ac6757c9b98bd1be0a8ed2f0
100644 blob 4eca752261f327712539fc04f81e7f335a69b429	Note
100644 blob 40754840a68f85ad8d963f1556a3f24b51cef4fa	Readme.md
100644 blob 54d0271a615735240d22dcd737b4bf26cbe9d43f	main.py

This shows the different files included in the commit. These files, while in git, are called blobs (more info [here](https://docs.github.com/en/rest/git/blobs)) but you can really just think of them as a file. The file that seems the most interesting is `main.py`, so I started there. It can be viewed with the same `cat-file` method:

ubuntu@thm-comitted:~/commited/commited/.git$ git cat-file -p 54d0271a615735240d22dcd737b4bf26cbe9d43f
import mysql.connector

def create\_db():
    mydb = mysql.connector.connect(
    host="localhost",
    user="root", # Username Goes Here
    password="REDACTED" # Password Goes Here
    )

    mycursor = mydb.cursor()

    mycursor.execute("CREATE DATABASE commited")

def create\_tables():
    mydb = mysql.connector.connect(
    host="localhost",
    user="root", #username Goes here
    password="REDACTED", #password Goes here
    database="commited"
    )

    mycursor = mydb.cursor()

    mycursor.execute("CREATE TABLE customers (name VARCHAR(255), address VARCHAR(255))")
    

def populate\_tables():
    mydb = mysql.connector.connect(
    host="localhost",
    user="root",
    password="REDACTED",
    database="commited"
    )

    mycursor = mydb.cursor()

    sql = "INSERT INTO customers (name, address) VALUES (%s, %s)"
    val = ("John", "Highway 21")
    mycursor.execute(sql, val)

    mydb.commit()

    print(mycursor.rowcount, "record inserted.")

create\_db()
create\_tables()
populate\_tables()

The password (redacted per TryHackMe's policy) is the flag we were looking for. All that is left is to submit the flag.
