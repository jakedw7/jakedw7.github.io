---
layout: post
title: Setting up a docker server
subtitle: Any port in a storm
cover-img: /assets/img/port.jpg
thumbnail-img: https://www.docker.com/wp-content/uploads/2022/03/horizontal-logo-monochromatic-white.png
tags: [homelab]
---
So now it is time to actually setup the server that will contain docker
along with docker itself.  I will finish up the docker setup by installing
[Portainer](https://www.portainer.io/) to manage the docker containers through
a web UI.  But before I can consider anything else, I actually have to determine
how many resources to allocate to my docker server based on the software 
requirements and my largest expected container workload.

# Overall system requirements
These are the overall system requirements maxed out for each element.

- 2 processor cores
- 4 GiB Ram
- 45 GB Hard Drive

# Database container needs
Of course, the system requirements for the database are more significant.

- 16 GiB Ram
- 50 GB Hard Drive

# Creating the linux VM
The first step is to create a linux VM to host docker containers.  I am
using Ubuntu Server 22.07 for that VM.  Otherwise, I am following the
tutorial mentioned on [Learn Linux TV](https://www.youtube.com/watch?v=xBUnV2rQ7do&ab_channel=LearnLinuxTV).
One thing I did change is that I told Ubuntu to go ahead and **install docker**
during installation.  This should cut down on a future step.  VM settings
I used:
- 2 Cores (of 8)
- 24 GiB RAM
- 128 GB Drive Space on the SSD.

Since I am putting this on the SSD, I did make sure to enable **Discard**
on the storage screen.  This all worked fine.

# Linux Setup
I am switching from Learn Linux TV to [Techno Tim](https://www.youtube.com/watch?v=ZsjK4VDopiE&ab_channel=TechnoTim)
as he provides a more detailed tutorial and, frankly, is a little easier
to listen to.  It also is a little more detailed.  Most of the instructions
were fine and the [docs](https://docs.technotim.live/posts/fist-13-things-linux/)
helped.  However, I did have the following issues:

## Machine not rebooting
When I installed qemu, I was unable to reboot the machine.  Here is
[the fix](https://dannyda.com/2020/05/11/how-to-fix-proxmox-pve-cant-lock-file-var-lock-qemu-server-lock-xxx-conf-got-timeout-proxmox-cant-shutdown-virtual-machine/) 
that I tried.  Remember, this is on the proxmox server itself, not
on the VM.  However, this also failed.  So, I scrolled down and manually
ran the "simple script" in the bonus section.

## UFW and Docker
Apparently the Universal Firewall does not play well with docker.  But,
as it is a common issue, as presented at [How-To Geek](https://www.howtogeek.com/devops/how-to-use-docker-with-a-ufw-firewall/) there is already a fix that is
available.  I installed ufw-docker and followed the instructions as
listed in this article.

# Installing Portainer
With the VM up and running with docker installed, the next step is
installing the Portainer container per the instructions 
[here](https://docs.portainer.io/start/install/server/docker/linux).  Of
course, I needed to remember to use the Community Edition versions of the
commands in the instructions.

## Issues with permissions
I did have an issue trying to perform any command line operations on docker
without using admin.  I finally gave up and did

```
sudo chmod 666 /var/run/docker.sock
```

## More firewall issues
And conflicts with UFW, as expected.  Since this is internal, I shutdown UFW
for now until I can figure out what is going on.

## No containers
No container option was listed in Portainer until I clicked on the environment
on the home screen.  Then it showed up.

# Credits
Cover image by Andres Alvarado.  https://www.flickr.com/photos/104082864@N04/51369131584