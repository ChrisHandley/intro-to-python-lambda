---
title: Login and Databases with Python and Flask
teaching: 20
exercises: 10
questions:
- "Login and session management with Flask?"
objectives:
- "Set up our login manager."
- "Access a database using python."
keypoints:
- "Python has many database libraries and queries are done using neat python syntax rather than SQL."
---

## Setup
Sooner or later we want an app that can manage user sessions and logging in and out so that users
can access their particular data and personalised preferences.

Install flask-sqlalchemy and flask-migrate

Let's return to our `__init__.py` and load in what we require.

~~~
from flask import Flask
from flask_bootstrap import Bootstrap
from flask_login import LoginManager
from flask_sqlalchemy import SQLAlchemy
from config import Config
import os

app = Flask(__name__)
app.config.from_object(Config)
db = SQLAlchemy(app)
login = LoginManager(app)
login.login_view = 'login'

# Flask-Bootstrap requires this line
Bootstrap(app)

from app import routes
~~~
{: .language-python}

We can see we have now imported the `LoginManager` from the library `flask_login`. This is a set of tools that extends the
app to monitor logins, current user sessions etc.

To use the LoginManager we are going to need a database, `db`, which we create using `SQLAlchemy`, which is a sql database type, and attach to the instance of the app, same as the login manager `login`. If the user is not logged in and tried to access pages that require authentification `login_view` informs the login manager which view to redirect to.

If we are to use a database, it will need to be created in the event it doesn't exist first. So we modify the main `microblog.py`.

~~~
from app import app, db

if __name__ == "__main__":
    db.create_all()
    app.run(debug=False)
~~~
{: .language-python}

Note we now also import the instance of the `db` database we defined above, and initiated if it is not present using `db.create_all()`.

You will also note in our `__init__.py` we now have more settings and paramters that we shift to a config file. The config file is imported as the `Config` object, using `from config import Config`, and that configuration is loaded into the app using `app.config.from_object(Config)`.

The `Config` object is defined in the main directory in `config.py` and has the following content.

~~~
import os
basedir = os.path.abspath(os.path.dirname(__file__))

class Config(object):
    SECRET_KEY = 'C2HWGVoMGfNTBsrYQg8EcMrdTimkZfAb'
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
~~~
{: .language-python}


~~~
microapp/
├── app
│   ├── __init__.py
│   ├── routes.py
|   ├── app.db
|   └── templates
|       ├── actor.html
|       ├── name.html
|       └── index.html
├── config.py
└── microblog.py
~~~
{: .terminal}

We import the os library so we can interact with the file directory system and set the absolute path where the app is run to
also be the location for the app databases.

The `Config` class is an object, and within it has some values we set.
The secret key is used to encrypt a session, and can then be stored in a cookie.
The database URI is set using the basedir.
And finally we set the database so we don't track modifications.

## Databases

To utilise the database we are going to need to define a table that stores our user information. This is done in `app/models.py`.

~~~
from app import db, login
import datetime
from flask_login import UserMixin
from werkzeug.security import generate_password_hash, check_password_hash

