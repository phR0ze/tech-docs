# Cloud Run
Cloud Run is one of Google's serverless offerings. It is fully managed, meaning that Google handles 
the infrastructure including scaling up and scaling down. You provide the container and Cloud Run 
provides the rest.

## Overview
* Pay per use
  * Free tier
* Simple and automated
* No infra management
* Increased developer velocity
* Accepts container or source
* Source will be built using Cloud Build and a generated Dockerfile
* Built in support for CI and can wire up your Git repository such that commits will kick off a build 
and deploy the new code immediately

### Cloud Run Integrations
New feature in preview allows for `1-click` integrations with other Google managed services like 
`Firebase`.

Cloud Run's automatically available generated public URL works but aren't pretty. Using the 1-click 
integrations feature to add the ***Firebase Hosting*** integration to pick a `NAME.web.app` subdomain 
and then click Submit. By clicking through to the Firebase control center you could also setup a 
fully custom domain name as well.

## Services
Automatically scaled request-driven services. Typically HTTP microservices

* Out of the box Public URL with TLS
  * Behind Google's global load balancer
  * Need to disable this if you don't want it public
* Scales down to zero when not being used
  * Can set to always have 1 instance for performance
* Built in traffic splitting for gradual rollout
  * Allows for blue/green deployments out of the box
* Can be triggered by events, websockets, HTTP/2 & gRPC
* Pay per request, or per instance lifetime

## Jobs
Set of containers which "run to completion"

* No requirement for HTTP
* Runs a specified number of tasks (instances)
  * Many can run in parallel
* Executed manually, or on a schedule
* Pay only while the job is executing

## Firebase
Firebase is Google's app development platform for building and hosting apps and games.

* Vanity `NAME.web.app` subdomain and free custom domains
* Access to a global CDN that requires effectively no additional configuration
