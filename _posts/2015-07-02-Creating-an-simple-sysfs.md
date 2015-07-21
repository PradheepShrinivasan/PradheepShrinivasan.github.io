---
layout: post
title: "Creating a simple sysfs module in linux kernel"
published: true
comments: true
tags : linux kernel sysfs
---
Sysfs is the commonly used method to export system information from the kernel space to the user space for specific devices.THe procfs is used to export the process specific information and the debugfs is used to used for exporting the debug information by the developer.

 The sysfs is tied to the device driver model of the kernel.To more about device driver model see [1]

So lets start building a simple sysfs that we will export to the user space.

the following is the basic structure of any simple device driver

{% highlight c linenos=table %}
#include <linux/module.h>
#include <linux/init.h>

static int __init mymodule_init (void)
{
        pr_debug("Module initialized successfully \n");
        return 0;
}

static void __exit mymodule_exit (void)
{
        pr_debug ("Module un initialized successfully \n");
}

module_init(mymodule_init);
module_exit(mymodule_exit);
MODULE_LICENSE("GPL");
MODULE_AUTHOR("pradheepsh");

{% endhighlight %}

At the heart of the sysfs model is the Kobject.Kobject is the glue that binds the sysfs and the kernel.

In this example lets write a simple code that will add a directory and a file to the sysfs.The file will allow reading and writing data to the sysfs of a fixed size of 4096 bytes.

The first step is to create a directory into the /sys/kernel directory. Thiscan be done by adding the following line in the mymodule_init function

{% highlight c linenos=table %}
example_kobject = kobject_create_and_add("kobject_example",
                                                 kernel_kobj);
if(!example_kobject)
    return -ENOMEM;

{% endhighlight %}

Dont forget to remove the kobject from the sysfs ,else the entry cannot be deleted untill the system is rebooted.

{% highlight c linenos=table %}

 kobject_put(example_kobject);

{% endhighlight %}

Now compile the kernel module and load it using insmod and you will see the following entry

```
pradheep@PradheepVM:$ ls /sys/kernel/
.. kobject_example  ..

```
The second step is creating the actual file attribute.There are loads of  helper function that can be used to create the kobject attributes.They can be found in header file sysfs.h

To create a single file attribute we are going to use 'sysfs_create_file'.One can use another function ' sysfs_create_group ' to create a group of attributes.

{% highlight c linenos=table %}

error = sysfs_create_file(example_kobject, &foo_attribute.attr);
if (error) {
 	pr_debug("failed to create the foo file in /sys/kernel/kobject_example \n");
} 

{% endhighlight %}

in the above code the second argument foo_attribute.attr helps in registering the callback methods that is needed to be used when the particular file example_kobject is accessed. It can be defined using the macro __ATTR as below

```
	static struct kobj_attribute foo_attribute =__ATTR(foo, 0660, foo_show,  foo_store);

```


what the above code essentially does is as set the corresponding attribute name, permission and the callback methods to be used when the file foo is accessed.

{% highlight c linenos=table %}

static struct device_attribute foo_attrute = {
	.attr = {
		.name = "foo",
		.mode = S_IWUSR | S_IRUGO,
	},
	.show = foo_show,
	.store = foo_store,
};

{% endhighlight %}


The third step is to use a define what needs to be done when  the particular file/attribute  foo is accessed. In this case we expect the user would allow user to write an integer value and when read we  give back the integer value that was written earlier.

{% highlight c  %}

static ssize_t foo_show(struct kobject *kobj, struct kobj_attribute *attr,
                      char *buf)
{
        return sprintf(buf, "%d\n", foo);
}

static ssize_t foo_store(struct kobject *kobj, struct kobj_attribute *attr,
                      char *buf, size_t count)
{
        sscanf(buf, "%du", &foo);
        return count;
}

{% endhighlight %}

The full implementation is as follows 

{% highlight c  %}

#include <linux/module.h>
#include <linux/printk.h>
#include <linux/kobject.h>
#include <linux/sysfs.h>
#include <linux/init.h>
#include <linux/fs.h>
#include <linux/string.h>

static struct kobject *example_kobject;
static int foo;

static ssize_t foo_show(struct kobject *kobj, struct kobj_attribute *attr,
                      char *buf)
{
        return sprintf(buf, "%d\n", foo);
}

static ssize_t foo_store(struct kobject *kobj, struct kobj_attribute *attr,
                      char *buf, size_t count)
{
        sscanf(buf, "%du", &foo);
        return count;
}


static struct kobj_attribute foo_attribute =__ATTR(foo, 0660, foo_show,
                                                   foo_store);

static int __init mymodule_init (void)
{
        int error = 0;

        pr_debug("Module initialized successfully \n");

        example_kobject = kobject_create_and_add("kobject_example",
                                                 kernel_kobj);
        if(!example_kobject)
                return -ENOMEM;

        error = sysfs_create_file(example_kobject, &foo_attribute.attr);
        if (error) {
                pr_debug("failed to create the foo file in /sys/kernel/kobject_example \n");
        }

        return error;
}

static void __exit mymodule_exit (void)
{
        pr_debug ("Module un initialized successfully \n");
        kobject_put(example_kobject);
}
{% endhighlight %}

We can change the above code to store any value (upto 4096 bytes) by just changing the logic as follows.



###Links

1. [Device driver model of Linux](http://lwn.net/Articles/31185/).
2. [Simple Makefile for Linux kernel] ()
3. [Good overview of Kobject] (http://lwn.net/Articles/51437/)
