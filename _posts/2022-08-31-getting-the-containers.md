---
layout: post
title: Getting the containers
subtitle: This should be easier
thumbnail-img: /assets/img/slow-loading.png
tags: [homelab]
---

This is post three in a development diary I am writing on setting
up a new homelab in my home office.  In the last post I described
setting up the Linux VM and its Docker software to manage docker-
based containers.  Now it was time to load some containers onto the
system.

The first few parts of the container install went very well.  With 
Portainer installed, it was just a matter of selecting items on
[Docker Hub](https://hub.docker.com) and then picking out the items
I needed from the command line examples to load the containers.

- [Heimdall](https://hub.docker.com/r/linuxserver/heimdall)
- [SQL Server Developer Edition 2017](https://hub.docker.com/_/microsoft-mssql-server)

For SQL Server, I used 2017-latest.  It would have helped if I got
the port mapping right the first time though.  I will file that in the
list of things to check later on.  Also, UFW was getting turned on at
startup which did not allow access to Docker containers.  I turned it
off for the moment, but I will need to fix that in the future.

# Autostarting
I am only auto starting Heimdall (which should serve as the entry point
into my homelab) and, of course, Portainer.  I'll manually start SQL Server
when I need it.

# The problem
Aside from mapping the wrong port, everything went fine.  Until I needed
to upload the 20GB database we use for testing.  Fox a user that is rather
rusty on Linux and a novice with Docker, that was a little more difficult.
Also, the sharing service I use claims to have FTP access, but I could not
make it work.  Here is what I finally did:

- Download via Windows to my desktop machine.
- Transfer the file via scp to the VM running docker (using the instructions at
[Linuxize](https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/)).
I used the transfer directory to hold the file.
- Install unzip and then decompress the file.
- Used [docker cp](https://docs.docker.com/engine/reference/commandline/cp/) to
move the file into /var/backups on the container
- Used [SSMS](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver16)
on my Windows machine to restore the backup using the default file locations.

Once this was done I did see a massive amount of memory being used on the VM. A
restart of the VM fixed the issue.



Thumbnail by [Delesign Graphics](https://iconscout.com/contributors/delesign) 
on [IconScout](https://iconscout.com)