---
layout: post
title: Size matters
subtitle: Some things are harder than they should be
thumbnail-img: /assets/img/disks_streched.jpg
tags: [homelab]
---

I have not been posting much as I have actually been using my setup for work.
This is a positive thing and it was working well until I had to do a full
dataset test.  To noone's surprise, I ran out of disk space - SQL Server is a
hog when it comes to log files and large operations.  However, just enlarging
the disk in proxmox was not enough to make my system work for this dataset.
Here is everything I had to do.

# Enlarge the VM Drive
OK, this part was easy.  Go into Proxmox, select the machine that serves my
Docker containers, go to the hardware, select the hard drive and resize it
using the Resize option in the Disk Action menu.  No big deal.  But, when I
logged into the VM and ran `df` nothing had changed!  This was not that
unexpected - after all, a Windows machine does the same thing when you add
a drive.  

Time for some help which was provided by Proxmox at 
[Proxmox Drive Resize Support](https://pve.proxmox.com/wiki/Resize_disks).
There was just one problem: these instruction were horribly unclear.

# Partitioning
This part was pretty much standard and easy to follow.  First, to see what 
dev my virtual hard drive was on:

`fdisk -l` 

which, of course, had to be run with elevated privileges (i.e.,
sudo).  This showed that my virtual drive was on /dev/sda and partitioned
into 3 logical drives (as most OEes are wont to do).  In my case, the 
partition with user accessible data was on /dev/sda3 (based on comparing
it with the results of `df`).

Expanding the data partition was simple.  Just run `parted` and resize the
appropriate partition.  No issue here although it would have been more clear
if the support file showed the results of the print command from their
example after fully expanding /dev/sda.

So now sda3 has the extra space, but the VM still can't use it.

# Expanding
Unlike Windows, the filesystem does not automatically expand to use the
available space with the increased volume post-partitioning.  As it turns out,
you have to do it manually in Linux.

First step is to resize the physical volume: `pvresize /dev/sda3`.  That part
was _kind of_ clear.  However, I did not realize that this had to be done on
/dev/sda3.  

Even though this was done, the logical volume still had to be expanded.  Data
is actually stored on an LVM partition, so the next step was to expand this.
`df` tells me the system is using /dev/mapper/ubuntu-vg--ubuntu-lv.  This is
done with lvresize:  `lvextend -l +100%FREE /dev/mapper/ubuntu-vg--ubuntu-lv`.
I got this from another website though so I did not run this with the 
`--resizefs` flag.  So, there was *one* more step.

Now the physical and logical volumes were both expanded.  But the filesystem,
a third layer in this system, had to be expanded.  This isn't hard once I
understood I had to do it: `resize2fs /dev/mapper/ubuntu-vg--ubuntu-lv`.  Now
I could actually start the DB server again and get back to testing!

# Conclusions
This is a diary, not a how to guide although if you have stumbled upon this
and it helps, I am glad to see it was useful to someone else.  Just keep in
mind that I am not an experienced Linux sysadmin (at least not these days!)
and some of what I have done may not match best practices in the system admin
realm.  If you have suggestions that will assist with this, please reach out
via email!