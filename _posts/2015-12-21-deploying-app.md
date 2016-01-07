---
layout: post
title: Deploying an App
modified:
categories: 
description: "Deploying App"
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
In the previous module, you learned how to create your own application platform on Azure with Deis. Now in this module, let's see how to deploy and scale an application to this platform using a simple `git push`. 

### Registering with Deis Controller
Before we can push apps to your Deis cluster, we need to first use `deis register` to register a user with the Deis Controller URL to create a new account. After a new user is registered, you will be logged in automatically. If you or your administrator already created a deis user account, you can use that to [login](#login-to-a-controller).

> Note: The domain you use here should match the one you configured with `deisctl config platform set domain=` in the [previous module](../provisioning-platform#configuring-domain-for-platform). When communicating with the controller, you should always use `deis.<domain>`.
{% highlight bash %}
$ deis register http://deis.mysubdomain.104.209.xxx.xxx.xip.io
username: deis
password: 
password (confirm): 
email: deis@example.com
Registered deis
Logged in as deis
{% endhighlight %}

### Login to a Controller
If you have not logged in and already have a Deis user account, user `deis login` to authenticate against the Deis Controller.
{% highlight bash %}
$ deis login http://deis.mysubdomain.104.209.xxx.xxx.xip.io
username: deis
password:
Logged in as deis
{% endhighlight %}

### Uploading SSH Public Key
In order to use `git push` to deploy applications to Deis, we need to add our default SSH public key to Deis keys using `deis keys:add` to upload it.
{% highlight bash %}
# In this case, id_rsa.pub is the default SSH key used for git push.
$ deis keys:add
Found the following SSH public keys:
1) id_rsa.pub
Which would you like to use with Deis? 1
Uploading /Users/myuser/.ssh/id_rsa.pub to Deis... done
{% endhighlight %}

Congratulations! Now you are ready to create apps and push for deployments!

### Creating an Application
Before you can deploy an application, you have to authenticate against the Deis Controller. Refer to the [login section](#login-to-a-controller).

Deis supports three different ways of deploying applications:
- [Heroku Buildpacks](https://devcenter.heroku.com/articles/buildpacks)
- [Dockerfiles](https://devcenter.heroku.com/articles/buildpacks)
- [Docker Images](https://devcenter.heroku.com/articles/buildpacks)

For the purpose of this training, we are going to deploy our application with builtin buildpacks. Deis will cycle through the `bin/detect` script of each buildpack to match the code you are pushing. During `git push`, you will see `-----> Node.js app detected` in the output.

Use `deis create` to create an application.
{% highlight bash %}
$ deis create
Creating Application... done, created quoted-rucksack
Git remote deis added
remote available at ssh://git@deis.mysubdomain.104.209.xxx.xxx.xip.io:2222/quoted-rucksack.git
{% endhighlight %}

To understand what is going on, let's see the list of remote repositories connected to this project.
{% highlight bash %}
$ git remote -v
deis	ssh://git@deis.mysubdomain.104.209.xxx.xxx.xip.io:2222/quoted-rucksack.git (fetch)
deis	ssh://git@deis.mysubdomain.104.209.xxx.xxx.xip.io:2222/quoted-rucksack.git (push)
origin	https://github.com/[test repo].git (fetch)
origin	https://github.com/[test repo].git (push)
{% endhighlight %}

### Git Push to Deploy
Use `git push deis [branch to deploy]` to deploy your application.
{% highlight bash %}
# In this example, we are pushing the latest in master branch to Deis.
$ git push deis master
{% endhighlight %}

Because a Heroku-style application is detected, the web process type is automatically scaled to 1 on the first deployment.
