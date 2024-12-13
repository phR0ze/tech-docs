# Traefik <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Traefik is an open source Application proxy that makes publishing your services a fun and easy 
experience. It receives requests on behalf of your system and identifies which components are 
responsible for handling them, and routes them securly.

**Features**
* Ingress controller, reverse proxy, load balancer and Web Application Firewall (WAF)
* Real-time Metrics, Distributed tracing, Ingress Dashboard
* Lightweight, Cloud Native, Canary deployments
* Automatically discovers the right configuration for your services based on labels
* Natively compliant with Kubernetes, Docker, legacy bare metal and much more

### Quick link
* [Overview](#overview)
  * [Deploy Traefik](#deploy-traefik)

## Overview
Traefik is an ***Edge Router***, this means it's the door to your platform, and that it intercepts 
and routes every incoming request: it knows all the logic and every rule that determines which 
services handle which requests (based on the ***path***, the ***host***, ***headers***, etc...).

### Terminology
* ***EntryPoints***
  * are the network entry points into Traefik. They define the port which will receive the packets 
    and whether to listen to TCP or UDP.

* ***Routers***
  * is in charge of connecting incoming requests to the services that can handle them.

* ***Middlewares***
  * attach to the routers, middlewares and can modify the requests or responses before they are sent 
    to your service

* ***Services***
  * are responsible for configuring how to reach the actual applications that will eventually handle 
    the incoming requests.

### Deploy Traefik
The simplest way to get started is to deploy Traefik with Docker.

**References**
* [Traefik - Quick start](https://doc.traefik.io/traefik/getting-started/quick-start/)
* [Christian Lempa - Traefik](https://github.com/ChristianLempa/boilerplates/tree/main/docker-compose/traefik)

```docker-compose.yaml
services:
  traefik:
    image: docker.io/library/traefik:v3.2
    container_name: traefik
    ports:
      - 80:80
      - 443:443
      # --> (Optional) Enable Dashboard, don't do in production
      # - 8080:8080
      # <--
    volumes:
      - /run/docker.sock:/run/docker.sock:ro
      - ./config/traefik.yaml:/etc/traefik/traefik.yaml:ro
      - ./data/certs/:/var/traefik/certs/:rw
      - ./config/conf.d/:/etc/traefik/conf.d/:ro
    environment:
      - CF_DNS_API_TOKEN=your-cloudflare-api-token # <-- Change this to your Cloudflare API Token
    networks:
      - frontend
    restart: unless-stopped
networks:
  frontend:
    external: true # <-- (Optional) Change this to false if you want to create a new network
```
