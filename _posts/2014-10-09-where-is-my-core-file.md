---
layout: post
title: "Where is my core file?"
published: true
comments: true
---
Recently i was testing [jemalloc](http://www.canonware.com/jemalloc/) for detecting memory corruptions of the program.This was tested by writing programs that intentionally crash and looking for jemalloc to abort and raise errors.

On my ubuntu machine, i found that program was not generating core. I was confused. So the first thing to check is to ulimit. 

```bash
pradheep@PradheepVM:~/jemalloc$ ulimit
unlimited 
```

The default output says unlimited. So looked at the man page of bash(1) this defaults to -f which is "The maximum size of files written by the shell and its children".

So looking at ulimit -a

```bash
 pradheep@PradheepVM:/$ ulimit -a  
 core file size     (blocks, -c) 0 
 data seg size      (kbytes, -d) unlimited  
 scheduling priority       (-e) 0  
```

Now we need to set the core value to unlimited.This can be done using
 ulimit -c unlimited

The output should be

```bash 
pradheep@PradheepVM:/$ ulimit -a  
 core file size     (blocks, -c) unlimited  
 data seg size      (kbytes, -d) unlimited  
 scheduling priority       (-e) 0  
 file size        (blocks, -f) unlimited  
 pending signals         (-i) 7402  
 max locked memory    (kbytes, -l) 64  
 max memory size     (kbytes, -m) unlimited  
 open files           (-n) 1024  
 pipe size      (512 bytes, -p) 8  
```

This is set only on the shell you have run, to set the value permentenly on all the new shells that are spawned you should edit `$HOME/.bashrc` to add `ulimit -c unlimited`

In all your new shell invocation you will be provided with unlimited core size dump.

More info on setting core dump can be found [here](http://www.akadia.com/services/ora_enable_core.html).

Additional info on core file dumps can be found in the following links

 1. [http://stackoverflow.com/questions/7732983/core-dump-file-is-not-generated] (http://stackoverflow.com/questions/7732983/core-dump-file-is-not-generated).
 2. [http://askubuntu.com/questions/295765/upstart-and-core-files] (http://askubuntu.com/questions/295765/upstart-and-core-files).
