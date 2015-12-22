---
layout: post
title: Creating a URL Shortener with Flask - Part 4
published: true
comments: true
categories : mongodb python flask pymongo
---
In this part 4 , i will discuss more on additional 2 tools which will help us keep the code consistent and also figure out ares of improvement.

# Code coverage


Code coverage is one of the oldest tools available for measuring the amount of test coverage. Earlier that code coverage was performed with manual test cases but with automation of unit test becoming a norm now a days , the test coverage can be run regularly. 

1. Python support of code coverage is done with a tool called `coverage`. It can be installed as below

    ```
        pip install coverage
    ```

2. Now that we have all the tests that are automated we can easily run the code coverage as below

    ```
    $coverage run --source app -m unittest discover 
    -------------------------------
    Ran 11 tests in 0.173s
    OK
    ```

    The above code runs all the unit tests and stores the coverage in a file called `.coverage` in the directory you ran the command.

3.  To see the report , we should see the coverage report command as 

    ```
        $ coverage report -m
        Name                              Stmts   Miss  Cover   Missing
        ---------------------------------------------------------------
        app/__init__.py                       3      0   100%   
        app/models/MongodbHandler.py          9      0   100%   
        app/models/__init__.py                0      0   100%   
        app/models/test_urlShortener.py      55      1    98%   90
        app/models/urlshortener.py           32      4    88%   32-33, 43-45
        app/views.py                         27      1    96%   48
        ---------------------------------------------------------------
        TOTAL                               126      6    95%   

    ```
    The output clearly points out the missing lines in the coverage .

4. Lets make it easy to run the coverage by putting all the coverage in a make file . Lets first create a new branch called `addCoverageSupport` to track all our coverage related changes.

    ```
        git branch addCoverageSupport
        git checkout addCoverageSupport
    ``` 

5. Though we can run it manually every time, it would be better if we could make the coverage also run automatically as part of our automatic build process(travis).To do this we have to use a online tool called [`coveralls`](https://coveralls.io/).

6. Sign up and create a new account for you and enable the repo on the coverall. Doing all this should be straight forward.

7. Install `coverall` for doing the code coverage as below 

    ```
        pip install coveralls
    ``` 

8. Add the `coveralls` to  `.travis.yml` for running code coverage as below

    ```
        # command to run tests
        script:
            coverage run --source=app -m unittest discover

        after_success:
            coveralls
    ```

9. Now update the `requirement.txt` and push the changes to the github so that it runs on travis


    ```
        pip freeze > requirement.txt
        git add .
        git commit -m "Added coveralls support"
        git push origin addCoverageSupport
    ```

10. When travis runs and is successful, it pushes all the code coverage report and the output should be somthing similar to below.

    ![travis](images/URLShortner/coveralls.png) 

    From the above, its clear that we have a code coverage of 94%, we will look more into how to get to a code coverage of 100%

11. Merge the changes into the master and we will create a new branch for making the code coverage to 100%


    ```
        git checkout master
        git merge  addCoverageSupport
    ```

12. Push the changes to github using

    ```
        git push master origin
    ```

# Improve code Coverage

    Now lets  work on to improve the code coverage to make sure we cover all the cases, by looking at the profile output from the coveralls.

    1. Lets first create a branch to do all the changes related to code coverage improvement.
        ```
            git checkout -b improveCode

        ```

    2. The code has no coverage for the case where we try to access a shortURL that is not available. In this case , the url shortener must return 404 not found. So lets add a functional test to improve code coverage in `testing/test_basicsite.py`
      
        ```
            # try to access a invalid shorturl and the code
            # must return error code 404 not found
            def test_get_invalidShortUrl(self):

                # invalid shortUrl
                shorturl = self.baseURL + '/' + '112111111'

                rv = self.client.get(shorturl)

                self.assertEqual(rv.status_code, 404)
        ```

    3.  The code has no coverage for case where the model fails to perform operation like save. In case of error trying to create a shortURL ,we should return a back to the index page and with possibly a flash messsage. For time being we will only check if the page returns back to the index.html

        ```
        # the case where we send a request to shorten the url and for
        # whatever reason , the code shortened url is not created
        def test_post_to_urlShortener_fail_in_model(self):

            # monkey patch the code to make sure that the saveUrl returns
            # false so we can check the return value.
            from app.models import urlshortener
            beforepatch = urlshortener.urlShortener.saveUrl
            urlshortener.urlShortener.saveUrl = self.stub_saveURL_returns_false
            post_data = {'url': 'http://www.google.com/'}

            rv = self.client.post('/urlshorten',
                                  data=post_data,
                                  follow_redirects=False)

            self.assertEqual(rv.status_code, 200)

            #cleanup
            urlshortener.urlShortener.saveUrl = beforepatch

        ```

With this we improve the code coverage to 96%. We will stop at this as we have some more code at model level to perform and we will look at improving the coverage more when we make changes.



