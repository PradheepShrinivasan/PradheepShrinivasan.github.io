---
layout: post
title: "Cloning linux next tree"
published: true
comments: true
---
I wanted to submit some patches to the kernel nothing big but some of the basic checkpatch.pl fixes. This would require me to clone the linux-next tree. 

First,some background on the linux-next tree.linux-next tree is the place where changes from different trees (mm tree, drivers tree,  etc) of the kernel are merged everyday. The merges are then built everyday with allmodconfig and tested by trying to boot up the kernel .  This process helps to find the conflicts between different kernel trees early and to fix it.The status of the builds of the everyday linux-next branch can be found in this link http://kisskb.ellerman.id.au/kisskb/branch/9

So in order to create a copy of the tree to my PC i have to clone the tree using git. I needed the path of the linux-next tree in the git.

As i searched through the different sites for the linux-next branch path to clone, all the sites gave me the address

```bash
git://git.kernel.org/pub/scm/linux/kernel/git/sfr/linux-next.git 
```
But when i tried to do a clone of the git using the command

```bash 
git clone git://git.kernel.org/pub/scm/linux/kernel/git/sfr/linux-next.git
```

I got the following error

```bash
git clone  git://git.kernel.org/pub/scm/linux/kernel/git/sfr/linux-next.git 
Initialized empty Git repository in /home/pradheep/temp/linux-next/.git/
fatal: The remote end hung up unexpected
````

I wanted to make sure that the path is correct so i trimmed the web address to the git/kenel.org and then i found that the linux-next tree has been moved to the new path (It must have happened after the kernel.org got hacked)

So the new path of the linux-next tree is

```bash
git://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git
```

and the http address is

```bash
http://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git
```

so at the end i ended up cloning using the following command

``` bash
git clone git://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git
```

and the entire linux-next was cloned to my PC :-)

Now off to doing some trivial patches so i can get my name in the linux kernel.
