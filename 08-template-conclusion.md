# Concluding with Templates

The goal of this lesson is to:

1. Setup a ```404 NOT FOUND``` page,
1. Setup dynamic links,
1. and build a layout for a consistent look and feel.

Let's not waste any time!

# 404 template

At the end of the previous lesson we left off with some broken URLs. I said we would talk about how to fix those. We're still going to do that, of course, but it would be nice if when we visited a non existent page we got a 404 page.

To do this, use the ```@app.errorhandler(404)``` decorator and create a function for this route just like we did all the other routes.

One difference here is that we're going to return two things. The first is the template, and the second is the 404 status code. We didn't put the status code for the others because the default is 200. We also take an argument for the function (in this case we called in error). This contains information about the error if you want to work with it or send it to the template to be displayed.

it's also good to know that we can create custom pages for [various status codes](https://flask.palletsprojects.com/en/1.1.x/patterns/errorpages/).

```python
@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html', error=error), 404
```

The template might look like this:

```html
<!DOCTYPE html>
<html>
<head>
    <title>404</title>
</head>
<body>
    <p>Page not found. Go to the <a href='/'>homepage?</a></p>
</body>
</html>
```

We'll want to make sure there is a homepage, of course. You can define the homepage and the ```/testimonials``` page to be the same thing by putting two ```@app.route()```s above the associated function. We'll also update the name to ```show_testimonials```:

```python3
@app.route('/')
@app.route('/testimonials')
def show_testimonials():
    return render_template('index.html', testimonials=testimonials)
```

At this point the home page should work and we just need to update the URLs to be more dynamic, which we will get to. First, we need to consider what happens when we request a testimonial that doesn't exist.

```http://localhost:5000/testimonials/5``` gives us an application error, but more realistically it should just go to the 404 page. To do this in Python code, we use the ```abort``` method, which needs imported from flask. ```from flask import abort```.

The new ```/testimonials/<id>``` will look like this:

```python3
@app.route('/testimonials/<id>')
def show_testimonial(id):

    for testimonial in testimonials:
        if testimonial.get('id') == int(id):
            return render_template('testimonial.html', testimonial=testimonial)
    abort(404)
```

The only difference being that we added ```abort(404)``` at the end. This works because if we iterate through the entire list of testimonials and we haven't found a match, we know the ID is not found.

Visiting an invalid URL like ```http://localhost:5000/testimonials/4``` will showcase this example because none of our testimonials have the ID of 4.

Next up, let's fix our broken URLs.

## Dynamic URLs

We will use a useful function ```url_for``` that is automatically available inside of templates. It is also usable in python files but you have to ```from flask import url_for```.

The first argument is the name of the function we want to hit. The fact that we put the function name instead of the route allows us to change the route to the function and this URL is still going to work.

The second argument (or additional arguments) are values for named parameters that we have as variables in the route, in our case the testimonial ID.

```html
<a href="{{url_for('show_testimonial', id=testimonial.id)}}"><button>More info</button></a>```
```

To showcase this, we can change the route for this function:

```python3
@app.route('/get/testimonial/<id>')
def show_testimonial(id):
...
```
Now, when we visit the homepage, the URLs are automatically updated to the correct location. Fantastic! We'll put it back to how it was (```@app.route('/testimonials/<id>')```), but this will be very helpful for building out URLs throughout our website.

Let's update the URLs also in the ```testimonial.html``` template.

We can also update the ```404.html``` template if we want, but I think an appropriate task of a 404 page is to always point to the root, so we'll leave it pointing to ```/``` in the ```<a>``` tag.

```testimonial.html``` on the other hand will now have this for the view all button:

```
<a href="{{url_for('show_testimonials')}}"><button>view all</button></a>
```

## Building a layout

At this point we have 3 templates. It's not too complicated right now, but that's because we our templates are very very basic, and we're not even using CSS or JavaScript.

When we visit a website in the real world, you'll often find that all of their pages have the same style and structure. We can do the same using a layout.

Let's create a ```layout.html``` file. Here we can build the HTML structure for our application and then have a small section that is filled up from the other templates that use this layout.

```html
<!doctype html>
<html>
<head>
    {% if title %}
         <title>{{title}}</title>
    {% else %}
        <title>Testimonials</title>
    {% endif %}
</head>
<body>
    <h1>Caleb's Testimonials</h1>
    {% block content %}

    {% endblock %}
</body>
</html>
```

```html
{% extends "layout.html" %}
{% block content %}
<p>Page not found. Go to the <a href='/'>homepage?</a></p>
{% endblock %}
```

We create a conditional for the title, if a ```title``` variable exists, we use that for the title. Otherwise, we default to ```Testimonials```. We also have an ```<h1>``` which you could remove if you don't want it on every page. Then, add an ```<h1>``` to any template directly. This is to just showcase how the template works. We can pass a title the same way we pass any data to a template, from the routes:

```python
@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html', error=e, title="Page Not Found"), 404
```
Anytime we want a page to have a specific title, we pass it in to the named parameter title for that route.

We can do the same for our other files:

```html
{% extends "layout.html" %}
{% block content %}
<p>Here are some great things people have said:</p>
{% for testimonial in testimonials %}
    <p>{{testimonial.message}}</p>
    <p>- {{testimonial.name}}</p>
    <a href="{{url_for('show_testimonial', id=testimonial.id)}}"><button>More info</button></a>
{% endfor %}
{% endblock %}
```

```html
{% extends "layout.html" %}
{% block content %}
<h1>Testimonial</h1>
<p>This is what {{testimonial.name}} has to say:</p>
    <p>{{testimonial.message}}</p>
<a href="{{url_for('show_testimonials')}}"><button>view all</button></a>
{% endblock %}
```

Now, we can update the entire website's look and feel from one file.

Let's use this setup to our advantage by linking a CSS file. We're not going to style our website but I'll show you how to get it all connected. You can also link JavaScript files in the layout.

```html
<link rel="stylesheet" type ="text/css" href="{{ url_for('static', filename='style.css')}}"/>
<script src="{{url_for('static', filename='script.js')}}" defer></script>
```

You have the option to use url_for here as well.

in the static folder, create a file ```style.css```, and make a really simple and obvious style:
```
*{
  background-color:red;
}
```

For this to show I have to do a hard refresh using ```command shift r```, beautiful, huh?

We can also make sure our JavaScript is wired up correctly, too:
```
alert("working")
```

Notice that this applies to all pages that extend this layout.

This CSS and JavaScript is a bit obnoxious, so we'll remove it for now.

## Conclusion

This concludes the basics of working with templates and building web pages without the use of an API. This should get you started, but there's still a lot more to know if we want create forms that modify data and so forth. In fact, we're still using dummy data. The next logical step would to replace this with an actual database.

We'll be doing a lot of the other concepts when we build out an API. If you want to build out the templates as we go, feel free. But overall, I think the API approach is best and just knowing the basics of templates is adequate.
