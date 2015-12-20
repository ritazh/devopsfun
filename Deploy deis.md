node0 - 168.62.236.222      10.0.0.5
node1 - 168.62.236.233      10.0.0.6
node2 - 168.62.232.240      10.0.0.4
domain - tsla.168.62.238.12.xip.io


node0 - 40.121.145.95       10.0.0.4
node1 - 40.121.148.67       10.0.0.6
node2 - 40.121.146.53       10.0.0.5
domain - tsla.40.121.144.218.xip.io

$ cd contrib/azure/
$ ./create-azure-user-data $(curl -s https://discovery.etcd.io/new)
Wrote file "/Users/ritazhang/Documents/work/deis-ritazh/deis/contrib/azure/azure-user-data" with url "https://discovery.etcd.io/fa8760af55265d28b804b1fcb0480cdb"

$ base64 azure-user-data
$ cat ~/.ssh/deis.pub 
Update parameter.json in ~/Documents/work/deis-ritazh/deis/contrib/azure

$ azure group create --name tesladeis2 --location "East US 2" --deployment-name deis --template-file arm-template.json --parameters-file parameters.json

/// FOR TSLA
$ azure group deployment create -g TeslaHackfest --name deistsla --template-file arm-template.json --parameters-file parameters.json

info:    Executing command group create
+ Getting resource group ritadeis2                                             
+ Creating resource group ritadeis2                                            
info:    Created resource group ritadeis2
+ Initializing template configurations and parameters                          
+ Creating a deployment                                                        
info:    Created template deployment "deis"
data:    Id:                  /subscriptions/04f7ec88-8e28-41ed-8537-5e17766001f5/resourceGroups/ritadeis2
data:    Name:                ritadeis2
data:    Location:            eastus2
data:    Provisioning State:  Succeeded
data:    Tags: null
data:    
info:    group create command OK

$ azure vm show deisNode0 --resource-group ritadeis2 | grep 'Public IP address'
/// For TSLA
$ azure vm show deisNode0 --resource-group TeslaHackfest | grep 'Public IP address'
data:            Public IP address       :168.62.236.222

data:            Public IP address       :104.209.209.1

$ eval `ssh-agent -s`
$ ssh-add ~/.ssh/deis

// set to one of the nodes in the cluster
$ export DEISCTL_TUNNEL=40.121.145.95
$ deisctl list
UNIT	MACHINE	LOAD	ACTIVE	SUB



$ deisctl config platform set sshPrivateKey=~/.ssh/deis
// domain is the load balancer ip + xip.io (NOTE: this is not the public ip of the deisnode)
$ deisctl config platform set domain=subdomain.104.209.212.89.xip.io
subdomain.104.209.212.89.xip.io



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

$ deisctl start platform

/// OUTPUT for TSLA

$ deisctl list
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

// Register with a Deis controller, register a new user

$ deis register http://deis.subdomain.40.78.68.187.xip.io
username: deis
password: 
password (confirm): 
email: deis@example.com
Registered deis
Logged in as deis

// Upload your ssh public key to deis to enable push app and pull from git with your git ssh key

$ deis keys:add
Found the following SSH public keys:
1) id_rsa.pub
Which would you like to use with Deis? 1
Uploading /Users/myuser/.ssh/id_rsa.pub to Deis... done

// Push a sample node app

$ git clone https://github.com/deis/example-nodejs-express
In case you get logged out:
deis login http://deis.tsla.168.62.238.12.xip.io

$ deis create
Creating Application... done, created quoted-rucksack
Git remote deis added
remote available at ssh://git@deis.40.78.68.187.xip.io:2222/quoted-rucksack.git

$ git remote -v
deis	ssh://git@deis.40.78.68.187.xip.io:2222/quoted-rucksack.git (fetch)
deis	ssh://git@deis.40.78.68.187.xip.io:2222/quoted-rucksack.git (push)
origin	https://github.com/deis/example-nodejs-express.git (fetch)
origin	https://github.com/deis/example-nodejs-express.git (push)

$ git push deis master
The authenticity of host '[deis.tsla.168.62.238.12.xip.io]:2222 ([168.62.238.12]:2222)' can't be established.
RSA key fingerprint is f8:f5:0b:b8:22:d2:ad:58:7a:26:50:40:31:34:ff:36.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[deis.tsla.168.62.238.12.xip.io]:2222,[168.62.238.12]:2222' (RSA) to the list of known hosts.
Counting objects: 184, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (82/82), done.
Writing objects: 100% (184/184), 33.35 KiB | 0 bytes/s, done.
Total 184 (delta 99), reused 184 (delta 99)
-----> Node.js app detected
       
