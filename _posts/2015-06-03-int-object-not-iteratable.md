---
layout: post
title: " TypeError: 'int' object is not iterable"
published: true
comments: true
tags : python static-typing dynamic-typing
---
Recently i started to learn python. For a long time, i wanted to learn a scripting language. i thought i would learn python by solving [spoj](http://www.spoj.com/) problems.spoj is a site that has a set of programming problems and one can write code in any language to solve it.On solving the problem ,the solution is submitted to a spoj server and is evaluated for its correctness by running it across a set of test cases.Its good site to get used to solving programming problems.

I started writing some of the basic problems to get an idea of working with python.One thing i stumble often is  dynamic typed feature of python, it could lead to subtle errors while programming. I not blaming the language but its something that people from static typed language find it a bit hard at first.In static typed language, the variables are checked at compile time their types and hence one can avoid common errors of type changes easily.

The problem i am faced was because of dynamic typing and it startled me for a second. It was fun and i wanted to kick myself when i found it.Here it goes

I was trying to solve the problem of finding the number of trailing zeros for factorial.This was a solution, to one of the spoj problem.In order to get the input, which will contain all the numbers to find the solution i ended up writing a code as below 
 
{% highlight python linenos=table %}
#finding the number of zeros in factorial
# the trick is to find the number of 5 and its factors 
AllInputs = []

#get the number of inputs
numberOfInputs = input()

# get all the inputs
for i in range(numberOfInputs) :
	AllInputs = input()

for number in AllInputs :
	# logic to handle the inputs
{% endhighlight %}


Now on executing the program on input i ended up getting the following error

```
Traceback (most recent call last):
  File "test4.py", line 13, in <module>
    for number in AllInputs :
TypeError: 'int' object is not iterable
```
In the above code line 13 is accessing all the input values. I was wondering that i have already declared `AllInputs` as in list in line 1. This got me confusing and on closer look at code the culprit was the below line 

{% highlight python  %}
# get all the inputs
for i in range(numberOfInputs) :
        AllInputs = input()
{% endhighlight %}


In the assignment section i have mistakenly converted the array into a integer and python being a dynamically typed language did not warn me of the change in type :)

The fix was of course a simple one 

{% highlight python  %}
for i in range(numberOfInputs) :
	AllInputs.append (input())
{% endhighlight %}

The above problem tought me more about static typed language and dynamic typed language more than i could have ever understood or learnt from book.
