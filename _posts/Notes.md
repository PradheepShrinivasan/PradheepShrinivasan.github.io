
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

-- 13 August 2015 --

Kernel playing around

to compile a kernel module alone run 

sudo make drivers/macintosh/mac_hid.ko

then you can insmod it using 

sudo insmod drivers/macintosh/mac_hid.ko 

However if you are getting an error message first look at the dmesg 
mac_hid: no symbol version for module_layout

if you are getting this error its one of the following
  1. You are having a mismatch from the kernel tree and the kernel thats running
  2. The kernel modules.order file is not generated. To do that 
  		 /home/pradheep/linux-kernel/linux make 

  		this will build all the kernel modules and module.order and hence will we can now create and insert our modules at will.

If you are having problem uninstalling any software in windows always use microsoft fix it. It works 99% of the time.

-- 20 Aug 2015 --

Following are some of the basic of HTTP and REST


1xx indicates an informational message only
2xx indicates success of some kind
3xx redirects the client to another URL
4xx indicates an error on the client's part
5xx indicates an error on the server's part

Operation       HTTP/RESE operation    common Error
Create           POST                   404(not found), 409(conflict) resource already present.
Read             GET					404 (not found), 400 bad request.
Update           PUT					404 (not found) , 204 (no content) 
Delete           Delete					404 not found,204 (no content)



The most common status codes are:
200 OK
204 No Content usually on PUT
404 Not Found
409 conflict if resource is already present.
301 Moved Permanently 
302 Moved Temporarily 
303 See Other (HTTP 1.1 only)
500 Server Error

References

1REST basics)[http://www.restapitutorial.com/lessons/httpmethods.html] for 

