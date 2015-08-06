---
layout: post
title: "Tracking linux next"
published: true
comments: true
---
I have been trying to get the latest version of the linux-next. As you guys already know all the latest changes of all the subsystem trees are pulled regularly into the linux-next tree and this helps in detecting merge conflicts earlier.I had already mentioned how I would clone the linux next tree in [here]({% post_url 2011-12-29-cloning-linux-next-tree-i-wanted-to-do %}).

Now i was trying to pull the latest change into the linux-tree using the following command

```
pradheep@PradheepVM:~/linux-next/linux-next$ git pull

```
However i would end up with loads of merge conflicts when pulling linux-next tree similar to the below

```	
CONFLICT (content): Merge conflict in arch/alpha/include/uapi/asm/mman.h
Auto-merging arch/Kconfig
Auto-merging Next/merge.log
CONFLICT (add/add): Merge conflict in Next/merge.log
Auto-merging Next/Trees
CONFLICT (add/add): Merge conflict in Next/Trees
Auto-merging Next/SHA1s
CONFLICT (add/add): Merge conflict in Next/SHA1s
Auto-merging MAINTAINERS
Auto-merging Documentation/ioctl/ioctl-number.txt
Auto-merging Documentation/devicetree/bindings/drm/msm/hdmi.txt
CONFLICT (content): Merge conflict in Documentation/devicetree/bindings/drm/msm/hdmi.txt
Automatic merge failed; fix conflicts and then commit the result.
```

This is because the linux-next tree contains the merges with conflicts and hence when we pull the linux-next tree we get all the merge conflicts.

So how do we get the latest changes without trying to merge all the conflicts?
One has to do the following to update linux-next 

```
$ git checkout master   
...
$ git remote update 
```

Then we can list out the recent tags 

```
pradheep@PradheepVM:~/linux-next/linux-next$ git tag -l next-* | tail
next-20150721
next-20150722
next-20150723
next-20150724
next-20150727
next-20150728
next-20150729
next-20150731
next-20150804
next-20150805
```

Then create the lastest linux-next branch by using the following command

```
pradheep@PradheepVM:~/linux-next/linux-next$ git checkout -b mynewbranch next-20150805
Switched to a new branch 'mynewbranch'
```

This branch has all the related changes of the merges that was performed recently.

I found most of these information from the kernel [doc](https://www.kernel.org/doc/man-pages/linux-next.html)