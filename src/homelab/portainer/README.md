# Portainer <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

***Portainer.io*** is an opensource fast and easy container management GUI. They are attempting to 
make containerization management approachable to the lay person without requiring advanced container 
orchestration systems knowledge. It works with Kubernetes, Docker, Docker Swarm and Azure.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Install on NixOS](#install-on-nixos)
  - [First run](#first-run)
  - [Templates](#templates)

## Overview

**References**
* [selfhosted templates](https://github.com/SelfhostedPro/selfhosted_templates)
* [500+ 1 click Portainer templates](https://github.com/Lissy93/portainer-templates)
* [Go to for portainer templates](https://portainer-templates.netlify.app/)
* [Qballjos templates](https://github.com/Qballjos/portainer_templates)

### Install on NixOS
Why install on NixOS when NixOS has arguably a better solution. Yes NixOS's modular declarative 
deployments gives you the ability to build a server and services in a repeatable way which solves 
this better and yes NixOS has declarative container solutions that are better for production but what 
Portainer brings is a avid community and widespread acceptance outside NixOS that makes pulling 
solutions off the shelf super fast and simple. Really this means the best of both worlds. I use 
Portainer to quickly test new software out in an isolated VM and then if I like it I can build out a 
production version in NixOS directly without Portainer.

[see portainer.nix](https://github.com/phR0ze/nixos-config/blob/main/options/services/cont/portainer.nix)

### First run

1. Navigate to `localhost:9000`
2. Create a new admin account e.g. `admin/insecure`
3. Add additional `App Templates`
   1. Navigate to `Settings >General >Application Settings >App Templates`
   2. Paste in your template `URL` e.g. `https://raw.githubusercontent.com/phR0ze/portainer-templates/main/templates_v3.json`
   3. Click `Save application settings` and repeat as desired
4. Navigate back to `Home` and select your `podman` local environment
5. Under `Templates >Application` you should now see a wide variety of options to install 

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

Note: most template repos use V2 but when V3 came out they started to fail:
* [Bug report on portainer](https://github.com/portainer/portainer/issues/12337)
* [Fix for app templates](https://github.com/Lissy93/portainer-templates/pull/67)
