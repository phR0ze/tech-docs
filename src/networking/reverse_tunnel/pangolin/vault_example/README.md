# Vaultwarden Example <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

A worked case study for locking down [Vaultwarden](https://github.com/dani-garcia/vaultwarden)
behind [Pangolin](../README.md) — the kind of setup that looks simple until the Bitwarden mobile
app enters the picture. Assumes the [Access Control](../README.md#access-control) section of the
main Pangolin doc, especially [Expose a service fronted by Caddy over
Pangolin](../README.md#expose-a-service-fronted-by-caddy-over-pangolin), is already read.

### Quick links
- [.. up dir](..)
- [Overview](#overview)
  - [Newt](#newt)
  - [Browser access](#browser-access)
  - [Native app access](#native-app-access)
  - [Device Approval](#device-approval)
  - [Browser vs. Native App Access](#browser-vs-native-app-access)
  - [Private HTTP resource gap](#private-http-resource-gap)
  - [Host mode resource solution](#host-mode-resource-solution)
- [Configure Vaultwarden for access outside your homelab](#configure-vaultwarden-for-access-outside-your-homelab)
  - [Configure Pangolin Resources](#configure-pangolin-resources)
    - [Prequisite Pangoline Resources](#prequisite-pangoline-resources)
    - [Create Public HTTP Resource](#create-public-http-resource)
    - [Create Private Host Resource](#create-private-host-resource)
  - [Device Setup Steps](#device-setup-steps)

## Overview

**Goal:** expose Vaultwarden through Pangolin so that only two named individuals — e.g. Bob and
Alice — can reach it, with independent auditing and revocation, and with Pangolin's identity layer
(not just Vaultwarden's own login) gating *all* traffic including the Bitwarden mobile app's API
calls — not merely the browser-based web vault. Also reachable on the LAN without going through
Pangolin at all (already solved locally by Caddy, untouched by anything below).

### Newt
***Newt*** — runs on *Bob's* side (the site connector), establishing the outbound WireGuard
tunnel from his homelab to the Pangolin VPS server. Newt then proxies from Pangolin to Caddy
internally as configured by the Pangolin public/private resource configurations. This allows
connectivity from the public world into the private homelab without any router ports being opened.

### Browser access
***Browser*** — runs on the *user's* device (Alice's computer), and what she connects with for
access to Pangolin public resources. Her browser then prompts for authentication with the Pangolin
SSO layer before granting access in her browser to the resource.

### Native app access
***Pangolin Client VPN*** — runs on the *user's* device (Alice's phone), and is what she connects
with to reach private resources for Android apps. The VPN client prompts for authentication with
the Pangolin SSO layer before granting access at the TCP layer for services that are not in the
browser.

### Device Approval
***Device fingerprinting*** — identifies each device by attributes like OS version and
hostname, so Bob can distinguish "Alice's iPhone" from any other device that might try to log in
with her credentials. This is required via the `Role` required for accessing the resource. You
specify `Require Device Approval`. Then Bod will have to manually approve the device in the Pangolin
portal before Alice can connect.

### Browser vs. Native App Access
Pangolin's SSO login wall (username/password + TOTP) is a ***browser-based redirect flow***. The
Bitwarden mobile/desktop app talks directly to the server's API — it doesn't open a browser,
follow redirects, or hold a session cookie — so the *SSO* gate specifically can't apply to it the
way it would to visiting a resource in Chrome/Safari. This means you need to use Pangolin's Private
(ZTNA) resources to authenticate at the *network* layer instead — the Pangolin VPN Client app on the
phone does the username/password + TOTP login to stand up a WireGuard tunnel, and nothing (not even a
TCP SYN) reaches the destination until that tunnel exists. Bitwarden itself never has to know
Pangolin is involved. This is the only path that satisfies putting the API behind Pangolin's own
identity layer, not just Vaultwarden's — a Public resource with path-exempting Resource Rules (the
community's simpler pattern for cases that don't need this) explicitly does not, since exempted paths
bypass Pangolin auth entirely.

### Private HTTP resource gap
Pangolin's Private HTTP resource has a genuine backend limitation, not just a UI one
The straightforward approach is a Private *HTTP* resource pointed at `host.containers.internal:443`,
mirroring the Caddy-fronted pattern used for the [public
resource](../README.md#expose-a-service-fronted-by-caddy-over-pangolin). It doesn't work: without a
way to override the backend TLS SNI or `Host` header, [Newt's known issue of not forwarding the
real SNI on its backend TLS connection](fosrl/pangolin#207) means a Private HTTP resource pointed
at a multi-tenant Caddy instance can't reach the right vhost — Caddy gets a TLS handshake with no
usable SNI and an HTTP request with the wrong `Host` header, and 404s or serves the wrong site.

***Checked directly against the upstream source (`fosrl/pangolin`, `main` branch, both the
open-source and Enterprise Edition code paths) — this turns out to be a real backend gap, not
merely a missing dashboard field:***
* [`server/db/pg/schema/schema.ts:183-184`](https://github.com/fosrl/pangolin/blob/main/server/db/pg/schema/schema.ts) —
  `tlsServerName`/`setHostHeader` are columns on the `resources` table, which is **Public resources
  only**. `PUT /v1/resource/{resourceId}` and its Zod schema in
  [`updateResource.ts`](https://github.com/fosrl/pangolin/blob/main/server/routers/resource/updateResource.ts)
  operate exclusively on that table.
* Private resources (every mode — Host, CIDR, HTTP, SSH) live in an entirely separate table,
  [`siteResources`](https://github.com/fosrl/pangolin/blob/main/server/db/pg/schema/schema.ts) (see
  the `siteResources = pgTable(...)` definition), managed through a completely different endpoint —
  `POST /v1/private-resource/{siteResourceId}` — with its own Zod schema in
  [`updateSiteResource.ts`](https://github.com/fosrl/pangolin/blob/main/server/routers/siteResource/updateSiteResource.ts).
  That schema has `mode`, `destination`, `destinationPort`, `scheme`, `alias`, `roleIds`, etc. —
  **no `tlsServerName` or `setHostHeader` field exists anywhere in it.**
* [`server/private/lib/traefik/getTraefikConfig.ts`](https://github.com/fosrl/pangolin/blob/main/server/private/lib/traefik/getTraefikConfig.ts) —
  the Enterprise Edition's own Traefik config generator confirms the same split: `tlsServerName`/
  `setHostHeader` are pulled from `resources.*` only. Its `siteResources` query (same file) is used
  exclusively to pre-provision TLS certs/maintenance-page placeholders for a private HTTP resource's
  domain — it never feeds SNI/Host-header data into the actual proxy config.
* [`server/lib/blueprints/privateResources.ts`](https://github.com/fosrl/pangolin/blob/main/server/lib/blueprints/privateResources.ts)
  has no `host-header`/`tls-server-name` mapping either, consistent with the field not existing on
  this resource type at all — this isn't a Blueprints-specific omission the way it first looked.

### Host mode resource solution
A `host`-mode private resource is a raw Layer-4 tunnel — Pangolin/Newt never touch TLS or HTTP at
all, they just relay encrypted bytes end-to-end between the Pangolin Client and the destination.
The original client's TLS `ClientHello` — real SNI included — reaches Caddy completely intact,
exactly as if the client were on the LAN talking to Caddy directly. There's nothing to override
because nothing in Pangolin is parsing the TLS session in the first place.

This is also why a dedicated Caddy listener for the private path (rather than reusing the public
resource's shared, SNI-multiplexed `443` vhost) earns its keep here: point `destination`/
`destinationPort` straight at that listener's real LAN `IP:port`, and it needs zero Caddy-side
changes — the SNI reaching it is genuinely correct, not routed around a gap.

## Configure Vaultwarden for access outside your homelab
The ***Ergonomics of the setup*** are that we want to be able to access our internal homelab
Vaultwarden service anywhere in the world through the Pangolin VPS but also to be able to acces the
Vaultwarden service onsite in the LAN without going through the Pangolin VPS and to do so with the
same service endpoings. To accomplish this we need 4 endpoints to be created:
* `https://vault.example.com` - the externally accessible Pangolin public resource using the
  preconfigured Cloudflare wildcard cert that Pangolin uses DNS-01 and Let's encrypt to generate
  valid certs for. This provides access to the Vaultwarden web portal through your browser.
* `https://vault-vpn.example.com` - the externally accessible Pangolin private resource that provides
  native apps, through the Pangolin VPN cliet, with access to the homelabe using the same
  preconfigured cert generation process.
* `https://vault.example.com` - the internally accessible Caddy proxy using a similar Cloudflare
  DNS-01 and Let's encrypt combination as well as local Adguard DNS to override the external
  Cloudflare DNS resolution to point to the internal Caddy service directly which in turn points to
  the actual service.
* `https://vault-vpn.example.com` - simply a Caddy/Adguard alias for vault.example.com to match
  ergonomics of how this will be setup on native apps so that URLs don't need to change.

### Configure Pangolin Resources

#### Prequisite Pangoline Resources
Other more general steps required for this to work.

1. You've already [configured your Pangolin server and hardened it](../README.md)
2. Ensure that the Organization has 2FA mandatory for all users
3. Create a user account per intended user e.g. `Bob` and `Alice`
4. Create a dedicated role for the service requiring device approval
5. Add the role to the users

#### Create Public HTTP Resource
This is for configuring Pangolin to proxy to Caddy meaning a HTTPS service with SNI requirements.

1. Navigate to `NETWORK >Resources >Public` in the left hand navigation then click `+ Add Resource`
2. Set the `Name` to e.g. `vault`
3. Choose the `Type` of service e.g. `HTTP`
4. Set `Subdomain` to e.g. `vault` and choose `Base Domain` e.g. `example.com`
5. Click `+ Add Target`
6. Choose the `Site` you configured for your server e.g. `testlab`
7. Set the `Address` to `https`, `host.containers.internal` and `443`
8. Click `Create Resource`
9. Switch to the `HTTP Settings` tab
10. Ensure `Enable TLS` is checked
11. Set `TLS Server Name` to `vault.example.com`
12. Set `Custom Host Header` to `vault.example.com`
13. Switch to the `Authentication` tab and add the custom role that requires device approval e.g.
    `Family Device Approval`
14. Click `Save Settings`

#### Create Private Host Resource
This is for configuring Pangolin proxy to Caddy over TCP for native applications.

1. Navigate in the Pangolin UI to `NETWORK >Resources >Private`
2. Set the name e.g. `vault-vpn`
3. Choose the `Type` as `Host`
4. Set the Alias to the name you'd like e.g. `vault-vpn.example.com`
5. Set the site e.g. `testlab`
6. Set the destination to your internal LAN caddy service address e.g. `192.168.x.x`
7. Set the `TCP` selection to `Custom` and the actual port value to your caddy service e.g. `433`
8. Set `UDP` to `Blocked`
9. Disable ICMP and click `Create Resource`
10. After creation switch to the `Authentication` tab and add the custom role that requires device
    approval e.g. `Family Device Approval`

### Device Setup Steps

* see [setup Android phone](../android_client/README.md)
