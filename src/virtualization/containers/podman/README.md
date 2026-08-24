# Podman <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [Overview](#overview)
  * [Podman vs Docker](#podman-vs-docker)
  * [Systemd Security Hardening](#systemd-security-hardening)
  * [Compose2Nix](#compose2nix)
  * [Install and configure in NixOS](#install-and-configure-in-nixos)
* [Networking](#networking)
  * [Static IP for Container](#static-ip-for-container)
  * [Stale Hostport NAT Rules After Restart](#stale-hostport-nat-rules-after-restart)

## Overview

### Podman vs docker
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

### Systemd Security Hardening
Containers and systemd only provide as much security as you apply yourself. Start with systemd's 
tooling e.g `systemd-analyze security podman-stirling-pdf`.

### Compose2Nix
Compose2Nix converts Docker Compose files into nix `oci-container` configs. The tool also creates all 
networks and volumes that are part of the compose project. 

Note: provides some great insights into how to set things up but you can get a cleaner configuration 
by hand if your willing to put in a little work.

**References**
* [Compose2Nix github](https://github.com/aksiksi/compose2nix)

1. Install via shell
   ```bash
   nix shell github:aksiksi/compose2nix
   ```
2. Convert compose file
   ```bash
   compose2nix -project stirling-pdf docker-compose.yml
   ```

### Install and configure in NixOS
Provides `podman` and `podman-compose`

```nix
{pkgs, ...}:
{
  virtualisation.podman = {
    enable = true;
    dockerCompat = true; # provide docker alias
    extraPackages = [
      pkgs.podman-compose
    ];

    # Allows docker containers to refer to each other by name
    defaultNetwork.settings.dns_enabled = true;

    # Removes dangling containers and images that are not being used. It won't remove any volumes by default
    autoPrune = {
      enable = true;
      dates = "weekly";
      # Removes stuff older than 24h and doesn't have the label important
      flags = [
        "--filter=until=24h"
        "--filter=label!=important"
      ];
    };
  };
}
```

### Declarative docker image deployment

**Nginx test**
```nix
{config, ...}:
{
  virtualisation.oci-containers.containers."nginx" = {
    image = "docker.io/nginx:alpine";
    environmentFiles = [
      config.age.secrets.secret1.path
    ];
  };
}
```

**Echo service**
```nix
{
  virtualisation.oci-containers.containers."echo-http-service" = {
    image = "hashicorp/http-echo:latest";
    extraOptions = ["-text='Hello, World!'"];
    ports = ["5678:5678"];
  };
}
```

### Declarative docker image building
```nix
{
  inputs.nixpkgs.url = "github:nixos/nixpkgs/nixos-22.05";
  outputs = { self, nixpkgs };
  let
    pkgs = nixpkgs.legacyPackages.x86_64-linux;
  in
  {
    my-image = pkgs.dockerTools.buildLayeredImage {
      name = "my-container";
      tag = "latest";
      contents = {
        pkgs.hello
      };
      config.Cmd = [ "hello" ];
    };
  };
}
```

```bash
nix build .#my-image
ls -lah result
docker load < result
docker run my-container
```

## Networking

### Static IP for Container
There are a number of ways to surface your container services. We can use `macvlan` networking to 
assign our container an IP address on our LAN and allow it to participate as a first class citizen on 
the LAN; however this poses some dangers as the container is running potentially untrusted code and 
has supply chain weaknesses and a rather large attack surface. If the container were compromised it 
is then in the heart of your LAN with access to all systems.

Best practice is to use a Docker bridge network per container which keeps that container in an 
isolated virtual network. Then use a reverse proxy to handle access to the container. No need to map 
ports to the docker host excpet for the reverse proxy's TCP/80 and TCP/443. The reverse proxy will 
handle all the port forwarding itself.










That said a static IP associated with a 
container can be very convenient. One way we can get both isolation and a static IP is to use the 
standard docker bridge networking with a dhcp internal address and then a separate host `macvlan` per 
service with the host macvlan having an assigned IP.

**Application specific static IP with isolation e.g. Stirling PDF**
1. Create a host based macvlan for your application
   1. Determine your host's primary NIC e.g. `ens18`
      ```bash
      $ ip a
      1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
          inet 127.0.0.1/8 scope host lo
             valid_lft forever preferred_lft forever
      2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
          link/ether bc:24:11:9f:c8:58 brd ff:ff:ff:ff:ff:ff
          altname enp0s18
          inet 192.168.1.50/24 scope global ens18
             valid_lft forever preferred_lft forever
      ```
   2. Create your app specific macvlan, name can only be about 14 chars
      ```bash
      $ sudo ip link add name stirling-pdf link ens18 type macvlan mode bridge
      $ sudo ip addr add 192.168.1.60/32 dev stirling-pdf
      $ sudo ip link set stirling-pdf up
      ```
2. Now start a container and forward a port to your new macvalan IP
   ```nix
   virtualisation.oci-containers.containers.pdf = {
     image = "docker.io/frooodle/s-pdf:latest";
     ports = [ "192.168.1.60:80:8080" ];
   };
   ```

### Stale Hostport NAT Rules After Restart
Podman/netavark have a long-standing, still-open bug:
[containers/podman#27516](https://github.com/containers/podman/issues/27516)
(originally [containers/netavark#302](https://github.com/containers/netavark/issues/302)). When a
container with a published port (`ports = [ "127.0.0.1:8082:8080" ]`) is stopped, netavark fails to
remove its hostport DNAT rule from the `ip nat` table — the cleanup step throws something like:

```
Error: cleaning up container <id>: removing container <id> network: netavark: setns: IO error: Invalid argument (os error 22)
```

On restart, netavark just appends a *new* DNAT rule for the same port rather than replacing the old
one. `nftables` evaluates rules top-down and the **first match wins** for NAT decisions, so the
stale rule — pointing at the now-dead IP of the previous container instance — keeps winning.
Symptoms: the container is healthy, `podman ps` looks fine, but the published port is unreachable
(connection times out, or `no route to host` if the old bridge/subnet is gone too). Worse: if the
whole podman *network* gets recreated (not just the container), it can land on a different
auto-assigned `10.89.X.0/24` subnet than before, leaving the host bridge interface holding a
leftover address for a subnet nothing routes to anymore.

No maintainer-side fix exists as of this writing — this is a workaround, not a resolution.

**Durable mitigation: pin the subnet and the container's IP**

If a container's IP never changes across restarts, a stale rule netavark fails to clean up ends up
*identical* to the fresh one — a harmless duplicate instead of a dead route. Pin both the network's
subnet and each container's address instead of relying on netavark's auto-IPAM:

```nix
# Pin the network to a fixed subnet instead of auto-IPAM
systemd.services."podman-network-myapp" = {
  serviceConfig = { Type = "oneshot"; RemainAfterExit = true;
    ExecStop = [ "${pkgs.podman}/bin/podman network rm -f myapp" ]; };
  script = ''
    if ! ${pkgs.podman}/bin/podman network exists myapp; then
      ${pkgs.podman}/bin/podman network create --interface-name myapp --subnet 10.89.101.0/24 myapp
    fi
  '';
};

virtualisation.oci-containers.containers.myapp = {
  networks = [ "myapp" ];
  ports = [ "127.0.0.1:8082:8080" ];
  extraOptions = [ "--ip=10.89.101.2" ];  # fixed IP within the pinned subnet
};
```

With this in place, ordinary restarts (`systemctl restart`, reboots, redeploys) stop being a problem
— the same IP gets re-requested every time, so there's nothing for a stale rule to conflict with.

**One-time manual fix after actually changing a network's subnet**

Changing `subnet`/`ip` in code (or migrating an existing service onto this pattern for the first
time) still requires a one-time manual reset, because the *old* subnet's rule is still sitting in
the NAT table pointing at a dead address. Simply removing and recreating the podman network does
**not** clean this up — the hostport DNAT rule lives in a separate chain keyed to the
container/port, not to the network, so `podman network rm` never touches it.

```
NAME=myapp
PORT=8082   # published host port; skip steps 4-5 if the service publishes no port
```

1. Stop the container and its network unit:
   ```bash
   sudo systemctl stop podman-$NAME podman-network-$NAME
   ```
2. Remove the podman network:
   ```bash
   sudo podman network rm -f $NAME
   ```
3. Force-delete the bridge interface if it's still lingering (network rm doesn't always clean this
   up either):
   ```bash
   ip -br addr show $NAME
   sudo ip link delete $NAME 2>/dev/null
   ```
4. Start the container — this recreates the network fresh on the new pinned subnet:
   ```bash
   sudo systemctl start podman-$NAME
   sudo podman network inspect $NAME --format '{{(index .subnets 0).subnet}}'  # sanity check
   ```
5. Find the stale rule — the actual fix. It's in the same `NETAVARK-DN-<hash>` chain as the fresh
   one:
   ```bash
   sudo nft -a list ruleset | grep -B1 -A1 "dport $PORT"
   ```
   The `jump` line names the chain (`NETAVARK-DN-<hash>`). Listing that chain shows **two**
   `dnat to` lines — one to the new pinned IP (correct), one to some other `10.89.X.NN` address
   (stale, from the old subnet). Note the `# handle N` on the stale `dnat to` line and the matching
   `saddr <old-subnet> ... jump NETAVARK-HOSTPORT-SETMARK` line just above it.
6. Delete the stale rule's two handles — this is what actually restores connectivity, not steps 1-4:
   ```bash
   sudo nft delete rule ip nat NETAVARK-DN-<hash> handle <stale-dnat-handle>
   sudo nft delete rule ip nat NETAVARK-DN-<hash> handle <stale-saddr-handle>
   ```
7. Verify:
   ```bash
   curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:$PORT --max-time 3
   ```

### NixOS notes
NixOS allows for advanced declarative networking to allow containers to communicate with each other.

```nix
{lib, ...}:
{
  virtualisation.oci-containers.containers."echo-http-service" = {
    image = "hashicorp/http-echo:latest";
    extraOptions = ["-text='Hello, World!'" "--network=web"];
    ports = ["5678:5678"];
  };

  # Create any networks on boot
  system.activationScripts.createPodmanNetworkWeb = lib.mkAfter ''
    if ! /run/current-system/sw/bin/podman network exists web; then
      /run/current-system/sw/bin/podman network create web
    fi
  '';
}
```

