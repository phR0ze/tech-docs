# Orbstack <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

OrbStack claims to b ea fast, light and easy way to run Docker containers and Linux VMs on MacOS. Its 
meant to be a Docker Desktop alternative.

### Quick links
- [.. up dir](../README.md)

## Orb cli
OrbStack has a cli `orbctl` aliased as `orb` that can be use to control OrbStack programmatically.

### Run a script in a VM
Target a VM by name using `-m` and pass in your local script

### Attach to the VM in a shell
You can simply click it in the UI or run `orb shell -m test-vm`

## VM Creation

### Example setting up Docker on Ubuntu
1. Create VM via the shell
   ```bash
   $ orb create --arch arm64 ubuntu test-vm
   ```
2. Shell into the new machine
   ```bash
   $ orb shell -m test-vm
   ```
3. Install dependencies
   ```bash
   # Install general dependencies
   sudo apt update && sudo apt-get install -y \
     ca-certificates \
     curl \
     gh \
     acl \
     apt-transport-https \
     ca-certificates \
     gnupg
   
   # Configure Docker repo
   sudo install -m 0755 -d /etc/apt/keyrings
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
   sudo chmod a+r /etc/apt/keyrings/docker.asc
   
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt-get update
   
   # Install Docker from repo
   sudo apt-get install -y \
     docker-ce \
     docker-ce-cli \
     containerd.io \
     docker-buildx-plugin \
     docker-compose-plugin
   
   # Setup socket permissions and validate working
   sudo chmod 0666 /var/run/docker.sock
   
   # Install docker-compose
   sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   
   # Install gcloud tooling
   curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg
   echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
   sudo apt-get update && sudo apt-get install -y google-cloud-cli
   ```
