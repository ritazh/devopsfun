---
layout: post
title: Managing App
modified:
categories: 
description: 
tags: []
image:
  feature: abstract-8.jpg
  credit:
  creditlink:
comments:
share:
date: 2015-12-10T15:03:36-08:00
---
## Managing Apps
In the previous module, we were able to continuously integrate and deploy updates of our application to the Dokku platform on Azure with a simple `git push`. Now in this module, let's see how we scale and configure the application to address production load.

### Scaling App
Dokku allows us to run multiple applications each backed by any number of containers. It comes with nginx out of the box, which provides HTTP load balancing and routing. With Dokku, we can horizontally scale our apps simply by running more containers on a single VM instance. 

> Note: Unlike other high-availability PaaS, since it is a single instance platform, scaling is only available at the container level, we cannot scale out by adding more hosts into our platform. Hence, Dokku is ideal for development and testing, not for production.

There are two ways to scale an app on Dokku. One is by creating a `DOKKU_SCALE` file in the root of our repository. Dokku expects this file to contain one line for every process defined in the Procfile. Two is by using the `ps:scale` command. This command can be used to scale multiple process types at the same time.

Let's increase the scale of our app:
{% highlight bash %}
# Before we increase the scale, let's see what the current scale is for our app:
$ dokku ps:scale hackathon-starter
-----> Scaling for hackathon-starter
-----> proctype           qty                                                                                                                                
-----> web                1

$ dokku ps:scale hackathon-starter web=2

-----> Scaling hackathon-starter:web to 2
-----> Releasing hackathon-starter (dokku/hackathon-starter:latest)...
-----> Deploying hackathon-starter (dokku/hackathon-starter:latest)...
-----> DOKKU_SCALE file found
=====> web=2
-----> Running pre-flight checks
       For more efficient zero downtime deployments, create a file CHECKS.
       See http://progrium.viewdocs.io/dokku/checks-examples.md for examples
       CHECKS file not found in container: Running simple container check...
-----> Waiting for 10 seconds ...
-----> Default container check successful!
...
{% endhighlight %}

### Adding a Domain
At this point, you should see our app running here: `http://hackathon-starter.<HOSTPUBLICIP>.xip.io`. What if we want to add a different domain? With `dokku domains:add <appname>`, you can add a custom domain name to your app.

{% highlight bash %}
$ dokku domains:add hackathon-starter devopsfun.<HOSTPUBLICIP>.xip.io
-----> Configuring hackathon-starter.<HOSTPUBLICIP>.xip.io...
-----> Configuring devopsfun.<HOSTPUBLICIP>.xip.io...
-----> Creating http nginx.conf
-----> Running nginx-pre-reload
       Reloading nginx
-----> Configuring hackathon-starter.<HOSTPUBLICIP>.xip.io...
-----> Configuring devopsfun.<HOSTPUBLICIP>.xip.io...
-----> Creating http nginx.conf
-----> Running nginx-pre-reload
       Reloading nginx
-----> Added devopsfun.<HOSTPUBLICIP>.xip.io to hackathon-starter
{% endhighlight %}

Navigate to `devopsfun.<HOSTPUBLICIP>.xip.io` in your browser, you should see the app.
