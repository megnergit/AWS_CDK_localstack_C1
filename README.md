# Let us practice cdk with localstack 

<!--==============================================-->

## Goal

... is to understand how AWS cdk works.

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
>  docker run \
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

### Prepare project

We will initialize the project, but first get into our 
working directory.

```
root@acf412775d98:~# cd /aws
root@acf412775d98:/aws# ls
```
This must be an empty directory. 

Then initialize the project.

```
root@acf412775d98:/aws# c init --language typescript
```
```c``` is aliased to ```cdklocal``` in the previous section.
Other languages one can use are

```
> c init help

....
 -l, --language           The language to be used for the new project (default
                           can be configured in ~/.cdk.json)
    [string] [choices: "csharp", "fsharp", "go", "java", "javascript", "python",
                                                                   "typescript"]
                                                                   ```

Now we should see the following directories created in our working directory. 

```
root@acf412775d98:/aws# ls -ltr
total 176
-rw-r--r--   1 root root    536 Jul 28 14:14 README.md
drwxr-xr-x   3 root root     96 Jul 28 14:14 bin
-rw-r--r--   1 root root    157 Jul 28 14:14 jest.config.js
drwxr-xr-x   3 root root     96 Jul 28 14:14 lib
drwxr-xr-x   3 root root     96 Jul 28 14:14 test
-rw-r--r--   1 root root    663 Jul 28 14:14 tsconfig.json
-rw-r--r--   1 root root   3490 Jul 28 14:14 cdk.json
-rw-r--r--   1 root root    518 Jul 28 14:14 package.json
-rw-r--r--   1 root root 157433 Jul 28 14:15 package-lock.json
drwxr-xr-x 224 root root   7168 Jul 28 14:15 node_modules
drwxr-xr-x   7 root root    224 Jul 28 14:27 cdk.out
```

Next we will do __bootstrap__. 
This step do 3 things. 

1. create an S3 bucket to store cdk files
2. create ESR repository
3. create IAM roles

I thought it will create a cloudfront instance, but no explanation included in the [official page](https://docs.aws.amazon.com/cdk/v2/guide/bootstrapping.html). 

__We do not create those resources actually on AWS.__
Here is the beauty of localstack. Localstack mocks as if 
those resources are created. 


### Edit .ts file

The important files are 

* ```bin/aws.ts``` and 
* ```lib/aws-stack.ts```

The ```aws.ts``` defines an __app__. app repreents the whole project. ```aws.ts``` calls ```aws-stack.ts```. ```aws-stacks.ts``` defines what resources we will deploy and how. So, let us work now with ```aws-stack.ts```.

```
root@acf412775d98:/aws/lib# ls
aws-stack.ts
```

oot@acf412775d98:/aws/lib# \cat aws-stack.ts
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3'
// import * as sqs from 'aws-cdk-lib/aws-sqs';

export class AwsStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // The code that defines your stack goes here

    // example resource
    // const queue = new sqs.Queue(this, 'AwsQueue', {
    //   visibilityTimeout: cdk.Duration.seconds(300)
    // });

  }
}

```

The details about ```cdk``` syntax should be found elsewhere. 
Here only one thing: __do not change the stack name__ ```AwsStack```. This is the 
name that __app__ calls the stack (check ```./bin/aws.ts```). 
If you change ```AwsStack```, __app__ will get lost what to call. 


### Play with cdk

Let us check a few things. 
Run the commands everytime at the working directory ```/aws```.

First check if we have a stack.
```
root@acf412775d98:/aws# c ls
`cdk synth` may hang in Docker on Linux 5.6-5.10. See https://github.com/aws/aws-cdk/issues/21379 for workarounds.
AwsStack
```

Yes, we have ```AwsStack``` already there. 

Next check if we have s3 resources (to be created in the next steps).

```
root@acf412775d98:/aws# a s3api list-buckets
{
    "Buckets": [
        {
            "Name": "cdk-xxxxxxxxxxx"
            "CreationDate": "2024-07-28T14:16:14.000Z"
        }
    ],
    "Owner": {
        "DisplayName": "webfile",
        "ID": "xxxxxxxxdx"
    }
}
```

The first bucket is the one that we created at bootstrapping. 

Let us deploy empty stack. 
```
root@acf412775d98:/aws# c deploy
`cdk synth` may hang in Docker on Linux 5.6-5.10. See https://github.com/aws/aws-cdk/issues/21379 for workarounds.

✨  Synthesis time: 10.9s

AwsStack: deploying... [1/1]

 ✅  AwsStack (no changes)

✨  Deployment time: 0.02s

Stack ARN:
arn:aws:cloudformation:xxxxxxxx:000000000000:stack/AwsStack/xxxxxxxx

✨  Total time: 10.92s


root@acf412775d98:/aws#
```



Let us check if the resourcesl

```
root@acf412775d98:/aws# c deploy
`cdk synth` may hang in Docker on Linux 5.6-5.10. See https://github.com/aws/aws-cdk/issues/21379 for workarounds.

✨  Synthesis time: 10.9s

AwsStack: deploying... [1/1]

 ✅  AwsStack (no changes)

✨  Deployment time: 0.02s

Stack ARN:
arn:aws:cloudformation:eu-central-1:000000000000:stack/AwsStack/a5901e24

