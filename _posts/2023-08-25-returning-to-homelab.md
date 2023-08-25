---
layout: post
title: Returning to the lab
subtitle: Digging a hole
cover-img:  /assets/img/wildhole.jpg
thumbnail-img: https://wp-cdn.pi-hole.net/wp-content/uploads/2016/12/Vortex-R.png
tags: [homelab]
---

Its been awhile.  But after quite some time away I wanted to add a couple of services to my homelab.  And if I am going to get back to it, I might as well start with an update of what I have been doing with the services setup so far.  Then I will get into what I am doing this time - setting up [Pi-hole](https://pi-hole.net/).

# The database server
The database server container has been going well.  I did need to expand the available drive size significantly, but it does work.  The biggest issue is that it consumes all the available memory on the Docker host machine when it is running full tilt.  This is expected as the databases I have been working with are now exceeding 100 GB in size.  It may be time in the near future to create a new development server on its own VM as opposed to being in a container.

# What is Pi-hole
Pi-hole is a DNS sink.  What is that and why is it valuable?  That requires a very short trip into how the internet works.

## The Domain Name Service
Every internet site has an address.  But that address is not, for example, google.com.  Instead it is a four part number such as 142.250.65.238 (one of google's addresses) just like old phone numbers known as in **IP address**.  But just like old phone numbers, they are hard to remember.  So, the Domain Name Service (DNS) translates a common name like "google.com" to that address.  You need a DNS to make the internet work and the typical user would just use a public DNS (such as Google's at 8.8.8.8 and 8.8.4.4) to make the internet useable.

## Why is a sink useful?
There are two main reasons.  First is so that a homelab user, like myself, can add entries to the DNS for their local machine.  I don't want these publicly available, but when I am at home I want to be able to access my other machines no matter what device is accessing the network.

The bigger reason, by far, though is because it is a hack to prevent ads.  For example, you have probably run across the infamous Taboola ads.  These are (one of) the ones with the ridiculous photoshopped images and the click bait titles.  If you need an example, try our local newspaper, [PennLive](https://www.pennlive.com/) who loves posting these ads all over the place.  I could use an ad blocker, but that triggers a popup.  Not only that, but who knows who is tracking me with off-site references.

Here is where the hack comes in - instead of having my browser block the ads, we will let the browser go ahead and do its thing.  But when we lookup the web address for the ad provider (i.e., ads.doubleclick.net), the DNS sink takes the request to translate that address to an IP address and returns noting.  And if you don't have an IP address, you can't get the requested data.  Not only that it works for websites, apps, on my TV, and on *anything* that uses an internet lookup.

As it turns out, the same hack to avoid ads *might* also be useful to avoid some hackers.  By blacklisting known hacker sites, Pi-Hole can make my internet a little more safe.

# Where to put Pi-hole & consequences of that decision
There is a significant trick to deploying Pi-Hole though.  Unlike the database server, this is an essential internet service.  If it goes down, the entire internet will seem to go down (for me at least).  That means it cannot go on the main Docker VM which is something I turn off when I don't need it.  So, it has to go on the one machine in my home that is always running - the Synology NAS.  No worries, Synology does docker!  And this is where the "but" lives.  The operating system of the Synology is called Disk Station Manager (DSM) which is different from Mac or Windows.  Instead it is a custom Linux distribution.  But for the purposes of this discussion, it is a separate OS.  On that OS, there is a docker app and there is a video on how to install Pi-Hole on the docker app provided by [WunderTech's Pi-Hole Video](https://youtu.be/OzI5DRfVAdc?si=g4u4dffbS4OVHRWF) and the [associated text instructions](https://www.wundertech.net/how-to-setup-pi-hole-on-a-synology-nas-two-methods/).  This was fine until I saw a [video](https://youtu.be/_iffYh7Ewy8?si=k1f2ga0Ot99ISXUf) about the latest version of DSM, 7.2.  The end result of this is that Docker is replaced by Container Manager in 7.2 and the instructions are slightly different.

# Installing Pi-hole
Fortunately, another user [Tech Me Out](https://youtu.be/Xk6DBMAK5Bk?si=IZNkiV6zPLT1pKPG) already did a video on how to install Pi-Hole on DSM 7.2 along with providing a script.  All I had to do was follow the instructions.  These included a docker compose file (this is a specially formatted file that tells Docker how to make a container along with the specific settings).  For reference, this is the file I used:
```
version: "3"

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    network_mode: "host"
    environment:
      TZ: 'America/New_York'
      WEBPASSWORD: 'THIS_IS_NOT_MY_PASSWORD'
      WEB_PORT: '9999'
    volumes:
      - '/volume1/docker/pihole/pihole:/etc/pihole'
      - '/volume1/docker/pihole/dnsmasq.d:/etc/dnsmasq.d'
    restart: unless-stopped
```
Note that I did change the web port as I was concerned it would conflict with some of my web development activities.  Also, I am using the latest pihole image from the Docker repository.

# Pointing to the hole
Next, I had to tell my machine(s) to use the Pi-Hole provided DNS instead of Google's.  This used to be simple, but now it is more complicated with the "simplification" of the Control Panel in Windows 10/11.  The instruction to change the DNS server are provided by [Windows Central](https://www.windowscentral.com/how-change-your-pcs-dns-settings-windows-10).  For the primary IP address of the DNS server, I used the address of my Synology NAS (where the Pi-Hole container sits).  For the secondary address, I used the IP address and added 1 to the last number which should lead to a server that does not have DNS services.  Since it is still and address that starts with 192.168 I don't have to worry - anything starting with that address is part of the local network (this is part of the specification).

# Fixing the Pi-Hole settings
The first time I pointed my laptop to the Pi-Hole DNS server, I broke the internet - at least on that machine.  After a bit of poking around, I found I had to change the Interface Settings to "Allow only local requests" as shown here:  ![Pi-Hole DNS Settings](/assets/img/PiSettings.png).

There is a way to fix this according to some users by adding a delay on Pi-Hole startup.  Check [this post](https://discourse.pi-hole.net/t/interface-eth0-does-not-currently-exist/51810) to see the suggestion.  It also seems that others are using the same approach but since the recommended setting seems to work for now, I have not tried it.

# Next Steps
This is only the first phase of the process.  I don't want to move on until I have fully tested the system.  However, there are two things that *need* to happen next and some other things that I need to do in the future.

## Moving the DNS to the router
At the moment, each machine has to know to access the DNS server directly.  This is kind of a pain and in some cases may not be possible (e.g., my television).  But once I have tested it, I do plan to move the DNS server reference to the router where any device will be able to use the DNS Sink provided by Pi-Hole.

## Making my Pi-Hole resilient
I also need to address the fact that DNS services are critical to the internet working.  To do that, I am going to need more than one Pi-Hole instance.  Initially, I can do this with my main Docker server (since it is on a different machine).  But there are some issues with setting up synchronization which are addressed in the [Techno Tim Video](https://youtu.be/IFVYe3riDRA?si=vZkM03vfMLhQx92d).

## Improving the blacklist
The blacklist in Pi-Hole is the list of sites that Pi-Hole returns no address for.  The efficacy of Pi-Hole depends on this list.  But there are better lists out there and I need to investigate where to pull those from.  Lets get the infrastructure in place first though and use the default lists until then.