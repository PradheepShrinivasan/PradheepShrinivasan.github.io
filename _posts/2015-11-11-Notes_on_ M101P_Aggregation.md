---
layout: post
title: "Mongodb M101P lecture 5(Notes) - Aggregation"
published: true
comments: true
tags : mongodb python
---

1. Mongodb provides a framework for aggregation similar to that of SQL database as below 

```
    The following query finds the sum of all items by particular seller.

    db.collection.aggregate([ {
        "$group" : {
            "_id":{maker:"$"merchent"},
            {"$numofItems":{"$sum":1}}
        }
    }
    ])
```

As you can see above the aggregation framework takes a array as input to function.Aggregation framework in mongodb can be used like a pipeline with a set of commands that can be passed as pipes (similar to pipes in unix). The following are aggregation pipeline elements supported in mongodb

```
        projection  - $project
        match       - $match
        group       - $group    - performs aggregation
        sort        - $sort
        skip        - $skip
        limit       - $limit 
        unwind      - $unwind   - unwinds array into unique elements
        redact      - $redact   - permission of documents viewing
        geonear     - $geonear  -  for geoloaction
```

2. The following commands can be used with $group commands 

```
        $sum    - performs sum or counting
        $avg    - Average
        $min    - Minimum value
        $max    - maximum value
        $push   - push a value to array
        $addtoset   - add a value to set if not already present
```

The below 2 commands must be used with sort to make any sense

```
        $first - returns the first element
        $last  - returns the last element
```
3.An example of aggregate pipeline with a project, sort and match sections.

```
        db.zips.aggregate([
            {$project : {city:{$substr:["$city",0,1]}}},
            {$sort : {city : 1}}, 
            {$match: {city: {$not: {$type: 2}}}},
        ]);
```
4. The aggregation pipeline is better alternative to most of map reduce framework supported in mongodb.