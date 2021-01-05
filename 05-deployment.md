# Deploying our Flask Application

WHAT!? We just started with a hello world application and now we are deploying?

My thoughts are that if we deploy early, we can build out the application with an end goal in mind. However, you are more than welcome to continue this course and come back to this lesson at a later date. But I recommend you go through it.

instead of building out a huge application that fails to deploy, we can deploy and improve our application progressively.

If you want a taste of real world web development, this is it. No more development just for localhost.

A word of warning: This can be complicated if you're new. Don't be discouraged if it doesn't work perfectly the first time. In fact, almost everything in this course can be followed with localhost. This means that if, for some reason, you're unable to get your application deployed, you can still study this course and come back to this section at a later date when you have more experience. Additionally, whenever we work with cloud providers it may cost money, so watch out for that.

For this process we will be storing our code in GitHub which is where our deployment server will get the latest code. Not only is this freakin' cool, but it's a best practice because we'll have all of our code safe in a repository off of our computer.

Using GitHub can also help us learn about open source software. If we decide to make our repository public, we'll have to take extra care that we design our code in such a way that it doesn't expose anything secret. You can use a private repo if you prefer, though.

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

We will be using **AWS Elastic Beanstalk**.

This is Amazon's **fully managed** solution. In [their words](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html):

> With Elastic Beanstalk, you can quickly deploy and manage applications in the AWS Cloud without having to learn about the infrastructure that runs those applications. Elastic Beanstalk reduces management complexity without restricting choice or control. You simply upload your application, and Elastic Beanstalk automatically handles the details of capacity provisioning, load balancing, scaling, and application health monitoring.

In a way, Elastic Beanstalk is bad as it's so easy to use without learning much technical detail. That being said, it allows us to focus on building our web application without investing all of our time in studying the infrastructure.

The underlying server infrastructure used with Elastic Beanstalk is known as **Amazon EC2**, which is the service to deploy servers.

You can additionally use **Amazon Route 53** to register a custom domain name.

Another service is **Amazon RDS**, for relational databases.

And another service made available, **Amazon S3**, can be used for static file storage such as images.

Most of what we need will be wrapped up nicely in Elastic Beanstalk.

Let's set it up. I'm not going to show photos of every single step as it's fairly intuitive and the user interface might change slightly over time.

1. Visit the [Elastic Beanstalk](https://console.aws.amazon.com/elasticbeanstalk/home) home.
1. You'll need to register for an AWS account if you don't have one already.
1. Create a new environment (web server tier), give it a name. We're going to be building an application to take testimonials, so you can call it ```testimonials```.
1. For the platform choose Python with the highest branch and version.
1. For the application code go with ```sample application```. We'll replace this code using our repo and AWS CodePipeline.
1. You can also configure a domain name if you're interested. You can still follow along without a domain name, but it's not going to be a pretty URL. I recommend you choose a domain name if you're building a project so that you have an official software application deployed in the real world.

Once your environment is initialized you should be able to go into the configuration and see options. For example, you can open the software category and see that it looks for a WSGI file called ```application```. So we will name our main file in Flask ```application.py```. Do that if you haven't already. We already did. These settings are also where you can set up things like load balancing, databases, etc.

You can launch the application and visit the home page once the server is done launching. Just click ```Go to the environment``` on the left panel.

## Step 3: CodePipeline

There are a bunch of tools in AWS for **CI/CD** (continuous integration, continuous delivery/deployment). You can read about the full list of options [here](https://aws.amazon.com/blogs/devops/complete-ci-cd-with-aws-codecommit-aws-codebuild-aws-codedeploy-and-aws-codepipeline/).

The summary is this:

**CodeCommit** - Source control service (an alternative to GitHub).

**CodeBuild** - Continuous integration (compiling and running tests).

**CodeDeploy** - Continuous deployment for automating releases to servers such as EC2.

**CodePipeline** - Continuous delivery to automate the entire release pipeline. This will take code from a repo in CodeCommit or GitHub (as well as some other options), run tests with CodeBuild, then brings it through to deployment through CodeDeploy.

1. We will be using CodePipeline, so go [set up a new pipeline](https://console.aws.amazon.com/codesuite/codepipeline/home).
1. You can use the name ```testimonials``` for the pipeline name.
1. Choose GitHub (I'm using version 2) and connect your account. Then, choose the correct repository and branch (likely the ```main``` branch)
1. Skip the build and deploy stages
1. Review and confirm

Now, whenever you push changes to the main branch in your GitHub repository, these changes will propagate to AWS and redeploy the web application. Cool!

Now, In a proper development setup, you would only deploy once you have *complete* changes. This is obviously a very subjective term, but ideally you only push code to the main branch when a feature is complete and everything is working properly.

If you've followed up to this point and gave your application a domain name, you have a real world application that you can show people or put on your resume. Yeah, it doesn't do a whole lot, but it does show that you can deliver ;)
