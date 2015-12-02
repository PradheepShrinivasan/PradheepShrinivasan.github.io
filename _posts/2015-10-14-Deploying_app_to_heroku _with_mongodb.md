---
layout: post
title: "Deploying app to heroku with mongodb - Part 2"
published: true
comments: true
categories: python mongodb 
---

Now that we have the application up and running in heroku its not time to hook up the backend i.e the database of mongodb to the blog.The following are the steps i performed to connect to the mongolab

1. create an account in mongolab [site](mongolab.com)

2.  Create a new database by clicking on the newdatabase button. 

3. Select amazon as the provider and single node as we are testing free service as of now.

4. Use a unique name for the database.

5. The database must now be displayed in the list of available database as below.Click on the database and enter the menu option

6. Create a new user name and password as this is this will be used to connect to database.

7. The usual way to connect to a mongodb server is as below in python using pymongo

```
connection_string = "mongodb://localhost"
connection = pymongo.MongoClient(connection_string)
database = connection.newblog
```

we change the connection string to include the name of the database which we got from step 5

```
connection_string = "mongodb://<dbuser>:<dbpassword>@ds059634.mongolab.com:59634/"
connection = pymongo.MongoClient(connection_string)
database = connection.newblog
```

8.Test locally with the actual value of dbuser and dbpassword locally and make sure that the application is working as expected.Now storing the password and login in code is a terrible idea, so we are going to use the heroku enviornment variable option to pass the login name and password.

9.Login to heroku, then go to your app->setting and then select revelConfig and add 2 enviornment variables like below to add your user login credentials.





10. Change the code to get the user name and login also from enviornment variable something like below 

```
db_user = os.environ['DBUSER']
db_password = os.environ['DBPASSWORD']
connection_string = "mongodb://"+ db_user + ":" + db_password +"@ds059634.mongolab.com:59634/newblog"
 connection = pymongo.MongoClient(connection_string)
database = connection.newblog
```

I have hosted the blog [here](https://infinite-shore-9738.herokuapp.com) and you can signup for your own account [here](https://infinite-shore-9738.herokuapp.com/signup) and the entire code can be found [here](https://github.com/PradheepShrinivasan/MongodbBlog)

P.S

Note the blog code is not mine and was provided as part of the mongodb course and i just used it as a blue print for hosting playing and running it on heroku and mongolab.
