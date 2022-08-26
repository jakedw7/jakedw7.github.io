---
layout: post
title: What is a homelab?
subtitle: And why I am bothering with it?
cover-img: /assets/img/HomeNetwork_SimeonW.jpg
thumbnail-img: /assets/img/MyServer.jpg
tags: [homelab]
---

This is post one in a development diary I am writing on setting up a new homelab
in my home office.  But before I can begin the post, I need to answer a pair of
important questions about this project that any project manager should be able
to answer immediately about any project they are developing, personal or
professional.  Why am I doing this and what do I hope to achieve?

To answer these two questions, I need to start with defining some terminology
and then discuss my specific needs.

# What is a homelab?
I am a newcomer to this kind of project and I will explain later on how I even
came across this community in the first place.  But because I am so new to this,
a definition is in order.  I am going to the definition provided by the
[Linux Handbook](https://linuxhandbook.com/homelab/#:~:text=In%20case%20you%20have%20never,for%20home%20and%20functional%20usage.) although similar definitions have also
been provided by others as well.
> Homelab is the name given to a server (or multiple server setup) that resides locally in 
> your home and where you host several applications and virtualized systems for testing and 
> developing or for home and functional usage.
In other words, it is an effort to replicate the services usually provided by
a professional IT group for a specific business or groups.  It is usually built
on leftover hardware or secondhand equipment as it isn't production critical.

## Why setup a homelab?
There are many reasons that people setup homelabs, but based on my initial
investigations, I can reduce the reasons to four common items:
- Curiosity - some people just want to know how the world works.
- Training - others are planning on a career in IT or want to learn something new without damaging production systems
- Testing - if you are putting together a project and need specific server infrastructure, a homelab is useful
- Control - there are a lot of basic services that other companies provide that a person may want to control themselves

# My home setup
Right now, I have a very simple setup.  I have a personal desktop computer at
home that I use for both gaming and development (personal and professional
projects).  In addition to that, there is almost always a laptop plugged into
my network, usually my work computer.  Finally, I have a small Synology NAS DS718+
which is now the [Synology DS720+](https://www.synology.com/en-us/products/DS720+)
in the current Synology lineup.  ![Synology DS720+](https://www.synology.com/img/products/detail/DS720plus/heading.png)

# The issue
I am currently developing an open source software package for transportation
agencies (think state-level departments of transportation or a city's department
of public works) called [Infrastructure Asset Management](https://repo.iam-demo.net/).
Our demo data uses a large database to run the system.  What I have done for
quite some time is run the database on the development machine.  But given the
database size, it is starting to bog down my desktop.  And if I am not developing,
I have to shutdown the database server manually to free up memory and processing
cycles.

## Option 1:  Run the database on my laptop
Since it is my work machine that is most often plugged into the network, using
that would be ideal.  After all, it does have the database running in
background on a regular basis.  However, my company is very protective of its
network (as it should be!).  So, I cannot access the database on my laptop from
my home machine.

## Option 2:  Develop on the work laptop
This is usually OK, but it is restricted by our IT department.  It is simply
easier to develop on my home machine.  And since this I am working on an open 
source product, I don't worry about the normal security issues related to working
off-network.  However, I also like running my communications software (email, chat, etc.)
on my laptop while developing as well.  So this option is out.

## Option 3:  Run in the cloud
Performance issues exist when I run the database in the cloud.

## Option 4:  Run a container on the Synology
The database is huge.  The Synology is great for a lot of basic network tasks but
a database of this size running in a Docker container on the Synology is not
realistic

# Conclusion
Given these options, it is time for the homelab route.  Also, I would like to
try some of the other suggested things that I have seen in the community
for myself.  So here we go!