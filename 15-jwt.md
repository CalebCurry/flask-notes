# Intro to JWTs

This is not going to be a complete introduction to the concept of JSON web tokens because we've talked about them numerous times throughout this course. Specific information can be found in lesson three.

This will be a more technical introduction designed to help us understand how to use them, how they're structured, etc.

But real quick, let's explain what a JWT is (based off of [this introduction](https://jwt.io/introduction)).

A **JWT** (JSON web token) is a JSON object used to  transmit data.

It's digitally signed, allowing the server to confirm the data has not been altered. That being said, the data itself is not guaranteed to be secure by default, so you shouldn't use JWTs to transmit sensitive data unless you have some level of encryption.

JWTs are used to confirm a message came from the right person and that the message has not be altered.

If done correctly, JWTs can be used to create private API endpoints that require a user to login.

The way we will have it setup is that if we want a private end point that not just anyone in the world can access, the user needs a valid JWT. **In other words, they need to be logged in.**

In order to get a valid JWT, the user will send a username and password to a ```/login``` API endpoint.

If the login credentials are correct, the server will issue a new valid JWT to this user.

The login credentials can be checked against a ```users``` table
in a database.

## How is this set up in the backend?

We will be using Flask JWT Extended, a package that allows us to easily work with JWTs inside of Flask.

Just like we have some secret information to connect to the database (the connection string), we are also going to have a secret key (a long, random string) that we don't want hard coded.

[Flask JWT Extended](https://flask-jwt-extended.readthedocs.io/en/stable/basic_usage/) will give us two useful imports. The first being ```create_access_token```. This function will generate a new JWT to send to the client when they successfully login.

The second is ```jwt_required```, which is a decorator placed on any API endpoints that require a valid JWT.

## How Does this Work for the Client?

The client will make a request to a ```/login``` API endpoint (or whatever you decide to name it on the server), passing a username and password.

Assuming the login info is correct, ```/login``` will generate a token and send it back in the body of the response.

From the client side, we will take this token, which looks like a long string of characters, and include that in our future requests.

How exactly do we include it? In an **Bearer Token** Authorization header. You can do set this up super easy in Postman.

This authorization gets sent with our requests. Assuming we are trying to access an endpoint labeled with ```@jwt_required```, the server reads this bearer token to see if it's valid. It does this by confirming it against the same secret key used to create the JWT in the first place.

## How we will use JWTs

I want anyone to be able to add a testimonial through the API. However, I want editing and deleting abilities to be limited.

Obviously the capabilities are endless.

- We could make it such that testimonials are hidden until they are approved.
- We could make it so that users can edit their own testimonials but not others.
- We can make it so that only authorized users can post testimonials in the first place, etc.

## WHat does a JWT Consist of

A JWT consists of three parts:

1. header
1. payload
1. signature

These parts are separated by dots and hashed using the HS256 hashing algorithm (by default). This algorithm uses one signing key to sign and confirm data. The end result may look like this:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

The header is metadata stuff, we don't have to worry about that too much.

The payload is where the actual data is stored. Here we can store information about the user such as their username, access level, etc. Here is also where information like the lifetime of the token exists, etc. Not their password though or anything else sensitive.

Finally the signature. This is a combination of everything hashed together.

The secret being hashed with the data prevents tampering. This means that if someone altered the payload on the client side, they wouldn't be able to make an acceptable JWT without knowing the secret key.

This is a bit hard to understand without some examples that we'll be showing in the next lesson.

## How does Logging in Work?

To login we will have a table in our database to store our user information. But it's widely considered an awful practice to store passwords in plain text in the database. Because of this, we will be using bcrypt, which is in fact another hashing algorithm. You can use many algorithms, but working with bcrypt is super easy.

Basically, we will hash our account password when we insert the data in to the database. Then, when we want to login, we will hash the submitted password to see if it matches the hash in the database.

This way, no passwords are stored in the database and we remove this liability.

## What about Roles?

Right now we are approaching this app very black and white. You're either a visitor leaving a testimonial, or you're an admin who has additional privileges.

Instead, you can introduce roles in to your app. For example, maybe you have a basic user role, an editor role, and an admin role.

These roles can be defined in the database and actually included in the JWT. That's because from the server we can embed whatever we want in the payload of the JWT.

Then, on certain actions, you can check the payload for the role to see if the person has access. You can find more info on this setup in the [Flask JWT Extended documentation](https://flask-jwt-extended.readthedocs.io/en/stable/add_custom_data_claims/).

You'll often hear the attributes in the payload called **claims**.
So we can have a claim for the persons role given in the application.

In our application, just build logic around that role.

Up next we are going to get started with these tools. See you then!




