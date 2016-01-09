---
layout: post
title: Deploying an App
modified:
categories: 
description: "Deploying an App"
tags: []
image:
  feature: abstract-1.jpg
  credit:
  creditlink:
comments:
share:
date: 2015-12-21T15:02:46-08:00
---
## Deploying an App
In the previous module, you learned how to create your own application platform on Azure with Dokku. Congratulations! Now you are ready to create apps and push for deployments to this platform using a simple `git push`. 

### Creating an Application

Dokku defaults to using Buildpack to deploy applications, unless a valid Dockerfile is detected at the root of your repository.

- [Buildpacks](http://dokku.viewdocs.io/dokku/deployment/buildpacks/): Using Heroku buildpacks to auto detect framework and runtime support for your application.
- [Dockerfiles](http://dokku.viewdocs.io/dokku/deployment/dockerfiles/): Dockerfiles are used to define a portable execution environment built on a base OS of your choosing.

For the purpose of this training, we are going to deploy our application with builtin buildpacks. During `git push`, Dokku will automatically detect the right Buildpack for your application, you will see `-----> Node.js app detected` in the output.

SSH into the host we just provisioned using the private key of the ssh key pair we used in the previous module:
{% highlight bash %}
$ ssh -i <your-ssh-private-key> <your-admin-user-name>@<DNSNAMEFORPUBLICIP>.<LOCATION>.cloudapp.azure.com
{% endhighlight %}

Create a new Dokku application:
{% highlight bash %}
$ dokku apps:create hackathon-starter
Creating hackathon-starter... done
{% endhighlight %}

### Adding MongoDB
Our Node.js application won't work without a database. Previously we were test our app locally with a MongoDB instance running locally. Now we we need to create a MongoDB instance for our application data on Dokku and link it to the application we just created by using the `Dokku Mongo plugin`:

{% highlight bash %}
# install the mongo plugin
# plugin installation requires root, hence the user change
$ sudo dokku plugin:install https://github.com/dokku/dokku-mongo.git mongo

# create a postgres service with the name nodeapp
$ dokku mongo:create nodeapp
-----> Starting container
       Waiting for container to be ready
=====> MongoDB container created: nodeapp
       DSN: mongodb://nodeapp:1080854ca703fc3de69fad128ea553ef@dokku-mongo-nodeapp:27017/nodeapp
{% endhighlight %}

Each official datastore offers a `link` method to link a service to any application. We will link the new mongodb instance to our Node.js application. Note the environment variable `MONGO_URL` has been set to the new instance. We will need this environment variable in a later step.

{% highlight bash %}
$ dokku mongo:link nodeapp hackathon-starterno config vars for hackathon-starter
-----> Setting config vars
       MONGO_URL: mongodb://nodeapp:1080854ca703fc3de69fad128ea553ef@dokku-mongo-nodeapp:27017/nodeapp
-----> Restarting app hackathon-starter
App hackathon-starter has not been deployed
{% endhighlight %}

To verify that we have linked the database and the Node.js application successfully:
{% highlight bash %}
$ dokku mongo:list
NAME       VERSION      STATUS   EXPOSED PORTS  LINKS
nodeapp    mongo:3.0.6  running  -              hackathon-starter 
{% endhighlight %}

### Add Dokku Remote Repo

From your local environment (not the Dokku VM), navigate to your project folder. Add a `dokku` remote to your local git repository using the `dokku` user name to push the app.
{% highlight bash %}
$ git remote add dokku dokku@ritadokku.eastus2.cloudapp.azure.com:hackathon-starter
$ git remote -v
dokku	dokku@<DNSNAMEFORPUBLICIP>.<LOCATION>.cloudapp.azure.com:hackathon-starter (fetch)
dokku	dokku@<DNSNAMEFORPUBLICIP>.<LOCATION>.cloudapp.azure.com:hackathon-starter (push)
origin	https://github.com/sahat/hackathon-starter.git (fetch)
origin	https://github.com/sahat/hackathon-starter.git (push)
{% endhighlight %}

### Update Environment File for Mongo
Before we push our app to Dokku, we need to update the `.env.example` file to tell the app to use the MongoDB instance we just created.

Recall this is the current `.env.example` configuration setting for Mongo:

```
MONGODB=mongodb://localhost:27017/test
```


We need to update this value to the `$MONGO_URL` environment variable set in the previous step when we linked the Mongo service to this app.

```
MONGODB=$MONGO_URL
```

Save this file, then use `git add` and `git commit` to push the change to your local master branch so that the updates will be deployed to Dokku when we push the app.

### Git Push to Deploy
Good job! We are almost there! Now we can finally push our application to Dokku.

Deploy the app with `git push` and Dokku will create the application on the server.
{% highlight bash %}
$ git push dokku master
{% endhighlight %}

Your output should look similar to below. Notice how Dokku automatically detects our app is a Node.js app.
{% highlight bash %}
Counting objects: 8446, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3376/3376), done.
Writing objects: 100% (8446/8446), 7.72 MiB | 650.00 KiB/s, done.
Total 8446 (delta 4909), reused 8437 (delta 4904)
-----> Cleaning up...
-----> Building hackathon-starter from herokuish...
-----> Adding BUILD_ENV to build environment...
-----> Node.js app detected
       
-----> Creating runtime environment
       
       NPM_CONFIG_LOGLEVEL=error
       NPM_CONFIG_PRODUCTION=true
       NODE_ENV=production
       NODE_MODULES_CACHE=true
       
-----> Installing binaries
...

-----> Restoring cache
       Skipping cache restore (new runtime signature)
       
-----> Building dependencies
       Pruning any extraneous modules
       Installing node modules (package.json)
...
{% endhighlight %}

The entire process builds a Docker image with all the dependencies our Node.js app needs. At the end, it will start the container with the recently built image deployed to the domain address we specified in the previous module. 
{% highlight bash %}
=====> hackathon-starter container output:
       > hackathon-starter@3.4.0 start /app
       > node app.js
       Express server listening on port 5000 in production mode
=====> end hackathon-starter container output
-----> Running post-deploy
-----> Creating new /home/dokku/hackathon-starter/VHOST...
-----> Setting config vars
       DOKKU_NGINX_PORT: 80
-----> Configuring hackathon-starter.<HOSTPUBLICIP>.xip.io...
-----> Creating http nginx.conf
-----> Running nginx-pre-reload
       Reloading nginx
-----> Setting config vars
       DOKKU_APP_RESTORE: 1

=====> Application deployed:
       http://hackathon-starter.<HOSTPUBLICIP>.xip.io
{% endhighlight %}

To verify the application has been deployed successfully, browse to the URL at the end of the output (e.g.:http://hackathon-starter.<HOSTPUBLICIP>.xip.io) and you will see the app running. Try creating a new account.

<figure>
	<img src="../images/deployapp.png"/>
	<figcaption>Screenshot of deployed app</figcaption>
</figure>