class User(UserMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password_hash = db.Column(db.String(128))

    def __repr__(self):
        return '<User {}>'.format(self.username)

    def set_password(self, password):
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        return check_password_hash(self.password_hash, password)

@login.user_loader
def load_user(id):
    return User.query.get(int(id))
~~~
{: .language-python}

Once more we import the database instance and the login manager instance. We also import from flask-login `UserMixin`. Flask-login requires a User model with the following properties:

- has an is_authenticated() method that returns True if the user has provided valid credentials
- has an is_active() method that returns True if the user’s account is active
- has an is_anonymous() method that returns True if the current user is an anonymous user
- has a get_id() method which, given a User instance, returns the unique ID for that object

UserMixin class provides the implementation of these properties. Its the reason you can call for example `is_authenticated` to check if login credentials provided are correct or not instead of having to write a method to do that yourself.

The `User` class we create is a table for our database. We can see that each piece of information is a `Column` and we can set the type of the data, if they are unique, a primary key, or if they are indexed for quick look up based on that attribute.

Note we also set some dunder methods which allow us to use the database User class to has the password as it is added, check the password with the hashed one, or return the username of an entry in the database.

We also define a further function, the user_loader. Each session of the app stores that user's ID. But to get the full information of the user it needs to be retrieved from the database using the ID.

## Forms

We have already made use of forms in the previous example, but here we elaborate more, and move our forms into a separate file,
`app/forms.py`.

~~~
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import ValidationError, DataRequired, Email, EqualTo
from app.models import User


class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
    remember_me = BooleanField('Remember Me')
    submit = SubmitField('Sign In')


class RegistrationForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    email = StringField('Email', validators=[DataRequired(), Email()])
    password = PasswordField('Password', validators=[DataRequired()])
    password2 = PasswordField(
        'Repeat Password', validators=[DataRequired(), EqualTo('password')])
    submit = SubmitField('Register')

    def validate_username(self, username):
        user = User.query.filter_by(username=username.data).first()
        if user is not None:
            raise ValidationError('Please use a different username.')

    def validate_email(self, email):
        user = User.query.filter_by(email=email.data).first()
        if user is not None:
            raise ValidationError('Please use a different email address.')

class NameForm(FlaskForm):
    name = StringField('Which actor is your favorite?', validators=[DataRequired()])
    submit = SubmitField('Submit')
~~~
{: .language-python}

We now have a few new field types, including BolleanField and PasswordField, and classes for some of these fields, such as Email, to make checking the data passed to the form is the correct format.

We have two new FlaskForms to utilise, the LoginForm, and the RegistrationForm. The LoginForm is much the same as the NameForm
we created previously, using validators, and a BolleanField that appears as a checkbox when rendered.

The RegistrationForm is one that has some associated functions, which allows us to check for the uniqueness of the username and email provided.

## Routes

With the infrastructure in place we are now in the place to create the endpoints for login, logout, and registration.

~~~
from app import app, db
from flask import request, render_template, redirect, url_for, flash
from flask_login import current_user, login_user, logout_user, login_required
from werkzeug.urls import url_parse
from app.data import ACTORS
from app.models import User
from app.forms import LoginForm, RegistrationForm, NameForm
from app.module import get_names, get_actor, get_id

...

@app.route('/login', methods=['GET', 'POST'])
def login():
    if current_user.is_authenticated:
        return redirect(url_for('index'))
    form = LoginForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.username.data).first()
        if user is None or not user.check_password(form.password.data):
            flash('Invalid username or password', 'alert-danger')
            return redirect(url_for('login'))
        login_user(user, remember=form.remember_me.data)
        next_page = request.args.get('next')
        if not next_page or url_parse(next_page).netloc != '':
            next_page = url_for('index')
        return redirect(next_page)
    return render_template('login.html', title='Sign In', form=form)


@app.route('/logout')
def logout():
    logout_user()
    return redirect(url_for('login'))


@app.route('/register', methods=['GET', 'POST'])
def register():
    if current_user.is_authenticated:
        return redirect(url_for('index'))
    form = RegistrationForm()
    if form.validate_on_submit():
        email = form.email.data
        email_domain = email.split('@')[-1]
        user = User(username=form.username.data, email=form.email.data)
        user.set_password(form.password.data)
        db.session.add(user)
        db.session.commit()
        flash('Congratulations, you are now a registered user!', 'alert-success')
        return redirect(url_for('login'))
    return render_template('register.html', title='Register', form=form)

@app.route('/index')
@login_required
def index():
    user = {'username': current_user}
    return render_template('index.html', title='Home', user=user)

...
~~~
{: .language-python}

Again we import the app and now also the database, `db`, and we now have the flash function for sending messages to the front end.

From `flask_login` we import all the functions for interaction with the login manager, mainly a way to recall the current user of the session, and to log in and out the user into that session.

From `werkzeug.urls` import url_parse, which checks if the url is present and then by interrogating netloc we determine if the url is to a page that is local.

With our forms also loaded in, we can refer to them for login, logout, and registration.

The logout route is simplest. Going to the route invokes the `logout_user` function, and then redirects the session to the login page.

