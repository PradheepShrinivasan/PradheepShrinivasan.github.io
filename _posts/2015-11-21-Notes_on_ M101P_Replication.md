---
layout: post
title: "Mongodb M101P lecture 6(Notes) - Replication"
published: true
comments: true
tags : mongodb python
---
## Replica Set
1. write concern dictates if the mongodb client must wait for the mongodb server acknowledgment before it starts executing the next instruction.
2. Journal is analogus to the transcation log in sql database that needs to be persisted for durability.
3. The default configuration of the mongodb is w=1, j= false, which means that mongodb client waits for acknowlegnment from the server that the transcation is successful before continuing processing of the next operation.
4. Since there is no commit to the disk in this scenario there is a small window in between the operation committing to disk and server failure of loss of data.
5. For strong consistency and for no loss of data the configuration that is suggested is w=1,j=true. However this has a slow performance compared to w=1,j=false as disk access is slow.
6. The configuration (old one) of setting w=0 is not recommended as the client does not wait for the operation to succeed.This is wrong because when the write fails due to any errors the application will not be able to detect and handle it.
7. There is a network connection between the mongodb client and mongodb server which can also be the cause of errors due to packet loss or any other network issues. 
8. Solution to the above problem is 2 things. In case of insert we need not worry about it as the retry from the client side would cause a duplicate key error and handling this is simple.
9. The problem occurs in cases where update is used on part of the document especially with *Inc* and *dec* operands. In this case its harder to handle as we are not sure what the new value must be.
10. In case of replication, mongodb supports 3 types of node
    a. Regular node - It acts as secondary node, that can act as a primary node in case of primary failure.
    b. Arbirter - It just participate in election,  casts votes and does not store data.Advantage is that is that its not resource hungry and used for testing.
    c. Delayed/Regular - It is mostly used as a backup node, usually hours behind.Priority is set as zero so that it cannot become primary.
    d.Hidden node - Used for analytics and has voting rights. Priority set to zero as it does not want to become primary.
11. For majority to happen, there should be at least 3 nodes that must cast a vote to select one as leader.In case of node failure, the nodes the other nodes hold an election and the node that contains the maximum oplog(transaction log) will become a leader.
12. By default all reads and writes are made from the primary node, so the data is strongly consistent.
13. We can configure data to be read from secondary, hence it the reads from the secondary may not be same what is at the primary.
14. The mongodb replication are replicated asynchronously from the primary to all the secondaries.
15. Reading from the secondary,means that the load is lesser than primary and it means we can balance some load.Its possible to make all data read and write of a particular database to primary and reading from load balancing  to secondary  for other database.
16. Replica sets are created using the following commands in mongo shell
```  
    mongod --replSet <replicaName>
```
all nodes of replication set must have the same relicaName.

17. ** rs.initiate() **, ** rs.status() ** command can be used to initiate a replica set  and show the the status of a replica set respectively.
18. ** rs.slaveOK() ** command must be used to allow reading from secondary node.
19.During election reads and writes are not allowed till election happens and a new primary is elected.
20.The oplog which records all operations and which is used for replication can be seen using
```
    use local
    db.rs.oplog.find()
```
21. The oplog is a capped collection which means if the network is slow or the database operations is replicated slowly because the secondary is a slower box, then there could be loss of data.
22. In case of secondary misses the oplog, then it restart the reading of entire database from primary and this is going to be a slow process.
23. The mongodb drivers are intelligent and once you given a valid node, it will detect all other nodes in replica set and failover to the new primary if needed automatically.
24. To handle the case of fail over during an insert is performed, one needs to handle 2 errors that are thrown by the mongodb client library

```
pymongo.errors.AutoReconnect - Thrown when a client reconnects to a new primary

```

To handle this case , the library must have a reconnect mechanism to make sure that we retry for n times. When we retry to insert, then we must insert get a duplicate key error since the network may have disconnected before the response to client.

