# Setting up a Deployment Database

In this lesson we are going to talk about two very important things.

1. How to start working with models on our local database
1. How to setup a deployment database for our deployment server

The initial deployment server structure will be generated form Python code so we don't have to worry about database migrations or SQL commands. The data in the development database will be for testing and development, where as the production database will be actual, real data.

## Basic Model

We are going to create our first database table and we do this through defining a model in Python. We will create a file ```models.py``` inside of our package directory (right next to ```__init__.py```). We can follow the example of [the documentation](https://flask-sqlalchemy.palletsprojects.com/en/2.x/quickstart/).

We will work with a Testimonial class, we can import that into our ```routes.py``` file now:

```python3
from testimonials import db
from testimonials.models import Testimonial
```

```python3
from testimonials import db
class Testimonial(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50), nullable=False)
    testimonial = db.Column(db.String(500), nullable=False)

    def __repr__(self):
        return f'{self.name} says {self.testimonial}'
```

In interactive mode, we can initialize our database:

```python3
from testimonials import db
db.create_all()
db.metadata.tables.keys()
from testimonials.models import Testimonial
testimonial1 = Testimonial(name='Claire', testimonial='saved my job')
db.session.add(testimonial1)
db.session.commit()
Testimonial.query.all()
```

If you're getting a warning from SQLAlchemy about tracking and deprecation, you can disable this by adding ```app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False``` above ```db = SQLAlchemy(app)```. This will be the default in the future. Read more [here](https://stackoverflow.com/questions/33738467/how-do-i-know-if-i-can-disable-sqlalchemy-track-modifications).

## Database Initialization code

We will add ```db.create_all()``` to our ```__init__.py``` file right after ```db = SQLAlchemy(app)```. ```db.create_all()``` can be invoked numerous times as it will only create tables that don't already exist. It's a really simple way to initialize your production database with tables that you need, but may be limited if you have a complex database that is regularly changing and in that situation you may want to use database migrations.

For this we will also want to import the models we want created right before our invocation to ```db.create_all()```:

```python3
app.config['SQLALCHEMY_DATABASE_URI'] = f'postgresql://{DB_USER}:{DB_PASSWORD}@{DB_HOSTNAME}/{DB_NAME}'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)
from testimonials.models import Testimonial
db.create_all()
```

I agree that all of the different imports and getting things connected is a huge pain and can be challenging. When I was first learning this I left out the import here, and ```db.create_all()``` wasn't creating any tables. I thought it was just broken, but turns out it just didn't see any tables to create. Where would we be without [StackOverflow](https://stackoverflow.com/questions/20744277/sqlalchemy-create-all-does-not-create-tables)?

## Basic API call

Confirm you have the correct imports ```from testimonials import db``` and ```from testimonials.models import Testimonial```, and write a basic GET request:

```python3
@app.route('/api/testimonials')
def get_testimonials():
    testimonials = Testimonial.query.all()
    return jsonify({'testimonials': testimonials})
```

This will give us an error by default. ```TypeError: Object of type Testimonial is not JSON serializable```

There are various solutions to this, but basically the issue is that we have a list and it's not immediately serializable. So we can iterate through each element and build out a dictionary, or we can use something that makes our life a bit easier.

## ```@dataclass``` Decorator

```python3
from dataclasses import dataclass

@dataclass
class Testimonial(db.Model):
    id: int
    name: str
    testimonial: str
```

Now, our request returns this response:
```
{
  "testimonials": [
    {
      "id": 1,
      "name": "Claire",
      "testimonial": "saved my job"
    }
  ]
}
```

We could skip the surrounding dictionary if we wanted by just invoking ```return jsonify(testimonials)```, but I think you're most likely going to want to have everything in a dictionary. Then, if you add, new keys, the overall structure stays the same and no consuming code needs updated.

Now that we have a basic function to give us some actual data from the database, we can try to get this all working on the deployment server.

## Set up the Deployment Database

We're going to create a database of the similar nature to our development database. One **very important note** is that I am still selecting the free / cheapest tier in AWS instead of selecting deployment database. If you're launching a legit app you can select deployment database, just prepare your wallet because it's not cheap. Be sure to look at the estimated costs before you just click stuff. The free tier is going to meet all of my needs.

When we are creating a production database we want to limit access.

The securest of them all is to make publicly available set to no. However, if you want to connect to your instance from pgAdmin, you may want to set this to yes and then just add your IP address.

When you create the database, you can create a new security group. This will later be added to the EB server to allow the app and database to communicate. Follow the [detailed guide here](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/rds-external-defaultvpc.html).

For the inbound rules you can set the ```Type``` to ```PostgreSQL``` and choose your own IP. This will allow you connect from your local machine using pgAdmin if the database is set to publicly accessible. You can also set an additional ```PostgreSQL``` type to ```sg-``` and the option to select your EB environment will show up for you to click.

You will then need to add this security group to your EB instance in the configuration.

## Environment variables

Earlier on we set some environment variables to connect to the database without a hard coded connection string. This works fine, but when we deploy our application, AWS has no knowledge of these environment variables. That's because **environment variables are tied to our environment**. That makes sense, but it's easy to forget. This can be one of the causes of "it works on my machine".

To have the production server use the same database, we just have to set environment variables on the server to match our local ones. But this is exactly where you can swap out the environment variables for new environment variables pointing to a new database. The result? Everything works the same on deployment but we're using a new database. It's like magic.

Environment variables are set in the ```environment properties``` of your server, and should match these names:

```
RDS_HOSTNAME
RDS_PORT
RDS_DB_NAME
RDS_USERNAME
RDS_PASSWORD
```

As a side note, for development you are given the option to tie a database to the life cycle of your EB application. This may make things simpler as it automatically sets everything up and configures the environment variables. We are just copying the names that are automatically created with an attached database but using values for a dedicated database.

## No Data

When we get everything set up and connect to our web page, we are not going to have the same data as our local development machine. This is normal and a good thing. Unfortunately though, we do not have any way to add data through our application (yet).

```
{"testimonials":[]}
```

We will talk about that in the next lesson.

## Potential Issues

My first time trying this I received a 500 Internal Server Error.

I checked the EB logs and saw this:

```
sqlalchemy.exc.OperationalError: (psycopg2.OperationalError) FATAL:  password authentication failed for user "None"
FATAL:  password authentication failed for user "None"
(Background on this error at: http://sqlalche.me/e/13/e3q8)
```

I accidentally missed the ```RDS_USERNAME``` environment variable :)

Again I received an issue:

```
sqlalchemy.exc.ProgrammingError: (psycopg2.errors.UndefinedTable) relation "testimonial" does not exist
LINE 2: FROM testimonial
^
[SQL: SELECT testimonial.id AS testimonial_id, testimonial.name AS testimonial_name, testimonial.testimonial AS testimonial_testimonial
FROM testimonial]
(Background on this error at: http://sqlalche.me/e/13/f405)
```

I forgot to initialize the database with ```db.create_all()``` in our ```__init__.py``` file as mentioned earlier.

I then received the same error because I forgot to import the models. You need to import the models you want initialized right before invoking ```db.create_all()```. Otherwise, it will not create any tables.

It's also helpful to connect using pgAdmin. That way you can see what's going on in the database a lot easier.

There are a million things to go wrong, so be patient and don't give up! Worse case scenario (I've been here before), reach out to AWS support or ask a question on StackOverflow.

