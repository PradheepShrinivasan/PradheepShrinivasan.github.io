---
layout: post
title: Creating a URL Shortener with Flask - Part 3
published: true
comments: true
categories : mongodb python flask pymongo
---
In this section we dive a bit more deep into looking into how we are going to have some basic functional testing for the application and then we look look into making a config file and finally merge all the changes and make them into our release 0.1

## Integration tests

Integration tests are integral part of system development. They make sure that the code works correctly, when we make changes and also validating code merge from several branches. The integration tests in this case will be written our well known unit test framework. 

1. Lets make a testing folder for functional testing

    ```
        mkdir testing
        cd testing
    ```

2. We will add a new file called `test_basicsite.py` in this folder and write the basic setup code

    ```
    import unittest
    import sys
    import urllib

    sys.path.append('..')


    class TestBasicUrlShortener(unittest.TestCase):
        def setUp(self):
            self.client = app.test_client()
            self.baseURL = 'http://localhost:5000'

        # this is one of the functions that must be
        # implemented for flask testing.
        def create_app(self):
            app = Flask(__name__)
            app.config['TESTING'] = True
            app.debug = True
            self.baseURL = 'http://localhost:5000'
            return app
    ```

3. To test `index` we must first send a `Get` request with the flask client application and check for the return value and the presence of input field and `url` field in the page that is received

    ```
     # Make sure that we have the index page working
    def test_get_to_index(self):
        rv = self.client.get('/')

        assert rv.status_code == 200
        assert 'name=\"url\"' in str(rv.data)
        assert 'input' in str(rv.data)

    # when we send a Get , we need to make sure that
    # it redirects to index
    def test_get_to_urlshortener(self):

        rv =self.client.get('urlshorten')

        self.assertEqual(rv.status_code, 302)
        assert 'localhost'in rv.location

    ```

4. In order to test the `post request`, we monkey patch the 'generate_shortURL' to make sure that it returns a fixed string so that we know the exact string to look for in the response

    ```
        # When we send a post we expect it to return a output
    # containing the baseURL and short url
    def test_post_to_urlshortener(self):

        # monkeypatch the generate shortURL so that we know
        # the correct value to expect and perform validation
        # accordingly
        from app.models import urlshortener
        urlshortener.urlShortener.generateShortUrl = self.generate_shortURL
        post_data = {'url': 'http://www.google.com/'}

        rv = self.client.post('/urlshorten',
                              data=post_data,
                              follow_redirects=False)

        self.assertEqual(rv.status_code, 200)
        shorturl = self.baseURL + '/' + self.generate_shortURL()
        assert shorturl in str(rv.data)

        #cleanup so next time it works
        urlshort = urlshortener.urlShortener()
        urlshort.removeUrl(self.generate_shortURL())

    ```

5. Now to check the URL redirect is working correctly we store a short url and then send a get request to make sure that we get a redirect response from our application.


    ```
       def test_get_shorturl(self):

            # monkey patch to a particular short url
            # store it in database and then
            # do a get with short url
            from app.models import urlshortener
            urlshortener.urlShortener.generateShortUrl = self.generate_shortURL_for_redirect
            post_data = {'url': 'http://www.google.com/'}
            self.client.post('/urlshorten',
                                  data=post_data,
                                  follow_redirects=False)

            shorturl = self.baseURL + '/' + self.generate_shortURL_for_redirect()
            rv = self.client.get(shorturl)

            self.assertEqual(rv.status_code, 302)
            self.assertEqual(rv.location, 'http://www.google.com/')

            #cleanup so next time it works
            urlshort = urlshortener.urlShortener()
            urlshort.removeUrl(self.generate_shortURL())
    ``` 

With these, we have covered all the basic integration test cases of our application.

## Making a config file for our application

Now all our configuration , we are getting it from our environment variables, but it would be better if we had a uniform configuration file for all our configuration.


1. we create a new file called `config.py` which will hold all our configuration in the top level directory.

2. The code structure looks like below 
    
    ```
        ├── LICENSE
        ├── Procfile
        ├── app
        │   ├── __init__.py
        │   ├── __init__.pyc
        │   ├── models
        │   ├── static
        │   ├── templates
        │   ├── views.py
        │   └── views.pyc
        ├── config.py
        ├── config.pyc
        ├── requirements.txt
        ├── run.py
        ├── testing
        │   ├── __init__.py
        │   ├── __init__.pyc
        │   ├── test_basicsite.py
        │   └── test_basicsite.pyc
        ├── tmp

    ```
    
3. In `config.py` we add all configuration to be read from the os environment variable and if not available it would not set a default value as below

    ```
        import os

        CONNECTION_URI = os.getenv('CONNECTION_STRING','mongodb://localhost:27017/')
        SITE_URL = os.getenv('SITE_URL', 'http://localhost:5000')
        PORT = os.getenv('PORT', 5000)

    ```

4. We import the configuration from all places where we would want to use the configuration as below 

    ```
        from config import PORT
        app.run(host='0.0.0.0', port=PORT,debug=True)
    ```
    This would import the `PORT` value from the configuration file.

After we have changed the configuration in all the other files where we were doing an `os.getenv`

5. Finally we run all the tests again to make sure that nothing is broken

    ```
        python -m unittest discover
        ...........
        ----------------------------------------------------------------------
        Ran 11 tests in 0.077s
        OK
    ```

## Merging code to main

1. Merge the code from `basicURLShortener` to `master`

    ```
        $ git checkout master
        Switched to branch 'master'
        
        $ git merge basicUrlShorterner 
        Updating 5c968d2..0f66d0b
        Fast-forward
         .travis.yml                     |  25 +++++++++++++++++++++++++
         app/__init__.py                 |   1 +
         app/models/Makefile             |   7 +++++++
         app/models/MongodbHandler.py    |  15 +++++++++++++++
         app/models/__init__.py          |   0
         app/models/test_urlShortener.py |  90 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
         app/models/urlshortener.py      |  51 +++++++++++++++++++++++++++++++++++++++++++++++++++
         app/templates/base.html         |  14 ++++++++++++++
         app/templates/index.html        |  23 +++++++++++++++++++++++
         app/views.py                    |  49 ++++++++++++++++++++++++++++++++++++++++++++++++-
         config.py                       |   6 ++++++
         requirements.txt                |   5 +++++
         run.py                          |   4 +++-
         testing/__init__.py             |   0
         testing/test_basicsite.py       | 102 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
         15 files changed, 390 insertions(+), 2 deletions(-)
         create mode 100644 .travis.yml
         create mode 100644 app/models/Makefile
         create mode 100644 app/models/MongodbHandler.py
         create mode 100644 app/models/__init__.py
         create mode 100644 app/models/test_urlShortener.py
         create mode 100644 app/models/urlshortener.py
         create mode 100644 app/templates/base.html
         create mode 100644 app/templates/index.html
         create mode 100644 config.py
         create mode 100644 testing/__init__.py
         create mode 100644 testing/test_basicsite.py
        (venv)Pradheep (master) UrlShortener $ 

    ```