# Cloud Build
<img align="left" width="48" height="48" src="../data/images/logo_256x256.png">
[Cloud Build](https://cloud.google.com/build/docs) is Google's CI/CD offering

## Overview
Builds are a series of build steps each of which is run in a Docker container. A build step can do 
anything that can be done from a container irrespective of the environment. To perform your tasks, 
you can either use the supported build steps provided by Cloud Build or write your own build steps.

By default Cloud Build runs in a fully managed environment with access to the public internet. Each 
build runs on its own worker and is isolated from other workloads. 

### Pricing
***GOOGLE CLOUD BUILD doesn't offer a self-hosted variant which makes its pricing cost prohibitive***

A build-minute is incured for every minute that a build initiated by Cloud Build is in process. 
Build-minutes are not incurred for queuing time.

First 2,500 build-minutes are free for the `e2-standard-2 2CPU 8GB` instances.

### Private Pools
***Private pools*** are private dedicated workers that offer greater customization, but are still 
hosted and fully-managed by Cloud Build. They are also more expensive due to the ability to customize 
them.

### Builder images
You can build a custom builder image with custom tools installed to be used by Cloud Build. 
Additionally there is a large list of community contributed builders available as well.

