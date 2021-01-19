# Databases and ORMs

Today we're going to focus on learning databases in a bit more depth.

Now, in general, the amount of knowledge required about SQL and databases is simplified thanks to **ORM**s (object relational mappers). The ORM we will be using is known as [**SQLAlchemy**](https://flask-sqlalchemy.palletsprojects.com/en/2.x/) and it works with all the [major relational databases](https://www.sqlalchemy.org/features.html#:~:text=Supported%20Databases,are%20published%20as%20external%20projects.).


That being said, it helps to have a decent understanding of databases because they are such a vital component of our application.

On top of that, they're one of the most dangerous parts of our application in terms of consequences if not done correctly. Data breaches are a **very** serious problem and there are a lot of security issues when it comes to databases.

## A database Crash Course

Data is stored in a relational database in tables. This is where the word **relational** comes from, **relations** (fancy word for table).

These tables have columns and rows. Columns define the attributes each row should have, and a row is an actual example of data we want to store that has values for each of these attributes.

```Extensions```

| ID | name      | Phone |
| -- | --------- | ----- |
| 17 | Caleb     | 1337  |
| 89 | Erin      | 1338  |

Every column is defined with a data type, describing what kind of data is allowed for that attribute. For example, we might have a string and a number.

We'll be using PostgreSQL. The entire list of data types can be found [here](https://www.postgresql.org/docs/9.5/datatype.html) and here is a more user-friendly tutorial on [Postgres data types](https://www.postgresqltutorial.com/postgresql-data-types/), however before studying all the PostgreSQL data types, read the next section.

## Generic Types

Because an ORM is designed to abstract away the SQL and database layer for application developers, we want to be careful that we're not using data types that are specific to only one database provider.

For example, PostgreSQL has an ```array``` data type that might not be available in every other database provider.

Instead of using a specific data type like this, we can use more generic or general data types that are part of the SQL standard and are available in all major SQL databases.

For SQLAlchemy, The entire list of [generic types](https://docs.sqlalchemy.org/en/13/core/type_basics.html#generic-types) is quite large, but gives you a wide variety of options to choose from. The most important thing to realize here is:
> SQLAlchemy will choose the best database column type available on the target database when issuing a CREATE TABLE statement.

This means as long as we use the generic types in SQLAlchemy, we can easily swap out underlying databases.

[Vendor specific types](https://docs.sqlalchemy.org/en/13/core/type_basics.html#types-sqlstandard) are still available if needed, but will likely need altered if switching databases.

## Other Attributes

**Primary key** - We set one column as the primary key. This is a column that is required and unique. Typically this is a number that will increment automatically.

**null** - In addition to defining the data type, we can say whether the column allows **nulls** (no value for that column). To do this, you would say nullable=True.

**Foreign key** - Not only can we have one table, we can have multiple tables and connect them using foreign keys. A foreign key column only allows values that match with another column (usually) in another table.

For example, we might have:

| ID | Employee ID | Phone |
| -- | ----------- | ----- |
| 17 | 3           | 1337  |
| 89 | 12          | 1338  |

Where the values for Employee ID refer to a row in an ```Employees``` table.

When working with an ORM, you'll be able to easily access related tables with a nested object. In this scenario if you wanted to grab employee info for a particular phone extension, it would look something like this:

```
ext = Extension.query.get(17)
name = ext.employee.name
```

## Models

How do these tables get represented in code? Using Python classes, one class for each table.

These classes are often called **models** and are the M in MVC.

Each row that we work with in the table becomes an object (an instance of the Python class).

We define our database structure by creating these Python classes and we make the class inherit from ```db.model```.

When we do this, SQL code is generated that represents the database structure.

Not only can we define the database structure, but changes in these structures can be tracked in **database migrations**, allowing us to easily change the database tables and migrate from one computer to another from source control.

When we check our code in to source control, the entire database structure is defined in Python code, meaning there is no need to transfer handwritten SQL scripts or database files around with the project.

We will see how this works soon.

## Database Hosting

As you've probably noticed by now, I support using cloud providers whenever possible to outsource responsibility, and of course many cloud providers have the ability to host database servers.

With AWS, you have a few options using [Amazon RDS](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/AWSHowTo.RDS.html) (relational database service).

1. Instance associated with Elastic Beanstalk environment
1. Dedicated instance outside of Elastic Beanstalk

The associated instance is an ideal setup for testing environments and learning, but it's recommended that you go with a dedicated instance for production.

The primary difference is that with the integrated database instance, it is tied to the life cycle of your application (deleting the environment deletes the database) and the database is isolated to just that application.

You can start with an associated instance and [decouple it from the environment](https://aws.amazon.com/premiumsupport/knowledge-center/decouple-rds-from-beanstalk/) later, but it might be easiest to just start with the dedicated instance.

Just watch the pricing! There is a free tier for 12 months if you're just signing up but this is not a forever free service. If you need something free you can practice on your local machine or find free database hosting out there.

## Local Development

In addition to deploying a database to AWS, we will have a development database instance. The data between these will not be shared but the structure will be synced thanks to the migrations we create. So if we setup a structure in our local code, when the code is deployed the remote database structure will be updated automatically.

You could just use the connection string for the remote database in AWS, but I think it's best to keep the deployment database and development database separate.

You can install PostgreSQL locally for development. Another option is you could have a development database in AWS deployed, then you do not have to install anything locally.

Enough chat though, let's just get started and try to get this all working!






