# Caddy <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Caddy is an extensible server platform that can be used as a web server or a reverse proxy server.

### Quick links
- [.. up dir](..)
- [Overview](#overview)
- [Configuration](#configuration)
  - [Let's Encrypt Cert Generation](#lets-encrypt-cert-generation)
  - [Create Subdomain for Homelab Service](#create-subdomain-for-homelab-service)

## Overview

### Caddy vs Nginx Proxy Manager
* Caddy auto-provisions with Let's Encrypt for free

## Configuration

### Let's Encrypt Cert Generation
`DNS-01` proves domain ownership by publishing a `TXT` record through the DNS provider's API
instead of serving a file over HTTP, so no port-80 exposure is needed — the right fit for
internal-only hostnames and wildcard certs (`*.example.com`).

`nixos-config`'s `options/services/raw/caddy` builds Caddy from source
(`buildGoModule`, see `package.nix`/`build.nix`) with the [`caddy-dns/cloudflare`](https://github.com/caddy-dns/cloudflare)
module compiled in, and its `default.nix` module always issues certs via Cloudflare DNS-01 — there's
no `tls internal`/self-signed fallback. Every proxied service gets its own HTTPS port (`port + 1000`
by default, or an explicit `httpsPort`) that terminates real, publicly-trusted TLS and forwards to
the backend's existing plain HTTP port over loopback; the backend's original port is left untouched,
so this is purely additive. See
[Cloudflare DNS - Let's Encrypt cert generation](../../dns/cloudflare_dns/README.md#lets-encrypt-cert-generation)
for creating the scoped Cloudflare API token (`Zone:DNS:Edit` + `Zone:Zone:Read`, not the Global API
Key) this build needs.

Store the token via [sops-nix](../../../system/nixos/secrets/sops_nix/README.md) under the
`caddy.cloudflareApiToken` key in a `secrets.enc.yaml` next to the machine's `configuration.nix`:

```yaml
# secrets.enc.yaml
caddy:
    cloudflareApiToken: <token-from-cloudflare_dns/README.md>
```

The module owns wiring that secret to Caddy's `environmentFile` itself — point `secrets` at the file
and set `domain` from `machine.domain` (sourced from `args.enc.json`/`args.nix`, keeping the literal
zone name out of tracked files) rather than hardcoding it:

```nix
{ config, ... }:
{
  services.raw.caddy = {
    enable = true;
    domain = config.machine.domain;
    secrets = ./secrets.enc.yaml;
  };
}
```

Each backend service is fronted by adding an entry to `proxies`, either directly:

```nix
services.raw.caddy.proxies = [
  { subdomain = "vault"; port = 8222; }   # -> https://vault.<domain>:9222
];
```

or, preferably, by having the app's own option contribute one automatically — e.g. Vaultwarden's
`services.raw.vaultwarden.caddy = { enable = true; subdomain = "vault"; }` populates
`services.raw.caddy.proxies` on its own via the shared `caddy_proxy` submodule type, so the proxy
entry lives next to the service it fronts instead of being repeated in the machine's
`configuration.nix`.

Once enabled, Caddy requests the cert, publishes the `TXT` record via the Cloudflare API, waits for
propagation, and renews automatically before expiry — no manual `Caddyfile` editing or intervention
needed after initial setup. Each `<subdomain>.<domain>` still needs an actual DNS record in
Cloudflare for clients to resolve it — see
[Create Subdomain for Homelab Service](#create-subdomain-for-homelab-service) below.

### Create Subdomain for Homelab Service
DNS-01 only proves control of the zone — it doesn't create routing, so each `<subdomain>.<domain>`
still needs an actual DNS record in Cloudflare before it'll resolve anywhere. In the Cloudflare
dashboard for the zone (e.g. `example.com`):

1. From the Cloudflare console navigate to `Domains >Overview`
2. Choose the options menu to the right of your domain e.g. `example.com` and click `Configure DNS`
3. Click the `Add record` button
4. Set `Type` to `A` and set `Name` to your service subdomain e.g. `vault`
5. Set the `IPv4 address` to your homelab's LAN IP e.g. `192.168.1.x`) if it's only ever reached locally, or its
   public IP if it should be reachable from outside.
6. Leave `TTL` at `Auto`, that's is fine.
7. Click `Save`