-----> Creating runtime environment
       
       NPM_CONFIG_LOGLEVEL=error
       NPM_CONFIG_PRODUCTION=true
       NODE_ENV=production
       NODE_MODULES_CACHE=true
       
-----> Installing binaries
       engines.node (package.json):  4.2.x
       engines.npm (package.json):   unspecified (use default)
       
       Resolving node version 4.2.x via semver.io...
       Downloading and installing node 4.2.3...
       Using default npm version: 2.14.7
       
-----> Restoring cache
       Skipping cache (new runtime signature)
       
-----> Building dependencies
       Pruning any extraneous modules
       Installing node modules (package.json)
       express@4.13.3 node_modules/express
       ├── escape-html@1.0.2
       ├── merge-descriptors@1.0.0
       ├── array-flatten@1.1.1
       ├── cookie@0.1.3
       ├── utils-merge@1.0.0
       ├── cookie-signature@1.0.6
       ├── content-type@1.0.1
       ├── methods@1.1.1
       ├── vary@1.0.1
       ├── fresh@0.3.0
       ├── range-parser@1.0.3
       ├── content-disposition@0.5.0
       ├── etag@1.7.0
       ├── path-to-regexp@0.1.7
       ├── serve-static@1.10.0
       ├── parseurl@1.3.0
       ├── depd@1.0.1
       ├── qs@4.0.0
       ├── finalhandler@0.4.0 (unpipe@1.0.0)
       ├── on-finished@2.3.0 (ee-first@1.1.1)
       ├── debug@2.2.0 (ms@0.7.1)
       ├── proxy-addr@1.0.10 (forwarded@0.1.0, ipaddr.js@1.0.5)
       ├── send@0.13.0 (destroy@1.0.3, statuses@1.2.1, ms@0.7.1, mime@1.3.4, http-errors@1.3.1)
       ├── accepts@1.2.13 (negotiator@0.5.3, mime-types@2.1.8)
       └── type-is@1.6.10 (media-typer@0.3.0, mime-types@2.1.8)
       
-----> Caching build
       Clearing previous node cache
       Saving 1 cacheDirectories (default):
       - node_modules
       
-----> Build succeeded!
       └── express@4.13.3
       
-----> Discovering process types
       Procfile declares types -> web
       Default process types for Node.js -> web
-----> Compiled slug size is 12M

-----> Building Docker image
remote: Sending build context to Docker daemon 12.06 MB
remote: build context to Docker daemon 
Step 0 : FROM deis/slugrunner
# Executing 3 build triggers
Trigger 0, RUN mkdir -p /app
Step 0 : RUN mkdir -p /app
 ---> Running in 301ef8ba913a
Trigger 1, WORKDIR /app
Step 0 : WORKDIR /app
 ---> Running in 7061120cee59
Trigger 2, ADD slug.tgz /app
Step 0 : ADD slug.tgz /app
 ---> f48e75e4b104
Removing intermediate container 301ef8ba913a
Removing intermediate container 7061120cee59
Removing intermediate container 8c4b195916c7
Step 1 : ENV GIT_SHA 85274ee71c0df4ab51b7aef1aaa47e9af59c1d56
 ---> Running in 6341d06680e1
 ---> 1e46a2af437d
Removing intermediate container 6341d06680e1
Successfully built 1e46a2af437d
-----> Pushing image to private registry

-----> Launching... 
       done, syrupy-traverse:v2 deployed to Deis

       http://syrupy-traverse.tsla.168.62.238.12.xip.io

       To learn more, use `deis help` or visit http://deis.io

To ssh://git@deis.tsla.168.62.238.12.xip.io:2222/syrupy-traverse.git
 * [new branch]      master -> master


------- if you see this error --------------
deis git push ssh: connect to host
https://github.com/deis/deis/issues/4495

- try to deisctrl restart builder
- if that does not work, deisctrl restart controller
- if either gets stuck, ssh core@ip_of_deisnode
- docker ps
$ etcdctl ls /deis/builder/
/deis/builder/sshHosted25519Key
/deis/builder/host
/deis/builder/port
/deis/builder/users
/deis/builder/sshHostrsaKey
/deis/builder/sshHostdsaKey
/deis/builder/sshHostecdsaKey
- back on laptop: deisctrl restart builder
docker ps in deishost should have created a new container for builder
- deis login...
- deis create


deis logs
deis run ls
deis releases
deis domains:add orderflow.tsla.40.121.144.218.xip.io
deis scale web=2

