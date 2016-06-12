---
layout: post
title: Creating a URL Shortener with Flask - Part 5
published: true
comments: true
categories : mongodb python flask pymongo login
---

In part 5 of this series, i will discuss how i added login support for the url shortener. If the user has logged in we will provide him with a set of links that he created and statistics regarding all the links he created. The first step is creating the user account so we can track all the user login.


1. As usual lets create a new branch for handling of user account 

    ```
        git branch -b addLoginSupport
    ```

2. I will be using a flask package called the `flask-login` that handles most of the login handling. 

    ```
        pip install flask-login
    ```

3.  Lets first add the backend storage , our database model is going to be very simple to start with , we will have a '_id' field which is the users email address and a 'password' stored in the 'password' field.

4. The code for storing is going to be straight forward similar to that of the short url insertion we saw in part 1. The code is as follows

    ```
    import pymongo
    from MongodbHandler import MongoDatabaseHandler


    class user_database(object):
        def __init__(self):
            """ save the username and password """
            databaseHandler = MongoDatabaseHandler()
            self.collection = databaseHandler.get_users_collection()

        def get_password(self, username):
            """ get password from a given username """
            doc = self.collection.find_one({'_id': username}, {'_id': 0, 'password': 1})
            if doc is None:
                return None
            return doc['password']

        def save_user(self, username, password):
            """save the username and password to the database
                return false if trying to add the same user again
            """
            try:
                self.collection.insert_one({'_id': username, 'password': password})
            except pymongo.errors.DuplicateKeyError:
                return False

            return True

        def delete_user(self, username):
            """  delete the user given an username  """

            result = self.collection.delete_one({'_id': username})
            if result.deleted_count:
                return True
            else:
                return False

    ```

5. The corresponding test coverage for the same is as follows. We as usual duct type to make sure that the database writes are to the test database

    ```
    import unittest
    import pymongo

    from user_database import user_database

    class TestUser(unittest.TestCase):
        def get_user_collection(self):
            return self.collection

        def setUp(self):
            connection = pymongo.MongoClient('mongodb://localhost:27017/')
            self.database = connection.test
            self.collection = self.database.user
            self.collection.drop()
            self.user = user_database()

            # monkey patch the mongodb handler to use the test database of user
            from MongodbHandler import MongoDatabaseHandler

            MongoDatabaseHandler.get_users_collection = self.get_user_collection

        def test_insert_user(self):
            """save a valid user to the database """
            user = 'testuser@gmail.com'
            password = 'testpassword'

            result = self.user.save_user(user, password)

            self.assertEqual(result, True)
            # read the user from database
            doc = self.collection.find_one({'_id': user})
            self.assertEqual(doc, {'_id': user, 'password': password})

            self.collection.delete_one({'_id': user})

        def test_insert_duplicateuser(self):
            """" save the user twice and that we return a false"""

            user = 'duplicatetestuser'
            password = 'duplicatetestuserpassword'
            password2 = 'duplicatetestuserpassword2'
            result = self.user.save_user(user, password)

            result = self.user.save_user(user, password)

            self.assertEqual(result, False)

            self.collection.delete_one({'_id': user})

        def test_remove_user(self):

            user = 'removeuser'
            password = 'removeuserpassword'
            self.user.save_user(user, password)

            result = self.user.delete_user(user)

            self.assertEqual(result, True)
            doc = self.collection.find_one({'_id':user})
            self.assertEqual(doc, None)


        def test_remove_non_existing_user(self):

            user = 'dummyuser'

            result = self.user.delete_user(user)

            self.assertEqual(result, False)


        def test_get_user(self):

            user = 'myuser'
            password = 'mypassword'
            self.user.save_user(user, password)

            result = self.user.get_password(user)

            self.assertEqual(result, password)

            self.collection.delete_one({'_id':user})

        def test_get_user_non_existing(self):

            result = self.user.get_password('nonExistingUser')

            self.assertEqual(result, None)

    ```

6.We will add a LoginForm which will handle all our login related text boxes. The code of `app/login_form.py` is as follows

    ```
        from flask_wtf import Form
        from wtforms import StringField, validators, PasswordField, SubmitField
        from wtforms.validators import email, data_required


        class LoginForm(Form):

            email = StringField('email', validators = [email(), data_required()])
            password = PasswordField('Password', validators = [data_required()])
            submit = SubmitField('Login')
            register = SubmitField('register')

    ```


7. We will add the login at the top of our `index` page, so i will be adding the code to a separate `login.html` as follows

    ```

        <div>
            <form action= "{{ url_for('login') }}" method="post" >
                {{ login_form.hidden_tag() }}

                {{ login_form.email }}
                {{ login_form.password }}

                {{ login_form.submit }}
            </form>
            <form action= "{{ url_for('register') }}" method="get" ></form>
                {{ login_form.register }}
            </form>

        </div>

    ```


8. The next part is the using the 'flask_login' initialization code in `app/__init__.py`.This will initialize the login manager of flask login which we will use shortly.

    ```
        from flask import Flask
        from flask_wtf.csrf import CsrfProtect
        from flask_login import LoginManager

        app = Flask(__name__)

        CsrfProtect(app)

        login_manager = LoginManager()
        login_manager.init_app(app)

        from app import views
    ```



# Reference

Many of the things in this series is inspired/learnt by studying and looking into several sites.The login is inspired by the code from this [blog](http://gouthamanbalaraman.com/blog/minimal-flask-login-example.html)