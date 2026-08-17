# Pangolin <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Pangolin is the closest approximation to Cloudflare Tunnels without the limitations. It has a
polished dashboard, auto SSL, and access control built in. Being able to hand out a custom domain
name instead of random IPs makes it easier for less technical family members - they just go to the
URL, log in with Google SSO, and use the service. This is the strongest self-hosted option.

Cloudflare Tunnel isn't an option for me because of the `100mb` upload limit and the ToS that blocks
the use of Jellyfin.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Security Advantage](#security-advantage)
  - [Concepts](#concepts)
- [Configure VPS](#configure-vps)
  - [Recommended Resources](#recommended-resources)
  - [Configure VPS](#configure-vps)
- [Configure Domain Name](#configure-domain-name)
  - [Purchase Domain Name](#purchase-domain-name)
  - [Configure DNS](#configure-dns)
- [Deploy Pangolin](#deploy-pangolin)
- [Access Control](#access-control)
  - [How Access Control Works](#how-access-control-works)
  - [Google as OAuth2 provider](#google-as-oauth2-provider)
  - [Case Study: Locking Down Vaultwarden](#case-study-locking-down-vaultwarden)
- [Configure Pangolin](#configure-pangolin)
  - [Add Site](#add-site)
  - [Install Newt](#install-newt)
  - [Create Resource](#create-resource)

## Overview
Pangolin uses `Traefik` as its reverse proxy and `Gerbil` for `WireGuard tunnel management`. It can
be setup on a VPS that has a publically reachable IP that accepts traffic on ports 80/443 and
securely tunnels it to your server. Your homelab just runs a lightweight `newt` client that makes an
outbound connection to the VPS. From that point on the VPS bridges internet users to your homelab
without your home IP ever being exposed. In this way the VPS is essentially playing the same role as
Cloudflare's edge network tunnel, but you own and control it. A VPS service like Racknerd only costs
about $4/month.

* [Pangolin - DB Tech](https://www.youtube.com/watch?v=a-a-Xk1hXBQ)

### Security Advantage
Pangolin has a security advantage over reverse proxies. 

**Pros**
* The VPS is the only public-facing endpoint
* No inbound ports are open on your home router
* Access control lives at the edge
* You own the relay, not TLS cracking
* WireGuard between VPS and homelab
* Blast radious containment

**Cons**
* The VPS is now a target you must maintain
* You are the security team
* VPS provider is a trust boundary
* Single point of failure for auth
* Misconfiguration risk
* No built-in WAF or bot protection
  - fail2ban, crowdsec, etc...

### Concepts

#### Sites
Sites in Pangolin are tunnels. You'll need a site per subnet you want to expose. The Wireguard tunnel
is established using `newt` running on your homelab host. 

#### Resources
Resources in Pangolin are the services you'd like expose over the tunnel (a.k.a. site).

## Configure VPS
The first thing we need is a Cloud VPS to host Pangolin on. After some cursory research I landed on
RackNerd as a budget option.

### Recommended Resources
Pangolin itself is lightweight — the VPS does no transcoding or storage. The binding constraint for
media-heavy workloads is monthly transfer, not compute.

**Minimum VPS specs (per official docs)**
- vCPU: 1
- RAM: 1.5 GB
- Storage: 8 GB SSD

**Recommended VPS specs**
- vCPU: 2
- RAM: 2 GB
- Storage: 20 GB SSD
- Transfer: sized to your workload (see [Cloud Budget Comparison](../../../../cloud/README.md#budget-comparison))

### Configure VPS
For a homelab serving Jellyfin (3× 1080p movies/day) and Immich photo/video browsing, estimated
transfer is ~0.85 TB/month (~1.7 TB with 2× headroom). The
[RackNerd 2 GB KVM](../../../../cloud/racknerd/README.md#pangolin-vps) at $35.99/yr is the
cheapest option that meets all requirements.

#### sysctl flags
Gerbil (Pangolin's WireGuard tunnel manager) needs two kernel network settings that aren't part of
a stock Ubuntu install. See [Kernel Parameters](../../../system/ubuntu/hardening/README.md#kernel-parameters)
for the general hardening pass this builds on — the table below is scoped to just what Pangolin
itself requires.


| Flag                               | Required value  | RackNerd default  | Status                         |
| ---------------------------------- | --------------- | ----------------- | ------------------------------ |
| `net.ipv4.conf.all.rp_filter`      | `2` (loose)     | `2`               | Already set — no action needed |
| `net.ipv4.conf.default.rp_filter`  | `2` (loose)     | `2`               | Already set — no action needed |
| `net.ipv4.ip_forward`              | `1`             | `0`               | **Missing — needs override**   |

* **`rp_filter=2`** — strict mode (`1`) drops a packet whenever its source address wouldn't be
  routed back out the same interface it arrived on. Once Gerbil is running, return traffic for a
  tunneled connection legitimately arrives on one interface (the public NIC) and leaves on another
  (the `wg`/tunnel interface), so strict mode drops it as spoofed. Loose mode (`2`) still filters
  obviously bogus source addresses but tolerates asymmetric routing across interfaces — RackNerd's
  image already ships this way, so nothing to change here.
* **`ip_forward=1`** — without it, the kernel won't forward packets between interfaces at all, so
  Gerbil can't route traffic between the public-facing side and the WireGuard tunnel back to your
  homelab. This is off by default on a standard Ubuntu server (it's not a router), so it's the one
  flag every fresh RackNerd VPS needs explicitly turned on for Pangolin to work.

**Check current values**
```bash
$ sysctl net.ipv4.conf.all.rp_filter net.ipv4.conf.default.rp_filter net.ipv4.ip_forward
```

**Create an override for anything missing** — don't edit RackNerd's existing drop-ins under
`/etc/sysctl.d/`; add a new file so this survives image updates:
```bash
$ sudo tee /etc/sysctl.d/98-pangolin.conf > /dev/null <<'EOF'
net.ipv4.ip_forward=1
EOF
$ sudo sysctl --system
```

**Verify it applied**
```bash
$ sysctl net.ipv4.ip_forward
```
If a flag you expect doesn't reflect the value you set, see the
[precedence note](../../../system/ubuntu/hardening/README.md#kernel-parameters) in the hardening
doc — a later-processed file (`99-sysctl.conf`, `/etc/sysctl.conf`) can silently override an
earlier one for the same key.

## Configure Domain Name

### Purchase Domain Name
A domain name is required to route public traffic to the VPS. [Cloudflare](../../../dns/cloudflare_dns/README.md)
is the recommended registrar — it keeps prices steady, has no surprise renewal markups, and the
domain integrates directly with Cloudflare DNS which handles the rest of the setup.

### Configure DNS
Pangolin needs a domain name pointing at the VPS's static public IPv4 address. See
[Configure DNS for your Domain Name](../../../dns/cloudflare_dns/README.md#configure-dns-for-your-domain-name)
for the full setup — Cloudflare is the recommended registrar and DNS provider as it integrates
cleanly with Pangolin's auto-SSL and is free for personal use.

## Deploy Pangolin

* [Thomas Wilde - Pangolin guide](https://www.youtube.com/watch?v=ISEP6SIrEVE)


## Access Control
Pangolin enforces access control at the edge, before traffic ever reaches a resource. Identity
providers, sites, and resources can each be scoped so that only authorized users reach a given
service.

### How Access Control Works
Resources in Pangolin are ***deny-by-default*** — nothing is reachable until you explicitly define
a policy for who can reach it. Two layers work together:

1. **Resource Rules** — network-level filtering (IP/CIDR, geography, ASN, URL path), evaluated
   *before* any authentication happens.
2. **RBAC + Authentication** — identity-level control. You attach a policy to the resource
   specifying exactly which users/roles may authenticate, and by which method.

### Google as OAuth2 provider
* [Setup GCP OAuth2](https://youtu.be/Bu8WFh1ns4c?t=655)

### Case Study: Locking Down Vaultwarden
**Goal:** expose Vaultwarden through Pangolin so that only two named individuals — e.g. you and
your daughter — can reach it, with independent auditing and revocation.

**Important Nuance: Browser vs. Native App Access**

Pangolin's SSO login wall (username/password + TOTP) is a ***browser-based redirect flow***. The
Bitwarden mobile/desktop app talks directly to the server's API — it doesn't open a browser,
follow redirects, or hold a session cookie — so the standard SSO gate doesn't apply to it the way
it would to visiting a resource in Chrome/Safari. This matters when choosing how to expose
Vaultwarden. There are two ways to handle it:

* ***Option A (Recommended): Private Resource + Pangolin Client app*** — expose Vaultwarden as a
  private/ZTNA resource rather than a public HTTP resource. It's reachable only through the
  dedicated Pangolin Client app (a WireGuard-based tunnel client), and Bitwarden connects to the
  internal address once that tunnel is active. This preserves full defense in depth: Pangolin
  identity + TOTP at the outer (network) layer, and Vaultwarden's own master password + 2FA at
  the inner (application) layer.
* ***Option B: Public Resource + HTTP Basic Auth header injection*** — Pangolin can inject HTTP
  Basic Auth in front of a resource instead of the SSO redirect. Native HTTP clients (including
  Bitwarden's) generally handle standard Basic Auth challenges automatically. Downside: Basic
  Auth is username + password only — no TOTP at this layer, so the second factor would need to
  live entirely in Vaultwarden's own 2FA. Simpler to set up, but a weaker outer layer.

**Recommendation: use Option A** — it keeps TOTP enforcement at the outer gate and works cleanly
with the native Bitwarden app.

**Recommended Setup (Option A)**

1. ***Two named user accounts — never a shared login***
   - One account per person, each with their own credentials.
   - Enables per-person audit logs and instant, independent revocation (e.g. if her phone is
     lost, you kill *her* access only).

2. ***Enforce TOTP 2FA on both accounts***
   - Pangolin has native TOTP with backup codes.
   - This is the single highest-value control here: a leaked or guessed Pangolin password alone
     is not enough to get in.

3. ***Create a dedicated role*** — e.g. `vaultwarden-family`
   - Add only the intended users to it.
   - Pangolin supports multiple roles per user, so if you expose other self-hosted services
     later, you can scope each one to its own role instead of giving blanket access.

4. ***Expose Vaultwarden as a Private (ZTNA) resource***
   - Attach it to the `vaultwarden-family` role only — no "any authenticated user," no
     public/anonymous fallback.
   - Access requires the Pangolin Client app to establish a tunnel; there is no direct public
     HTTP endpoint to scan or brute-force.

5. ***Add Resource Rules as a network-level filter***
   - Geo-block to the country/countries the user will realistically connect from — this cuts
     most scanner and credential-stuffing traffic before it ever reaches the login page.
   - Skip IP allowlisting when the user roams (dorm wifi, cellular, coffee shops) — it's
     impractical to maintain.

6. ***Avoid shared secrets as the primary gate***
   - Pangolin also offers resource-specific pins/passwords and email-OTP whitelisting. These are
     handy for one-off sharing but weaker than per-user identity + TOTP for something as
     sensitive as a password vault — a shared pin can't be individually revoked or audited.
   - Skip these for a resource like this.

7. ***Separate admin and resource access boundaries***
   - Non-owner accounts should be plain members of the resource's role only — never
     admin/owner on the Pangolin org itself.

**Private (ZTNA) Resources: How They Actually Work**

Pangolin has two resource types, and they behave differently:

| | Public Resource | Private Resource (ZTNA) |
|---|---|---|
| Access method | Browser, no client needed | Requires the Pangolin Client app |
| What's exposed | An HTTP(S) endpoint via reverse proxy | A specific host/IP or CIDR range, at the network layer |
| Auth | SSO redirect + cookie session | Login to the client app itself |
| Best for | Web apps used in a browser | Native apps, SSH, databases, anything not browser-based |

For a private resource, you define a ***destination*** — either a single host/IP (e.g.
Vaultwarden's internal container address) or a CIDR block — plus which ports/protocols are
allowed (TCP/UDP/ICMP). You can optionally give it a friendly internal DNS name instead of
remembering an IP.

There are two different tunnel roles at play:
* ***Newt*** — runs on *your* side (the site connector), establishing the outbound WireGuard
  tunnel from your home network/VPS to the Pangolin server. This is what makes Vaultwarden
  reachable at all, without opening inbound ports on your firewall.
* ***Pangolin Client (Olm)*** — runs on the *user's* device (her phone), and is what she connects
  with to reach resources she's been granted. Traffic is scoped only to the specific resources
  her role allows — not full-network VPN access.

As of Pangolin 1.15, there are official iOS and Android apps (built on Olm, the same client core
used on desktop), so private resources are genuinely usable from her phone, not just a laptop.
Pangolin 1.15 also added device-level zero-trust controls worth enabling for her phone:
* ***Device fingerprinting*** — identifies each device by attributes like OS version and
  hostname, so you can distinguish "her iPhone" from any other device that might try to log in
  with her credentials.
* ***Device approval (deny-by-default for new hardware)*** — even after correct password + TOTP,
  a brand-new unrecognized device is blocked until you explicitly approve it from the dashboard.
  Turn this on for the `vaultwarden-family` role.
* ***Instant block on lost/compromised device*** — one click on the dashboard immediately cuts
  off a specific device (e.g. if she loses her phone), without touching her account's other
  access. Devices are archived, not deleted, preserving an audit trail.
* ***Posture checks*** (optional) — can require things like disk encryption before granting
  access. Likely more than needed for a personal use case, but worth knowing about.

**Setup Steps**

1. Vaultwarden is already reachable via Newt as part of your Pangolin site.
2. Create a Private Resource pointing at Vaultwarden's *internal* host:port (not its
   public-facing address) — e.g. its container address on your Docker network.
3. Restrict the port policy to just what Vaultwarden needs (its HTTPS port) — nothing broader.
4. Assign the resource to the `vaultwarden-family` role only.
5. Enable device approval for that role.
6. She installs the Pangolin Client app (Play Store / App Store), logs in with password + TOTP,
   and you approve her device the first time from the dashboard. From then on, Bitwarden talks
   to the internal address through the tunnel automatically.

**Her Phone Experience, Step by Step**

1. She opens the Pangolin Client app (separate from Bitwarden).
2. It prompts her to sign in to your Pangolin org — she enters her username and password.
3. She's prompted for her TOTP code — she opens her authenticator app, reads the 6-digit code,
   and enters it.
4. The Pangolin Client establishes the tunnel in the background, scoped only to the resources
   her role (`vaultwarden-family`) can reach.
5. She opens the Bitwarden app, configured with your self-hosted server URL. Since the tunnel is
   up, it connects.
6. She logs into Bitwarden as normal: master password, then Vaultwarden's own 2FA (if enabled).

In practice this isn't a "re-auth every time" ritual — the Pangolin Client can stay connected or
reconnect automatically, with re-auth frequency controlled by the session length you configure.

**Important:** don't store the Pangolin TOTP secret inside the Bitwarden vault it's protecting —
that creates a circular dependency (locked out of the vault means locked out of the code needed
to unlock the vault). Use a separate authenticator app on her phone, and keep printed/offline
backup codes somewhere safe.

**Resulting Access Flow**

Two independent authentication layers, both scoped to named individuals, both auditable, and
both revocable independently:

1. **Outer layer (network):** Pangolin Client login — username/password + TOTP — establishes the
   tunnel.
2. **Inner layer (application):** Vaultwarden login — master password + Vaultwarden 2FA.

**Next Steps (not yet covered)**
- [Ubuntu VPS hardening](../../../system/ubuntu/hardening/README.md) (covers `Crowdsec` setup)
- Pangolin server hardening (admin panel exposure, rate limiting)
- Vaultwarden hardening
- Network segmentation / isolation design


## Configure Pangolin

### Add Site
Create a new site for your homelab subnet

1. Set the `Name` e.g. `redfish`
2. Set the `Method` to `Newt`
3. Copy the newt configuration line 
4. Check `I have copied the config`
5. Click `Create Site`

Note: the new site will be offline until you install the `Newt` agent on your homelab host using the
supplied configuration.

### Install Newt
In order for your site to be active you need to complete the tunnel configuration with `Newt`

1. Browse to the [Pangoline newt instructions](https://docs.pangolin.net/manage/sites/install-site#docker-compose)
2. Grab the docker compose configuration
```yaml
services:
  newt:
    image: fosrl/newt
    container_name: newt
    restart: unless-stopped
    environment:
      - PANGOLIN_ENDPOINT=https://app.pangolin.net
      - NEWT_ID=2ix2t8xk22ubpfy
      - NEWT_SECRET=nnisrfsdfc7prqsp9ewo1dvtvci50j5uiqotez00dgap0ii2
```
3. Update the environment variables with the configuration copied in the [Add Site](#add-site) section
   1. Set `PANGOLIN_ENDPOINT`
   2. Set `NEWT_ID`
   3. Set `NEWT_SECRET`

4. Once newt is up and running you should see your site go `active`

### Create Resource
Creating a resource is essentially configuring your homelab application to be accessible over the
tunnel that the site configuration and newt created.

1. Set the `Name` e.g. `Jellyfin`
2. Set the `Subdomain` e.g. `jellyfin.example.com`
3. Select the correct site e.g. `redfish`
4. Click `Create Resource`
5. Flip the toggle to `Enable SSL (https)`
6. Enter the IP address e.g. `192.168.0.3` of Jellyfin
7. Enter the Port e.g. `8096` of Jellyfin
8. Click `Add Target` then `Save Target`

