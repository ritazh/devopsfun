---
layout: post
title:  "Install Node"
date:   2015-12-04 03:38:55 -0800
categories: Preparation
permalink: /install-node-tutorial/
---

### Install Node.js
Download install package from [https://nodejs.org](https://nodejs.org)

When you install Node.js, you'll want to ensure your `PATH` variable includes your install path so you can call Node from anywhere.

### Test your install
Create a new directory named `hello-world`, add a new `app.js` file with the following content:

```js
/* app.js */
console.log('Hello World!');
```

In the command prompt (or terminal on Mac) , run `node app.js`.

If you get any error regarding Node is not found, open a new command prompt (or terminal) to reflect the new environment variable for Node.

To stop the application, run `Ctrl+C`.
