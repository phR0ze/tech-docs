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
A TCP Tunnel can be created with IAP that allows for connectivity to private VMs. This can be done 
via `gcloud compute start-iap-tunnel` to create a tunnel through which another process can create a 
connection e.g. SSH, RDP.

* [IAP TCP Forwarding docs](https://cloud.google.com/iap/docs/tcp-forwarding-overview)
* [Independent IAPC project](https://github.com/cedws/iapc)

### gcloud tunnle examples
* [gcloud docs for iap-tunnel](https://cloud.google.com/sdk/gcloud/reference/compute/start-iap-tunnel)

* Open a tunnel to the instances RDP port 3389 on an arbitrary local port
  ```
  gcloud compute start-iap-tunnel my-instance 3389
  ```
* Open a tunnel to the instances RDP port 3389 to a specific local port
  ```
  gcloud compute start-iap-tunnel my-instance 3389 --local-host-port=localhost:3333
  ```

