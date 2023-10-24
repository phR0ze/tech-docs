# Podman
<img align="left" width="48" height="48" src="../data/images/logo_256x256.png">

### Quick links
* [Podman vs Docker](#podman-vs-docker)

# Podman vs docker
Podman is essentially just an implementation of docker that doesn't have the commonly complained 
about shortcomings. Podman was built to seamlessly replace Docker in development flows.

|                | Docker                         | Podman
| -------------- | ------------------------------ | ----------------------------------
| Daemon         | Uses docker daemon             | Daemonless architecture
| Root           | Runs containers as root only   | Runs containers as root or non-root 
| Images         | Can build containers images    | Uses Buildah to build container images
| Platform       | Monolithic                     | Different binaries
| Docker swarm   | yes                            | no
| Docker compose | yes                            | yes
| Cross platform | Linux, macOS, Windows          | Linux, macOS, Windows (with WSL)

<!-- 
vim: ts=2:sw=2:sts=2
-->
