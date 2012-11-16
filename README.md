# bosh-example (in progress) #

This example will walk through the steps necessary to use BOSH with Cinderella. This is not a best practice but instead is a proof of concept to illustrate Cinderella's EC2 capabilities.

## What is BOSH? ##

Cloud Foundry BOSH is an open source tool chain for release engineering, deployment and lifecycle management of large scale distributed services.

## Prerequisites ##

1. An AWS (Amazon Web Services) account.
2. A [CloudFoundry.com](http://cloudfoundry.com) account.
3. A vCloud Director org with credentials.
4. A Mac or *nix computer with the following installed: 
    * Ruby 1.9+ and rubygems 1.8+
    * Java 1.6+
    * Maven 3.0.4+

## Overview ##

![Interaction of Components](https://github.com/cinderella/bosh-example/raw/master/bosh-example.png)

## BOSH CLI Setup ##

From command line:
```
$ gem install bosh_deployer
```
To verify the install, 
```
$ bosh help
```

## Micro BOSH Setup ##

The Micro BOSH instance will be setup on EC2. 

Create a key pair and name it "bosh".

Create a security group called "bosh" with the following opened ports:

* 22
* 2825
* 4222
* 6868
* 25250
* 25555
* 25777
* 25888

Allocate an Elastic IP for the micro bosh instance.

Third, create supporting directory structure:

```
$ mkdir ~/deployments
$ cd ~/deployments
$ mkdir aws
```

Create `~/deployments/aws/micro_bosh.yml` using the following template:

```
---
name: aws

logging:
  level: DEBUG

network:
  type: dynamic
  vip: x.x.x.x

resources:
  persistent_disk: 20000
  cloud_properties:
    instance_type: m1.small
    availability_zone: sa-east-1a

cloud:
  plugin: aws
  properties:
    aws:
      access_key_id: YOUR_ACCESS_KEY_ID
      secret_access_key: YOUR_SECRET_ACCESS_KEY
      default_key_name: bosh
      default_security_groups: ["bosh"]
      ec2_private_key: ~/.ssh/bosh
      ec2_endpoint: ec2.sa-east-1.amazonaws.com

apply_spec:
  agent:
    blobstore:
      address: x.x.x.x
    nats:
      address: x.x.x.x
  properties:
    aws_registry:
      address: x.x.x.x
```
Make the following changes to the template:

1. Replace all instances of 'x.x.x.x' with the Elastic IP address you allocated above.
2. Update YOUR_ACCESS_KEY_ID and YOUR_SECRET_ACCESS_KEY to match your AWS credentials.
3. Update the default_key_name and ec2_private_key to match yours.
4. Update the availability zone and endpoint settings to match your availability zone. For instance, if you deploy to us-east-1 you should set the availability zone to either us-east-1a or us-east-1b.

At this point, you're ready to start the deployment of the micro bosh to AWS. Make sure you're in the deployments directory you created above:

```
$ cd ~/deployments
```

Select the deployment you've created:
```
bosh micro deployment aws
```

*Note:* don’t be concerned by seemingly inaccurate message WARNING! Your target has been changed to `http://aws:25555!



Start the deployment using the AMI from the BOSH AMIs section below:

```
bosh micro deploy ami-xxxxxxxx
```

* ap-northeast-1: ami-7656eb77
* ap-southeast-1:	ami-64d59436
* eu-west-1:	ami-874c4af3
* sa-east-1:	ami-6280597f
* us-east-1:	ami-69dd6900
* us-west-1:	ami-4f3e1a0a
* us-west-2:	ami-7ac7494a

Within 20 minutes your instance of micro BOSH will be deployed. After the ‘Done’ message appears, you have a running micro BOSH instance.

Target the newly deployed micro bosh, again substituting 'x.x.x.x' with your Elastic IP address:
```
$ bosh target http://x.x.x.x:25555
```

Login to the micro bosh using the default credentials:
```
$ bosh login
```

Then get the status of the micro bosh:
```
$ bosh status
```
## Cinderella Setup ##

Follow the Getting Source, Build and Configuration steps in the [Cinderella README.md](https://github.com/cinderella/cinderella/blob/master/readme.md)

Once you've successfully built Cinderella you can deploy to CloudFoundry.

First, install the command line client for CloudFoundry using the instructions here: [Installing the Command-Line Interface (vmc)](http://docs.cloudfoundry.com/tools/vmc/installing-vmc.html)

Deploy Cinderella to CloudFoundry:
```
$ cd /path/to/cinderella
$ vmc push YOUR_APP_NAME
Would you like to deploy from the current directory? [Yn]: n
Deployment path: cinderella-web/target/cinderella.jar
Detected a Standalone Application, is this correct? [Yn]: y
1: java
2: java7
3: node
4: node06
5: node08
6: ruby18
7: ruby19
Select Runtime [java]: 1
Selected java
Start Command: java $JAVA_OPTS -jar cinderella.jar
Application Deployed URL [None]: YOUR_APP_NAME.cloudfoundry.com
Memory reservation (128M, 256M, 512M, 1G, 2G) [512M]: 1G
How many instances? [1]: 1
Create services to bind to 'cinderella2'? [yN]: n
Would you like to save this configuration? [yN]: n
Creating Application: OK
Uploading Application:
  Checking for available resources: OK
  Processing resources: OK
  Packing application: OK
  Uploading (0K): OK   
Push Status: OK
Staging Application 'YOUR_APP_NAME': OK                                           
Starting Application 'YOUR_APP_NAME': OK 
```

Add environment variables required by Cinderella (these should match the properties in `~/.cinderella/ec2-service.properties`):
```
$ vmc env-add YOUR_APP_NAME aws_key_YOUR_AWS_ACCESS_KEY=YOUR_AWS_SECRET_KEY
$ vmc env-add YOUR_APP_NAME vcd_endpoint=YOUR_VCD_ENDPOINT
$ vmc env-add YOUR_APP_NAME vcd_useratorg=YOUR_VCD_USERATORG
$ vmc env-add YOUR_APP_NAME vcd_password=YOUR_VCD_PASSWORD
$ vmc env-add YOUR_APP_NAME vcd_network=YOUR_VCD_NETWORK
```

If everything went smoothly, you should be able to open `http://YOUR_APP_NAME.cloudfoundry.com/` in your browser and you'll see:
```
OK!
```

You can also test with an EC2 call:
```
$ ec2-describe-availability-zones -U http://YOUR_APP_NAME.cloudfoundry.com/api/ -O YOUR_AWS_ACCESS_KEY -W YOUR_AWS_SECRET_KEY -v --debug --request-timeout 120
```

## Update Micro BOSH

In order to leverage Cinderella's orchestration of vCloud resources, we need to update the micro bosh and point it to Cinderella instead of the existing EC2 endpoint.

Start by ssh'ing to the micro bosh running in EC2:
```
$ ssh -i ~/.ssh/YOUR_KEYPAIR_NAME.pem vcap@YOUR_ELASTIC_IP
```
Change to root user:
```
$ sudo su -
```
The default root password is `C10udc0w`

Now edit `/var/vcap/jobs/director/config/director.yml.erb` and update the "cloud:" section at the bottom:
```
cloud:

  plugin: aws
  properties:
    agent:
      ntp: []
      blobstore:
        plugin: simple
        properties:



          endpoint: 'http://YOUR_ELASTIC_IP:25250'
          user: agent
          password: agent


      mbus: nats://nats:nats@YOUR_ELASTIC_IP:4222

    aws:
      access_key_id: YOUR_AWS_ACCESS_KEY
      secret_access_key: YOUR_AWS_SECRET_KEY
      ec2_endpoint: YOUR_APP_NAME.cloudfoundry.com/api/
      default_key_name: YOUR_KEYPAIR_NAME
      default_security_groups: []
    registry:
      endpoint: http://YOUR_ELASTIC_IP:25777
      user: admin
      password: admin
    stemcell:
      kernel_id: 
```

Save your changes, then restart the director: 
```
$ monit restart director
```

At this point, the BOSH commands you issue from the cmd line will call into Cinderella with EC2 calls and in turn, issue vCloud Director API calls to your vCloud.

## Deploy an application to vCloud

TODO

## References

* [BOSH Documentation](https://github.com/cloudfoundry/oss-docs/blob/master/bosh/documentation/documentation.md)
* [Deploying to AWS Using Cloud Foundry BOSH](http://blog.cloudfoundry.org/2012/09/06/deploying-to-aws-using-cloud-foundry-bosh/)
* [Installing the Command-Line Interface (vmc)](http://docs.cloudfoundry.com/tools/vmc/installing-vmc.html)






