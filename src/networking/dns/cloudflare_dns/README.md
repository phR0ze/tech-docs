# Cloudflare DNS <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Cloudflare is not a replacement for web hosting. Cloudflare is used as your name server and provides 
DNS lookups for your domain i.e. `vanity name` => `Cloudflare DNS` => `homelab web server`. Because 
of Cloudflare's giant network they will likely have a server close to your potential client and will 
speed things up. Cloudflare is free to use for personal or hobby projects for basic DNS, Global CDN, 
Unmetered DDoS Protection, Universal SSL Certificate, Free Managed Ruleset and Simple bot 
mitigation.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Homelab overview](#homelab-overview)
  - [Domain Name Registrar](#domain-name-registrar)
- [Let's Encrypt cert generation](#lets-encrypt-cert-generation)
  - [Cloudflare API token](#cloudflare-api-token)
  - [Caddy](#caddy)
  - [Pangolin](#pangolin)

## Overview

**References**
* [Craylor overview](https://www.youtube.com/watch?v=IXpvUD5SDzA)
* [Craylor setup guide](https://www.youtube.com/watch?v=f1EoQygkJ0E)

### Homelab overview

#### HTTPs for your service
Every homelab will eventually need a domain name and DNS setup to handle networking and services
properly with HTTPS. For example lets take the Vaultwarden service. In order to be able to manage
your own secrets across all your device anywhere in the world you'll need a Domain name `example.com`
that you'll register with Vaultwarden's `DOMAIN` env var. This is required for WebAuthn/passkey/FIDO2
registration (origin-bound by the WebAuthn spec), push notification config and email links. So you'll
register a security key or passkey for `vault.example.com`. This also enables `HTTPS` for the
service.

#### Ingress vs LAN
The same URL `vault.example.com` with the same WebAuthn origin can be used in either the external or
internnal cases as each proxy service will get its own certificate issued independently from
Cloudflare over `DNS-01` so each will terminate TLS on a different path. TLS certs aren't a single
global resource per hostname. Anyone can issue a cert for a name they can prove control of. Thus two
independent processes each holding a valid-but-different cert for the same name is completely
standard. Browsers don't care which cert they got as long as its valid, matches the hostname, and
chains to a trusted CA (usually Let's Encrypt in both cases).

* **Externally** Pangolin gets its cert for `vault.example.com` via DNS-01 and its own Cloudflare API
  token and Cloudflare public DNS resolves to the VPS where Pangolin's ingress tunnel points to
  Vaultwarden.
* **Internally** Caddy gets its own cert for `vault.example.com` via DNS-01 using a Cloudflare API
  token and the DNS gets overridden by Adguard's local DNS to point to Caddy's proxy port to
  Vaultwarden. 

### Domain Name Registrar
The first thing you need is to purchase a domain name to then use with Cloudflare DNS. There are 
numerous registrars but Cloudflare is reliable, keeps prices steady and integrates well with the 
rest of their system.

## Let's Encrypt cert generation

***DNS-01*** is an ACME challenge type where Let's Encrypt proves you control a domain by asking
you to publish a specific `TXT` record under `_acme-challenge.<hostname>`. Unlike `HTTP-01`,
DNS-01 doesn't require port 80 to be reachable from the internet, which makes it the right choice
for internal-only services and wildcard certs (`*.example.com`). Both Caddy and Pangolin can drive
this automatically against Cloudflare using an API token, so no manual DNS editing is needed.

### Cloudflare API token
Both clients need a scoped Cloudflare API token rather than your Global API Key see
[Caddy DNS for Cloudflare](https://github.com/caddy-dns/cloudflare)

1. In the Cloudflare dashboard go to **Manage account** > **Account API tokens**.
2. Click `Create a token` and name it e.g. `Caddy example.com`
3. Switch the `Permission policies >Edit policy` to `All Domains`
4. Under `DNS & Zones`
   1. Check `Edit` for `DNS`
   2. Check `Read` for `Zone`
5. Set `No expiration` and click `Review token` then `Create token`
6. Save the generated token somewhere safe

Use a separate token per client (Caddy, Pangolin, etc.) scoped similarly so each can be
revoked independently, matching the [Ingress vs LAN](#ingress-vs-lan) split described above.

### Caddy
Caddy will request a cert via DNS-01, publish the `TXT` record through the Cloudflare API,
wait for propagation, and renew automatically before expiry.

Configure the token as an environment variable and reference the `tls` directive with the `dns`
subdirective in your `Caddyfile`:

```caddyfile
vault.example.com {
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }
    reverse_proxy vaultwarden:80
}
```

```bash
# .env or secret store
CLOUDFLARE_API_TOKEN=<token-from-above>
```

### Pangolin
Pangolin issues its own certs via DNS-01 using the same style of Cloudflare API token, configured
in its ACME/cert settings (either through the web UI or its config file, depending on version):

```yaml
# pangolin config.yml (example)
certificates:
  acme:
    email: admin@example.com
    dns_provider: cloudflare
    dns_challenge:
      provider: cloudflare
      config:
        api_token: <token-from-above>
```

Once configured, Pangolin requests and renews certs for the hostnames it proxies (e.g.
`vault.example.com`) the same way Caddy does internally — independently, and without needing
inbound port 80/443 open just for the ACME challenge.

