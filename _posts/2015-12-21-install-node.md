---
layout: post
title: Install Node
modified:
categories: 
description: "Install and configure node.js locally"
tags: []
image:
  feature: abstract-10.jpg
  credit:
  creditlink:
comments:
share:
date: 2015-12-21T22:27:40-08:00
---
### Install Node.js
Download install package from [https://nodejs.org](https://nodejs.org)

When you install Node.js, you'll want to ensure your `PATH` variable includes your install path so you can call Node from anywhere.

### Test Your Install
Create a new directory named `hello-world`, add a new `app.js` file with the following content:

{% highlight js %}
/* app.js */
console.log('Hello World!');

{% endhighlight %}

In the command prompt (or terminal on Mac) , run 
{% highlight bash %}
$ node app.js
{% endhighlight %}

If you get any error regarding Node is not found, open a new command prompt (or terminal) to reflect the new environment variable for Node.

To stop the application, run `Ctrl+C`.