✨  Total time: 10.92s
```

Check the resources, 
```
root@acf412775d98:/aws# a cloudformation list-stack-resources --stack-name AwsStack
{
    "StackResourceSummaries": [
        {
            "LogicalResourceId": "CDKMetadata",
            "PhysicalResourceId": "AwsStack-CDKMetadata-c29623e9",
            "ResourceType": "AWS::CDK::Metadata",
            "LastUpdatedTimestamp": "2024-07-28T14:27:39.474000Z",
            "ResourceStatus": "UPDATE_COMPLETE",
            "DriftInformation": {
                "StackResourceDriftStatus": "NOT_CHECKED"
            }
        }
    ]
}
root@acf412775d98:/aws#
```

All right, still no resources deployed except stack itself. 


### Now deploy S3 bucket

Add s3 __construct__ as follows. 
```
root@acf412775d98:/aws# \cat lib/aws-stack.ts
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3'
// import * as sqs from 'aws-cdk-lib/aws-sqs';

export class AwsStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // The code that defines your stack goes here

    // example resource
    // const queue = new sqs.Queue(this, 'AwsQueue', {
    //   visibilityTimeout: cdk.Duration.seconds(300)
    // });

// FROM HERE
    const mys3 = new s3.Bucket(this, 'FirstBucket', {
      bucketName: 'firstbucket-28418'
    }) ;

// TO HERE

  }
}
```
(Note: because we mounted the working directory to the container, we can edit the file from outside the container.)

Then deploy it.
```
root@acf412775d98:/aws# c deploy
`cdk synth` may hang in Docker on Linux 5.6-5.10. See https://github.com/aws/aws-cdk/issues/21379 for workarounds.

✨  Synthesis time: 9.57s

AwsStack: deploying... [1/1]
AwsStack: creating CloudFormation changeset...

 ✅  AwsStack

✨  Deployment time: 5.07s

Stack ARN:
arn:aws:cloudformation:eu-central-1:000000000000:stack/AwsStack/xxxxxxxx

✨  Total time: 14.64s

root@acf412775d98:/aws#
```

All right. Let us check if the S3 resource is there.

```
root@acf412775d98:/aws# a s3api list-buckets
{
    "Buckets": [
        {
            "Name": "cdk-hnb659fds-assets-000000000000-eu-central-1",
            "CreationDate": "2024-07-28T14:16:14.000Z"
        },
        {
            "Name": "firstbucket-28418",
            "CreationDate": "2024-07-28T16:53:53.000Z"
        }
    ],
    "Owner": {
        "DisplayName": "webfile",
        "ID": "xxxxxxxxxxxxxx"
    }
}
```

Yes, now 'firstbucket-28418' is there. Check cloudformation as well. 

```
root@acf412775d98:/aws# a cloudformation list-stack-resources --stack-name AwsStack
{
    "StackResourceSummaries": [
        {
            "LogicalResourceId": "CDKMetadata",
            "PhysicalResourceId": "AwsStack-CDKMetadata-c29623e9",
            "ResourceType": "AWS::CDK::Metadata",
            "LastUpdatedTimestamp": "2024-07-28T16:53:53.215000Z",
            "ResourceStatus": "UPDATE_COMPLETE",
            "DriftInformation": {
                "StackResourceDriftStatus": "NOT_CHECKED"
            }
        },
        {
            "LogicalResourceId": "FirstBucket8E7B2622",
            "PhysicalResourceId": "firstbucket-28418",
            "ResourceType": "AWS::S3::Bucket",
            "LastUpdatedTimestamp": "2024-07-28T16:53:53.238000Z",
            "ResourceStatus": "CREATE_COMPLETE",
            "DriftInformation": {
                "StackResourceDriftStatus": "NOT_CHECKED"
            }
        }
    ]
}
```

Yes, S3 bucket is registered to cloudformation as well. 

### Delete resource

In order to delete resource, we do __not__ destroy them, but __deploy empty stack__. 

So, remove S3 part from ```./lib/aws-stack.ts```.
```
root@acf412775d98:/aws# cat lib/aws-stack.ts
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3'
// import * as sqs from 'aws-cdk-lib/aws-sqs';

export class AwsStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // The code that defines your stack goes here

    // example resource
    // const queue = new sqs.Queue(this, 'AwsQueue', {
    //   visibilityTimeout: cdk.Duration.seconds(300)
    // });

    /* LIKE THIS
    const mys3 = new s3.Bucket(this, 'FirstBucket', {
      bucketName: 'firstbucket-28418',
      versioned: false
    }) ;
   */

  }
}
```

Before deploying it, let us see what will happen

```
root@acf412775d98:/aws# c diff
`cdk synth` may hang in Docker on Linux 5.6-5.10. See https://github.com/aws/aws-cdk/issues/21379 for workarounds.
Stack AwsStack
Hold on while we create a read-only change set to get a diff with accurate replacement information (use --no-change-set to use a less accurate but faster template-only diff)
Could not create a change set, will base the diff on template differences (run again with -v to see the reason)
Resources
[-] AWS::S3::Bucket FirstBucket FirstBucket8E7B2622 orphan


✨  Number of stacks with differences: 1

```

cdk is saying that it will remove one S3 buekct.

Then we will actually delete it by deploying empty stack.

```
root@acf412775d98:/aws# c deploy
`cdk synth` may hang in Docker on Linux 5.6-5.10. See https://github.com/aws/aws-cdk/issues/21379 for workarounds.

✨  Synthesis time: 9.38s

AwsStack: deploying... [1/1]
AwsStack: creating CloudFormation changeset...

 ✅  AwsStack

✨  Deployment time: 5.11s

Stack ARN:
arn:aws:cloudformation:eu-central-1:000000000000:stack/AwsStack/a5901e24

✨  Total time: 14.49s

```

Check ```s3api``` and ```cloudformation```.

```
> a s3api list-buckets
....

> a cloudformation list-stack-resources --stack-name AwsStack
```

All right. 

<!--==============================================-->
## Files 

I will put only ```./lib/aws-stack.ts``` in the repository. 
All the else will be created automatically when you run ```cdklocal init```.

------
# END