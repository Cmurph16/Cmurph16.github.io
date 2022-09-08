---
title: "100 Red Team Projects - Intro and 00"
date: "2021-09-19"
categories: 
  - "projects"
tags: 
  - "100rtp"
  - "python"
  - "scripting"
coverImage: "red.png"
---

Recently I've been doing some more scripting with work, and it really got me back into it. I've been working on the eCPPT class from eLearnSecurity and had fallen off with scripting, and found the perfect excuse to get back into it on Reddit. This excuse was [100 Red Team Projects](https://github.com/kurogai/100-redteam-projects). It's a list from GitHub user Kurogai that has 100 projects of various difficulty designed to teach you to build your own tools. I thought this was an awesome idea and started off with the first one, a TCP or UDP server just to receive messages. It will be improved and edited in some of the subsequent projects, but the first part is pretty basic. Most of the posts I'll do about this will have more than one project, but as this had the introduction about the topic I stuck with just the first.

I set up a GitHub repo for this project and others to make all the code accessible. This one is at [https://github.com/Cmurph16/100redTeamProjects/blob/master/00tcpReceiveServer.py](https://github.com/Cmurph16/100redTeamProjects/blob/master/00tcpReceiveServer.py), but the others will be in that 100redTeamProjects repo. I'm not going to cover every section of the code in this post, so definitely check out the GitHub to get everything.

The first main block sets up the server. It uses python sockets and starts up a TCP listener on port 80.

#creating a TCP socket
s = socket.socket(socket.AF\_INET, socket.SOCK\_STREAM)
# bind the socket only accessible to our machine and port 80
s.bind(('localhost', 80))

While the servers running, this block of code runs. The basic idea of what it's doing is accepting connections to the server, and assigning each new connection a unique user ID number. It uses a dictionary to store the id associated with a particular address and port tuple. On a connection, it also prints out the IP address and port, as well as the assigned name.

clientsocket, (address, port) = s.accept()
# if the socket already exists, get it's userID
if (address,port) in userNameToSocket:
    name = userNameToSocket\[(address,port)\]
# otherwise assign it the next sequential userID
else:
    name = userID
    userNameToSocket\[(address,port)\] = userID
    userID+=1
print("Connection from {} started on port {}. User ID: {}".format(address,port, name))

The last block prints out the data received. It constantly listens for 1024 bytes of data, and prints the given data with the userID.

while clientsocket:
        # receive input
        data = clientsocket.recv(1024)
        if not data: break
        # print out the userID and the input they gave
        print("{} > {}".format(name, data))

All in all, this code is pretty simple and bare bones. A lot of the project ideas in the basic section deal with servers like this, so I'll be updating it and making a better server as it goes on. In it's current state there is a known problem I've run into, which is it can only have one connection at a time. The easy fix for this is to make it multi threaded, which is project number 03 so I'm going to leave it til then to get it working.

## Resources

These are some of the resources I used to complete the project, in case you're interested in learning more:

- [https://docs.python.org/3/library/socket.html](https://docs.python.org/3/library/socket.html)
- [https://stackoverflow.com/questions/7174927/when-does-socket-recvrecv-size-return](https://stackoverflow.com/questions/7174927/when-does-socket-recvrecv-size-return)
- [https://pythonprogramming.net/sockets-tutorial-python-3/](https://pythonprogramming.net/sockets-tutorial-python-3/)
- [https://docs.python.org/3/howto/sockets.html#creating-a-socket](https://docs.python.org/3/howto/sockets.html#creating-a-socket)
