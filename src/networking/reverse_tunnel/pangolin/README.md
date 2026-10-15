# Pangolin <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Pangolin is the closest approximation to Cloudflare Tunnels without the limitations. It has a
polished dashboard, auto SSL, and access control built in. Being able to hand out a custom domain
name instead of random IPs makes it easier for less technical family members - they just go to the
URL, log in with Google SSO, and use the service. This is the strongest self-hosted option.

Cloudflare Tunnel isn't an option for me because of the `100mb` upload limit and the ToS that blocks
the use of Jellyfin.

Everything on the hosts is deployed declaratively from
[nixos-config](https://github.com/phR0ze/nixos-config) - both the VPS running Pangolin and the
homelab running Newt. The modules' header comments are the reference for *how* it's deployed and
*why* it differs from upstream. This page covers what the config can't do for you: the DNS records,
the third-party accounts and keys, and the clicks in the Pangolin dashboard.

### Quick links
- [.. up dir](..)
- [Overview](#overview)
  - [Security Advantage](#security-advantage)
  - [Concepts](#concepts)
    - [Sites](#sites)
    - [Resources](#resources)
  - [Where It Lives in nixos-config](#where-it-lives-in-nixos-config)
- [Configure Domain Name](#configure-domain-name)
  - [Purchase Domain Name](#purchase-domain-name)
  - [Configure DNS](#configure-dns)
  - [Verify the Wildcard Record](#verify-the-wildcard-record)
  - [Create the Cloudflare API Token](#create-the-cloudflare-api-token)
- [Prepare the VPS Host](#prepare-the-vps-host)
  - [Recommended Resources](#recommended-resources)
  - [Args and Secrets](#args-and-secrets)
  - [CrowdSec Credentials](#crowdsec-credentials)
  - [Ports](#ports)
- [Configure Pangolin](#configure-pangolin)
  - [Create Admin Account](#create-admin-account)
  - [First Run Experience](#first-run-experience)
    - [Create the initial organization](#create-the-initial-organization)
    - [Enable MFA on Your Account](#enable-mfa-on-your-account)
  - [Enterprise Edition](#enterprise-edition)
  - [Create a Site for the Homelab](#create-a-site-for-the-homelab)
  - [Verify the Tunnel End-to-End](#verify-the-tunnel-end-to-end)
  - [Access Control](#access-control)
    - [Expose a Caddy-fronted service](#expose-a-caddy-fronted-service)
    - [Require Device Approval on Private Resources](#require-device-approval-on-private-resources)
    - [Enforce MFA Organization-Wide](#enforce-mfa-organization-wide)
    - [Remove restrictions from public service](#remove-restrictions-from-public-service)
    - [Google as OAuth2 provider](#google-as-oauth2-provider)
    - [Case Study: Locking Down Vaultwarden](#case-study-locking-down-vaultwarden)
  - [Using the Pangolin API](#using-the-pangolin-api)
  - [Configure Pangolin Client](#configure-pangolin-client)
    - [Android Client](#android-client)
    - [Linux CLI (NixOS)](#linux-cli-nixos)
  - [Configure Organization](#configure-organization)
    - [Configure Domain](#configure-domain)
- [Troubleshooting](#troubleshooting)

### Linked pages
* [Android Client](android_client/README.md)
* [NixOS Client](nixos_client/README.md)
* [Extension Client](extension_client/README.md)
* [Vaultwarden Example](vault_example/README.md)
* [Using the Pangolin API](api/README.md)

## Overview
Pangolin uses `Traefik` as its reverse proxy and `Gerbil` for `WireGuard tunnel management`. It runs
on a VPS with a publicly reachable IP that accepts HTTPS on 443 and securely tunnels it to your
homelab. The homelab just runs a lightweight `newt` client that makes an outbound connection to the
VPS. From that point on the VPS bridges internet users to your homelab without your home IP ever
being exposed. The VPS plays the same role as Cloudflare's edge network, but you own and control
it. A budget VPS like RackNerd costs about $3-4/month.

* [Pangolin - DB Tech](https://www.youtube.com/watch?v=a-a-Xk1hXBQ)

### Security Advantage
Pangolin has a security advantage over exposing a reverse proxy directly.

**Pros**
* The VPS is the only public-facing endpoint
* No inbound ports are open on your home router
* Access control lives at the edge
* You own the relay, not TLS cracking
* WireGuard between VPS and homelab
* Blast radius containment

**Cons**
* The VPS is now a target you must maintain
* You are the security team
* VPS provider is a trust boundary
* Single point of failure for auth
* Misconfiguration risk
* No built-in WAF or bot protection by default - this setup adds CrowdSec + AppSec for that

### Concepts

#### Sites
Sites in Pangolin are tunnels. You'll need a site per network you want to expose. The WireGuard
tunnel is established by `newt` running on your homelab host.

#### Resources
Resources in Pangolin are the services you'd like to expose over the tunnel (a.k.a. site).

### Where It Lives in nixos-config
| Piece                                   | Module                          | Runs on |
| --------------------------------------- | ------------------------------- | ------- |
| Pangolin + Gerbil + Traefik + CrowdSec  | `services.oci.pangolin`         | VPS     |
| Hardening, US-only geo-block, sshd      | `layers.console.server.harden`  | VPS     |
| CrowdSec LAPI + firewall bouncer        | `services.native.crowdsec`      | VPS     |
| Alerts (ntfy)                           | `services.native.alerts`        | Both    |
| Newt site connector(s)                  | `services.oci.newt`             | Homelab |
| Caddy (what Newt actually targets)      | `services.native.caddy`         | Homelab |

The points that shape the manual steps below:
* **The VPS is an isolated host** (`hosts/vps1/.isolated`) with its own sops key - it shares no args
  or secrets with the rest of the fleet. `hosts/vm-vps1` + `hosts/vm-homelab` are its local test twins.
* **One CrowdSec per host** - the stack's CrowdSec container is only an agent (log parsing + AppSec)
  reporting to the host's LAPI, so SSH, port-scan, Traefik and AppSec detections share one decision
  list, one allowlist and one Console identity, enforced by both Traefik and the host firewall.
* **Geo-blocking is done once, on the host** - a prerouting nftables rule drops non-US connections
  to every port, including the container-published ones.
* **No port 80** - certificates use the DNS-01 challenge with a wildcard cert.
* **Newt can only reach Caddy** - its egress is limited to Caddy on its own gateway plus the VPS, so
  every Resource targets `host.containers.internal:443`.

## Configure Domain Name

### Purchase Domain Name
A domain name is required to route public traffic to the VPS. [Cloudflare](../../dns/cloudflare_dns/README.md)
is the recommended registrar - it keeps prices steady, has no surprise renewal markups, and the
domain integrates directly with Cloudflare DNS, which the DNS-01 certificate challenge below uses.

### Configure DNS
Pangolin needs a wildcard record pointing at the VPS's static public IPv4 address. See
[Configure DNS for your Domain Name](../../dns/cloudflare_dns/README.md#configure-dns-for-your-domain-name)
for the general setup.

1. From the Cloudflare console navigate to `Domains >Overview`
2. Choose the options menu to the right of your domain e.g. `example.com` and click `Configure DNS`
   * Alternately if you're already on the target domain config page use `DNS >Records`
3. Click the `Add record` button
4. Set `Type` to `A` and set `Name` to wildcard `*`
5. Set the `IPv4 address` to your VPS's public IP address
6. Flip the `Proxied` toggle to disable it
7. Leave `TTL` at `Auto`
8. Click `Save`

***The public wildcard and the homelab's split-horizon DNS work together*** - LAN clients use the
homelab's AdGuard, which rewrites `*.<domain>` to the homelab's own Caddy, so LAN traffic never
hairpins through the VPS. Anything that should reach the VPS from the LAN - at minimum the
dashboard, `pangolin.<domain>` - needs its own exact rewrite to the VPS IP (exact matches beat the
wildcard), set through `services.native.adguardhome.dnsRewrites` in the homelab's args.

The gotcha with a public wildcard isn't a clash, it's a silent fallback: any subdomain you *haven't*
explicitly defined resolves publicly to the VPS instead of failing with `NXDOMAIN`, so a newly added
internal service that "isn't reachable" may simply be resolving to the VPS. Check DNS first.

### Verify the Wildcard Record
Before anything is deployed you can confirm the DNS layer - this is purely resolution, not HTTP.

**Confirm the wildcard resolves to your VPS's public IP** from public resolvers (not your own, in
case something is cached or rewritten locally), for a made-up subdomain with no explicit record:
```bash
$ dig @1.1.1.1 randomtest123.example.com +short
$ dig @8.8.8.8 randomtest123.example.com +short
$ dig @9.9.9.9 randomtest123.example.com +short
```
All three should return the VPS's real IP. A Cloudflare edge IP (e.g. `104.x.x.x`, `172.6x.x.x`)
means the `Proxied` toggle didn't take.

### Create the Cloudflare API Token
Certificates are issued with the ACME **DNS-01** challenge against Cloudflare, which is what lets
port 80 stay closed and what makes a wildcard certificate possible. Create a scoped token - see
[Cloudflare API token](../../dns/cloudflare_dns/README.md#cloudflare-api-token) - with only
`Zone:DNS:Edit` + `Zone:Zone:Read` on this zone, never the Global API Key. Name it something like
`Pangolin example.com` so it's identifiable and revocable on its own. It goes into the VPS host's
secrets as `pangolin/cloudflareApiToken` (see [Args and Secrets](#args-and-secrets)).

## Prepare the VPS Host

### Recommended Resources
**Minimum (per official docs)**: 1 vCPU, 1.5 GB RAM, 8 GB SSD

**Recommended**: 2 vCPU, 2 GB RAM, 20 GB SSD, transfer sized to your workload (see
[Cloud Budget Comparison](../../../../cloud/README.md#budget-comparison)). For Jellyfin (3× 1080p
movies/day) plus Immich browsing, estimated transfer is ~0.85 TB/month (1.7 TB with 2× headroom).
Pangolin is lightweight and the VPS does no transcoding or storage, so for media-heavy use the limit
is monthly transfer, not compute. A RackNerd 2 GB KVM is enough: the full stack uses ~1 GB.

### Args and Secrets
These are the values you have to supply by hand. Everything else is in the host's
`configuration.nix`.

`hosts/vps1/args.enc.yaml` (`sops` to edit):
```yaml
host:
  network:
    domain: example.com           # base domain: wildcard cert, dashboard at pangolin.<domain>
    allowList:                    # trusted IPs/CIDRs: skip geo-block + CrowdSec, never banned
      - 198.51.100.7              # e.g. the homelab's public IP
  services:
    oci:
      pangolin:
        acmeEmail: admin@example.com
```
Keep the allowlist to addresses you actually control - never shared/CGNAT ranges, which would
exempt strangers too.

`hosts/vps1/secrets.enc.yaml`:

| Key                           | Where it comes from                                             |
| ----------------------------- | --------------------------------------------------------------- |
| `pangolin/serverSecret`       | `openssl rand -base64 32` - rotating it logs everyone out       |
| `pangolin/cloudflareApiToken` | [The scoped DNS-01 token](#create-the-cloudflare-api-token)     |
| `crowdsec/capiCredentials`    | [`cscli capi register`](#crowdsec-credentials) - unique per host |
| `crowdsec/consoleEnrollKey`   | [CrowdSec Console](#crowdsec-credentials)                       |
| `alerts/ntfyTopic`            | The ntfy.sh topic alerts push to                                |
| `users/admin/...`             | The host's admin user, same as any fleet host                   |

Then `./clu build` on the VPS. Once the stack is up, get the one-time setup token for
[Create Admin Account](#create-admin-account):
```bash
$ sudo podman logs pangolin 2>&1 | grep -A 2 -B 2 'SETUP TOKEN'
```

### CrowdSec Credentials
The host's CrowdSec uses two different credentials. Both are needed:

**CAPI credentials** - give this engine the community blocklist. ***Each host needs its own***: the
Console identifies an engine by its CAPI login, not by its name, so a host whose secrets were copied
from another (e.g. `vm-vps1` → `vps1`) never shows up in the Console. NixOS's `cscli` wrapper runs
as the `crowdsec` user even under `sudo`, so write the file somewhere that user can write to:
```bash
$ d=$(sudo -u crowdsec mktemp -d); sudo cscli capi register -f "$d/capi.yaml"; sudo cat "$d/capi.yaml"
```
Paste the contents into `crowdsec/capiCredentials`.

**Console enroll key** - links the engine to the [CrowdSec Console](https://app.crowdsec.net) for its
dashboard and extra blocklists:
1. Log in to `app.crowdsec.net` → `Security Engines` → `Engines` → `Enroll command`
2. Copy just the key from the shown `cscli console enroll <key>` into `crowdsec/consoleEnrollKey`
3. `./clu build` - enrollment runs once, automatically
4. Back in the Console, **accept** the engine, which appears under the host's name

To re-enroll, e.g. after replacing the CAPI credentials, delete
`/var/lib/crowdsec-console-enroll/enrolled` and rebuild.

### Ports
Make sure the VPS provider's firewall (if it has one) allows these:

| Port    | Protocol | Purpose                              |
| ------- | -------- | ------------------------------------ |
| `443`   | TCP      | Dashboard + HTTPS resources          |
| `51820` | UDP      | Site tunnels - Newt → Gerbil         |
| `21820` | UDP      | Pangolin Client (Olm) → Gerbil relay |
| `2222`  | TCP      | sshd (hardened)                      |

## Configure Pangolin

### Create Admin Account
1. Visit `https://pangolin.<your-domain>`
2. Enter the [setup token](#args-and-secrets) from the pangolin container's log
3. Create your admin account with a strong password

### First Run Experience

#### Create the initial organization
1. Login using your new credentials
2. Set `Organization Name` e.g. `your-domain-name`
3. Click `Create Organization`

#### Enable MFA on Your Account
This is a public facing portal, so add TOTP to your account. This only covers *your own* account -
see [Enforce MFA Organization-Wide](#enforce-mfa-organization-wide) to require it for everyone.

1. Click on your profile image in the top right
2. Choose the `Enable Two-factor` menu option
3. Enter your password for confirmation
4. Use your authenticator app to complete the standard process

**References**
* [Configure MFA - Pangolin docs](https://docs.pangolin.net/manage/access-control/mfa)
* [Security Keys - Pangolin docs](https://docs.pangolin.net/manage/access-control/security-keys)

### Enterprise Edition
Pangolin's Enterprise Edition (EE) is free for personal use and for businesses under $100K gross
annual revenue. It's the same codebase as Community Edition behind a different image tag, so
switching is a tag change plus a license key. EE-only features used below include
[device approval](#require-device-approval-on-private-resources) and
[org-wide MFA](#enforce-mfa-organization-wide).

1. Create an account at `app.pangolin.net`, create an organization there, then
   `ORGANIZATION >Billing & Licensing >Licenses` → `+Generate License Key` → check
   `Personal use only (free license - no checkout)`
2. Set `pangolinTag = "ee-<version>";` (pin an exact `ee-` tag, never `ee-latest`) and `./clu build`
3. In the dashboard, `Server Admin >License` (`/admin/license`) → enter the key and activate

### Create a Site for the Homelab
A site is one Newt connection - one homelab host exposing any number of services. Each Pangolin
server the homelab joins is its own Newt instance (`services.oci.newt.instances.<name>`, container
`newt-<name>`), so the same steps work for a second server, e.g. production plus the test VPS.

1. Navigate to `NETWORK >Sites` in the left hand navigation then click `+ Add Site`
2. Choose `Pangolin Site (Recommended)` and set the `Name` e.g. `new-a`
3. Save the three credentials - `Endpoint`, `ID` and `Secret`. ***The secret is only ever shown
   here*** - there's no way to view it again, only to recreate the site
4. Uncheck the `Allow Pangolin SSH` option to reduce attack surface
5. Click `Create Site` and disable `Enable Docker Blueprint`
6. Put them into the homelab's config. In `args.enc.yaml`:
   ```yaml
   host:
     services:
       oci:
         newt:
           instances:
             a:                                      # instance name, 3 chars max
               pangolin:
                 url: https://pangolin.example.com   # the site's "Endpoint"
                 ip: 203.0.113.10                    # the VPS's IPv4 - egress is pinned to it
               id: <newt id>                         # the site's "ID"
               subnet: 10.89.120.0/24                # this instance's own podman network
               ip: 10.89.120.2                       # Newt's fixed address in it
   ```
   and in `secrets.enc.yaml`:
   ```yaml
   newt:
       a:
           clientSecret: <newt secret>               # the site's "Secret"
   ```
7. `./clu build` on the homelab

Dashboard-created sites always get generated credentials. (Pangolin's integration API's
`PUT /org/{orgId}/site` accepts your own `newtId`/`secret`, but that API is off in this setup;
Blueprints can't create sites at all - their `sites:` section only renames/configures existing ones.)

**Confirm it connected** on the homelab:
```bash
$ sudo podman ps --filter name=newt --format '{{.Names}} {{.Status}}'   # (healthy) after the 60s start period
$ sudo podman logs --tail 10 newt-a 2>&1    # "Tunnel connection to server established successfully!"
```
and the site shows **Online** in the dashboard.

### Verify the Tunnel End-to-End
A connected site only proves Newt registered. Prove traffic flows by
[exposing](#expose-a-caddy-fronted-service) one existing Caddy-fronted homelab service (e.g. Homarr)
as a public resource, then from outside the LAN (or from a LAN client resolving the name to the VPS):
```bash
$ curl -I https://home.example.com
```
That service's response means VPS → tunnel → Newt → Caddy works. A throwaway container on an
arbitrary port (the classic `traefik/whoami` test) is *not* reachable through Newt - only Caddy is.

### Access Control
Pangolin resources come in two flavors:

|                 | Public Resource                       | Private Resource (ZTNA)                                 |
|-----------------|---------------------------------------|---------------------------------------------------------|
| Access method   | Browser, no client needed             | Requires the Pangolin Client app                        |
| What's exposed  | An HTTP(S) endpoint via reverse proxy | A specific host/IP or CIDR range, at the network layer  |
| Auth            | SSO redirect + cookie session         | Login to the client app itself                          |
| Best for        | Web apps used in a browser            | Native apps, SSH, databases, anything not browser-based |

All Pangolin resources are ***deny-by-default***.

***Both kinds can only target Caddy*** - Newt's egress allows only Caddy on its own gateway, so a
Resource's target is always `host.containers.internal` port `443`, never a LAN IP or another port.
Private resources come in through Gerbil's relay and use `Host` mode so Caddy routes them by the
client's own hostname. They can be turned off per instance with `allowClients = false`.

#### Expose a Caddy-fronted service
Every homelab service reachable through Newt sits behind Caddy, which terminates TLS and multiplexes
apps on 443 by hostname. The target is therefore always the same:

1. `NETWORK >Resources >Public` → `+ Add Resource`, `Name` e.g. `homarr`, `Type` `HTTP`,
   `Subdomain` e.g. `home.example.com`
2. `+ Add Target`: `Site` = the homelab, `Scheme` = `https`, `Address` = `host.containers.internal`,
   `Port` = `443`
3. If the public subdomain differs from the Caddy vhost, under
   `HTTP Settings >Additional Proxy Settings` set **TLS Server Name** and **Custom Host Header** to
   the vhost (e.g. `home.example.com`). Caddy routes on `Host`; a mismatch 502s
4. Click `Create Resource`

#### Require Device Approval on Private Resources
Even after correct credentials, a brand-new device is blocked until an admin approves it; devices
are fingerprinted and can be revoked individually. **EE or Pangolin Cloud only.**

***Per-role, not global*** - enable `Require Device Approval` on every non-admin role with a Private
Resource attached (`Roles` → role → toggle). A forgotten role leaves its resources reachable from any
device. New devices then show as `Pending Approval` on the dashboard's approvals page.

**References**
* [Device Approvals - Pangolin Docs](https://docs.pangolin.net/manage/access-control/approvals)

#### Enforce MFA Organization-Wide
Blocks every internal-account user from resources until they've set up TOTP. **EE or Pangolin Cloud
only**, one org-wide toggle. Doesn't cover external IdP accounts (Google SSO users are governed by
Google's MFA).

1. `Organization Settings` → `Security`
2. Toggle `Require Two-Factor Authentication for All Users` on
3. Set `Maximum Session Length` to `7 days`

#### Remove restrictions from public service
1. Edit your resource
2. Click the `Authentication` tab
3. Toggle `Platform SSO` off

#### Google as OAuth2 provider
* [Setup GCP OAuth2](https://youtu.be/Bu8WFh1ns4c?t=655)

Auto-provisioning ("log in with a Google email, no pre-created account") is available on Community
as of [Pangolin 1.4.0](https://github.com/orgs/fosrl/discussions/718):
1. Configure Google as a generic OAuth2/OIDC identity provider
2. Enable **"Auth Provision Users"** on that IDP
3. Set a role/org mapping (e.g. a JMESPath rule on email domain); every referenced role/org must
   already exist with an exact name match
4. On first successful login Pangolin creates the account and applies the mapping

#### Case Study: Locking Down Vaultwarden
See [Vaultwarden Example](vault_example/README.md). In this setup its Private resource must target
`host.containers.internal:443` (Caddy), not the Vaultwarden host directly.

### Using the Pangolin API
See [Using the Pangolin API](api/README.md). The key-gated integration API (port 3003) is off in
this setup. It can be enabled temporarily by hand - the module re-renders `config.yml` on every
activation, which turns it back off - and reached over SSH at the container's bridge address, so
3003 is never published.

### Configure Pangolin Client

#### Android Client
See [Android Client](android_client/README.md) for connecting from a phone, including the Private
DNS and full-tunnel routing gotchas.

#### Linux CLI (NixOS)
See [NixOS Client](nixos_client/README.md) for installing `Pangolin CLI` via `nixpkgs`, logging in,
and how routing scope and DNS overrides work so it doesn't take over the machine's network.

### Configure Organization
In the top left corner click `Organization` then `+ New Organization`

#### Configure Domain
1. Click `Network >Domains > + Add Domain`
2. Set the name `example.com`
3. Click `Create Domain`
4. You should see DNS Records created
   ```
   example.com      A   Auto  <internal-ip-address>
   *.example.com    A   Auto  <internal-ip-address>
   ```

## Troubleshooting
| Symptom | Cause / fix |
| ------- | ----------- |
| Newt: `get token ... status code: 400`, `No newt found with that newtId` | The site doesn't exist on *this* Pangolin (different instance, deleted, DB reset). Create it and update `id` + `newt/<instance>/clientSecret` |
| Newt: `Secret is incorrect` | ID exists, secret doesn't match - secrets are only shown at creation, so recreate the site |
| Newt healthy, site never Online / no traffic | UDP 51820 not getting through - provider firewall, or `pangolin.ip` wrong in the instance's args |
| Resource returns 502 | Public subdomain differs from the Caddy vhost - set **TLS Server Name** and **Custom Host Header** |
| Dashboard `403` / connection refused from a trusted client | Geo-block or a CrowdSec ban - check the address is in `allowList`, then `sudo cscli decisions list` (`sudo cscli decisions delete --ip <ip>` to unban) |
| Host missing from the CrowdSec Console | Its `crowdsec/capiCredentials` are shared with another host - [register new ones](#crowdsec-credentials) |
| Internal service "unreachable" from the LAN | It has no AdGuard rewrite, so it resolves through the public wildcard to the VPS - see [Configure DNS](#configure-dns) |
