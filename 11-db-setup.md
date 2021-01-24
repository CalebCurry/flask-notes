# Database Setup

[Reference](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/rds-external-defaultvpc.html)

We will be deploying two databases to Amazon Web Services to store our data. You could also create one of the databases locally, but I'll be using AWS for consistency.

Get started with that [here](https://console.aws.amazon.com/rds/home).

When you create a database, you're given the option to set the creation method to ```Easy create``` but this is not recommended when connecting to the database with elastic beanstalk.


You'll want to make sure that you select free tier with **```db.t2.micro```** as the server size. Please, don't mistakingly go for the production database (unless necessary) and pay a significant amount of money.

You can get more info on the free tier and also check out the AWS RDS pricing to see [what it could cost](http://aws.amazon.com/rds/pricing).


Now the important thing to understand is that the more complex the connection info, the less likely it is for someone to be able to guess the info needed to connect to your database. That being said, you want to make sure you have it memorized or have it in a safe location.

1. For the development database, we will keep the security fairly loose. Obviously we don't just want to grant access to the entire world, but we also don't want to make things overly difficult for ourselves.
1. For the production database, we will make it as secure as possible while only allowing access from our EB (Elastic Beanstalk) application.

## Setup

Let's build things out in this order:

1. Deploy development database
1. Connect to development database locally and confirm connection is working
1. Connect to development database from our deployment server by matching environment variables
1. Deploy production database with tighter security
1. Connect to production database from deployment by updating environment variable values on the production server.

In this setup process we:

1. Have a progressive build-up instead of launching everything at once and hoping it works
1. Have a nice way to know if we run in to any issues whether it is coming from the way we connect to the database (this will come up when we connect to our development database) or our security settings on the production database (development database works from EB, deployment does not). Maybe once you know what you're doing you don't have to worry about being this granular.

For setup, I suggest that you **auto generate a password**.

This will be shown to you once for you to save somewhere.

There are a few important pieces of information for when we want to connect to our database from code:

1. ```Database Port``` - This is defaulted to 5432 and is sometimes part of the connection string.
1. ```VPC Security Group``` - The rules the database adheres to, we will edit these and make sure we can connect from the right place. You can read more about [VPCs](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.html).
1. ```Public access``` - We will want to be set to ```yes```. If it's set to ```no```, AWS says this:

> RDS will not assign a public IP address to the database. Only Amazon EC2 instances and devices inside the VPC can connect to your database.

```No``` would be an appropriate option if we were creating a production database that we wanted to limit access to EC2.

## Changing Database Configuration

We will need to update the database instance VPC security groups. These define what incoming and outgoing traffic is allowed.

For our development database, let's make allow all incoming traffic to make things as easy as possible for us.

Edit the inbound rules and set ```Type``` to ```All traffic``` and set the ```Source``` to ```Anywhere```. This plus the database being publicly accessible will allow us to easily connect from our development environment.

Before moving on, you may want to take note of these things (from the ```Connectivity & security``` section on the AWS database overview):

1. hostname
1. port
1. db name
1. username
1. password

## Connecting to the database from our local machine

Whenever I'm working with a database locally for a decent period of time, I like to download a client application to make working with and seeing the database easier.

The default tool for PostreSQL is called [pgAdmin](https://www.pgadmin.org/). pgAdmin needs you to set a master password that is required to open the app. Don't forget it.

This allows us to easily check our connection before we try connecting in code which can require a bit more work sometimes.

Create a new server in pgAdmin and put all of your connection details. Confirm that you can connect. If not, check to make sure your database is publicly accessible and has open inbound rules for the VPC security group.

## Connecting from Python

In ```__init__.py```, **near the end right before importing the routes/views file**, add your connection string in this fashion
(exact connection string will depend on your db settings in AWS):

```python3
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://flaskpractice:SXR4Vt3MbFvMqhfDP3IA@flask-practice.cyr69gg58bpb.us-east-2.rds.amazonaws.com/postgres'
db = SQLAlchemy(app)
```

The general format should be:

```python3
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://username:password@endpoint/databasename'
db = SQLAlchemy(app)
```

We can check the connection via interactive mode (issue ```python3``` in the terminal while in the root directory and while having an activated virtual environment):

```python3
from testimonials import db
db.engine.connect()
```
```db.engine.connect()``` is not required to use the database but can be used to confirm the connection. If no errors are thrown, then you are good.

More than likely, you'll get an error like this:

> Is the server running on host "flask-practice.cyr69gg58bpb.us-east-2.rds.amazonaws.com" (172.31.22.107) and accepting
TCP/IP connections on port 5432?

To fix this, exit interactive mode with ```exit()``` and issue:
```
pip3 install psycopg2-binary
pip3 freeze > requirements.txt
```

This gets us the necessary binaries to work with Postgres.

Now, try connecting again from interactive mode and you shouldn't get any errors when you check the connection. That being said, we're not making any changes, so nothing actually happens to the database. but we can confirm that we have the connection.

## Reading the connection string from environment variables

Following [these variable naming conventions](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/rds-external-defaultvpc.html), we're going to break our connection string up into a bunch of values that we will read from environment variables. That way, we don't have our connection string hard coded.
```
export RDS_DB_NAME=postgres
export RDS_USERNAME=flaskpractice
export RDS_PASSWORD=SXR4Vt3MbFvMqhfDP3IA
export RDS_HOSTNAME=flask-practice.cyr69gg58bpb.us-east-2.rds.amazonaws.com
```

For the following code, you will want to ```import os```:
```python3
DB_NAME = os.environ.get('RDS_DB_NAME')
DB_USER = os.environ.get('RDS_USERNAME')
DB_PASSWORD = os.environ.get('RDS_PASSWORD')
DB_HOSTNAME = os.environ.get('RDS_HOSTNAME')
#DB_PORT = os.environ['RDS_PORT'],

app.config['SQLALCHEMY_DATABASE_URI'] = f'postgresql://{DB_USER}:{DB_PASSWORD}@{DB_HOSTNAME}/{DB_NAME}'
db = SQLAlchemy(app)
```

```python3
from testimonials import db
db.engine.connect()
```

At this point you should be connected using only environment variables.

The downside to using export to set environment variables is that they are temporary. When you start a new terminal session you have to reset the environment variables, just like how you set ```FLASK_APP``` to application. It's a pain.

Each operating system has a way to make these more permanent, which you can research if interested. This is certainly optional, so I'm not going to show it on each OS. On mac, we can put them in our zprofile profile. This used to be the bash profile but was changed to zprofile at some update by Apple.

Open up your user directory and hit ```command shift .``` to toggle hidden files and folders. Open up ```.zprofile```.
You can make the file if it doesn't exist.

Add our exports to the end of the file:

```
export FLASK_APP=application
export FLASK_ENV=development
export RDS_DB_NAME=postgres
export RDS_USERNAME=flaskpractice
export RDS_PASSWORD=SXR4Vt3MbFvMqhfDP3IA
export RDS_HOSTNAME=flask-practice.cyr69gg58bpb.us-east-2.rds.amazonaws.com
```

Keep in mind that this is system wide, so if you're working on multiple projects you'll need to use the export command in the terminal to replace these for that terminal session. These are now the defaults.

You can check this by opening a new terminal and using ```printenv```.

## What's next?

In the next lesson we will learn how to start using this database in our Python code and how to set up a deployment database for our deployment server. Stay tuned!


