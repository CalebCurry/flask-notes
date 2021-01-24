# GET Requests and Templates

GET is the default HTTP method when we request a webpage using our browser. Because of this, it's really easy to test! We can create an endpoint to accept GET requests and then visit that web address in the browser.

In fact, that's exactly what we did with our previous requests:

```python3
@app.route('/')
def index():
    return "Hello!"
```

This will accept GET requests at the root of our website.

Fascinating that we don't actually have to specify that this accepts GET requests. That's because it's the default for Flask end points.

If you want, you can manually specify that this is for GET requests by passing in an additional argument like ```methods = ['GET']``` to route. Seems kind of pointless now, but once we get into other HTTP methods, we can specify things like POST here to specify that the endpoint is design for POST, not GET.

Right now, we're returning a string. Obviously this is not a very useful thing to give the client if we're trying to build the next big thing. Instead, we should send HTML. We can type out the HTML directly here, like so:

```python3
@app.route('/')
def index():
    return '<h1>hello World</h1>'
```

This sort of works but doesn't give us a lot of freedom to build out web pages without polluting our routes file.

Thus, we can bring the HTML code to a separate file and then just return that file here instead.

FYI, this is the approach you would follow to process and send data directly using Flask. It's important to understand how this works first before we start building out an API (which we're going to start doing in a few lessons).

```python3
from flask import render_template

@ app.route('/')
def index():
return render_template('index.html')
```

Now, inside of a directory called ```templates``` in our module, create index.html. In Sublime I was able to type HTML and hit tab to generate a basic HTML structure. But if this doesn't work for you just type it out.

Feel free to modify the HTML as you desire, but here is a basic homepage:

```html
<!DOCTYPE html>
<html>
<head>
<title> Welcome to my Website!</title>
</head>
<body>
<h1>Testimonials!</h1>
<p> Here are some great things people have said: </p>
</body>
</html>
```

# Jinja templating

Right now this is just basic HTML. Flask is not offering anything more useful than just writing this out in HTML our self and not worrying about Flask.

Flask becomes a lot more handy when we pair it with a templating language, such as Jinja. Taking a look at your requirements.txt, you'll find that this was installed by default when we installed Flask.

Jinja allows us to integrate Python code in our HTML structure. This will render out to just HTML sent to the browser.

We can do things like iterate through elements in a list, do conditionals, and more! It would be nice if we could say something like:

```
for each testimonial:
display the message and name in HTML
```

In order to do this though, we need to think about the structure of testimonials. What are they going to look like?

Let's define some basic dummy testimonial data in Python.

# Creating Testimonial Example Data

We will build out our sample data as a list of dictionaries(key - value pairs). This is the structure typical when working with databases and we will see more of this later on.

We have to be careful of the variable name here if we have a function already named ```testimonials``` from our experimentation earlier. We can rename or remove that function(you could rename it ```get_testimonials``` for right now.

```python3
testimonials = [
    {
        'id': 10,
        'name': 'Connor',
        'message': 'Your courses helped me land a job at McDonalds!'
    },
    {
        'id': 35,
        'name': 'Sarah',
        'message': 'Never have I understood OOP until now.'
    },
    {
        'id': 43,
        'name': 'John',
        'message': 'I watched all 200 hours straight!'
    },
]
```

Don't worry too much about the data, it's just for learning!

We can pass this data to the template by passing it with a named parameter to ```render_template()```:

```python3
return render_template('index.html', testimonials=testimonials)
```

This will be reachable using the data variable inside of our template:

```html
<p>Here are some great things people have said:</p>
{% for testimonial in testimonials %}
    <p>{{testimonial.message}}</p>
    <p>- {{testimonial.name}}</p>
{% endfor %}
```

# Variables in GET route

Next up, I want to be able to select any of these testimonials to bring up a dedicated page for the selected testimonial. This will help us learn a few important concepts:

1. How a template can be used for different data(the same template will be used for any of the testimonials),
1. How pages link together dynamically,
1. How to parameterize a route to take an ID or other data.

```python3
@app.route('/<id>')
def show_testimonial(id):

    for testimonial in testimonials:
        if testimonial.get('id') == int(id):
            return render_template('testimonial.html', testimonial=testimonial)
```

Don't forget to create a new template for an individual testimonial. We'll copy and paste ```index.html``` and change it up some. For example, we can put the individual testimonial in the title.

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{testimonial.name}}'s testimonial</title>
</head>
<body>
<h1>Testimonial</h1>
<p>This is what {{testimonial.name}} has to say:</p>
    <p>{{testimonial.message}}</p>
<a href='/'><button>view all</button></a>
</body>
</html>
```

We can access this by passing in an ID to the URL:

```
http://localhost:5000/35
```

We can create links to each individual testimonial from the homepage at the end of each iteration of the for loop:

```
<a href='/{{testimonial.id}}'><button>More info</button></a>
```

## Potential Modifications

It's important to understand that there are many different ways to design an application. The most important thing is to be consistent.

For example, you may not want to take the ID as a query parameter in the URL, something like

```
http://localhost:5000/?id=35
```

In this situation, we would still visit the homepage but have the ability to pass in additional data. in the ```/``` route you would check for a submitted query parameter and build a condition on whether or not it exists and is valid. I personally prefer the way we have it.

Here's an example on [how to retrieve query parameters](https://stackoverflow.com/questions/11774265/how-do-you-get-a-query-string-on-flask).

Another variation would be to have everything in a designated subdirectory of the URL structure, such as:


```http://localhost:5000/testimonials```

and:

```http://localhost:5000/testimonials/35```

This is easy! Just change the ```@app.route()```s to the path you want!

For example, let's change it to ```@app.route('/testimonials')``` and ```@app.route('/testimonials/<id>')```

Visiting ```http://localhost:5000/testimonials``` and ```http://localhost:5000/testimonials/35``` work now, but the ```<a>``` URL's are broken! From ```/testimonials```, the ```<a>``` URL doesn't exist, and from ```/testimonials/<id>``` the ```<a>``` URL goes back to the homepage, which also no longer exists.

Yikes! In the next lesson we'll talk about how to fix this in an elegant way and set up our template structure in a way that is more scalable.

