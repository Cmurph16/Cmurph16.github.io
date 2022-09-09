---
layout: post
title: "The Importance of Backups"
date: "2021-08-15"
categories: "general"
tags: "backup"
---

[Where'S Our Backup GIF](https://tenor.com/view/i-dont-know-wheres-our-backup-police-the-simpsons-gif-11625986)from [I Dont Know GIFs](https://tenor.com/search/i+dont+know-gifs)

<script type="text/javascript" async src="https://tenor.com/embed.js"></script>

To some, this may seem like an odd first post, especially with how different it is to the other future (at the time of writing) posts. There's a good reason for that, which put simply is that this is not the first post. I had somewhere in the range of 10 Hack the Box write-ups as well as a few random posts, which you may have noticed are nowhere to be found. Here is the story of how I got burned and why I'm now a big fan of excessive backups.

Before I get to the story, security in general has the idea of defense in depth. You never want a single point of failure, and you want as many layers as possible to protect against failures in any of the layers. I recently heard a good description of how to think of defense in depth which I think makes it very easy to visualize (I can't remember the specific source unfortunately but I want to say Eric Conrad in SANS' MGT414 class). Every layer can be thought as a slice of swiss cheese that has holes in random places. If you have one layer, all that needs to happen to defeat it is something to line up with one of the holes. If there is two layers, than not only does something have to line up with a hole on the first slice, there also has to be another random hole in the second slice that is in line with the first. As you can see, adding more slices (layers of defense) adds more protections in the case where something was going to get through the slice (the defenses in general). I included the explanation of defense in depth to show why adding another layer, in this case a backup, can make all the difference. Backups are especially important because depending on the frequency of backups and the changes to what's being backed up, you can basically escape all the other layers failing with no issues and no loss of data.

Now to the story of how I lost all the posts. When I created this site initially, I did it in college to help push me to learn more about security and to share with others who may be trying to learn as well. What I didn't consider was that I wouldn't be in college forever and with that, my college email would not exist forever. Some colleges and universities offer the ability to extend or keep your email, but the one I went to did not. On it's own, not too big a deal. Sure, I may not be able to get notices and other information from the hosting company, but definitely not the biggest issue in the world. As you may have guessed, this was not the only issue I ran into. The second issue, which combined with the previous issue and my own lack of attention on the mater led to my problem, revolved around automatic payments.

<iframe src="https://giphy.com/embed/3orifboWZvOhWnYRvq" width="480" height="366" frameborder="0" class="giphy-embed" allowfullscreen></iframe>

[via GIPHY](https://giphy.com/gifs/thesimpsons-the-simpsons-3x12-3orifboWZvOhWnYRvq)

Automatic payments are great and remove a lot of the worrying about remembering everything you have to pay for in the course of a month or year. My downfall with them was the fact the card I used had a relatively soon expiration date. I didn't think anything of it when I signed up, because the date was around 2 years out and I thought worst case scenario I just get the notice about the card not working and at that time update it to a newer one. My problem now becomes very apparent. I can not get the notice about an expired card because I don't have access to the email, and I forgot that card was used for the website. I then didn't update or check in with the site for a little while and bam, everything is gone. As in most cases, human error was the downfall, but it could've been avoided with backups.

Since this incident, I've become much more diligent with backups. I have backups set up for my notetaking platform every day, week, and month, and am also setting that up this site alongside writing this post. Backups are one of the things everyone knows is important, but it really doesn't sink in until you're burned without having them. I hope that this post can help others not make the same mistake I did, because it is really not fun to try and start from scratch.
