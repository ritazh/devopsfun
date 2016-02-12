---
layout: post
title: Installing Node
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
date: 2016-01-06T22:27:40-08:00
---
### Installing Node.js
Download install package from [https://nodejs.org](https://nodejs.org). At the time of writing this, we have tested and validated the sample application against Node 5.5.0 and npm 3.3.12. Download the specific release of Node 5.5.0 from [here](https://nodejs.org/en/download/releases/) to ensure later modules in this training work properly.

When you install Node.js, you'll want to ensure your `PATH` variable includes your install path so you can call Node from anywhere. Node comes with npm installed so you should have a version of npm. To install a specific version of npm globallly, run the following:
{% highlight js %}
npm install -g npm@3.3.12
{% endhighlight %}

### Testing Your Install
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