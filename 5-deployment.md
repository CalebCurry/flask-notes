# Deploying our Flask Application

WHAT!? We just started with a hello world application and now we are deploying?

My thoughts are that if we deploy early, we can build out the application with an end goal in mind. However, you are more than welcome to continue this course and come back to this lesson at a later date. But I recommend you go through it.

instead of building out a huge application that fails to deploy, we can deploy and improve our application progressively.

If you want a taste of real world web development, this is it. No more development just for localhost.

A word of warning: This can be complicated if you're new. Don't be discouraged if it doesn't work perfectly the first time. In fact, almost everything in this course can be followed with localhost. This means that if, for some reason, you're unable to get your application deployed, you can still study this course and come back to this section at a later date when you have more experience. Additionally, whenever we work with cloud providers it may cost money, so watch out for that.

For this process we will be storing our code in GitHub which is where our deployment server will get the latest code. Not only is this freakin' cool, it's a best practice because we'll have all of our code safe in a repository off of our computer.

It can also help us learn about open source software. If we decide to make our repository public, we'll have to take extra care that we design our code in such a way that it doesn't expose anything secret. You can use a private repo if you prefer, though.

## Step 1: Source Control

First, Download [Git](https://git-scm.com/). If you're on Mac or Linux, you'll use Git from the Terminal. If you're on windows, the Git installation will give you the option to install [MinGW](http://www.mingw.org/), which I recommend. This is a software that will emulate a Unix-like terminal (making following along with this course 100x easier).

Once Git is installed, type ```git``` in the terminal to confirm it's recognized as a command.

```
git init
touch .gitignore
```
paste gitignore content

```
git add .
git commit -m "initial commit"
```

Now, create the repository in GitHub
```
git remote add origin https://github.com/CalebCurry/cuddly-octo-bassoon.git
git branch -M main
git push -u origin main
```

Make sure you use the correct origin URL (replace it with the repo URL in your GitHub account).

## Step 2: Deployment Server

We will be using Amazon Web Services Elastic Beanstalk.

This is Amazon's **fully managed** solution. In [their words](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html):

> With Elastic Beanstalk, you can quickly deploy and manage applications in the AWS Cloud without having to learn about the infrastructure that runs those applications. Elastic Beanstalk reduces management complexity without restricting choice or control. You simply upload your application, and Elastic Beanstalk automatically handles the details of capacity provisioning, load balancing, scaling, and application health monitoring.

In a way, Elastic Beanstalk is bad as it's so easy to use without learning much technical detail. That being said, it allows us to focus on building our web application and not investing all of our time in studying the infrastructure.

The underlying infrastructure used from AWS is known as EC2, which is the service to deploy servers.

You can additionally use Amazon's **Route** 53 to register a custom domain name.

Another service is **Amazon RDS**, for relational databases.

And another service made available, **Amazon S3**, can be used for static file storage such as images.

Most of what we need will be wrapped up nicely in Elastic Beanstalk.

Let's set it up.
## Step 3: CodePipeline

