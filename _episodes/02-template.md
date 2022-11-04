---
title: Html Templating
teaching: 10
exercises: 10
questions:
- "How can I use python to render webpages?"
objectives:
- "Templates"
- "Passing data to the front end"
keypoints:
- "You can do a lot of front end work with Python."
---

We can build webapps where the back end is all python and the front end client is Vue, for example.
But Python, and Flask, also lets us do some fun with templates and forms, so we don't have to touch a front
end language if we don't want to.

## Hello World with some style

~~~
@app.route('/index')
def index():
    user = {'username': 'Chris'}
    return '''
<html>
    <head>
        <title>Home Page - Microblog</title>
    </head>
    <body>
        <h1>Hello, ''' + user['username'] + '''!</h1>
    </body>
</html>'''
~~~
{: .language-python}

So the route `/index` has the index function that defines a dictionary with a key and value, `user`, and then returns to the client (the browser), the html.

We can see that the values in the dictionary can be accessed just as they would in regular python code.

Of course this is not the best way to define the html, as it can be subject to change, so we separate the functionalty
from the html

~~~
from flask import render_template
from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'Chris'}
    return render_template('index.html', title='Home', user=user)
~~~
{: .language-python}

The above is now our `routes.py` file in our app directory. See how the index function now returns using the
render_template function, and we pass to it the name of the template, and pass to it some arguments.

~~~
{% raw %}
<!doctype html>
<html>
    <head>
        <title>{{ title }} - Microblog</title>
    </head>
    <body>
        <h1>Hello, {{ user.username }}!</h1>
    </body>
</html>
{% endraw %}
~~~
{: language-html}

Note the change in how the values are used on the front end. They are referenced using the curly double brackets.
The `Jinja2` module in the background is performing this replacement for us.
We can also use conditional statements. The above html is in the `index.html` file, and is placed in the templates directory.

~~~
microapp/
├── app
│   ├── __init__.py
│   ├── routes.py
|   └── templates
|       └── index.html
└── microblog.py
~~~
{: .terminal}

~~~
{% raw %}
<!doctype html>
<html>
    <head>
        {% if title %}
        <title>{{ title }} - Microblog</title>
        {% else %}
        <title>Welcome to the Microblog!</title>
        {% endif %}
    </head>
    <body>
        <h1>Hello, {{ user.username }}!</h1>
    </body>
</html>
{% endraw %}
~~~
{: language-html}

We can also pass information and loop through it in the template.

~~~
from flask import render_template
from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'Chris'}
    posts = [
        {
            'author': {'username': 'John'},
            'body': 'Beautiful day in Portland!'
        },
        {
            'author': {'username': 'Susan'},
            'body': 'The Avengers movie was so cool!'
        }
    ]
    return render_template('index.html', title='Home', user=user, posts=posts)
~~~
{: .language-python}

~~~
{% raw %}
<!doctype html>
<html>
    <head>
        {% if title %}
        <title>{{ title }} - Microblog</title>
        {% else %}
        <title>Welcome to the Microblog!</title>
        {% endif %}
    </head>
    <body>
        <h1>Hi, {{ user.username }}!</h1>
        {% for post in posts %}
        <div><p>{{ post.author.username }} says: <b>{{ post.body }}</b></p></div>
        {% endfor %}
    </body>
</html>
{% endraw %}
~~~
{: language-html}

Note the for the list iteration follows python styles, just encased by the Jinga2 decorator curly brackets.

But how do we dynamically send data from the front end to the back end?

## Flask Forms to the Rescue

Flask supports defining forms that are pushed to the front end. In the `app` directory we can make a new file `forms.py`

Also install `wtforms` using pip.

~~~
from flask_wtf import FlaskForm
from wtforms import StringField
from wtforms.validators import DataRequired

class NameForm(FlaskForm):
    name = StringField('Which actor is your favorite?', validators=[DataRequired()])
    submit = SubmitField('Submit')
~~~
{: .language-python}

~~~
from flask import redirect, url_for
from app.data import ACTORS
from app.module import get_names, get_actor, get_id

@app.route('/name', methods=['GET', 'POST'])
def name():
    names = get_names(ACTORS)
    # you must tell the variable 'form' what you named the class, above
    # 'form' is the variable name used in this template: index.html
    form = NameForm()
    message = ""
    if form.validate_on_submit():
        name = form.name.data
        if name.lower() in names:
            # empty the form field
            form.name.data = ""
            id = get_id(ACTORS, name)
            # redirect the browser to another route and template
            return redirect( url_for('actor', id=id) )
        else:
            message = "That actor is not in our database."
    return render_template('name.html', names=names, form=form, message=message)
~~~
{: .language-python}

This new route we takes the `ACTORS` object from a python file, `datas.py`, which is just a dictionary of names, ids, and urls and stores that object in the variable names. We place data.py in the app directory.

~~~
ACTORS = [
{"id":26073614,"name":"Al Pacino","photo":"https://placebear.com/342/479"},
{"id":77394988,"name":"Anthony Hopkins","photo":"https://placebear.com/305/469"},
{"id":44646271,"name":"Audrey Hepburn","photo":"https://placebear.com/390/442"},
{"id":85626345,"name":"Barbara Stanwyck","photo":"https://placebear.com/399/411"},
{"id":57946467,"name":"Barbra Streisand","photo":"https://placebear.com/328/490"}
]

~~~
{: .language-python}

The NameForm class is associated to the variable form. Note the Field stypes we use. There are many prebuilt, like radio buttons etc. Associated with the Field is a string of text, the allows for a description of the Field to be rendered.

