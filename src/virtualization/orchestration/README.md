# Orchestration <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

With the rise of Docker and the application container paradigm the industry responded with tooling to 
manage the plethora of application containers that we now need to manage. Systems like Docker Swarm, 
and Kubernetes were created in the Enterprise space for large scale production systems while OrbStack, 
Docker Desktop sprang up in the development space and the likes of Proxmox, Incus, Yacht, Portainer, 
Dockge, and others for other needs.

### Quick links
- [.. up dir](../README.md)
- [Incus](incus/README.md)
- [Kasm](kasm/README.md)
- [Kubernetes](kubernetes/README.md)
- [Orbstack](orbstack/README.md)
- [Portainer](portainer/README.md)
- [ProxMox](proxmox/README.md)

## Comparison
Comparing orchestration tooling. I'm only looking at free options that are reasonably simple to 
configure for a `single server` home lab i.e. Kubernetes, although great requires a cluster.

Note: any solution that requires a specific operating system other than NixOS is being automatically 
dismissed e.g. `Proxmox` as the upgrade path for other distributions is fraught with peril that I no 
longer care to expose myself to.

| Tool        | VMs | Containers  | Deployment methods        |
| ----------- | --- | ----------- | ------------------------- |
| CasaOS      |     |             | Bare metal                |
| Dockge      |     |             | Compose                   |
| DockSTARTer |     |             | apk, apt, pacman pkgs     |
| Incus       | yes | app, system | Compose                   |
| NixOS       | yes | all         | Bare metal                |
| Portainer   |     |             | Docker, Swarm, K8s        |
| Yacht       |     |             | Docker, Compose, Podman   |

### First run impressions

| Feature                      | Port. | Casa. | Dkge. |  Comments
| -----------------------------| ----- | ----- | ----- | -------------------
| Elegant clean design         | yes   | yes   | yes   |
| Password setting first run   | yes   | yes   | yes   |
| Featureful                   | yes   | yes   | no    | Dockge is bare bones
| Convert docker run to compose| yes   | yes   | no    | Dockge is bare bones
| Adopt legacy apps            | no    | yes   | no    | Casa supports imports of running apps

### Template repo
Most of the docker orchestration solutions provide some kind of pre-defined list of templates or the 
ability to add addiitional pre-defined templates.

| Feature                      | Port. | Casa. | Dkge. |  Comments
| -----------------------------| ----- | ----- | ----- | -------------------
| Pre-defined template repo    | yes   | yes   | no    |
| 3rd party template repos     | yes   | yes   | no    | Casa supports many, Portainer only one at a time

### Template custom deployment
| Feature                      | Port. | Casa. | Dkge. |  Comments
| -----------------------------| ----- | ----- | ----- | -------------------
| Deploy from uploaded compose | yes   | yes   | yes   |

### Template deployment in UI
Selecting an application template from the available templates in Portainer takes you to a 
configuration page to view and modify the template before deployment as needed.

Reviewing Portainer's UI vs CasaOS's UI for deploying templates

| Feature                      | Port. | Casa. | Dkge. |  Comments
| -----------------------------| ----- | ----- | ----- | -------------------
| Custom Install/Compose files | yes   | yes   |       |
| Name/Title                   | yes   | yes   |       |
| Docker image and version     | no    | yes   |       |
| Icon URL                     | no    | yes   |       | Casa allows for changing this in the UI
| Web UI                       | no    | yes   |       | Casa dashboard launcher component
| Network from drop down       | yes   | yes   |       |
| PUID, PGID                   | yes   | yes   |       | Casa supports with environment vars
| Port mappings w/TCP/UDP btns | yes   | yes   |       | Casa's layout is way cleaner
| Custom template fields       | yes   | yes   |       | Casa supports with environment vars
| Volume mapping               | yes   | yes   |       | Casa's layout is way cleaner
| Host file entries            | yes   |       |       |
| Labels                       | yes   |       |       |
| Hostname                     | yes   | yes   |       |
| Privileges                   | no    | yes   |       | 
| Environment Variables        | no    | yes   |       | 
| Devices                      | no    | yes   |       | 
| Container command override   | no    | yes   |       | 
| Memory limit                 | no    | yes   |       | Only a few selectable options
| CPU Shares                   | no    | yes   |       | Only a few selectable options
| Restart policy               | no    | yes   |       |

### Dockge
**Pros**
* Supports 3x docker compose files
* Quick `Docker Run` entry and converts to compose
* Creates a directory per app with its compose inside
* Can read any compose file not just once it created
  * Portainer can't do this
* Compose output is shown in the console
  * Container logs are immediately shown as they boot
  * Portainer just shows spinner and no output
* Provide a link to click through to the application page
  * Portainer requires you click around

**Cons**
* Still new

### DockSTARTer
**Pros**

**Cons**

### Incus
**Pros**

**Cons**

### Portainer
**Pros**
* Installs via Docker
* Templates come out of the box
  * Nice list of apps to install
* Shell into the container
* More featureful than Yacht
* Add more registries

**Cons**
* Default dashboard not as nice as Yacht's
  * Hard to see CPU and Memory consumption
* Only works with compose files created with Portainer
  * i.e. kidnaps the compose files. they have to be stored in the Portainer DB
* Doesn't show streaming logs during boot
* Enterprise focused

### Yacht
**Pros**
* Installs via Docker, Compose or Podman
* Nice card view of all your containers and CPU/Mem usage
  * Apps view gives a nice list
* Create templates i.e. compose yaml
* Update using watchtower

**Cons**
* No templates by default
* No shell into the container
* Brand new, not as many features

### Docker Swarm
Docker Swarm provides the ability to take Docker into a clustered type configuration in a very simple 
way. It simply lets you manage docker across more machines without the complexity of K8s.

