---
layout: post
title: Creating a URL Shortener - Part 2
published: true
comments: true
categories : mongodb python flask pymongo
---
In this part, i will discuss how i wrote the basic url shortener, in which i  will implemented a web page which returns a shortened URL. THer is no user based login stuff, no stats, no setting preffered short form and so on.Just enter the URL and the site would return a shorter version which is of fixed length of 6.


## Model implementation 

In the first part of this blog. i will discuss the implementation of the backend i.e database storage and retrieval.


1. We would want to make changes to a branch in git. We use branches to make sure that we can make group of changes independent of the main branch and merge code when its ready. So the main branch(also called master in git) is always present with a ready to ship code and we can multiple branches for each individual features. 

    ```
        $ git branch basicUrlShorterner
        $ git checkout basicUrlShorterner 
        Switched to branch 'basicUrlShorterner'
    ```

2. Since we are going to write unit test in alongside code , we need to install the basic unit testing framework of python i.e pyttest.

    ```
        pip install pytest
    ```

3. Lets now start creating a folder called models. Models in MVC architecture hold the code that does all the database handling ie all CRUD operation to the database.

    
    ```
        $ mkdir models
        $ cd models
    ```

4. Lets create a class`(urlShortener)` that performs all the operations to the database for url shortening. The class which performs the basic handling of storing to the database would look like below

    ```
    class urlShortener:

    def __init__(self, collection):
        self.collection = collection

    #  Save short Url and url
    # The short Url is stored as index as all looks
    # find and deletes will be only using short Url
    def saveUrl(self, shortUrl, url):
        saveQuery = {'_id': shortUrl, 'url':url}

        try:
            self.collection.insert_one(saveQuery)
        except:
                return False
        return True


    ```
    In the above code, the shortURL is kept as index as all operations in the database is going to be performed in most of the case and its required to be unique.Hence it makes a wonderful unique key to index the database.

5. So we write a unit test to check that the code works properly as below 

    ```
    class TestUrlShortener(unittest.TestCase):
        def setUp(self):
            connection = pymongo.MongoClient('mongodb://localhost:27017/')
            self.database = connection.test
            self.collection = self.database.urlshortener
            self.urlShortener = urlShortener(self.collection)

        def test_saveUrl_Unique(self):
            # setup
            url = "http://www.google.com"
            shortUrl = "gl"

            result = self.urlShortener.saveUrl(shortUrl, url)

            # Assertions
            self.assertEqual(result, True)
            doc = self.collection.find_one({'_id': shortUrl})
            self.assertEqual(doc, {'_id': shortUrl, 'url': url})

            # cleanup so that next time we dont get duplicateKeyError
            self.collection.delete_one({'_id': shortUrl})

        def test_saveUrl_duplicate(self):
            shortUrl = 'orig'
            url = 'http://www.google.com'
            urldup = 'http://www.yahoo.com'

            self.urlShortener.saveUrl(shortUrl, url)
            result = self.urlShortener.saveUrl(shortUrl, urldup)

            self.assertEqual(result, False)
            doc = self.collection.find_one({'_id': shortUrl})
            self.assertEqual(doc, {'_id': shortUrl, 'url': url})

            self.collection.delete_one({'_id': shortUrl})

    ```

6. Now that we have methods to save, the next important method is `findURL` from the database. The code is simple

    ```
    # Finds a url from shorUrl that is sent from the user
    def findUrl(self, shortUrl):

        try:
            doc  = self.collection.find_one({'_id': shortUrl})
        except:
            return None
        if doc is None:
            return None

        return doc['url']
    ```

    When the database cannot find a short URL we must send a `NotFound` message. 

7. The test case for `findURL` is as follows

    ```
    def test_findUrl_Existing(self):

        shortUrl = 'findUrl'
        url = 'http://www.google.com'
        self.urlShortener.saveUrl(shortUrl, url)

        resultUrl = self.urlShortener.findUrl(shortUrl)

        self.assertEqual(resultUrl, url)

        self.collection.delete_one({'_id': shortUrl})

    def test_findUrl_NonExisting(self):
        shortUrl = 'findDuplicate'

        resultUrl = self.urlShortener.findUrl(shortUrl)

        self.assertEqual(resultUrl, None)
    ```

8. And finally methods for deleting and generating a alpha numerical code of fixed length for short URL as below 

    ```
        def removeUrl(self, shortUrl):

        try:
            result = self.collection.delete_one({'_id': shortUrl})
        except:
            # we would want to log an error message as of now
            return False
        return True


        # generates an shorturl needed from so
        #  http://stackoverflow.com/questions/2257441/random-string-generation-with-upper-case-letters-and-digits-in-python/23728630#23728630
        def generateShortUrl(self, length=6):
            return ''.join(random.SystemRandom().choice(string.ascii_uppercase + string.digits) for _ in range(length))

    ```

    And test cases 

    ```

    def test_removeURL_Existing(self):
        shortURL = 'removeURL'
        url = 'http://www.google.com'
        self.urlShortener.saveUrl(shortUrl, url)

        result = self.urlShortener.removeUrl(shortURL)

        self.assertEqual(result, True)


    def test_removeURL_NonExisting(self):
        shortURL = 'NonExisting'

        result = self.urlShortener.removeUrl(shortURL)

        self.assertEqual(result, True)


    def test_generateRandom(self):

        self.assertEqual(len(self.urlShortener.generateShortUrl()), 6)
        self.assertEqual(len(self.urlShortener.generateShortUrl(7)), 7)

        self.assertEqual(self.urlShortener.generateShortUrl().isalnum(), True)

    ```

