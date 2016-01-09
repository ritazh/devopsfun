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
date: 2015-12-20T15:03:36-08:00
---
## Managing Apps
In the previous module, we were able to deploy our application to the Dokku platform on Azure with a simple `git push`. Now in this module, let's see how we can continously deploy and make updates to the application.

### Scaling App
Dokku allows us to run multiple applications each backed by any number of containers. It comes with nginx out of the box, which provides HTTP load balancing and routing. With Dokku, we can horizontally scale our apps simply by adding more instances behind a load balancer.

There are two ways to scale an app on Dokku. One is by creating a `DOKKU_SCALE` file in the root of our repository. Dokku expects this file to contain one line for every process defined in the Procfile. Two is by using the `ps:scale` command. This command can be used to scale multiple process types at the same time.

Let's increase the scale of our app:
{% highlight bash %}
# Before we increase the scale, let's see what the current scale is for our app:
$ dokku ps:scale hackathon-starter
-----> Scaling for hackathon-starter
-----> proctype           qty                                                                                                                                
-----> web                1

$ dokku ps:scale hackathon-starter web=2
{% endhighlight %}
