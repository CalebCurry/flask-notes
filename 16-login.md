## Using JWTs with a Login System

```
pip install flask-bcrypt
pip install flask-jwt-extended
pip freeze > requirements.txt
```

Inside of ```__init__.py```:
```
from flask_jwt_extended import JWTManager
from flask_bcrypt import Bcrypt

app = Flask(__name__)
bcrypt = Bcrypt(app)

app.config['JWT_SECRET_KEY'] = 'super-secret'  # Change this!
jwt = JWTManager(app)
```

For the secret key, the [JWT homepage](https://jwt.io/) recommends a 256 bit secret. This translates to a 32 character string or a 64 digit hexadecimal number.

[Generate a secret key](https://www.allkeysgenerator.com/Random/Security-Encryption-Key-Generator.aspx). We will go with the hex box checked. Each hex digit coordinates to 4 binary digits. So a 64 digit hexadecimal number is a 256 bit binary number.

```
app.config['JWT_SECRET_KEY'] = '26452948404D635166546A576E5A7234753777217A25432A462D4A614E645267'
jwt = JWTManager(app)
```

Now, this is not as complicated as the database connection string. Whenever we want we can just replace this secret key with a new one and it will invalidate all JWTs that have been sent out.

We can change this and set this as an environment variable, which we will do later to showcase how this works.

## User Model

It's recommended to set the password size to 60, which is the length of the hash from bcrypt.

Because we have ```db.create_all()``` in our ```__init__.py``` file, it will create this table for us automatically.
```
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(30), unique=True, nullable=False)
    password = db.Column(db.String(60), nullable=False)
```

Notice that there is nothing about hashing here. It's all done in application code.

## Creating Users and Logging In
Now, let's take a look at ```routes.py```:
```
from drink_ratings import bcrypt, jwt
from flask_jwt_extended import jwt_required, create_access_token

```

##Basic login concepts

We can see how this works by first ignoring the password. Assume only a username is needed.
```
@app.route('/api/login', methods=['POST'])
def login():
    username = request.get_json().get('username')

    user = User.query.filter_by(username=username).first()

    return user.username
```

Now, we just need to add some additional functionality to get the password, compare it to the database, and issue a JWT.

```
@app.route('/api/login', methods=['POST'])
def login():
    username = request.get_json().get('username')
    candidate = request.get_json().get('password')

    if not username or not candidate:
        return {'error': 'username and password data required'}

    user = User.query.filter_by(username=username).first()

    if not user:
        return {'error': 'invalid username or password'}, 400
    # return jsonify(bcrypt.check_password_hash(user.password, candidate)) #helpful to see

    if bcrypt.check_password_hash(user.password, candidate):
        access_token = create_access_token(
            identity={'username': username, 'role': 'admin'})
        return {'access_token': access_token}, 200

    else:
        return {'error': 'invalid Username or password'}, 400
```

This will give back a JWT like so:

```
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpYXQiOjE2MTEyNTEwOTYsIm5iZiI6MTYxMTI1MTA5NiwianRpIjoiMTgyY2IyMzgtYTIwMy00MjMyLTk0ZjgtMDFjOGI3NjU1YmEwIiwiZXhwIjoxNjExMjUxOTk2LCJpZGVudGl0eSI6eyJ1c2VybmFtZSI6ImNhbGViIiwicm9sZSI6ImFkbWluIn0sImZyZXNoIjpmYWxzZSwidHlwZSI6ImFjY2VzcyJ9.t0J-toWbO4n_4NFOjwmHuFu-ySA_XjqSHB2f-muBI8A"
}
```

Copy the value of ```access_token``` (without the quotes) and paste it in the Authorization tab of Postman with ```Bearer Token``` as the ```TYPE```.

When building out a client application, you can write code to do this automatically.

Now, add the decorator to our desired routes:

```
@app.route('/api/testimonials/<id>', methods=['DELETE'])
@jwt_required

...


@app.route('/api/testimonials/<id>', methods=['POST', 'PUT'])
@jwt_required
```
Now, hitting these routes work the same way as expected.

If you remove the authorization in Postman, you get:
```
{
    "msg": "Missing Authorization Header"
}
```

You want to put that back in. At some point you might find that your JWT expires:

```
{
    "msg": "Token has expired"
}
```

When this happens, you'll need to log back in to get a new token.

This is a huge pain. It seems like you're only logged in for .5 seconds. We can fix this by configuring the JWT settings on the server. Here is a page to see [all the settings](https://flask-jwt-extended.readthedocs.io/en/stable/options/#configuration-options). This is in the ```__init__.py``` file and can take an integer for the number of seconds till expiration.

```
app.config['JWT_ACCESS_TOKEN_EXPIRES'] = 604800
jwt = JWTManager(app)
```

You also have the ability to use refresh tokens, which will give you a new token with a new expiration. This can be used if you want a log out after, say, 10 minutes of inactivity.

## Environment Variable

Now what I want to do is remove this secret key from our code as nobody should be able to see it.

We're going to move it to an environment variable just like we did with the database info.

Before we set an environment variable, I want to get a valid token and confirm that it is working. When we change the secret key, all tokens will be invalidated. This is the surest way to log everyone out. Change the secret key.

```

```
**NOTE** when you export a new environment variable, you'll need to launch the flask app in a new terminal window for it to catch this change.

```
export JWT_SECRET_KEY=58703273357538782F413F4428472B4B6250655368566D597133743677397924
```

Again, this is a new secret key value.

(or the windows equivalent with ```set```)

You can of course add this to your zprofile as we did with the database info.

Now, when we hit the same path that was working, we get this:

```
{
    "msg": "Signature verification failed"
}
```

This is essentially what would happen if someone tried to fake a JWT from the client. They would most likely have an incorrect signing key and it would be rejected.

We can of course login again to get a new, valid JWT.

## Deployment

For the deployment server, we will set the same environment variable. We talked about how to do this in an earlier episode.

Now, just replace the address from localhost to the server URL and confirm everything is working just as expected!

You'll need to create a new account to login, as the data on the development server does not match the deployment server.


## Conclusion

We have come a long ways. At this point you've learned the foundation to learn on your own. You should be able to do research and experiment to build the software that you need using JWTs. It can take a lot more time when you're not following a video tutorial, but it can be a lot more satisfying!

Three things you need to do:

1. Research security. I'm not liable for any security concerns in your applications and this course is by no means comprehensive. It's ultimately your responsibility to research potential threats.
1. Research costs. Again, computing can be expensive. Make sure you understand your budget and expenses! I am not liable for any unexpected expenses.
1. API consumption. Now that you have an API, you should learn the proper way to consume the API from a client application and how to do more advanced things in the API. So much to learn!

Thanks for following along!






