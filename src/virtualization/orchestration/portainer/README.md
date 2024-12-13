# Portainer <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

***Portainer.io*** is an opensource fast and easy container management GUI. They are attempting to 
make containerization management approachable to the lay person without requiring advanced container 
orchestration systems knowledge. It works with Kubernetes, Docker, Docker Swarm and Azure.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Install Portainer](#install-portainer)

## Overview

### Install in Proxmox

**References**
* [Install Docker and Portainer in LXC](https://www.cynicalsignals.com/installing-portainer-in-a-proxmox-lxc/)

1. Create a new LXC container based on Ubuntu 22.04
2. Install Docker
3. Install Portainer

### Templates
Portainer provides what is calls `templates` which can be either a single Docker file a.k.a. `container` or a Docker 
compose file i.e. `stack`. There are many templates available directly from the Portainer UI but it 
is clearly directed more at Enterprise software rather than the homelab scene. None of the 
traditional homelab applications are available by default.

Although the default templates don't provide much out of the box it can be augmented with [community 
templates](https://github.com/Lissy93/portainer-templates) to round it out.

Portainer is able to pull templates from your own custom github repository and even promotes and 
makes this path easy to do in their own documentation 
[Build and host your own app templates](https://docs.portainer.io/2.24/advanced/app-templates/build).
Portainer even guides users through how to mount the JSON templates file locally so you can modify it 
and even seen live changes. ***This is a fantastic solution for homelab or any highly customized 
solution space.***

**References**
* [Mycroftwilde - portainer templates](https://github.com/mycroftwilde/portainer_templates/tree/master/TableOfContents/Portainer)

1. Clone the portainer templates repo
   ```bash
   $ git clone https://github.com/portainer/templates
   ```
2. Edit the `templates/templates.json` file as desired
3. Update your Portainer `App Templates` URL
   1. Login to the UI
   2. Navigate to `Settings >General`
   3. In the top section change the `URL` field to something else e.g. [mycroftwilde templates](https://raw.githubusercontent.com/mycroftwilde/portainer_templates/master/Template/template.json)

