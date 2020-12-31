# A Brief but Important Update...part 2?

Previously we talked about the basics of how a Python web development project will be setup as well as the most common request types.

Now, I want to talk about some more concepts.

## Responses

With each request we get can respond with data. such as a response status code, headers, and a body.

If you ```GET``` a list of users, you'll likely get that in the response body.

if instead you ```POST``` a new user, you might get something like the new users ID, or a confirmation message saying that the user was created succesfully.

In addition to the body, there are **status codes**.

When we open our web browser and make a request to a web server, the web server responds. This is typically a status code ```200 OK```, which means everything worked fine, but there are a bunch of other [status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status).

Another common one you might see is ```404 NOT FOUND```, which means the particular page you are looking for cannot be found.

A third popular one is ```500 INTERNAL SERVER ERROR```, which means there has been an error on the server and the page can't be delivered. Something with the web application isn't working, but we weren't given any details.

As the web developers, we have the abilty to choose the content type that is given back to the client in resposne to a request. This is defined in a header called **Content-Type**.

Two of the most common content types you will see from web apps are ```text/html``` and ```application/json```. The first is the typical response if the client is requesting a web page and the second is if the client is requesting JSON.

![](./img/content-type.png)

In flask, we can return HTML by invoking ```render_template()``` for a route and then returning the result. That is if you're using templates as we spoke about in the first lesson.

alternatively, we can return JSON by invoking ```jsonify()``` with some data and returning the result. This is what we'll be using to build our API and return JSON.

## Databases

## ORMs

## Authorization

## Secrets
1. response (template or JSON-dictionary automatic)
1. database, sqlite, postgres
1. sqlalchemy, ORM, and models
1. Authorization and authentication
1. secrets / hiding connection strings
