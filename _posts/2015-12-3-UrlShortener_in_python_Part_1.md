---
layout: post
title: Creating a URL Shortener - Part 1
published: true
comments: true
categories : mongodb python flask pymongo
---

I wanted to play around with python more in terms of web application, rather than a scripting language.So i decided to create a URL shortener to understand the frameworks better. I chose flask because its a simpler framework than Django and is well supported.

Here are the steps i followed

## Setting up the enviornment.

1. Create a folder for my project 
    
    ```
        mkdir UrlShortener
    ```
2. I wanted to use a virtual env as it helps managing dependencies better.

    ``` 
    Pradheep github $ cd UrlShortener/
    Pradheep UrlShortener $ virtualenv venv
    New python executable in venv/bin/python2.7
    Also creating executable in venv/bin/python
    Installing setuptools, pip, wheel...done.
    
    ```
    Now we have all the virtual enviornment all set and ready to go.

3. Finally i activate the virtual enviornment by using activate command as follows

    ```
    Pradheep UrlShortener $ source venv/bin/activate
    (venv)Pradheep UrlShortener $ 

    ```

4. This project is going to use pymongo, flask so i did the following

    ```
    pip install Flask pymongo coverage
    ```

5. We then initialize an empty git repositary using git init

    ```
        git init .
    ```
    With this we have setup all the necessary tools.

6. I created a github repo and i pulled the repo to my local directory.

```
    git remote add origin https://github.com/PradheepShrinivasan/UrlShortener.git
    git pull
```

With this the entire setup basic setup of our system is done.

## Setting up Flask 

1. Create folders for application 

    ```
        mkdir app app/static app/template tmp
    ```
2. create an ``` app/__init__.py ``` file with the following contents

    ```
        from flask import Flask

        app = Flask(__name__)
        from app import views
    ```

3. create a view file in ``` app/view.py ```

    ```
    from app import app

    @app.route('/')
    @app.route('/index')
    def index():
        return "URL Shortener"
    ```

4. Now to start the application, we create  ``` run.py  ``` which starts our application

    ```
        #!flask/bin/python
        from app import app
        app.run(debug=True)
    ```

5. The directory structure of application is as follows

    ```
    $tree  ./ -l 2
    ./
    ├── LICENSE
    ├── app
    │   ├── __init__.py
    │   ├── static
    │   ├── template
    │   ├── views.py
    │   └── views.pyc
    ├── run.py
    ├── tmp
    └── venv
    ```

6. Now run the application using the following  command 

    ```
        $ python run.py 
         * Restarting with stat
         * Debugger is active!
         * Debugger pin code: 163-315-710
    ```

7. Go to a web browser and look up `http://localhost:5000` and you will be able to will see " URL Shortener!"

8. Commit all changes to you have made till now to git master branch


## Deploying to heroku 

1. Install heroku toolbelt if you have not already installed.

2. Create add a remote of 

    ```
        $ heroku create
        Creating enigmatic-temple-7361... done, stack is cedar-14
        https://enigmatic-temple-7361.herokuapp.com/ | https://git.heroku.com/enigmatic-temple-7361.git
        Git remote heroku added
    ```
3. Create a requirement.txt that helps installtion on heroku by following command 

    ```
    $ pip freeze > requirements.txt

    ```
4. Create a Procfile that helps heroku identify how to start the application 

    ```
    web: python ./run.py
    ```
5. Now change the ```run.py``` a bit to make it run on heroku. Heroku expects that you use the ``` PORT ``` enviornment variable 

    ```
    app.run(host='0.0.0.0', port=int(os.environ['PORT']), debug=True)

    ```
6. commit all changes and push the changes to heroku using 

    ```
    git add Procfile requirments.txt run.py
    git commit -m "add changes to Procfile, requiements.txt"
    git push heroku master
    ```
This moves the code to heroku and start the application

7. Now open the application using `heroku open`

    ```
    heroku open
    ```

You must see the same output as you have seen earlier.


With this we have completed installation of application on Flask and hosted it on heroku.

The heroku hosted site is [here](http://enigmatic-temple-7361.herokuapp.com)

## Reference:

I looked up this micro blog for some ideas and basic structure of the flask application. I wish i would write a full blown application like the author and i highly recommend reading the [blog](http://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)