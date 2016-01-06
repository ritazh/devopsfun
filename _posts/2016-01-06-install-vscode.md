---
layout: post
title: Install Vscode
modified:
categories: 
description: "Install Visual Studio Code"
tags: []
image:
  feature: abstract-6.jpg
  credit:
  creditlink:
comments:
share:
date: 2016-01-06T10:02:38-08:00
---
### Install Visual Studio Code
If you don't already have VS Code installed, go to the official [VS Code Download page and download it](https://code.visualstudio.com/Docs/editor/setup). 

#### Get VS Code for Mac
To get the latest version of VS Code, [download the binary installer for OS X](http://go.microsoft.com/fwlink/?LinkID=534106).

Click on `VSCode-osx.zip` to open the contents. Drag `Visual Studio Code.app` to the `Applications` folder. Now you can search for the app and launch it. 

> If you want to launch VS Code from terminal, add the following to your `~/.bash_profile`. 
{% highlight bash %}
function code () { VSCODE_CWD="$PWD" open -n -b "com.microsoft.VSCode" --args $*; }
{% endhighlight %}

Now, from terminal, go to any folder with your project, type `code .` will launch VS Code with files in that folder.


#### Get VS Code for Windows
To get the latest version of VS Code for Windows, [download the binary installer](http://go.microsoft.com/fwlink/?LinkID=534107).

Click on `VSCodeSetup.exe` to initiate the install process. Once the install is complete, you can launch the app. 

> To launch VS Code from console, all you need to do is type `code .` in a folder, VS Code will be launched with all the files in that folder. Note, the installer added VS Code to your `PATH` environment variable. You may need to log off then log in after a fresh install to ensure `PATH` has been updated. 

#### BONUS: Know More VS Code
Now you have installed and setup VS Code. Here are more details for how to use VS Code.

- [The Basics](https://code.visualstudio.com/docs/editor/codebasics) - The Basics of Visual Studio Code
- [Editing Evolved](https://code.visualstudio.com/docs/editor/editingevolved) - VS Code features, Lint, IntelliSense, Lightbulbs, Peek and Goto Definition and more
- [Debugging](https://code.visualstudio.com/docs/editor/debugging) - VS Code's built-in debugger 

