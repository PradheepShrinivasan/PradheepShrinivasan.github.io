---
layout: post
title: "Mounting iso disk on Linux"
published: true
comments: true
---
Recently i was asked to set up linux boxes for our lab systems and so i started installed CentOS in the linux machines. The installation process was easy, mostly using the default configuration.However there was one issue, the lab machines did not have cd players and access to the internet was restricted.

We needed a mechanism to set up to install packages to without using DVD drive or connecting to the internet. We thought why not copy the iso images to the linux boxes and set the package manager (YUM) to use the iso image ? We did the same and i will now explain how we set up them image.

First lets assume that the DVD image (.iso) is copied to some directory say /home/centos/Centos.iso.The first step is to mount the image to the file system.

```bash
$mkdir /mnt/centos
$mount -o loop -t iso9660 /home/centos/centos.iso /mnt/centos
```

Now you can view the content of the centos. check it using

```bash
$ls /mnt/centos
```

The next step is to configure yum to look for the file we have just now mounted

```bash
$vim /etc/yum.repos.d/CentOS-Media.repo
```

add the mount path to the file and then set the enabled in the file to 1

```bash
file:///mnt/centos
enabled =1
```
Save the changes and exit.Note:You have to be root to do all these changes

Now you have to disable the base repo else yum will be complaining unable to load the base repository file

```bash
$vim /etc/yum.repo.d/CentOS-Base.repo
```

Add enabled to 0 in all the repository and if enabled is set to 1 change it  0

```bash
enabled=0
```
Save changes and exit from the editor

Now we need to ask yum to clean all the meta data and start the update all

```bash
$yum clean all
$yum update all
```

You will see that the files are read from the mount point in the log that is printed.You can now install any package by using the following command
$yum install pacakgename

We can make the mounting to the filesystem automatic  by adding the following to `/etc/fstab`

```bash
$vim /etc/fstab
/home/centos/centos.iso     /mnt/centos   iso9660 loop,auto,ro  0 0 
```

Save the changes and exit and the next time you boot up , you need not do the above configuration once again.
