# bosh-example (in progress) #

This example will walk through the steps necessary to use BOSH with Cinderella. This is not to be meant to be an example of a best practice but instead is a proof of concept to illustrate Cinderella's capabilities.

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

Create ~/deployments/aws/micro_bosh.yml using the following template:

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



## Cinderella Setup ##

1. Follow the Getting Source, Build and Configuration steps in the [Cinderella README.md](https://github.com/cinderella/cinderella/blob/master/readme.md)


## References

* [BOSH Documentation](https://github.com/cloudfoundry/oss-docs/blob/master/bosh/documentation/documentation.md)
* [Deploying to AWS Using Cloud Foundry BOSH](http://blog.cloudfoundry.org/2012/09/06/deploying-to-aws-using-cloud-foundry-bosh/)







