---
layout: post
title: "++ operator in python"
published: true
comments: true
tags: python ++operator unary 
---

If you are one of the guys who are used to programming in c/c++ , one commonly ++ operator in loops.Recently i have been taking a look at python because its one of the most common scripting language around with loads of interesting library/bindings. 

So i started writing the usual code for reading a file and counting the number of lines as below

{% highlight python linenos=table %}
fh = open("words.txt")
count = 0
for line in fh:
    ++count
    
print "count %d" %count

{% endhighlight %}

And running the program i got the following output

```
	count 0
```

I was suprised to see the output as there is nothing that can go wrong.OH!! God how naive of me.On turing to my most trused source, Google ofcourse i found the reason as follows [1]

Simple increment and decrement aren't needed as much as in other languages. You don't write things like ``` for(int i = 0; i < 10; ++i) ``` in Python very often; instead you do things like ```for i in range(0, 10)```.
 
This make sense because we hard make loops as above in python.

Now that we undestand the reason for such a decision,lets explore a little more. 

Why did the interpretor did not throw any error ?

The reason is simple because the + operator is evaluated as unary operator.From the doc of the python 

````
The unary + (plus) operator yields its numeric argument unchanged.
```
So in the above case the python interprator evalutes to two unary + evaluation and hence the value of the count remains unchanged.

If we try to convert the above code into post increment operator (i.e. count++) the interpretor throws an error as below.

{% highlight python %}
>>> count = 1
>>> count++
  File "<stdin>", line 1
    count++
          ^
SyntaxError: invalid syntax
{% endhighlight %}

The above error is thrown as unary operator(+) does not have any operand.


[1] [stack overflow answer](http://stackoverflow.com/questions/3654830/why-are-there-no-and-operators-in-python) 
