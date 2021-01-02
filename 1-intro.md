# Introduction to Flask and Python Web Development

## What is Flask?
Flask is a framework to build web applications in Python. This is to build the back end of a website, so you will use flask to do data processing and handle requests, work with databases, send responses, etc.

To understand where Flask fits into the larger picture, you will still use front end technologies such as JavaScript, and you will still use database technologies like MySQL, MongoDB, or Postgres. And you will still need a web server, such as NGINX.

Flask competes with other frameworks such as Django, and FastAPI.

Now fortunately, if you’re new to back end application development, a lot of the concepts for Flask are going to be very similar to other frameworks for Django. I wouldn’t worry too much about which one to choose or start with. And if you’re watching this, then I’m assuming you’re pretty convinced Flask may be the way to go!

## A Simple Architecture Example
A simple application architecture that is going to be seen across most web frameworks is that you have a function, let’s say it’s called ```hello()```, that is going to handle certain web requests.

The next piece in this architecture is that we associate this function with a particular route, likely ```/hello```. This is where the web server comes in. The web server’s job is to take requests over the internet (HTTP).

The ```hello()``` function will be hit, some processing will take place, and then something is returned to the client (the computer requesting this route). This will always be JavaScript, HTML, CSS, images, and other static content. The client never receives Python code as it is strictly a server side language.

Another way to think about it is that we are using Python and Flask to create dynamic websites that generate HTML to be sent to the client's browser.

You can learn more about web servers [here](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/What_is_a_web_server).

## Why Flask?

Ultimately I think most of what we are going to do could be done with other web frameworks as well, so it ultimately comes down to personal preference or what you are required to use for a team project.

I first started learning Django, but ended up switching to Flask because it is so incredibly easy to get started.

Even the [Flask documentation](https://flask.palletsprojects.com/en/1.1.x/quickstart/#quickstart) gives an example ```hello world``` application that is only 5 lines long:

```python3
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```
Obviously as our application needs grow it becomes a bit more complicated. Flask was good for me because it allowed me to build an application one line at a time and understand a lot of the pieces as I went instead of just generating a website template where I had no idea how to update or configure anything. Plus, the skills needed to understand flask also apply for much of Django as well.

## Two Approaches of Development

When I was planning this series I was struggling because I wanted to keep the content very useful and also very thin. This is a challenge with web development because it's such a **huge** topic.

The challenge of web development is that with what seems like a million ways of doing things, it's hard to know which way is best.

Because of this, I'm going to share two main approaches of web development, and what approach I suggest.

## Approach 1: Templates

As soon as you start learning Flask, you'll start learning about templates. Templates are sort of like HTML, but with added code components built in. These templates get rendered server-side and only pure HTML is sent to the browser.

So, we might have a template ```index.html```:

```
{% for user in users %}
        <p>{{user.name}}</p>
{% endfor %}
```

Notice how it is like HTML mixed with code. And the end result sent to the user would just be a series of paragraphs such as:

```
<p>Caleb</p>
<p>Margot</p>
<p>Elizabeth</p>
```
The exact syntax required for the template is defined by a template engine. The template engine most used in Flask is known as [**Jinja**](https://jinja.palletsprojects.com/en/2.11.x/).

### The Problem with Templates

Templates make it very easy to start making dynamic applications, but the problem is that when we have two distinct things being mixed together. Python code, and HTML structure. For many applications this will be just fine, but when we use templates:

1. Our code is tightly coupled (logic and presentation are tied)
2. We cannot easily create single page applications as every page requires a template
3. We limit our application to only the web

## Approach 2: APIs

The second main approach to developing for the web is to only give the **data** to the client and not any HTML structure or CSS styling. This data will typically be sent in a [**JSON**](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON) format. Our previous example would instead send:

```
{
    "users": ["Caleb", "Margot", "Elizabeth"]
}
```

An architecture like this is known as an API (application programming interface) and allows for apps to communicate with one another as long as a standard communication means and language or notation is followed (in this case, HTTP is the means and JSON is the notation)

We could then write an application to read this data and present it nicely to the user. The end result? An application split into two main sections: the back end and the front end. This course is going to focus on the back end, so building the API, but you could easily integrate with the API.

I believe this is the best approach to web development.

1. We have our data processing separate from our presentation
2. We can create single page apps that do not require a a refresh to see new data.
3. We can consume this single API from numerous sources and maintain consistent behavior across apps. Example front end applications could include a React web application, a Python console application, a Swift mobile application, or a C++ desktop application.

Although potentially more scalable, the downside to APIs is that it introduces more challenging programming concepts and architectures. Now not only do we need to worry about the Python and our templates, but likely JavaScript, Asynchronous callbacks, security concerns, and more.

In fact, you can build out an entire API and still not have a beautiful front end application to see your code in action. However, you can test all the API functionality using tools like Postman or you can create a client application to consume the API.

## Our approach

For simplicity and sanity sake, we will be focusing on building our application as an API. We will of course take a brief look at templates and how they work, but we will quickly switch over to sending JSON to the client instead of HTML.

## Conclusion

If this was all review for you, great!

If you're new to web development, this can all be very overwhelming. Fortunately, you don't necessarily have to be an expert at everything. Instead, you can figure out a system and structure that works for you and excel at it.

In fact, in many development companies you will focus on just one piece of a larger web application. Knowing Python back end development but not necessarily how to build a full stack application from beginning to end is actually okay.

Since you're watching this course, I recommend you first become proficient in back end development and once you have that down you can experiment with creating front end software. But if you try to do it all at once it can be overwhelming.




