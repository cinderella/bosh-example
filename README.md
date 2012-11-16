# bosh-example (in progress) #

This example will walk through the steps necessary to use BOSH with Cinderella. This is not to be meant to be an example of a best practice but instead is a proof of concept to illustrate Cinderella's capabilities.

## What is BOSH? ##

Cloud Foundry BOSH is an open source tool chain for release engineering, deployment and lifecycle management of large scale distributed services.

## Prerequisites ##

1. An AWS (Amazon Web Services) account.
2. A CloudFoundry.com account.
3. A vCloud Director org with credentials.
4. A Mac or *nix computer with Ruby 1.9+ and rubygems 1.8+ installed.

## Overview ##

![Interaction of Components](https://github.com/cinderella/bosh-example/raw/master/bosh-example.png)

## BOSH CLI Setup ##

From command line:
```
$ gem install bosh_deployer
```


## Micro BOSH Setup ##

The Micro BOSH instance will be setup on EC2. 

First, create a security group called "bosh" with the following opened ports:

* 22
* 2825
* 4222
* 6868
* 25250
* 25555
* 25777
* 25888

Second, allocate an Elastic IP for the micro bosh instance.

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
      access_key_id: AKIAIYJWVDUP4KRWBESQ
      secret_access_key: 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
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

Replace all instances of 'x.x.x.x' with the Elastic IP address you allocated.


## Cinderella Setup ##

TODO


## References

* [BOSH Documentation](https://github.com/cloudfoundry/oss-docs/blob/master/bosh/documentation/documentation.md)
* [Deploying to AWS Using Cloud Foundry BOSH](http://blog.cloudfoundry.org/2012/09/06/deploying-to-aws-using-cloud-foundry-bosh/)







