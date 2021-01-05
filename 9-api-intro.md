# Web Architecture

We've already talked about the basics of APIs, so not all of this should be new.

in this video I wanted to discuss the approach to building an API.

First thing is that you should already have a decent knowledge of how we set up a web page using templates. With this approach the data is tightly coupled with the HTML, making it more challenging to turn this in to a scalable application.

## Stateless

We want to make our API as close as possible to stateless.

What this means is that each invocation to the API has no memory of previous invocations.

Often times when you build a website using templates and go all the way to creating user accounts and login capabilities, you use sessions and store information on the server.

This approach *does* work, but the web server essentially has a memory of who you are.

You can read more about sessions on the Flask quickstart, if interested.

We really want each component of our applications to be self contained. This is a concept known as separation of concerns and dictates the way we design pretty much everything.

Even within a Flask application, we can follow an **MVC-like structure** (model, view, controller). Once we implement SQLAlchemy we will have defined our **model**, we use a template for a **view**, and routes for our **controller**. This way, each individual section is partially isolated from the others.

## Reintroduce Yourself

Imagine you went out for a drink and had to present your ID. Then just a minute later you order another drink and are required to show you ID again, to the same person.

Maybe this person has memory loss, or maybe taking statelessness a little too seriously.

This is exactly how it works with our API and where JWTs come in. Each invocation of the API requires us to reintroduce ourselves and prove that we are authorized to reach a particular route. Even if we just made a request 1 second ago, the web server has no concept of who you are the next time you make an invocation.

I think all analogies fail at some point, but let's take it one step further. Eventually, your ID is going to expire and you have to go get a new one. The same thing can happen with JWTs when you have to get re-authenticated to receive a new JWT. This is based on the the expiration time of JWTs.

Back to the example of the bartender asking for your ID twice. It seems odd from a person-to-person standpoint, but makes sense for computer software. If the bar's protocol was to ask for identification anytime someone ordered a drink, we could easily swap out bartenders without ever having to worry about accidentally giving a drink to someone who was underage. Probably not very practical in the human world, but for computers, sending the authorization with your request is fairly easy, and it allows for the back end to swap out web servers entirely because there is nothing significant stored on each individual web server.

If we follow this pattern enough, we can consider services like **serverless computing**, which allows you to pay for cloud computing by the request, instead of by the server.

## API methods

We've talked about GET and POST methods. There are two more we are going to discuss as well as some important convention decisions that we need to talk about. We use the term **resource** which generally means an object or record in the database. In our case, a resource would be a testimonial for our testimonial website.

```GET``` - Designed to retrieve a resource

```POST``` - Used to add a resource

```PUT``` - Used to replace a resource

```DELETE``` - Used to delete a resource

These coordinate with the **CRUD** (create, read, update, delete) capabilities needed to work with a database.

There is not 100% agreement on how an API should be set up. A common argument is how to use POST and PUT. Some people choose to use POST for everything, some people say you should use PUT to add a resource, etc.

Plus, there is not universal agreement on what you should return from these API calls either.

There is one thing everybody agrees on, though. And that is that PUT requests need to be **idempotent**. This means that invoking a PUT API request 1 time or 100 times does the same thing.

This is why PUT is ideal for replacing a resource with a new updated version.

Say somebody left a testimonial on our website and they had a typo. They decided to edit the testimonial and fix the spelling mistake. When they click the update button they hit it a few times. Basically what this will do is replace the original with the updated version the first click, and then replace the updated version with the same exact updated version on the second and following clicks. The ID for the testimonial will be exactly the same, so it doesn't really matter how many times we replace it.

This is in contrast to POST, which is not conventionally idempotent. This is often used to add a resource, so hitting submit to add a new testimonial a few times will actually work the first time, and then either insert a duplicate the second time or cause a database error. Because the first and following invocations were not the same, it's not idempotent.

This is why when you're filling out some web page and you hit submit twice, you get a browser warning saying that you're about to submit the form a second time which can cause issues.

## routes and returns

I tend to follow these conventions for the routes:

```
GET resources - read all resources
POST resources - add one resource

GET resources/<id> - read one resource
PUT/POST resources/<id> - update one resource
DELETE resources/<id> - remove one resource

```

In this situation, you could use PUT or POST to update a particular record.

I was largely inspired to structure my routes this way based on the Stack Exchange API and URL structure.

For example, ```https://stackoverflow.com/questions/42045479/mysql-date-before-year-1000-allowed```

This is essentially a GET request to resource/<id> (where the resource is questions and the id is 42045479).

As for return data, I don't think there are any standards, but some common conventions. To explain what I mean, let's say you update a resource. You could:
1. return the count of rows updates (it should be 1)
1. return the resource with the added ```ID``` column retrieved from the database
1. just return something like ```True```, saying it worked.

Don't worry too much about it starting out. Just pick a convention and stick to it across your API code.
