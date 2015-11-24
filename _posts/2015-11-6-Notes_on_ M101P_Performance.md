---
layout: post
title: "Mongodb M101P lecture 4(Notes) - Indexing for Performance"
published: true
comments: true
tags : mongodb python
---

The following are the notes on mongodb Lecture 4 

1. Pluggable storage engines introduced into mongodb from 3.x version.

2. 2 storage engines supported - MMAPV1 and WiredTiger

3. MMAPV1 is the default , uses mmap command , os handles all the pages and other stuff . 

4. WhiteTiger - newer one better handling of pages and perform better for most workloads.

5. Index as in any database can be used in mongodb to improve performance of query.Of course there is overhead in space and write operation as there is a index that needs to be updated every time.

```

        db.collection.createIndex({scores:1}, {unique:true}) // creates index in ascending order
```

    a. unique is optional and is used to guarantee the uniqueness and throw an error when we try to insert something wrong.

    b. Sparse index can be used to Index for items that may not exits in all documents.However sparse index will not be used in sort query.

    c. Indexing can be done in foreground and background.The default <b>createIndex</b> indexes on foreground.During foreground all read and write requests are blocked for the database. Background Index is at background and is slower. One way to do is to use the background creation in secondary .


6.Possible to create index on sub elements using the dot(.) notation as below

        db.collection.createIndex({scores.score:1}) // creates index in ascending order

7.To list all the available index in a database we can use 

        db.collection.getIndexes()
    

8.Multikey index is an index on array. you cannot create multi key index on 2 or more arrays . Index are created as tuple in output.

        db.collection.createIndex({scores:1, subject:1})

In the above example either scores or subject can be an array but not both.

9.Explain command can be used to analyze how a command was executed in a find command.

        db.collection.explain().find({score.scores: {$gt: 90}})

In the output look for the "winning plan" as it contains all information about the query execution

One of the options of the explain command is the <b>"executionStats"</b> which provide more information regarding the execution time and so on.

Read all the stats from the bottom for easier understanding.

"All plan execution" option can be run to get details about how the query optimizer runs periodically to figure out which query plan to figure out which one is the best way to run the query.


10. Covered query is one that uses only the index to satisfy and need not go and look at the document. Make sure that the output is only the one the index you are using and if not it will become non covered query.

11. Index perform better when they  are in memory and to measure the size of the index use the stats method of collection


``` 
    db.students.stats()
    {
        "ns" : "school.students",
        "count" : 200,
        "size" : 48000,
        "avgObjSize" : 240,
        "numExtents" : 3,
        "storageSize" : 172032,
        "lastExtentSize" : 131072,
        "paddingFactor" : 1,
        "userFlags" : 1,
        "capped" : false,
        "nindexes" : 1,
        "totalIndexSize" : 8176, <------Index size
        "indexSizes" : {
            "_id_" : 8176
        },
        "ok" : 1
    }
```

12.Mongodb automatically logs all queries longer than 100ms to stdout of the mongod logs.
One can enable profiling level using  

    db.setProfilingLevel(1,value).
    0 - no Profiling, 1- Slower queries > value and 2 All queries.

a. db.getProfilingLevel to get the profiling level. 

b. db.profile.find() will give the output of the profiling. Note its a constant buffer so it will overwrite older data.

13.mongotop and mongostat are used similar to top and stat for mongodb.
