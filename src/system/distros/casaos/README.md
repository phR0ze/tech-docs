# CasaOS <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

[CasaOS](https://github.com/IceWhaleTech/CasaOS) is a modern take on an operating system that focuses 
on a homelab private cloud type experience. It is a opensource and provides an elegant web interface 
and a slick management interface for deploying Docker based applications from an app store. These 
deployed applications are then added to the web interface in its integrated app dashboard.

What's more it is able to be deployed as a set of applications directly mostly any Linux based 
distribution with Debian 12 being the recommended base. The benefits of this are that it is able to 
avoid the Docker in Docker performance degradation typical of most most orchestration tools.

### Quick links
* [CasaOS Overview](#casaos-overview)
  * [Install on Ubuntu](#install-on-ubuntu)

## CasaOS Overview

### Install on Ubuntu
1. Create an Ubuntu 22.04 LTS VM
2. Update, upgrade and install utilities
   ```bash
   $ sudo apt install curl vim
   $ sudo apt update && apt upgrade -y
   ```
3. Run the CasaOS curl bash line
   ```bash
   $ curl -fsSL https://get.casaos.io | sudo bash
   ```
   * smartmontools net-tools udevil samba cifs-utils mergerfs unzip docker 
   * CasaOS will be served from your IP e.g. `http://192.168.1.200`
4. Login and create your account

### Add additional app stores
CasaOS has the ability to add in other app stores simply and easily via their UI.

1. Launch the `App Store`
2. Click the `apps` drop down on the right and choose `More`
3. The launched website will show a list of alternative app stores
4. Back in the `App Store` you can select `More apps` in that same location
5. Then paste in a new app store location to add more stores

### Test apps
* Portainer