```
for i in range(0,500):
    for retry in range (3):
        try:
            things.insert_one({'_id':i})
            print "Inserted Document: " + str(i)
            time.sleep(.1)
            break
        except pymongo.errors.AutoReconnect as e:
            print "Exception ",type(e), e
            print "Retrying.."
            time.sleep(5)
        except pymongo.errors.DuplicateKeyError as e:
            print "duplicate..but it worked"
            break
```

Note the above is not perfect as it might not work when the election takes more than 14 seconds.

25.In case of read, we should keep on retrying read and we need not worry about duplicate key error as we are just reading and not writing.

```
for i in range(0,500):
    for retry in range (3):
        try:
            things.find_one({'_id':i})
            print "read document: " + str(i)
            time.sleep(.1)
            break
        except pymongo.errors.AutoReconnect as e:
            print "Exception ",type(e), e
            print "Retrying.."
            time.sleep(5)
```

26.update is kind of tricky in case of non idopotent cases like $push , $inc $dec since when we retry the values it might have been  incremented already.

```
    for i in xrange(500):
        time.sleep(.1)  # Don't want this to go too fast.
        for retry in xrange(3):
            try:  # to read the doc up to 3 times.
                votes = things.find_one({'_id': i})["votes"] + 1
                break
            except pymongo.errors.AutoReconnect as e:  # failover!
                print ("Exception reading doc with _id = {_id}. " +
                       "{te}: {e}").format(_id=i, te=type(e), e=e)
                print "Retrying..."
                time.sleep(5)
        else: 
            print "Unable to read from the database. Aborting."
            exit()
        for retry in xrange(3):
            try:  # to read the doc up to 3 times.
                things.update_one({'_id': i}, {'$set': {'votes': votes}})
                print "Updated Document with _id = {_id}".format(_id=i)
                break
            except pymongo.errors.AutoReconnect as e:  # failover!
                print ("Exception writing doc with _id = {_id}. " +
                       "{te}: {e}").format(_id=i, te=type(e), e=e)
                print "Retrying..."
                time.sleep(5)
        else:  # If no break, we failed to write the document. Abort.
            print ("We have failed to increment the 'votes' field for " +
                   "the document with _id = {_id} to {votes}. Exiting."
                  ).format(_id=i, votes=votes)
            exit()

```

This leads to 2 ways to handle these cases 
    
    1. If the value is very important that the data to be correct then we must convert the insert from inc to a read operation and then increment the value and write the new value to the database.
    Note: In this case, if you application has multiple threads, then one thread you could have thread races and result in data loss between the increments in threads.Locks could be a solution in application but it can be a problem if your application is distributed. 
    2. If its okay for value to be a little bit more or less from application perspective then we can either decide to retry in the AutoReconnect error scenario or we could ignore the error altogether and retry(not a good one since we might start losing values)

27. The write concern for a mongodb replica can be set at database level, collection level or connection level, the values can be set to an ** integer ** (denotes the number of nodes to write to before acknowledging) or ** majority **
28. Specifying ** majority ** is preferred as it prevents loss of data due to secondaries not in sync. Of course there is a increase in response time waiting for acknowlegement for the majority of nodes. 
29. *** Wtimeout *** can be used to specify the maximum time the server must wait for majority of time the client must wait for acknowledgement.
30. Read preference can be used to specify which node that the client must read from. The following are the options supported.
    1.Primary - Default - read only from primary node.
    2.Primary Preffered - Reads from secondary if primary is not available.
    3.Secondary - Reads from secondary only
    4.SecondaryPreffered -Reads from primary if no secondary is available.
    5.Nearest - The node thats nearest to the client. 

## sharding

1. We shard to achieve horizontal scalability i.e. we make sure that there not only one server is able to handle all load but its distributed across servers
2. Each shard is a replica set itself which means each replica set is highly available.
3. In order to ensure unique key across replica each unique index must contain shard key.
4. Shard keys must be used in all queries else the mongos will send the queries to all shared sets and provide the aggregated result.
5. On sharding, the client libraries(drivers) connect to the mongos which is responsible for figuring which shred that the operation must be sent to.
6. Mongos are lightweight and usually deployed in the same application server and its possible to have multiple mongos running for high availability.

