# A Brief but Important Overview...part 2?

Previously we talked about the basics of how a Python web development project will be setup as well as the most common request types.

Now, I want to talk about some more concepts.

## Responses

With each request we get can respond with data. such as a response status code, headers, and a body.

If you ```GET``` a list of users, you'll likely get that in the response body.

if instead you ```POST``` a new user, you might get something like the new users ID, or a confirmation message saying that the user was created successfully.

In addition to the body, there are **status codes**.

When we open our web browser and make a request to a web server, the web server responds. This is typically a status code ```200 OK```, which means everything worked fine, but there are a bunch of other [status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status).

Another common one you might see is ```404 NOT FOUND```, which means the particular page you are looking for cannot be found.

A third popular one is ```500 INTERNAL SERVER ERROR```, which means there has been an error on the server and the page can't be delivered. Something with the web application isn't working, but we weren't given any details.

As the web developers, we have the ability to choose the content type that is given back to the client in response to a request. This is defined in a header called **Content-Type**.

Two of the most common content types you will see from web apps are ```text/html``` and ```application/json```. The first is the typical response if the client is requesting a web page and the second is if the client is requesting JSON.

![](./img/content-type.png)

In flask, we can return HTML by invoking ```render_template()``` for a route and then returning the result. That is if you're using templates as we spoke about in the first lesson.

alternatively, we can return JSON by invoking ```jsonify()``` with some data and returning the result. This is what we'll be using to build our API and return JSON.

We'll get into the details later.

## Databases

Databases give us the ability to persist data between website visits.

Python actually ships with a database, **SQLite**, which is extremely convenient! Python has made it easier than ever to start working with databases. It takes like .02 seconds to get started.

SQLite is amazing because it's very light weight and can easily ship with your software without requiring a dedicated database server. SQLite is said to be an embedded database and it just ships with your Python code.

That being said, SQLite is not typically the database of choice for web applications, for two reasons:

1. You don't need an embedded database with a web application, because everyone using your web application visits in a centralized location.
2. SQLite is not designed to be massively scalable and may introduce some limitations into your application.

That being said, SQLite is not as limited as people make it seem. Even the [SQLite website](https://sqlite.org/whentouse.html) handles millions of requests a month while being powered by SQLite.

Regardless of your opinion on SQLite, I don't believe it's the best intended tool for the job or what you would expect to find in a real world project. Thus, we are going to use Postgres.

Postgres is an open source relational database and in terms of use is going to be pretty much the same as MySQL, SQL Server, or any other relational database. If you're looking for an unstructured database, you could try MongoDB.

## SQLAlchemy and ORMs

SQLAlchemy is an example of an **ORM** (object relational mapper). The purpose of SQLAlchemy is to abstract away SQL from the developer so we can focus on application development and not database development.

AN ORM will essentially take a bunch of rows of a database table and convert them into a collection of objects that match the database structure.

There is a SQLAlchemy module specifically for Flask called [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/) (shocking, right?). Instead of writing SQL (which may change from database to database), you can grab all the rows of a database table with a Python statement as simple as ```User.query.all()```.

## Authorization

As we build out our API, we may want restrict certain access to certain people. Maybe you want to build out an admin panel with more capabilities, or only allow registered users to use the API. This is all included in **authentication** and **authorization**. Authentication is confirming your identity, and authorization is confirming your permission levels.

For this step, we will be using **JSON web tokens** (JWTs). These are digitally signed JSON data packets called **tokens**.

Two fantastic introductions to JWT are these articles from [JWT](https://jwt.io/introduction/) and [AuthO](https://auth0.com/blog/stateless-auth-for-stateful-minds).

The workflow described from the JWT website is that the user will login using user credentials for authentication. Then, the user will send a JSON web token with each request to confirm they are authorized to work with a particular API endpoint. For authorization, you can use a service like AuthO, or use your own database for user information.

With JSON web tokens, no session info is stored on the server. This means that the user is essentially "logged in" as long as they have their JWT. We don't keep track of this information on the server.

We may not want users logged in for eternity. Think of a JWT as a keycard that carries with it various permissions. If a keycard was misplaced, we would want to mark it as invalid. We can do a few things:

1. We could put an expiration on the JWT so that each JWT expires after a certain length of time. This is the easiest but doesn't give us the ability to invalidate specific JWTs.
2. We could expire all tokens forcing everyone to login again. This would invalidate all tokens and fix the previous problem, but is inconvenient and not scalable. This is done by changing the secret, which we'll talk about in the next concept on secrets.
3. Keep track of invalid tokens (a blacklist / denylist). This could potentially defeat the purpose of using JWTs because now we have to check against a list of tokens, and one of the main benefits of JWTs is that we don't have to do it. But there are a lot of resources out there to make this possible without adding much overhead.


## Secrets

For various things in development there are things that should be secret. Database connection strings, encryption keys, etc. The two main things we are going to work with are:

1. A connection string to the database
2. A signing key used for JWTs

When you're starting out development, it's very common to hard code these in the application code. For experimentation, this is usually OK, but it's really recommended to store these outside of the source code, especially if you plan on open sourcing the project.

If you check in code to a public GitHub repository that has database connection strings or secrets used for digital signatures, then the security of your application has been compromised.

In order to store your code publicly in GitHub while also having functional code that uses things that should be secret, we can store those in a different location and read them from GitHub.

Locally, we could literally just store them in a text file and read the text file. Or we can store them in a certain Python file, import the file into our project. When storing secrets in a separate file, make sure you add this file to the .gitignore so that we don't accidentally check it in to source control. It should stay out of the repository at all times. And if you accidentally commit it in to the repository but then remove it, it's already to late as the history of the repository is in tact. If this happens, you'll need to refresh all of your secrets and update the credentials to the database.

In the case of JWTs, it's perfectly fine to change the signing key. In fact, it's probably recommended to change it every now and again. The only consequence is that all tokens will be invalidated, forcing everyone to login to get new tokens. This is actually how you force a re-login across the system for everybody, if you're concerned about unauthorized access.

The ultimate solution for secrets is to use environment variables. Flask has examples on how to retrieve info [from files](https://flask.palletsprojects.com/en/1.1.x/config/#configuring-from-files) and also from [environment variables](https://flask.palletsprojects.com/en/1.1.x/config/#configuring-from-environment-variables). We'll talk about this more when the time comes.
