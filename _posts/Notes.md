
This contains all my notes regarding various stuff


-- 26 March 2015 --

Kernel playing around 

dmesg -c would clear the dmesg buffer so that we get only the latest updates

pr_debug,pr_err, pr_info must be used instead of the printk set of functions.

In order to use the pr_debug to be printed at the dmesg add the following to the kernel makefile 

FLAGS_[filename].o := -DDEBUG.eg CFLAGS_kobject_uevent.o := -DDEBUG

The above just defines the variable DEBUG. One can just #define DEBUG at the top of the cpp file.

Always run the checkpatch.pl before submitting any patch.The below line gets you to run out of tree checkpatch.pl
~/linux-kernel/linux/scripts/checkpatch.pl -terse -file -no-tree



