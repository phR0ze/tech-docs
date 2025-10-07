# Docker <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [Docker Overview](#docker-overview)
  * [Install and configure in NixOS](#install-and-configure-in-nixos)
* [Docker Networking](#docker-networking)
  * [Docker Networking](#docker-networking)
* [Bild Docker Image](#build-docker-image)
  * [Reproducible with Nix](#reproducible-with-nix)

## Docker Overview

### Install and configure in NixOS
NixOS works well for managing Docker containers

```nix
{pkgs, ...}:
{
  virtualisation.docker = {
    enable = true;

    # Remove dangling containers and images. It won't remove any volumes by default
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

## Docker Networking

### Inspect network
You can get the details of an network by inspecting it i.e. IP address ranges, active containers and 
their DNS names.

```bash
$ sudo podman network inspect immich
```

## Buid Docker Image

### Reproducible with Nix
```nix
{
  inputs.nixpkgs.url = "github:nixos/nixpkgs/nixos-22.11";
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

1. Show flake contents
   ```bash
   nix flake show
   git+file:///home/joel/docker-example
   |__packages
      |__x86_64-linux
         |__dockerImage: package 'otl-nix-demo.tar.gz'
   ```
2. Build the container using the flake syntax
   ```bash
   nix build .#dockerImage
   ls -l result
   result -> /nix/store...otl-nix-demo.tar.gz
   ```
3. Load the image into docker
   ```bash
   docker load < result
   ```
4. Run the docker image
   ```bash
   docker run -p 8080:80 otl-nix-demo:latest
   ```

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
## Agenix


## Networking in Podman
NixOS allows for advanced declarative networking to allow containers to communicate with each other.

```nix
{lib, ...}:
{
  virtualisation.oci-containers.containers."echo-http-service" = {
    image = "hashicorp/http-echo:latest";
    extraOptions = ["-text='Hello, World!'" "--network=web"];
    ports = ["5678:5678"];
  };

  # Create anynetworks on boot
  system.activationScripts.createPodmanNetworkWeb = lib.mkAfter ''
    if ! /run/current-system/sw/bin/podman network exists web; then
      /run/current-system/sw/bin/podman network create web
    fi
  '';
}
```



## Build Secrets
A build secret is any piece of sensitive information, such as a password or API token, consumed as 
part of your application's build process.

**References**
* [Docker docs - Build secrets](https://docs.docker.com/build/building/secrets/)

Build arguments and environment variables are inappropriate for passing secrets to your build, 
because they persist in the final image. Instead, you should use secret mounts or SSH mounts, which 
expose secrets to your builds securely.

### Secret mounts
To pass a secret to a build use the `docker build --secret` flag. To then consume the secret in the 
Dockerfile and make it accessible to the `RUN` instruction use the `--mount=type=secret`.

The source of the secret can be either a file or an environment variable. The type will be detected 
automatically.

**Build Example:**
```bash
$ docker build --secret GITHUB_TOKEN .
```

**Dockerfile Example:**
```Dockerfile
RUN --mount=type=secret,id=GITHUB_TOKEN,env=GITHUB_TOKEN \
  git clone https://oauth2:$GITHUB_TOKEN@github.com...
```
