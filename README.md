# Let us practice cdk with localstack 

<!--==============================================-->

## Goal is 

to understand how AWS cdk works.

In order to do so, I need to practice cdk as many times as possible.
In order to do so, I need a safe environment that would not get charged (= zero cost).
Then let us start with __localstack__, in a __containered__ environment so that we can
delete the environment clean, once we are done. 


Overview

1. set up docker.
2. start localstack in a containner.
3. create s3 bucket from there.

<!--==============================================-->

## Set up localstack

[__localstack__](https://docs.localstack.cloud/overview/) is a mock environment for AWS.

Not only cdk but also other AWS services, such as SQS, one can practice there without
actually touching AWS. It is a comvenient way to learn how AWS works.


### Docker 

We will use localstack in side a container this time. First let up set up docker on my mac.
The easiest way is to go for [docker desktop for mac](https://docs.docker.com/desktop/install/mac-install/).

* Download the package
* Install docker desktop
* Start it

That is it. Open a terminal and check


```
> docker --version
https://docs.docker.com/desktop/install/mac-install/
```

We will use docker hub. You need to register yourself there.
After registering yourself, you will log in like following.

```
> echo [your passowrd] | docker login -u [username] --password-stdin
   Login Succeeded
```   

Test it.

```
> docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete
Digest: sha256:1408fec50309afee38f3535383f5b09419e6dc0925bc69891e79d84cc4cdcec6
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```


### Start localstack in a container

First create an exercise directory (arbitrarily named 'e3' here).

```

> mkdir e3
> cd e3
>  ls
total 0

```
Let us keep the directory __empty__ (we will fill the directory at ```cdk init```, but
the command only works when the directory is empty).

Start ```docker run ``` from there.

```
meg@elias ~/aws/cdk]$ docker run \
  --rm -it -d \
  -p 127.0.0.1:4566:4566 \
  -p 127.0.0.1:4510-4559:4510-4559 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v ~/.aws:/root/.aws \
  -v $(pwd):/aws \
  localstack/localstack

```
This will

* mount your config file ```~/.aws/config``` to the root directory inside the container
* mount the current working directory ```$(pws)``` to ```/aws``` where we work most of the time.


Check if the localstack container is running.
```
> docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED       STATUS                 PORTS                                                                    NAMES
acf412775d98   localstack/localstack   "docker-entrypoint.sh"   2 hours ago   Up 2 hours (healthy)   127.0.0.1:4510-4559->4510-4559/tcp, 127.0.0.1:4566->4566/tcp, 5678/tcp   busy_cannon

```

Get into the container
```
> docker exec -it acf bash
root@acf412775d98:/opt/code/localstack# ls
Makefile  __pycache__  localstack-core	package-lock.json  pyproject.toml
VERSION   bin	       node_modules	package.json	   setup.py
root@acf412775d98:/opt/code/localstack# cd
```
You only need first few letters to identify the container.

Install packages required for ```cdk```.

```
npm install aws-cdk-local aws-cdk
```
 
Check if ```cdklocal``` (same as ```cdk``` but is used inside ```localstack```) is there.
```
root@acf412775d98:~# ls
node_modules  package-lock.json  package.json
root@acf412775d98:~# ls node_modules/.bin
cdk  cdklocal
```

Let us create an alias to it. Also to ```awslocal``` (same with ```aws``` but is used inside ```localstack```)

```
alias c=$HOME/node_modules/.bin/cdklocal
alias a=/usr/local/bin/awslocal
```
Let us check,

```
root@acf412775d98:~# c --version
2.150.0 (build 3f93027)
root@acf412775d98:~# a configure
AWS Access Key ID [****************xxxx]:
AWS Secret Access Key [****************xxxx]:
Default region name [xxxxxxxxx]:
Default output format [None]:
```

All right. Next we will work with cdk inside localstack

<!--==============================================-->
## CDK in localstack 

We will initialize the project, but first get into our 
working directory.

```
root@acf412775d98:/cd
aws# ls
```
This must be an empty directory. 















------
# END