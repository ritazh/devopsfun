---
layout: post
title: Setup CI for Your App
modified:
categories: 
description: "Setup CI for Your App"
tags: []
image:
  feature: abstract-7.jpg
  credit:
  creditlink:
comments:
share:
date: 2015-12-15T15:02:46-08:00
---
## Adding Continuous Integration for Your App
In the previous module, you learned how to manually push an application to a test environment from our local dev machine. Now what about running tests, deploy to staging and production, and checkstyle reports? In this module, we will look at how to setup and configure a Jenkins instance to create an end-to-end continuous integration pipeline for our Node.js application. We start the pipeline by integrating with GitHub to automatically trigger this CI pipeline to run when a change is pushed to GitHub.  

### What is Jenkins
[Jenkins](https://jenkins-ci.org/) is an open source automation tool written in Java. It provides continous integration services for application development. With a huge open source community, it provides hundreds of plugins to support building, testing, deploying, and automation for virtually any project. 

It enables developers to automate end-to-end build, test, and deployment process from pushing code updates to a source repository to deploying the latest features to a production environment in minutes.

### Provisioning Resources for Jenkins on Azure 
To setup a Jenkins instance on Microsoft Azure, we need to provision all the required resources for Jenkins. At the end of this step, we should have two virtual machines (one Jenkins master node on an Ubuntu virtual machine and a Jenkins slave node on another VM), a storage account, virtual network, availability sets, public IP addresses and network interfaces required by the installation.

Now, let's login to Azure CLI. If you have not installed the Azure CLI, please install the [Azure CLI](https://azure.microsoft.com/en-us/documentation/articles/xplat-cli-install/) to provision resources using an Azure Resource Manager (ARM) template.

First login to the Azure CLI:
{% highlight bash %}
$ azure login
{% endhighlight %}

Instruct the client to switch to ARM mode:
{% highlight bash %}
$ azure config mode arm
{% endhighlight %}

Next, download [azuredeploy.json](https://github.com/Azure/azure-quickstart-templates/blob/master/jenkins-on-ubuntu/azuredeploy.json) and [azuredeploy.parameters.json](https://github.com/Azure/azure-quickstart-templates/blob/master/jenkins-on-ubuntu/azuredeploy.parameters.json) locally. Edit `azuredeploy.parameters.json` to provide values to the parameters required for the cluster. You can also refer to a sample `azuredeploy.parameters.json` [here](https://github.com/ritazh/devopsfun/blob/gh-pages/provisionjenkins/azuredeploy.parameters.json)for your reference.

- For `storageAccountPrefix`, `virtualNetworkName`, and `dnsName`, you need to provide unique values for these fields.

To kickoff deployment on Azure, type the following in terminal:
{% highlight bash %}
$ azure group create --name devopsfunrgjenkins --location "West US" --deployment-name devopsfun --template-file azuredeploy.json --parameters-file azuredeploy.parameters.json
{% endhighlight %}

Actual deployment time may vary. It should take less than 5 minutes for provision to complete.

To verify, you can check the `ProvisionState` by running the following command in terminal. When the deployment is completed, the `ProvisionState` should go from `Running` to `Succeeded`. 
{% highlight bash %}
$ azure group deployment show devopsfunrgjenkins devopsfun

# Sample output:
+ Getting deployments                                     
data:    DeploymentName     : devopsfun
data:    ResourceGroupName  : devopsfunrgjenkins
data:    ProvisioningState  : Running
...
{% endhighlight %}

You can also verify by logging into the [Azure Portal](https://portal.azure.com/) and once deployment is done, you should see the following resources:

<figure>
  <img src="../images/jenkinsonazure.png"/>
  <figcaption>Screenshot of Azure portal</figcaption>
</figure>

### Configuring Jenkins Security
Once we have all the resources provisioned on Azure, let's configure Jenkins security settings to complete the setup. By default, Jenkins allows anyone to do anything in the system. Let’s change this.

Browse to the new Jenkins instance by navigating to the URL below in your browser. Using the `dnsName` and `region` values you provided in the previous step and hit the Jenkins master server on port 8080. For example with the values we provided in the screenshot, this would be the URL: http://devopsfunjenkins.westus.cloudapp.azure.com:8080

```
http://<dnsName>.<region>.cloudapp.azure.com:8080
```

From your browser, click `Manage Jenkins` in the navigation sidebar and then click `Configure Global Security`. From the `Configure Global Security` page, check `Enable security`. Next, scroll to `Access Control` and select `Jenkins’ own user database`. Then, click `Save`.

<figure>
  <img src="../images/jenkinssecurity.png"/>
  <figcaption>Screenshot of Configuring Jenkins Security</figcaption>
</figure>

Let's lock down the Jenkins instance to allow only your admin user to create new users and to view/administer the instance. You can always add more administrators later. First, create a user by clicking `Sign up` and filling out the form, then submit.

<figure>
  <img src="../images/jenkinsadmin.png"/>
  <figcaption>Screenshot of Configuring Jenkins Security</figcaption>
</figure>

Back to the `Global Security` Page again. Uncheck `Allow users to sign up`.

<figure>
  <img src="../images/jenkinsnosignup.png"/>
  <figcaption>Screenshot of Configuring Jenkins Security</figcaption>
</figure>

Then, under `Authorization` select `Project-based Matrix Authorization Strategy`. Under the matrix table, add the recently created username "devopsfunadmin" and then check `Administer` under the permissions. Then Save.

<figure>
  <img src="../images/jenkinsauthorization.png"/>
  <figcaption>Screenshot of Configuring Jenkins Security</figcaption>
</figure>

Job well done! You have locked down your Jenkins instance.

### Installing Plugins
Jenkins provides hundreds of plugins to support building, testing, deploying, and automation for virtually any project. 

From your browser, click `Manage Jenkins` in the navigation sidebar and then click `Manage Plugins`. From the `Available` tab, search and select the following plugins:
- GitHub Plugin
- Embeddable Build Status Plugin
- NodeJS Plugin

Click `Install without restart` to install the plugins and all the dependencies for these plugins. Once all plugins indicate `Success`, click the checkbox next to "Restart Jenkins" at the bottom to finish the installation.

<figure>
  <img src="../images/jenkinsrestart.png"/>
  <figcaption>Screenshot of Restarting Jenkins</figcaption>
</figure>

### Integrating with GitHub
To create an end-to-end CI pipeline for an application, we need to start with cloning the code from the code source repository. Jenkins supports many source code management systems. With the GitHub plugin, we can integrate with each one of our GitHub repository through the GitHub API. Let’s add GitHub credentials for cloning repositories.

First, we need to generate a GitHub token to use with the GitHub API to ensure the GitHub account has administrative rights to the repos. From your browser, go to GitHub and ensure you are logged in with the account you want to use. Then, visit the `Settings` page, then select `Personal access tokens` from the sidebar. To the right, click `Generate new token`. In the form, set the relevant permissions to enable the hooks to work properly in Jenkins by giving the hook a name, check `repo` and then clicking `Generate token`. Copy and save the generated key.

<figure>
  <img src="../images/jenkinsgengithubtoken.png"/>
  <figcaption>Screenshot of Jenkins GitHub Plugin</figcaption>
</figure>

From your browser, click `Manage Jenkins` then click `Configure System`. Under `GitHub Plugin Configuration`, click `Add GitHub Server Config` to add credentials. Click `Add` next to the `Credentials` dropdown. From `Add Credentials` view, select `Secret text` from the dropdown. Set `Scope` to `Global` and paste the GitHub token in `Secret` text field. Provide a description, for example `github jenkins`. Then click `Add`. Now the `Credentials` dropdown should be set to the Jenkins Credential we just created. Click `Verify credentials` to verify connectivity. Click Save.

<figure>
  <img src="../images/jenkinsgithubtoken.png"/>
  <figcaption>Screenshot of Jenkins GitHub Plugin</figcaption>
</figure>

##### Installing Git on Jenkins
Since we are using git to clone our repo to Jenkins servers and we are using git to push to our Dokku test environment, we need to install git on Jenkins servers. First ssh into Jenkins master VM:
{% highlight bash %}
$ ssh <adminUsername>@<dnsName>.<LOCATION>.cloudapp.azure.com
# Use the same adminUsername, adminPassword as you provided for azuredeploy.parameters.json
{% endhighlight %}

Once logged into the Jenkins VMs, run the `sudo apt-get install git` to install git.

#### Creating Jenkins Job
Now let's setup a new Jenkins job for our Node.js project. From your browser, click `New Item`, provide a name for the job, for example `nodesample`, select the `Freestyle project` option. Then hit OK. In the configuration page, check the `GitHub project` box, then for `Project url`, provide the http url to your Node.js app GitHub repo. Under `Source Code Management`, select the `Git` radio button. For `Repository URL`, provide the http url to your Node.js app GitHub repo. 

<figure>
  <img src="../images/jenkinsscm.png"/>
  <figcaption>Screenshot of Jenkins Git repo</figcaption>
</figure>

Next, make sure to check the `Build when a change is pushed to GitHub` option as that will automatically trigger this job to run whenever a change has been commited to the master branch of the GitHub repo. Click Save.

Now go ahead and checkin some changes to your repo. You should see this Jenkins job automatically running as soon as you do a `git push`. 

<figure>
  <img src="../images/jenkinsjobrun.gif"/>
  <figcaption>Screenshot of Jenkins job triggered by git push</figcaption>
</figure>

### Setting up Node
To build and test Node.js applications with Jenkins, we need to configure the NodeJS plugin to build applications against different versions of Node. It also handles the installations and writes Jenkins scripts in Node. From Jenkins, click `Manage Jenkins` and `Configure System`. Under `NodeJS` section, click `NodeJS installations`. For NodeJS Name, provide a value so you can easily refer to it in a job. Check the `Install automatically` box, click `Add Installer` dropdown to select `Install from nodejs.org`. Select `NodeJS 5.5.0` for `Installation`. Leave npm packages empty, since we will build dependencies locally for each job. Click Save.

<figure>
  <img src="../images/jenkinsnodeinstall.png"/>
  <figcaption>Screenshot of Setting up Node on Jenkins</figcaption>
</figure> 

Now from Jenkins, let's configure our job to use a specific Node installation to build the application.
From your browser, click the job for your Node application, for example `nodesample`. Scroll down to `Build Environment` and check `Provide Node & npm bin/ folder to PATH`. Then, select the desired Node installation to build this application.

<figure>
  <img src="../images/jenkinssetnodeforjob.png"/>
  <figcaption>Screenshot of Set Node version for a Job</figcaption>
</figure> 

To test our installation of Node, let's build our Node app on Jenkins. Scroll down to `Build` section, click `Add build step`. Let's confirm the Node and npm versions and let's execute `npm install` to build the app. Click Save.

<figure>
  <img src="../images/jenkinsbuildshell.png"/>
  <figcaption>Screenshot of Build shell for job</figcaption>
</figure> 

Let's build this job by clicking `Build Now`. To see the output of this job, click on the running instance and `Console Output`. The output should contain something like the following:

{% highlight bash %}
+ node -v
v5.5.0
+ npm -v
3.6.0
+ npm install
...
Finished: SUCCESS
{% endhighlight %}

### Exposing Build Badges
Build badges can be added to your Github readme.md and elsewhere that indicate the state of the build. From your browser, click `Manage Jenkins` then click `Configure Global Security`. Under `Authorization` and `Project-based Matrix Authorization Strategy`, in the matrix table, for the `Anonymous` user, check  `ViewStatus` under the `Job` category. Then Save.

<figure>
  <img src="../images/jenkinsanonymousviewstatus.png"/>
  <figcaption>Screenshot of Configuring Jenkins anonymous view access</figcaption>
</figure>

To get the build badge for our application, from Jenkins, click into the job for our application. Click the `Embeddable Build Status` icon in the project sidebar to reveal embeddable markup. Copy and paste the relevant format for your readme file. Make sure to copy the unprotected markup so it can be accessed anonymously.

<figure>
  <img src="../images/jenkinsbuildbadgemarkdown.png"/>
  <figcaption>Screenshot of Get Jenkins Job Build Status</figcaption>
</figure>

Now add the badge status markup to the top  the `README.md` file in your node.js sample app. Commit your changes and `git push origin master`. This will kickoff the job we just created in Jenkins and the real-time status of the current build will be visible on the main page of your GitHub repo.

<figure>
  <img src="../images/jenkinsbuildbadge.png"/>
  <figcaption>Screenshot of Build Badge on GitHub</figcaption>
</figure> 