This form communicates back to this backend if it is valid, and from the form, the name is extracted, compared to our list,
and the form is then cleared and prepared for another query. Using the name, the id is extracted, and the url obtained and
we are redirected to a the `actor` end point.

The name.html uses Bootstrap for rendering so we update the main app build in `__init__.py`.

~~~
from flask import Flask
from flask_bootstrap import Bootstrap

app = Flask(__name__)

# Flask-WTF requires an encryption key - the string can be anything
app.config['SECRET_KEY'] = 'C2HWGVoMGfNTBsrYQg8EcMrdTimkZfAb'

# Flask-Bootstrap requires this line
Bootstrap(app)

from app import routes
~~~
{: .language-python}

Boostrap prepares the app with bootstrap templates that can be used for building the pages.
We also update the app config to use a secret key for some added security for the user session.

~~~
{% raw %}
{% extends 'bootstrap/base.html' %}
{% import "bootstrap/wtf.html" as wtf %}

{% block styles %}
{{ super() }}
        <style>
                body { background: #e8f1f9; }
        </style>
{% endblock %}


{% block title %}
Best Movie Actors
{% endblock %}


{% block content %}

<div class="container">
  <div class="row">
    <div class="col-md-10 col-lg-8 mx-lg-auto mx-md-auto">

      <h1 class="pt-5 pb-2">Welcome to the best movie actors Flask example!</h1>

      <p class="lead">This is the index page for an example Flask app using Bootstrap and WTForms. Note that only 100 actors are in the data source. Partial names are not valid.</p>

      {{ wtf.quick_form(form) }}

      <p class="pt-5"><strong>{{ message }}</strong></p>

    </div>
  </div>
</div>

{% endblock %}
{% endraw %}
~~~
{: .language-html}

The `name.html` above shows the use of forms again, and is placed in the templates directory. In this case the form is rapidly built using `wtf.quick_form`.
That's it, nothing else to do. On acceptance of a valid name, the `name` route redirects us to `actor` route, using the id
to form the full url.

~~~
@app.route('/actor/<id>')
def actor(id):
    # run function to get actor data based on the id in the path
    id, name, photo = get_actor(ACTORS, id)
    if name == "Unknown":
        # redirect the browser to the error template
        return render_template('404.html'), 404
    else:
        # pass all the data for the selected actor to the template
        return render_template('actor.html', id=id, name=name, photo=photo)
~~~
{: .language-python}

In the actor route, using the id, we obtain the id, name, and url for the photo. We add thsi to our routes.py file.

~~~
{% raw %}
{% extends 'bootstrap/base.html' %}

{% block styles %}
{{ super() }}
        <style>
                body { background: #e8f1f9; }
        </style>
{% endblock %}


{% block title %}
Selected Actor: {{ name }}
{% endblock %}


{% block content %}

<div class="container">
  <div class="row">
    <div class="col-sm-6">

      <!-- note, the id is not used in the template -->

      <h1 class="pt-5 pb-2">{{ name }}</h1>

      <p><a href="{{ url_for('index') }}">Return to search page</a></p>

          <!--
              notice how the URL in the link above is rendered for
              Flask - it needs to name the FUNCTION, not the route
      -->

          <p>Images are from <a href="https://placebear.com/">placebear</a>.</p>

          <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Donec vel mi ut lorem porta porttitor. Nullam vestibulum nulla quis ligula lobortis, se
d commodo dolor lobortis. Cras accumsan justo quis sem dictum ultricies. Pellentesque placerat, dolor vitae fringilla pretium, orci dolor convallis odio, vit
ae cursus nibh sapien sit amet enim. Mauris imperdiet hendrerit risus, quis congue justo molestie in. Sed et lobortis nunc, sed rutrum nibh.</p>

    </div>
    <div class="col-sm-6">

      <img src="{{ photo }}" alt="Photo: {{ name }}" class="img-fluid pt-5">

      <p><em>Above: {{ name }}</em></p>

    </div>
  </div>
</div>
{% endblock %}
{% endraw %}
~~~
{: .language-html}

This is out page, `actor.html` which we are redirected to when we search for an actor from our list.

You will note in the `name` route, there are a few new functions that we call; `get_names`, `get_id`. The `actor` route contains the function `get_actor`.

All of these functions we define in the `module.py` file in the app directory. Note how in the above code changes for routes.py how we import these functions.

~~~
from app.module import get_names, get_actor, get_id
~~~
{: .language-python}


~~~
def get_names(source):
    names = []
    for row in source:
        # lowercase all the names for better searching
        name = row["name"].lower()
        names.append(name)
    return sorted(names)
~~~
{: .language-python}

`get_names` simply takes the content of `data.py` and extracts from it a list of names and places it into a list that we search.


~~~
def get_id(source, name):
    for row in source:
        # lower() makes the string all lowercase
        if name.lower() == row["name"].lower():
            id = row["id"]
            # change number to string
            id = str(id)
            # return id if name is valid
            return id
    # return these if id is not valid - not a great solution, but simple
    return "Unknown"
~~~
{: .language-python}

Provided with the name of the actor, the id number is returned, and this is then used to form the url for that page.

The name route then using the id redirects tot he actor route.

~~~
def get_actor(source, id):
    for row in source:
        if id == str( row["id"] ):
            name = row["name"]
            photo = row["photo"]
            # change number to string
            id = str(id)
            # return these if id is valid
            return id, name, photo
    # return these if id is not valid - not a great solution, but simple
    return "Unknown", "Unknown", ""
~~~
{: .language-python}

In the actor route, the id that is passed is used to get the actor information which is passed to the `actor.html` template.



Provided with the 

{% include links.md %}
