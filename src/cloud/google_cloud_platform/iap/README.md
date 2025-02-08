# IAP <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Identity-Aware Proxy is Google's control layer for accessing cloud resources. It supports verifying 
a user's identity and context to determine if a user should be granted access. It allows for working 
from untrusted networks without the use of VPNs. It is a zero-trust access model.

**Key Features**
* Centralized access control - IAP provides a single point of control for managing user access to web 
  applications and cloud resources
* Works with cloud and on-premises apps - IAP can protect access to applications hosted on Google 
  Cloud, other clouds and on-premises
* TCP Forwarding - IAP can protect SSH and RDP access to your VMs hosted on Google Cloud and your VM instances won't 
  event need public IP addresses

### Quick links
* [Overview](#overview)
* [Programmatic authentication](#programmatic-authentication)
* [Getting user identity](#getting-user-identity)
* [Enabling IAP for GKE](#enabling-iap-for-gke)
* [TCP Forwarding](#tcp-forwarding)

## Overview
Google’s Identity-Aware Proxy (IAP) acts as a crucial gatekeeper for applications and resources 
deployed on Google Cloud Platform (GCP), such as those within Google App Engine. By leveraging user 
identity and request context, IAP ensures that only authorized users can access your deployed 
services.

When a request is made to an IAP-protected resource, IAP intercepts the request and forwards it to 
Google's authentication system. This step verifies the identity of the user or service making the 
request. Following successful authentication, IAP evaluates the configured access policies to decide 
if the request should be granted access. This decision is based on the user's identity and the 
specifices of their request.

IAP employs a unique bearer token called the `IAP bearer token` that is different than standard
OAuth2 tokens which are used throughout GCP resources.

**References**
* [IAP - GCP console](https://console.cloud.google.com/security/iap)

## Programmatic authentication
IAP employs a unique bearer token called the `IAP bearer token` that is different than standard
OAuth2 tokens which are used throughout GCP resources. To generate an IAP bearer token, you need the 
OAuth2 client ID associated with the IAP-protected resource (this is the OAuth2 client that was 
created when the IAPI was activated). With the IAP bearer token, the server process can authenticate 
its requests to the IAP-protected resource. The token is included in the Authorization heder of the 
HTTP request as a bearer token.

**References**
* [Programmatic authentication - gcp docs](https://cloud.google.com/iap/docs/authentication-howto)
* [IAP protected resources - medium](https://medium.com/google-cloud/access-google-cloud-iap-protected-resources-via-cli-e04e26475485)
* [IAP token - git-remote-https-iap](https://github.com/adohkan/git-remote-https-iap/tree/master/internal/iap)

### Create a OAuth 2.0 client
IAP needs an OAuth 2.0 client of type Desktop app created for API access.

* Download the OAuth client's credentials and store them as client.json

### Fetch token for user
Use the client's credentials file and SCOPES to make a token fetch call that will then receive a 
callback with the token to a local server you setup to receive it. Once the token is received then 
you can make your actual API request of your app.

* headers `Authorization: Bearer ${TOKEN}`

## Getting user identity
To ensure a request to your app was authorized by IAP, your app must validate every request by 
checking the `x-goog-iap-jwt-assertion` HTTP request header.

**References**
* [Getting the user's identity - gcp docs](https://cloud.google.com/iap/docs/identity-howto)

## Enabling IAP for GKE
IAP is integrated through Ingress for GKE. This integration enables you to control resource-level 
access for employees instead of using VPN. In a GKE cluster incoming traffice is handled by HTTP(S) 
Load Balancing, a component of Cloud Load Balancing. The HTTP(S) load balancer is typically 
configured by the `K8s Ingress controller`. The ingress controller gets configuration information 
from the K8s ingress object by associating a `Service` with a `BackendConfig` object. `BackendConfig` 
is a custom resource definition (CRD) that is defined in the kubernetes/ingress-gce repo. The K8s 
Ingress Controller reads configuration information from the BackendConfig and sets up the load 
balancer accordingly. A BackendConfig holds configuration information that enables you to define a 
separate configuration for each backend service.

**References**
* [Enabling IAP for GKE - gcp docs](https://cloud.google.com/iap/docs/enabling-kubernetes-howto)

## TCP Forwarding
IAP's TCP forwarding feature allows users to connect to arbitrary TCP ports on Compute Engine 
instances. For general TCP traffic, IAP creates a listening port on the local host that forwards all 
traffic to a specified instance. IAP then wraps all traffic from the client in HTTPS. Users gain 
access to the interface and port if they pass the authentication and authorization check of the 
target resource's Identity and Access Management (IAM) policy.

**References**
* [IAP TCP Forwarding docs](https://cloud.google.com/iap/docs/tcp-forwarding-overview)
* [Independent IAPC project](https://github.com/cedws/iapc)
* [go based gcloud replacement](https://github.com/gartnera/gcloud)

### gcloud tunnel
* [gcloud docs for iap-tunnel](https://cloud.google.com/sdk/gcloud/reference/compute/start-iap-tunnel)

1. Optionally filter on parent folder id to get projects that are part of a folder
   ```bash
   $ gcloud projects list --filter=parent.id=$PARENT_ID --format='value(project_id)'
   ```
2. Optionally list all VMs for the project filtering on a preset tag e.g. `allow-iap`
   ```bash
   $ gcloud compute instances list --filter='tags: allow-iap' --project $PROJECT_ID --format='value(name, zone)'
   mysql-vm use-east4
   ```
3. Optionally extract db access secrets for the target db vm
   ```bash
   $ gcloud secrets versions access latest --secret $SECRET_NAME --project $PROJECT_ID
   ```

4. Opening the tunnel to your mysql instance requires the `VM_NAME`, `VM_ZONE`, `VM_PROJECT`, 
   `REMOTE_PORT` and optionally the `LOCAL_PORT` to use.
   ```bash
   $ gcloud compute start-iap-tunnel --project=$VM_PROJECT $VM_NAME $REMOTE_PORT --zone=$VM_ZONE --local-host-port=localhost:$LOCAL_PORT
   ```
