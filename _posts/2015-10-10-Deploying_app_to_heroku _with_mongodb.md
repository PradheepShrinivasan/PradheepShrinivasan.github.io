---
layout: post
title: "Deploying app to heroku with mongodb - Part 1"
published: true
comments: true
categories: python mongodb 
---
As part of the mongodb course, one of the assignment required to complete the backend of the blog.I wanted to get my hands a bit more dirty and so i decided to deploy the entire application to heroku with mongolab as the backend.The following are the steps that i followed to make the site work in heroku and mongolab.

## Deploying the site to Heroku

Heroku has build packs which are essentially set of language tools that are needed for deploying your application. Since in heroku we deploy application by pushing code, build packs provide the basic infrastructure thats needed to compile/interpret your code.

1. Install the [heroku toolbet](https://toolbelt.heroku.com/) as it provides a set of command line tools for deploying application easily to heroku.

2. login to heroku using the following
   
    ```
       $ heroku login
        Enter your Heroku credentials.
        Email: adam@example.com
        Password (typing will be hidden):
        Authentication successful.

    ```

3.  Create a new application using *** heroku create *** .This creates a dyno that can be used to deploy your application.

    ```
        Pradheep (master #) hw4-3 $ heroku create
        Creating infinite-shore-9738... done, stack is cedar-14
        https://infinite-shore-9738.herokuapp.com/ | https://git.heroku.com/infinite-shore-9738.git
        Git remote heroku added
    ```

4. Now build your application and test it in your local box to make sure that its working as expected and commit all the changes to master. 
        

5. Push the changes changes to heroku using the command 

    ```
        Pradheep (master) hw4-3 $ git push heroku master
        Counting objects: 44, done.
        Delta compression using up to 4 threads.
        Compressing objects: 100% (41/41), done.
        Writing objects: 100% (44/44), 4.39 MiB | 664.00 KiB/s, done.
        Total 44 (delta 8), reused 0 (delta 0)
        remote: Compressing source files... done.
        remote: Building source:
        remote: 
        remote: -----> Python app detected
        remote: -----> Installing runtime (python-2.7.10)
        remote: -----> Installing dependencies with pip
        remote:        Collecting bottle (from -r requirements.txt (line 1))
        remote:          Downloading bottle-0.12.9.tar.gz (69kB)
        remote:        Collecting pymongo (from -r requirements.txt (line 2))
        remote:          Downloading pymongo-3.1.1.tar.gz (462kB)
        remote:        Installing collected packages: bottle, pymongo
        remote:          Running setup.py install for bottle
        remote:          Running setup.py install for pymongo
        remote:        Successfully installed bottle-0.12.9 pymongo-3.1.1
        remote: 
        remote: -----> Discovering process types
        remote:        Procfile declares types -> web
        remote: 
        remote: -----> Compressing... done, 40.3MB
        remote: -----> Launching... done, v3
        remote:        https://infinite-shore-9738.herokuapp.com/ deployed to Heroku
        remote: 
        remote: Verifying deploy.... done.
        To https://git.heroku.com/infinite-shore-9738.git
         * [new branch]      master -> master
    ```

6. Now the application is deployed to heroku, we need to make sure that its working and we can open the application using 

```
    Pradheep (master) hw4-3 $ heroku open
    Opening infinite-shore-9738... done
```

7.When i opened the application page it was throwing page not found error.So in order to figure out the error we can see the logs by using command 

```

Pradheep (master) hw4-3 $ heroku logs
2015-12-01T18:54:17.334789+00:00 heroku[web.1]: Starting process with command `python ./blog.py`
2015-12-01T18:54:19.342818+00:00 app[web.1]: Bottle v0.12.9 server starting up (using WSGIRefServer())...
2015-12-01T18:54:19.342862+00:00 app[web.1]: Listening on http://localhost:8082/
2015-12-01T18:54:19.342874+00:00 app[web.1]: Hit Ctrl-C to quit.
2015-12-01T18:54:19.342876+00:00 app[web.1]: 
2015-12-01T18:55:17.799116+00:00 heroku[web.1]: Error R10 (Boot timeout) -> Web process failed to bind to $PORT within 60 seconds of launch  <--------------- This is error 
2015-12-01T18:55:17.799116+00:00 heroku[web.1]: Stopping process with SIGKILL
2015-12-01T18:55:18.475966+00:00 heroku[web.1]: State changed from starting to crashed
2015-12-01T18:55:18.458035+00:00 heroku[web.1]: Process exited with status 137
2015-12-01T18:55:19.155515+00:00 heroku[router]: at=error code=H10 desc="App crashed" method=GET path="/" host=infinite-shore-9738.herokuapp.com request_id=79d4746f-8a61-409e-b970-49c99f7ef3c3 fwd="72.177.207.73" dyno= connect= service= status=503 bytes=
2015-12-01T18:55:19.701594+00:00 heroku[router]: at=error code=H10 desc="App crashed" method=GET path="/favicon.ico" host=infinite-shore-9738.herokuapp.com request_id=a38c3ce7-eb7e-4b88-ad3d-a79eabbe0b61 fwd="72.177.207.73" dyno= connect= service= status=503 bytes=

```

8. Looking at the logs above its clear that we are not binding to the correct port. Revel applications must bind to the port that the heroku application specify using the enviornment variable $PORT. So we modify the application to make it listen to the $PORT.

```
bottle.run(host='0.0.0.0', port=int(os.environ['PORT']))  # Start the webserver running and wait for requests                                                                                                  
```
In the above example, as the default application used bottle i modified the application a bit to use the enviornment variable *$PORT*

9. Heroku also needs a Procfile to help it understand how it needs to start the application.

    ```
    Pradheep (master *) hw4-3 $ cat Procfile 
    web: python ./blog.py 
    ```

In the above, the * web * specifies that the application needs to accept traffic from outside.

10. The application must also have a * requirements.txt * to specify the files that are needed to deploy the application. 

    ```
    Pradheep (master *) hw4-3 $ cat requirements.txt 
    bottle
    pymongo

```
Now push the changes again using git push and look at the heroku logs to see what went wrong and fix it untill the application is running properly. 

Also make sure that you are trying to access a page that is doesnot use database query as we have not yet connected a mongodb to the application yet.

11. To open the heroku application logs or login to the application use * heroku run bash* command as below 

```
Pradheep (master *) hw4-3 $ heroku run bash
Running bash on infinite-shore-9738... up, run.5908
```

12. If you application has crashed and you need to start it again then use the * ps:webscale * option

```
 heroku ps:scale web=1

```
The default free tier provides only 1 tier so set the value is set to 1 above.


In the next post i will discuss how i hooked up mongolab to our blog application.






