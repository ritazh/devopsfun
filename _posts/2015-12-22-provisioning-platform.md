---
layout: post
title: Creating Your Own Application Platform
modified:
categories: 
description: "Creating Your Own Application Platform"
tags: []
image:
  feature: abstract-12.jpg
  credit:
  creditlink:
comments:
share:
date: 2015-12-22T15:02:13-08:00
---
## Creating Your Own Application Platform
Now that we have built our app, let's create an environment for our developers to push their components often to an environment for continous integration and continous deployment.

In this module, you will learn how to create your own application platform on Azure and in the next module you will learn how to deploy and scale an application to this platform on Azure.

### What is Deis
[Deis](http://docs.deis.io/en/latest/) is an open source, lightweight application platform that deploys and scales Twelve-Factor apps as Docker containers across a cluster of CoreOS machines. 

It enables developers to self-host a platform implementing the Heroku workflow without being dependent on the operations team. Deis can deploy any application or service that can run inside of a Docker container. It also enables developers to choose from various development frameworks for their applications via Heroku Buildpacks. 

### Provisioning Resources for Deis on Azure 
Before we install a 3-node Deis cluster on Microsoft Azure, we need to provision all required resources for our Deis cluster.

### Prerequisites
Let's install client tools for Deis so that we can install, configure, and deploy apps to the Deis cluster.

> WARNING: `deisctl` commands should be done by an administrator of the Deis cluster.  `deisctl config` and `deisctl restart platform` will reconfigure the entire cluster. Please use with caution.

#### Installing deisctl CLI

> Note: Make sure the version for `deisctl` CLI matches the version of Deis you want to provision. Check the latest release on [Deis GitHub](https://github.com/deis/deis/releases).

{% highlight bash %}
# On Mac
# For this example, we are getting version 1.12.2 for this CLI.
$ curl -sSL http://deis.io/deisctl/install.sh | sh -s 1.12.2
$ sudo ln -fs $PWD/deisctl /usr/local/bin/deisctl
{% endhighlight %}

#### Installing deis CLI

> Note: Make sure the version for `deisctl` CLI matches the version of Deis you want to provision. Check the latest release on [Deis GitHub](https://github.com/deis/deis/releases).

{% highlight bash %}
# On Mac
$ curl -sSL http://deis.io/deis-cli/install.sh | sh -s 1.12.2
$ ln -fs $PWD/deis /usr/local/bin/deis
{% endhighlight %}

#### Installing Python and PyYAML
Since the cluster creation tool uses Python to generate a configuration file, we need to install Python and PyYAML, a Python library.
{% highlight bash %}
# On Mac
$ brew install python
$ sudo pip install pyyaml
{% endhighlight %}

{% highlight bash %}
# On Ubuntu
$ sudo apt-get install -y python-yaml
{% endhighlight %}

#### Installing Azure CLI
Since we are creating resources on Azure, we need to [install Azure CLI](https://azure.microsoft.com/en-us/documentation/articles/xplat-cli-install/). 

With the Azure CLI, first login to Azure:
{% highlight bash %}
$ azure login
{% endhighlight %}

> Note: Deis provision makes use of [Azure Resource Manager](https://azure.microsoft.com/en-us/documentation/articles/resource-manager-deployment-model/) to submit a template describing the infrastructure we need to create. Make sure you have created an [organizational account](http://www.brucebnews.com/2013/04/the-difference-between-a-microsoft-account-and-an-office-365-account/) on Azure in order to use this template.

Configure Azure CLI to use the ARM mode:
{% highlight bash %}
$ azure config mode arm
{% endhighlight %}

#### Generating a New SSH Key Pair
For this cluster, we need to [create a new SSH key pair](https://help.github.com/articles/generating-ssh-keys/) to be used for accessing the hosts using `ssh-keygen`.

### Creating Resources on Azure

Get the latest Deis project from GitHub. 

> Check the latest release on [Deis GitHub](https://github.com/deis/deis/releases).

{% highlight bash %}
$ git clone https://github.com/deis/deis.git
# To get the latest, check the latest version on the Deis.io site.
$ cd deis
$ git checkout v1.12.2
{% endhighlight %}

For Deis hosts to discover eachother, generate a new discovery URL for this cluster:
{% highlight bash %}
$ cd contrib/azure/
$ ./create-azure-user-data $(curl -s https://discovery.etcd.io/new)
{% endhighlight %} 

Now let's edit `parameters.json` to specify values for deploying this cluster. 

- For `customData`, copy the base64-encoded version of the `azure-user-data` file that was just generated in the previous step. 
{% highlight bash %}
$ base64 azure-user-data
{% endhighlight %} 

- For `sshKeyData`, use the public key for the SSH key you generated in [Generating a New SSH Key Pair](#generating-a-new-ssh-key-pair) to use to log into the hosts. 
{% highlight bash %}
# For example, to get the public key of deis key pair:
$ cat ~/.ssh/deis.pub
{% endhighlight %}

[Here](?) is a sample `parameters.json` file.

To kickoff deployment on Azure, execute the following:
{% highlight bash %}
# This command will create an Azure Resource Group: deisresourcegroup
# It will also create all the resources specified in parameters.json.
$ azure group create --name deisresourcegroup --location "West US" --deployment-name deis --template-file arm-template.json --parameters-file parameters.json

# Sample output
info:    Executing command group create
+ Getting resource group deisresourcegroup                                             
+ Creating resource group deisresourcegroup                                            
info:    Created resource group deisresourcegroup
+ Initializing template configurations and parameters                          
+ Creating a deployment                                                        
info:    Created template deployment "deis"
data:    Id:                  /subscriptions/[omitted]
data:    Name:                deisresourcegroup
data:    Location:            westus
data:    Provisioning State:  Succeeded
data:    Tags: null
data:    
info:    group create command OK
{% endhighlight %}

##### Validating Deployment
Once deployment is complete, you can run the following to get the public IP of one of the VMs in the cluster:
{% highlight bash %}
$ azure vm show deisNode0 --resource-group deisresourcegroup | grep 'Public IP address'
data:            Public IP address       :40.121.xxx.xxx
{% endhighlight %}

> Here is a sample output of a specific deployment:
{% highlight bash %}
# Here are the Deis nodes specific to a cluster:
deisNode0 - 40.121.xx.xx       10.0.0.4
deisNode1 - 40.121.xx.xx       10.0.0.6
deisNode2 - 40.121.xx.xx       10.0.0.5
{% endhighlight %}

### Installing the Deis Platform
After we have provisioned all the resources needed by the Deis cluster on Azure, let's setup and configure Deis.

We will use `deisctl` CLI to install the Deis platform. If it is not the same version as the Deis you have cloned from GitHub, refer to the [Installing deisctl CLI section](#installing-deisctl-cli).
{% highlight bash %}
# Validate this is the correct version of CLI
$ deisctl --version
1.12.2
{% endhighlight %}

Before we can communicate with our CoreOS nodes on Azure, we need to ensure the local SSH agent is running with the private SSH key we created for this deployment in the [Generating a New SSH Key Pair](#generating-a-new-ssh-key-pair) step.
{% highlight bash %}
$ eval `ssh-agent -s`
$ ssh-add ~/.ssh/deis
{% endhighlight %}

To enable `deisctl` to communicate with the Deis CoreOS cluster, we need to set the `DEISCTL_TUNNEL` environment variable to the public IP address of one of our nodes. You can lookup the public IP address from the [Validating Deployment](#validating-deployment) step or access the [Azure portal](https://ms.portal.azure.com/).

{% highlight bash %}
$ export DEISCTL_TUNNEL=40.121.xx.xx
{% endhighlight %}

To allow Deis to connect to remote hosts for `deis run` commands, we need to add the private SSH key we generated for this deployment to the platform:
{% highlight bash %}
$ deisctl config platform set sshPrivateKey=~/.ssh/deis
{% endhighlight %}

We also need to tell the Deis controller the domain name we want to use for deploying applications. In our case, we want to use the public ip of our load balancer together with [xip](http://xip.io/) as our domain, (note, this is not the public IP of any of the nodes).
{% highlight bash %}
$ deisctl config platform set domain=deis.104.209.xxx.xxx.xip.io
{% endhighlight %}

To validate communicate has been setup correctly, test with the following:

{% highlight bash %}
$ deisctl list
# Expected output should be one line since we have not installed the platform:
UNIT	MACHINE	LOAD	ACTIVE	SUB
{% endhighlight %}

Finally ready to install Deis!

{% highlight bash %}
$ deisctl install platform

● ▴ ■
■ ● ▴ Installing Deis...
▴ ■ ●
Storage subsystem...
deis-store-metadata.service: loaded
deis-store-volume.service: loaded
deis-store-daemon.service: loaded
deis-store-monitor.service: loaded
deis-store-gateway@1.service: loaded
Logging subsystem...
deis-logspout.service: loaded
deis-logger.service: loaded
Control plane...
deis-builder.service: loaded
deis-registry@1.service: loaded
deis-database.service: loaded
deis-controller.service: loaded
Data plane...
deis-publisher.service: loaded
Router mesh...
deis-router@1.service: loaded
deis-router@2.service: loaded
deis-router@3.service: loaded
Done.

Please run `deisctl start platform` to boot up Deis.
{% endhighlight %}

After Deis is installed, let's start boot up the platform.

{% highlight bash %}
$ deisctl start platform
{% endhighlight %}

After it's completed, let's check the status of all our components in Deis cluster to ensure they are loaded, active, and running. Your output should look something similar to below.

{% highlight bash %}
$ deisctl list

# Sample output

UNIT				MACHINE			LOAD	ACTIVE	SUB
deis-builder.service		8d9bee8e.../10.0.0.4	loaded	active	running
deis-controller.service		44f1530d.../10.0.0.5	loaded	active	running
deis-database.service		7a1a3941.../10.0.0.6	loaded	active	running
deis-logger.service		7a1a3941.../10.0.0.6	loaded	active	running
deis-logspout.service		44f1530d.../10.0.0.5	loaded	active	running
deis-logspout.service		7a1a3941.../10.0.0.6	loaded	active	running
deis-logspout.service		8d9bee8e.../10.0.0.4	loaded	active	running
deis-publisher.service		44f1530d.../10.0.0.5	loaded	active	running
deis-publisher.service		7a1a3941.../10.0.0.6	loaded	active	running
deis-publisher.service		8d9bee8e.../10.0.0.4	loaded	active	running
deis-registry@1.service		8d9bee8e.../10.0.0.4	loaded	active	running
deis-router@1.service		44f1530d.../10.0.0.5	loaded	active	running
deis-router@2.service		7a1a3941.../10.0.0.6	loaded	active	running
deis-router@3.service		8d9bee8e.../10.0.0.4	loaded	active	running
deis-store-daemon.service	44f1530d.../10.0.0.5	loaded	active	running
deis-store-daemon.service	7a1a3941.../10.0.0.6	loaded	active	running
deis-store-daemon.service	8d9bee8e.../10.0.0.4	loaded	active	running
deis-store-gateway@1.service	44f1530d.../10.0.0.5	loaded	active	running
deis-store-metadata.service	44f1530d.../10.0.0.5	loaded	active	running
deis-store-metadata.service	7a1a3941.../10.0.0.6	loaded	active	running
deis-store-metadata.service	8d9bee8e.../10.0.0.4	loaded	active	running
deis-store-monitor.service	44f1530d.../10.0.0.5	loaded	active	running
deis-store-monitor.service	7a1a3941.../10.0.0.6	loaded	active	running
deis-store-monitor.service	8d9bee8e.../10.0.0.4	loaded	active	running
deis-store-volume.service	44f1530d.../10.0.0.5	loaded	active	running
deis-store-volume.service	7a1a3941.../10.0.0.6	loaded	active	running
deis-store-volume.service	8d9bee8e.../10.0.0.4	loaded	active	running

{% endhighlight %}

Job well done! Now let's go push some apps!

