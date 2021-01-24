# Flask Large Application Setup

So far we created a very simple Flask application. This makes Flask very likable (it's easy to get started), but doesn't give us much room to build a larger application.

Because of this, Flask gives a recommended file structure for [larger applications](https://flask.palletsprojects.com/en/1.1.x/patterns/packages/).

You can reference this documentation if needed, but ours may be just a tad different. Specifically, I use the name ```application.py``` instead of ```setup.py``` and I use ```routes.py``` instead of ```views.py```. We also have the additional ```.venv``` directory, ```requirements.txt```, and our ```.gitignore``` file.

For reference, our end goal structure looks something like this:
```
/testimonials
    application.py
    requirements.txt
    .gitignore
    /.venv
    /testimonials
        __init__.py
        routes.py
        /static
            style.css
            ...
        /templates
            layout.html
            index.html
            login.html
            ...
```

To start this off right, we will create the appropriate files and folders so that we have this structure.

We already have the file application.py where we put all of our code, but we still have to create a subdirectory for all of our application code.

Create the directory ```testimonials``` and within this folder create two files, ```__init__.py``` and ```routes.py```. You may also often see routes.py titled views.py, but I've grown to prefer ```routes.py```.

You can now also create two more subdirectories inside of this new directory, ```static``` and ```templates```. We're not going to have anything in them quite yet, but it's good to have them placed in the right place for when we need them.

At this point, you have the correct structure! Now, we need to take the code from ```application.py``` and split it up into these new files.

## ```application.py```

When we set up our project so that all of our code is in its own directory with an ```__init__.py``` file, we are creating a **package**.

The new job of ```application.py``` is to now import the package, and we will move the rest of the code to ```__init__.py``` for right now.

When we set our environment variable, we said ```export FLASK_APP=application.py```, so it looks for this file and specifically looks for a flask app. Thus, we will import our app variable from our new package into this module.

```python3
from testimonials import app
```

Now, if you're just developing locally, this should be fine, but I know that I'm deploying to AWS and AWS looks for a flask instance called application, not app, so we can ```import as``` to give it the name ```application```:

```python3
from testimonials import app as application
```

> Using ```application.py``` as the filename and providing a callable ```application``` object (the Flask object, in this case) allows Elastic Beanstalk to easily find your application's code ([src](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create-deploy-python-flask.html)).

## ```__init__.py```

The main purpose of ```__init__.py``` is to initialize a new Flask application. This is where we invoke Flask and create a new variable conventionally called ```app```.

We can now keep our initialization code separate from our routing or view logic. Let's move the rest of our code (everything after we create ```app```) and replace it with an import of ```routes.py```. Be sure that this import is at the very bottom of the ```__init__.py``` file.

```python
import testimonials.routes
```

## ```routes.py```

The actual routes for our applications will need to import app (as this variable is used in the code).

```python
from testimonials import app

```

## Final Structure

Our final structure and the code for each file is as follows:
```
/testimonials
    application.py
    /testimonials
        __init__.py
        routes.py
```
(I left out some files, refer to the structure at the beginning for the full structure).


```application.py```
```python
from testimonials import app as application
```

```__init__.py```
```python
from flask import Flask
app = Flask(__name__)

import testimonials.routes
```

```routes.py```

```python
from testimonials import app

@app.route('/')
def index():
    return "Hello!"

@app.route('/api/testimonials')
def testimonials():
    return {'testimonials': ["great", "its ok", "fantastic"]}
```

## Packages and Modules

I didn't talk about it much, but what we did was convert our Flask application from a module to a package. A **module** being a single file with all of code and a **package** being a collection of files (or modules). When we create our application as a package, that's when we introduce the ```__init__.py``` file.

Obviously working with packages is a bit more complicated up front, but as we expand on our application in the next few lessons, it will be a lot more manageable and organized!

Before we move on, test the code locally. If you've setup continuous delivery, push your changes to GitHub and confirm everything is still working on your deployment server. Remember that when you make a change in GitHub it can take a few minutes to deploy on AWS. You can watch this process in CodePipeline.

## Circular imports

We set up our code correctly, so we don't have any issues, but we're really close to having a big issue known as circular imports. This is because our ```__init__.py``` file imports ```routes.py``` and ```routes.py``` imports from ```__init__.py```.

The only reason this is working is because we imported the routes after we defined the app variable. [Corey Shafer](https://www.youtube.com/watch?v=44PvX0Yv368) has a great video explaining circular imports in detail if you want to know the details.