The registration route is a bit more complex as it utilises the database. The `current_user` object keeps information about the user for the current session and if they are authenticated. This means we can refer to this attribute to allow or deny access to views, routes, and backend functionality. In this case going to register if you are already logged in lead to redirection to the main page.

In the case the user is not authenticated they may being the registration process. We pass the RegistrationForm to form, and this is then rendered on the `register.html`, using the `render_template` function. When the form is filled in and submitted on the frontend, the form is returned and data can be extracted from it.

An instance of the database `User` entry is created and passed to it is the form data, and using the instance the password can be set using the built in password functions created earlier.

Then the user instance is added to the database in the User table, using `db.session.add(user)` and once all additions are made, the session is committed, and a redirection to the login page can occur.

The login route is similar, with a form, this time the `LoginForm` passed to form and used as part of the rendering of the `login.html` page. Redirection occurs first if the user is already authenticated for the session. In the case of login being required the form is inspected to see if it has been submitted and data is extracted from the fields. `User.query.filter_by(username=form.username.data).first()` searches the database to retrieve the User entry instance that is compared to to confirm passwords. The use of the `next` argument is used, because this has content if we try to access pages that require login, and so once we complete login, the redirect is to this page we were trying to login into.

The `login_user` function then takes the user information and the remember_me argument, to inform the session that the user is logged in.

Finally, we have made use of a new decorator on our index route. `@login_required` checks to see if the current session is logged in, and if so this route can be accessed.

## Templates

The `login.html` template is, in the minimal form really simple.

~~~
{% raw %}
{% extends "base.html" %}

{% block content %}
    <h1>Sign In</h1>
    <form action="" method="post" novalidate>
        {{ form.hidden_tag() }}
        <p>
            {{ form.username.label }}<br>
            {{ form.username(size=32) }}<br>
            {% for error in form.username.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>
            {{ form.password.label }}<br>
            {{ form.password(size=32) }}<br>
            {% for error in form.password.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>{{ form.remember_me() }} {{ form.remember_me.label }}</p>
        <p>{{ form.submit() }}</p>
    </form>
    <p>New User? <a href="{{ url_for('register') }}">Click to Register!</a></p>
{% endblock %}
{% endraw %}
~~~
{: .language-html}

Once more the Jinja2 package allows us to pass from python Flask app to the html using {{ }}, in this case the form content. WE can also pick up errors in the form `form.errors` which can then be used to trigger an error message.

Similarly the `register.html` template is equally simple.

~~~
{% raw %}
{% extends "base.html" %}

{% block content %}
    <h1>Register</h1>
    <form action="" method="post">
        {{ form.hidden_tag() }}
        <p>
            {{ form.username.label }}<br>
            {{ form.username(size=32) }}<br>
            {% for error in form.username.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>
            {{ form.email.label }}<br>
            {{ form.email(size=64) }}<br>
            {% for error in form.email.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>
            {{ form.password.label }}<br>
            {{ form.password(size=32) }}<br>
            {% for error in form.password.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>
            {{ form.password2.label }}<br>
            {{ form.password2(size=32) }}<br>
            {% for error in form.password2.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>{{ form.submit() }}</p>
    </form>
{% endblock %}
{% endraw %}
~~~



> ## Blog Posts
>
>
> What would the database look like to accomodate blog posts from users?
> Criteria:
> - The post needs to contain a body of just 140 characters length
> - We need to capture the time stamp of when the blog post was created
> - We need to capture the user who creates the blog post
>
> > ## Solution
> > 
> > ~~~
> > class Post(db.Model):
> >     id = db.Column(db.Integer, primary_key=True)
> >     body = db.Column(db.String(140))
> >     timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)
> >     user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
> > 
> >    def __repr__(self):
> >        return '<Post {}>'.format(self.body)
> >
> > ~~~
> > {: .language-python}
> > {: .solution}
>
> How do we update the User table to accomodate a 1 to many relationship
> between user and the posts they make?
>
> > ~~~
> > class User(db.Model):
> > ...
> > posts = db.relationship('Post', backref='author', lazy='dynamic')
> > ...
> > ~~~
> > {: .language-python}
> > post.author will return the user object related to that post.
> > {: .solution}
> 
> 


{% include links.md %}