9. We can always run the test code at any time using the below command at top level

    ```
        python -m unittest discover
    ```

10. Finally we create a `mongodbHandler.py` to handle all the connection information for url Handler. The code is as follows

    ```
    import pymongo
    import os

    # TODO: Make this class read from a config file
    # Now it reads from the enviornment variable
    class MongoDatabaseHandler:

        def __init__(self):
            connection_string = os.environ['CONNECTION_STRING']
            self.connection = pymongo.MongoClient(connection_string)
            self.database = self.connection.urlshortener

        def get_ShortURLCollection(self):
            return self.database.shorturls

    ```
    The above code reads from enviornment variable `CONNECTION_STRING`, make sure that your export it in your shell as `export CONNECTION_STRING = "mongodb://localhost:27017/"`

## Designing of the web page(View)

Now that we have got the basic structure of the backend all taken care, we can look designing the front end. I wanted the page to be simple and so i created a basic mockup as follows.

![URLShortener Home page](images/URLShortner/URLShorterner_HomePage.png) 

1.The basic web page design in our case is very simple.All we need to do is have a webform that takes an URL and a submit button.

2. The html front end is divided into 2 parts, the basic include stuff like header that is common between all pages that we are planning to create and the page specific sections. The basic form is as follows


    {% highlight html  %}
    {% raw %}

    <html>
        <head>
        {% if title %}
            <title>{{ title }} - URLShortener</title>
        {% else %}
            <title>Welcome to URLShortener</title>
        {% endif %}
        </head>
        <body>
            {% block content %}

            {% endblock %}
        </body>
    </html>

    {% endraw %}
    {% endhighlight %}
    
3.The landing page (also called the `index.html`) which we had a UI design above is as follows

    {% highlight html  %}
    {% raw %}

    <!-- extend from base layout -->
    {% extends "base.html" %}

    {% block content %}
        
        <h1>URL Shoterner!</h1>
        
        <form action="/UrlShorten" method="post" >
          <div>
              <input type="url" id="url"  name="url"/>
          </div>
          <div>
              <input type="submit" value="Shorten">
          </div>
        </form>
        
        {% if shortURL %}
            <div class="shortUrlOutput">
                {{ shortURL }}
            </div>
        {% endif %}
    
    {% endblock %}

    {% endraw %}
    {% endhighlight %}


The above code sends a `post` method to the link `/UrlShorten`, where we will have a handler that will recieve the `url` value from the form and create a response with the shorturl.


Also you will find the code contains a conditional `include shortURL` which we will use to send the output.

## Handling of requests (Controller)

The final part we need to design the controller which is responsible for handling requests and sending the response.

1. When a user enters the site, we provide him with the `index.html` page. so to render the page we have the code as below

    ```
        from app import app
        from flask import render_template

        @app.route('/')
        @app.route('/index')
        def index():
             return render_template('index.html')
    ```
    In the above `@app.route` which is a decorator provided by the `Flask` framework to define functions that should handle when browser sends requests.

2. Now on the handler that redirects the shortened url to the full url. The code is simple, it just reads from the database using the shortened url as the key.If a page is found it redirects to that page or it sends a 404 not found code.

    ```
    @app.route('/<shorturl>')
    def getURL(shorturl):

        url_shortener_handler = urlShortener()
        url = url_shortener_handler.findUrl(shorturl)

        app.logger.debug("value of url(%s) is for short url(%s) ", url, shorturl)

        if url is not None:
            return redirect(url, code=302)
        else:
            return abort(404)
    ```

3. Finally the controller handler which generates creates the shortURL is as follows

    ```
    @app.route('/UrlShorten', methods=['POST', 'GET'])
    def shortenUrl():
        if request.method == 'POST':
            url = request.form['url']
            url_shortener_handler = urlShortener()

            #TODO have a mechanism for handling duplicate key error
            short_url = url_shortener_handler.generateShortUrl()

            if url_shortener_handler.saveUrl(short_url, url):
                # TODO move the site_prefix to a config file
                site_url = os.environ['SITE_URL']
                return render_template('index.html', shortURL=site_url+short_url)
            else:
                #TODO change this to temporary error message
                return render_template('index.html', shortURL=None)
        else:
            return redirect('/')

    ```
In case of web form submission, the code just creates a instance of the `UrlShortener` that handles all the backend storage and if there is an error in generation of short URL it just redirects to homepage.


Also, we are importing a global variable called 'SITE_URL' which is needed as site prefix in the short url generation.For testing locally `export SITE_URL="http://localhost:5000/"`

To have a working model of the site look [here]( http://picourl.herokuapp.com/)




