# bosh-example (in progress) #

This example will walk through the steps necessary to use BOSH with Cinderella. This is not to be meant to be an example of a best practice but instead is a proof of concept to illustrate Cinderella's capabilities.

## What is BOSH? ##

Cloud Foundry BOSH is an open source tool chain for release engineering, deployment and lifecycle management of large scale distributed services.

## Prerequisites ##

1. An AWS (Amazon Web Services) account.
2. A CloudFoundry.com account.
3. A vCloud Director org with credentials.

## Overview ##

![Interaction of Components](https://github.com/cinderella/bosh-example/raw/master/bosh-example.png)


## Setup Micro BOSH ##

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







