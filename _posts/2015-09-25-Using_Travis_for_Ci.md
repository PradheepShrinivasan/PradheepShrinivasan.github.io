---
layout: post
title: "Addding Travis for CI of redis"
published: true
comments: true
tags : Continuous Integration, Travis
---
 For those people who don't know what CI is, continue reading this paragraph . The general way most development team used to work earlier was , team members develop their code in isolation in separate branches and after everyone is done , the integration team would pull all the changes and integrate everything.This is a hard process, as there are merge conflicts due to overlapping changes,memory leaks detection becomes hard as loads of changes, bug isolation is hard as no one is sure which code introduced it and so on. In general, significant dev team was spent on integration and testing.However,CI makes all these process simple by automating tests with unit tests, integration tests so that bugs are detected early and in isolation.More on CI [here](https://www.thoughtworks.com/continuous-integration)

But running Continuous integration, for developers of open source was bit difficult.One has to setup a Jenkins server locally and run it for every merge. It would be better, if we are able to run CI on each pull requests that others send. The solution is [Travis](https://travis-ci.org/) and it works with all github projects out of the box.

 I was excited to see Travis support by default in github and free for all projects that are open sourced. I decided to give it a shot. Though i could have done a small code to test its no fun but nothing beats automating a full project.So looking around i found redis has not updated itself with travis support. Redis in fact runs all the tests in http://ci.redis.io/ .I felt running these tests as part of the each pull requests to redis would be awesome. So here is how i got the basic travis setup to work. 

 Adding travis to any project in github is easy. Just add `.travis.yml`  in the top directory of your repo and you are done.

 So the first step in .travis.yml file is to define the language and the compiler you want the project to compile. 
 Natively travis support gcc and clang. So i added the below lines .travis.yml

{% highlight bash %}

language: c 

compiler:
  - clang
  - gcc

{% endhighlight %}

Travis provides travis-lint that can check for errors in travis.yml file . You can install travis-lint using the following command

```
gem install travis-lint

```

And then you can check .yml file using the command 
```
travis-lint [path to your .travis.yml]
```

Adding the above changes to the repo and pushing it to the github was all the basic configuration that was needed.Then i had to add my repo to travis.The steps to add a repo into travis is very straight forward and step by step instructions can be found in this [site](http://code.tutsplus.com/tutorials/travis-ci-what-why-how--net-34771)

 The travis.yml file has different sections similar to rpm packaging to control the build, test and deployment. To build and test your application the script sections must contain your application specific build and test specification.So i added the below lines to .travis.yml

{% highlight bash %}

script:
  - make
  - make test

{% endhighlight %}

Pushing the above to github automatically triggers travis to run the test.Result of one such build is [here](https://travis-ci.org/PradheepShrinivasan/redis/jobs/81861415).Travis was reporting error and looking at the logs it was clear that the setup needed tcl and it was not available causing the test to fail.So the next step was to add tcl package.

Travis tests all the applications on top of Ubuntu and installs some of language specfic packages.But not all packages are installed and one can install any package by using the apt-get command.Since the packages must be added before the make test, i added install tcl 8.5 in before_script section to .travis.yml

{% highlight bash %}

 before_script:
 - sudo apt-get install -qq tcl8.5

{% endhighlight %}

The before_script section, as you might have guessed, is executed before the script section.This installs tcl and since there was no dependency problem anymore tests ran [successfully](https://travis-ci.org/PradheepShrinivasan/redis/builds/81863361).If you look at the link you will see tha the tests are run both in clang and gcc. This is because we had asked for clang and gcc as compilers in the compiler section.

The next step was to make the code run in both in OSX and linux.Currently travis only supports 64 bit OS Linux and support for OSX is in beta. There is plans to add support for FreeBSD.Back to what I was doing, to add OS support i added the following lines

{% highlight bash %}

os:
   - linux
   - osx

{% endhighlight %}

But there was one more thing, i need to install tcl for OSX. So i ended up modifying the before_script with the following lines 
{% highlight bash %}

 before_script:
    - if [ "$TRAVIS_OS_NAME" == "linux" ]; then  sudo apt-get install -qq tcl8.5; fi
    - if [ "$TRAVIS_OS_NAME" == "osx" ] ; then brew update; brew tap homebrew/dupes; brew install tcl-tk;fi

{% endhighlight %}

Brew is the unofficial package manager for OSX.

Cautious readers would notice that i added brew tap homebrew/dupes.This is because OSX comes preloaded with tcl and in order to make sure that users dont override the default package, brew uses the dupes to make sure that people who are installing knows what they are doing.

Finally i sent a pull request to redis and i am currently not sure when it will get integrated.The entire pull request is [here](https://github.com/antirez/redis/pull/2785).The changes are the basic and if the pull request is accepted i am planning to add matrix build and also move the changes to new test infrastructure.

The entire .travis.yml that was pulled

{% highlight bash %}

language: c 
os:
   - linux
   - osx

compiler:
  - clang
  - gcc

before_script:
    - if [ "$TRAVIS_OS_NAME" == "linux" ]; then  sudo apt-get install -qq tcl8.5; fi
    - if [ "$TRAVIS_OS_NAME" == "osx" ] ; then brew update; brew tap homebrew/dupes; brew install tcl-tk;fi

script:
  - make
  - make test

{% endhighlight %}

Useful links

1. [Travis ci](https://travis-ci.org/)
2. [Travis Documentation](http://docs.travis-ci.com/)
