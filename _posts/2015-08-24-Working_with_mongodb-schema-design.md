---
layout: post
title: " Working with mongodb : Schema  design"
published: false
comments: true
tags :  mongodb database BDB
---
I have been working in oracle BDB a NOSQL database for some time.It was time that i get  to work on some other NOSQL database , so started getting my hands dirty with document datbase. One of the primary difference between Mongodb and Oracle BDB is that mongodb is s a document database whereas BDB is a key value store.What it boils down is that in mongodb there is no need for serailization and in BDB you need to take care of serialization.

To learn mongodb i thought i would try to think of a simple use case in mongodb, and decided i would try to think of how one would implement a mail application. So the first thing in any database design is to design the entity relationship. 

 The entity relationship of a simple mail client is as below 

![Entity relationship diagram]( /images/simple_mail_ER.jpeg) 

The next step is to convert this entity and relationship into set of tables, columns and row or in mongodb terms set of collections and documents.

Lets first understand the problem in bit more, what we have to design is a mailbox so obviously we would want it to be a seperate database.This database will contain all the other relevent collections for implementing the mailbox.