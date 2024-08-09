# IAP
Identity-Aware Proxy is Google's control layer for accessing cloud resources. It supports verifying 
a user's idenity and context to determine if a user should be granted access. It allows for working 
from untrusted networks without the use of VPNs. It is a zero-trust access model.

**Key Features**
* Centralized access control - IAP provides a single point of control for managing user access to web 
  applications and cloud resources
* Works with cloud and on-premises apps - IAP can protect access to applications hosted on Google 
  Cloud, other clouds and on-premises
* TCP Forwarding - IAP can protect SSH and RDP access to your VMs hosted on Google Cloud and your VM instances won't 
  event need public IP addresses

## Using IAP for TCP Forwarding
IAP's TCP forwarding feature allows user sto connect to arbitrary TCP ports on Compute Engine 
instances. For general TCP traffic, IAP creates a listening port on the local host that forwards all 
traffic to a specified instance. IAP then wraps all traffic from the client in HTTPS. Users gain 
access to the interface and port if they pass the authentication and authorization check of the 
target resource's Identity and Access Management (IAM) policy.

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
