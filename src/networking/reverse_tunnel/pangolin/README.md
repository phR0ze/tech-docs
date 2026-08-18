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
    - [Sites](#sites)
    - [Resources](#resources)
- [Configure Domain Name](#configure-domain-name)
  - [Purchase Domain Name](#purchase-domain-name)
  - [Configure DNS](#configure-dns)
    - [Verify the Wildcard Record](#verify-the-wildcard-record)
- [Configure VPS](#configure-vps)
  - [Recommended Resources](#recommended-resources)
  - [OS Requirement](#os-requirement)
  - [Subnet Conflict Check](#subnet-conflict-check)
  - [Port Requirements](#port-requirements)
  - [sysctl flags](#sysctl-flags)
- [Deploy Pangolin](#deploy-pangolin)
  - [Pre-Flight Check](#pre-flight-check)
  - [Run the Installer](#run-the-installer)
  - [CrowdSec: Two Separate Engines by Design](#crowdsec-two-separate-engines-by-design)
  - [Log Rotation for Traefik's Access Log](#log-rotation-for-traefiks-access-log)
  - [Extend Monitoring & Alerts for Pangolin](#extend-monitoring--alerts-for-pangolin)
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

1. From the Cloudflare console navigate to `Domains >Overview`
2. Choose the options menu to the right of your domain e.g. `example.com` and click `Configure DNS`
   * Alternately if your already on the target domain config page use `DNS >Records`
3. Click the `Add record` button
4. Set `Type` to `A` and set `Name` to wildcard `*`
5. Set the `IPv4 address` to your VPS's public IP address
6. Flip the `Proxyied` toggle to disable it
6. Leave `TTL` at `Auto`, that's is fine.
7. Click `Save`

***Doesn't clash with existing internal Caddy `A` records for LAN IPs*** — DNS resolution always
prefers an exact match over a wildcard (RFC 4592), regardless of which record was created first. A
specific record like `caddy.example.com A 192.168.x.x` will keep resolving to that LAN IP; the `*`
wildcard only fills in for subdomains that have no explicit record of their own.

The actual gotcha isn't a clash, it's a silent fallback: any subdomain you *haven't* explicitly
defined — including a new internal service you spin up later and forget to add a record for — now
resolves publicly to the VPS instead of failing with `NXDOMAIN`. That request lands on
Traefik/Pangolin rather than erroring out, so "why isn't my new service reachable on the LAN"
can quietly turn into "it's resolving to the VPS, not my LAN" — a different failure mode than
you'd hit without the wildcard in place. Worth checking DNS resolution first if a newly added
internal service seems unreachable.

### Verify the Wildcard Record
Before Pangolin is even deployed, you can confirm the DNS layer is correct — this is purely DNS
resolution, not an HTTP reachability test, since nothing is listening on the VPS yet.

**Confirm the wildcard resolves to your VPS's public IP** — query a public resolver (not your own,
in case your router/ISP has something cached) for a made-up subdomain that has no explicit record:
```bash
$ dig @1.1.1.1 randomtest123.example.com +short
```
Should return your VPS's public IP directly.

**Confirm it isn't being Cloudflare-proxied** — since the `Proxied` toggle was disabled, the result
above should be the real VPS IP, not a Cloudflare edge IP (always within their published ranges,
e.g. `104.x.x.x`, `172.6x.x.x`). If you see something that isn't your VPS's actual IP, the proxy
toggle didn't take.

**Check propagation across a couple of resolvers**, since DNS caching means one resolver updating
doesn't guarantee all have:
```bash
$ dig @8.8.8.8 randomtest123.example.com +short
$ dig @9.9.9.9 randomtest123.example.com +short
```
All three should match.

**Confirm existing internal Caddy records still win** — test one of those specific hostnames the
same way, validating the exact-match-over-wildcard behavior noted above:
```bash
$ dig @1.1.1.1 caddy.example.com +short
```
Should return the LAN IP, not the VPS IP.

Once these check out, HTTP-level testing (`curl -v https://randomtest123.example.com`) isn't
meaningful yet — that has to wait until Traefik is actually running and listening on 80/443 after
[Run the Installer](#run-the-installer) below.

## Configure VPS
The first thing we need is a Cloud VPS to host Pangolin on. After some cursory research I landed on
RackNerd as a budget option.

***Required: complete the [Ubuntu VPS hardening](../../../system/ubuntu/hardening/README.md) doc
on this VPS before deploying Pangolin.*** Pangolin makes this box the sole public-facing ingress
point into the homelab, so it needs to already be locked down — SSH hardened, firewall/GeoIP/
fail2ban/Crowdsec in place — before it starts fronting real traffic. The sysctl and subnet checks
below assume that hardening pass is already done, not a substitute for it.

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

For a homelab serving Jellyfin (3× 1080p movies/day) and Immich photo/video browsing, estimated
transfer is ~0.85 TB/month (~1.7 TB with 2× headroom). The
[RackNerd 2 GB KVM](../../../../cloud/racknerd/README.md#pangolin-vps) at $35.99/yr is the
cheapest option that meets all requirements.

### OS Requirement
Per [official docs](https://docs.pangolin.net/self-host/quick-install), Pangolin requires
`Ubuntu 20.04+` or `Debian 11+`. A fresh RackNerd Ubuntu image comfortably clears this (24.04 LTS
as of writing) — called out here mainly for anyone following this doc on an older or different
distro, since the installer doesn't check this for you.

### Subnet Conflict Check
Gerbil defaults to the CGNAT range `100.89.137.0/20` for its internal tunnel addressing (a `/24`
block per site, `/30` per-site allocation within that). This must not overlap with any subnet
already in use — your homelab LAN, or any other WireGuard/VPN already running that uses CGNAT
space (`100.64.0.0/10`). Check for a conflict *before* initial site registration in Pangolin, not
after:
```bash
$ ip route show | grep -E '100\.(6[4-9]|[7-9][0-9]|1[01][0-9]|12[0-7])\.'
```
If nothing matches, you're clear. If something does, Gerbil's subnet is configurable — change it
before creating your first site rather than after, since resubnetting an active site means
reconfiguring every connected `newt` client.

### Port Requirements
Per [DNS & Networking](https://docs.pangolin.net/self-host/dns-and-networking#port-configuration),
not all four ports opened in the [hardening doc's firewall section](../../../system/ubuntu/hardening/README.md#firewall)
are unconditionally required — two are, two depend on how you use Pangolin:

| Port    | Protocol | Purpose                                 | Required?               | Decision         |
|---------|----------|-----------------------------------------|-------------------------|------------------|
| `443`   | TCP      | Dashboard + HTTPS-secured resources     | Always                  | Open             |
| `51820` | UDP      | Site tunnels — Newt → Gerbil            | Always                  | Open             |
| `80`    | TCP      | Let's Encrypt HTTP-01 validation        | Conditional — see below | Close use DNS-01 |
| `21820` | UDP      | Pangolin Client Olm → Gerbil (tunnels)  | Conditional — see below | Open             |

**`80/tcp` can be closed if you switch to DNS-01 certificate validation.** By default, Traefik's
ACME resolver uses HTTP-01, which needs port 80 reachable for the challenge. Since this doc already
recommends Cloudflare as DNS provider, a `letsencrypt-dns` resolver with `dnsChallenge.provider:
cloudflare` avoids that entirely — validation happens via a DNS TXT record instead of an inbound
HTTP request, and it's also the only way to get a wildcard cert. See
[Wildcard Domains](https://docs.pangolin.net/self-host/advanced/wild-card-domains) for the
`cert_resolver`/`prefer_wildcard_cert` config. If you make this switch, close the port:
```bash
$ sudo ufw delete allow 80/tcp
```
Skip this if you're not ready to reconfigure the cert resolver — HTTP-01 on port 80 is the default
and works fine, this is purely an optional attack-surface reduction.

**`21820/udp` is only needed if you expose Private/ZTNA resources** — it's the port the Pangolin
Client (Olm) app uses to reach Gerbil. The [Vaultwarden case study](#case-study-locking-down-vaultwarden)
below relies on it (Alice's phone uses the Pangolin Client to reach a Private resource), so it stays
open for *this* deployment. For a Pangolin instance that only ever exposes Public resources
(browser-based, no client app), this port isn't needed and can stay closed.

### sysctl flags
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

## Deploy Pangolin

* [Thomas Wilde - Pangolin guide](https://www.youtube.com/watch?v=ISEP6SIrEVE)
* [Official Quick Install docs](https://docs.pangolin.net/self-host/quick-install)

### Pre-Flight Check
Confirm nothing is already bound to 80/443 before running the installer — a leftover web server or
another container can silently make Traefik fail to bind without an obvious error pointing at why:
```bash
$ sudo ss -tulpn | grep -E ':80|:443'
```
Empty output means you're clear to install.

### Run the Installer
The installer places its files in whatever directory you run it from — `cd` into the default
install location used elsewhere in this doc (`/opt/pangolin`, referenced by the log rotation and
container-health sections below) before downloading it, rather than letting it land somewhere else:
```bash
$ sudo mkdir -p /opt/pangolin
$ cd /opt/pangolin
$ curl -fsSL https://static.pangolin.net/get-installer.sh | bash
$ sudo ./installer
```

The installer prompts for, in order:
1. **Edition** — Community or Enterprise (Community for this setup).
2. **Base Domain** — your root domain, e.g. `example.com`.
3. **Dashboard Domain** — defaults to `pangolin.example.com`, or enter your own.
4. **Let's Encrypt Email** — used for SSL certs and as the initial admin account's contact.
5. **Install Gerbil?** — yes, this is what gives Pangolin its tunneling capability; without it
   you'd just have a reverse proxy with no way to reach the homelab.
6. **SMTP email** (optional) — skip unless you already have a mail sender configured.

If you've decided to also run Pangolin's own CrowdSec engine (see below), enable it now rather
than adding it after the fact — check `sudo ./installer --help` for the exact current flag syntax,
since it isn't consistently documented across Pangolin's own docs pages.

Once confirmed, the installer pulls and starts three containers — `pangolin`, `gerbil`, `traefik`
(plus `crowdsec` if enabled) — which takes a couple of minutes.

**Retrieve the initial setup token** from the pangolin container's logs:
```bash
$ sudo docker compose logs pangolin | grep -i token
```

**Complete setup** by visiting `https://<your-dashboard-domain>/auth/initial-setup`, entering the
token, and creating your admin account with a strong password. This is also where you'll create
your first organization — see [Configure Pangolin](#configure-pangolin) below for adding a site
and exposing your first resource.

### CrowdSec: Two Separate Engines by Design
Pangolin's installer has a `--crowdsec` flag that sets up its **own**, Docker-based CrowdSec
engine — distinct from the host-level one already configured in the
[hardening doc](../../../system/ubuntu/hardening/README.md#crowdsec). They don't overlap, so this
isn't redundancy to eliminate, it's two engines each scoped to what they're actually watching:

|             | Host-level CrowdSec (hardening doc)         | Pangolin's CrowdSec (`--crowdsec` flag) |
| ----------- | ------------------------------------------- | --------------------------------------- |
| Watches     | `auth.log`, `syslog`, `kern.log`            | Traefik access logs                     |
| Protects    | SSH, host-level system                      | HTTP(S) traffic through Traefik         |
| Runs as     | native host service                         | Docker container, own Local API on `8080` |
| Collections | `crowdsecurity/sshd`, `crowdsecurity/linux` | HTTP/Traefik-focused (e.g. `crowdsecurity/traefik`) |

Pangolin's CrowdSec has no visibility into SSH/host logs unless you deliberately mount them into
its container and add the `sshd` collection there — don't assume enabling `--crowdsec` gives you
the SSH brute-force protection the host-level one already provides. Keep both running
independently: `cscli` on the host reports SSH/system decisions, `cscli` inside the Pangolin
Docker stack reports HTTP/Traefik decisions.

**Enabling `--crowdsec` turns on Traefik access logging**, which grows unbounded by default —
set up log rotation for it before traffic ramps up, same rationale as any other unbounded log file.

### Log Rotation for Traefik's Access Log
With `--crowdsec` enabled, Traefik writes its access log to
`config/traefik/logs/access.log` relative to the installer's default install directory
(`/opt/pangolin`), so the full host path is `/opt/pangolin/config/traefik/logs/access.log`. Nothing
rotates this by default.

**Create a logrotate config**:
```bash
$ sudo tee /etc/logrotate.d/pangolin-traefik > /dev/null <<'EOF'
/opt/pangolin/config/traefik/logs/access.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
}
EOF
```

***`copytruncate` is required, not optional*** — Traefik runs inside a container holding an open
file handle to this log. A normal rotate-then-reopen cycle needs the process to notice the file was
renamed and open a fresh one, which means signaling the containerized process specifically
(`docker kill -s HUP <container>`, if Traefik even honors that signal for log reopening). Since this
log only feeds CrowdSec's traffic analysis rather than serving as a long-term audit trail,
`copytruncate` (copy the current contents, then truncate the original file in place) is the
simpler, container-agnostic option — Traefik keeps writing to the same inode the whole time, no
signal needed. The small tradeoff is a handful of log lines written in the brief window between the
copy and the truncate can be lost, which is an acceptable loss for this use case.

**Verify it's picked up**:
```bash
$ sudo logrotate -d /etc/logrotate.d/pangolin-traefik
```
`-d` is a dry run — confirms the config parses and shows what it *would* do without actually
rotating anything.

### Extend Monitoring & Alerts for Pangolin
The `ntfy` alerting set up in the
[hardening doc's Monitoring & Alerts](../../../system/ubuntu/hardening/README.md#monitoring--alerts)
section only watches host-level state — it has two blind spots once Pangolin's stack is running,
both worth closing using the same `ntfy` topic already configured.

**1. Docker container health isn't covered by `check-failed-units.sh`**

That script greps `systemctl --failed`, which only sees systemd units. If Pangolin, Gerbil,
Traefik, or Newt crash-loops or exits, `docker.service` itself stays `active` the whole time —
nothing in the existing alerting notices. Add a parallel check scoped to the Pangolin stack,
following the same diff-on-change pattern as `check-failed-units.sh` so a still-down container
doesn't re-page every run:
```bash
$ sudo tee /usr/local/sbin/check-pangolin-containers.sh > /dev/null <<'EOF'
#!/bin/bash
set -euo pipefail
COMPOSE_FILE=/opt/pangolin/docker-compose.yml
STATE_FILE=/var/lib/check-pangolin-containers.state
DEFINED=$(docker compose -f "$COMPOSE_FILE" config --services | sort)
RUNNING=$(docker compose -f "$COMPOSE_FILE" ps --services --filter "status=running" | sort)
DOWN=$(comm -23 <(echo "$DEFINED") <(echo "$RUNNING"))
PREV=$(cat "$STATE_FILE" 2>/dev/null || true)
if [ "$DOWN" != "$PREV" ]; then
  if [ -n "$DOWN" ]; then
    curl -sf -H "Title: Pangolin container(s) down on $(hostname)" -H "Priority: high" \
      -d "$DOWN" https://ntfy.sh/<your-private-topic-name>
  else
    curl -sf -H "Title: Pangolin containers recovered on $(hostname)" \
      -d "All Pangolin containers are running again" https://ntfy.sh/<your-private-topic-name>
  fi
fi
echo "$DOWN" > "$STATE_FILE"
EOF
$ sudo chmod +x /usr/local/sbin/check-pangolin-containers.sh
```
`comm -23` reports service names present in `DEFINED` but absent from `RUNNING` — i.e. anything
defined in the compose file that isn't currently up. This is a best-effort check, not a
substitute for Docker health checks: a container mid-restart-loop can flap between `running` and
`restarting` faster than the polling interval, so a persistent crash loop is guaranteed to be
caught eventually but a brief blip might be missed between runs.

**Schedule it** on the same 5-minute cadence as `check-failed-units.timer`:
```bash
$ sudo tee /etc/systemd/system/check-pangolin-containers.timer > /dev/null <<'EOF'
[Unit]
Description=Check Pangolin container health every 5 minutes

[Timer]
OnBootSec=2min
OnUnitActiveSec=5min

[Install]
WantedBy=timers.target
EOF
$ sudo tee /etc/systemd/system/check-pangolin-containers.service > /dev/null <<'EOF'
[Unit]
Description=Check Pangolin container health

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/check-pangolin-containers.sh
EOF
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now check-pangolin-containers.timer
```

**2. The Security Review Digest only reports the host-level CrowdSec engine**

Per the [Two Separate Engines](#crowdsec-two-separate-engines-by-design) decision above, Pangolin's
`--crowdsec` runs its own engine inside Docker for Traefik/HTTP traffic — a completely separate
`cscli` context from the host one the
[Security Review Digest](../../../system/ubuntu/hardening/README.md#security-review-digest) script
already queries. Extend `security-digest.sh` with a second `cscli` call, run inside the container,
so one daily digest covers both engines instead of just the host one:
```bash
$ sudo sed -i \
  '/^CS_DECISIONS=/a CS_HTTP_DECISIONS=$(docker exec crowdsec cscli decisions list -o raw 2>/dev/null | tail -n +2 | wc -l || echo 0)' \
  /usr/local/sbin/security-digest.sh
$ sudo sed -i \
  's/^Active Crowdsec bans: \$CS_DECISIONS$/Active Crowdsec bans (host\/SSH): $CS_DECISIONS\nActive Crowdsec bans (Traefik\/HTTP): $CS_HTTP_DECISIONS/' \
  /usr/local/sbin/security-digest.sh
```
`crowdsec` here is the container name Pangolin's compose stack uses for its bundled CrowdSec
engine — confirm this matches your actual `docker ps` output before relying on the sed, since a
custom compose override could rename it. Verify the edit landed correctly with
`sudo cat /usr/local/sbin/security-digest.sh` before the next scheduled run, and re-run the digest
manually once to confirm both counts appear in the `ntfy` push.

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

**Auto-provisioning itself is available on Community**, and it's the actual mechanism for
"log in with a Google email, no pre-created account needed" — don't confuse it with the native
Google IDP type above. As of [Pangolin 1.4.0](https://github.com/orgs/fosrl/discussions/718), the
project moved to full feature parity between Community and Professional editions: *"All features will
always be available in BOTH the Professional and Community Edition of Pangolin."* (One older docs
page still states auto-provisioning is Cloud/Enterprise-only — that appears to be stale relative to
the 1.4.0 change; the GitHub announcement is the more specific, dated source and is confirmed
accurate.) To use it:

1. Configure Google as a generic OAuth2/OIDC identity provider (as above).
2. Enable the **"Auth Provision Users"** toggle on that IDP.
3. Set a role/org mapping — e.g. a JMESPath rule matching on email domain — using fixed roles, the
   mapping builder, or a raw expression. Every role/org referenced must already exist in Pangolin
   with exact, character-for-character name matching.
4. The user visits the resource and authenticates with Google. On first successful login, Pangolin
   auto-creates their account and applies the mapped role/org — no manual pre-provisioning needed.

### Case Study: Locking Down Vaultwarden
**Goal:** expose Vaultwarden through Pangolin so that only two named individuals — e.g. Bob and
his daughter Alice — can reach it, with independent auditing and revocation.

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
   - Enables per-person audit logs and instant, independent revocation (e.g. if Alice's phone is
     lost, Bob kills *her* access only).

2. ***Enforce TOTP 2FA on both accounts***
   - Pangolin has native TOTP with backup codes.
   - This is the single highest-value control here: a leaked or guessed Pangolin password alone
     is not enough to get in.

3. ***Create a dedicated role*** — e.g. `vaultwarden-family`
   - Add only the intended users to it.
   - Pangolin supports multiple roles per user, so if Bob exposes other self-hosted services
     later, he can scope each one to its own role instead of giving blanket access.

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

For a private resource, Bob defines a ***destination*** — either a single host/IP (e.g.
Vaultwarden's internal container address) or a CIDR block — plus which ports/protocols are
allowed (TCP/UDP/ICMP). He can optionally give it a friendly internal DNS name instead of
remembering an IP.

There are two different tunnel roles at play:
* ***Newt*** — runs on *Bob's* side (the site connector), establishing the outbound WireGuard
  tunnel from his home network/VPS to the Pangolin server. This is what makes Vaultwarden
  reachable at all, without opening inbound ports on his firewall.
* ***Pangolin Client (Olm)*** — runs on the *user's* device (Alice's phone), and is what she
  connects with to reach resources she's been granted. Traffic is scoped only to the specific
  resources her role allows — not full-network VPN access.

As of Pangolin 1.15, there are official iOS and Android apps (built on Olm, the same client core
used on desktop), so private resources are genuinely usable from Alice's phone, not just a laptop.
Pangolin 1.15 also added device-level zero-trust controls worth enabling for her phone:
* ***Device fingerprinting*** — identifies each device by attributes like OS version and
  hostname, so Bob can distinguish "Alice's iPhone" from any other device that might try to log in
  with her credentials.
* ***Device approval (deny-by-default for new hardware)*** — even after correct password + TOTP,
  a brand-new unrecognized device is blocked until Bob explicitly approves it from the dashboard.
  Turn this on for the `vaultwarden-family` role.
* ***Instant block on lost/compromised device*** — one click on the dashboard immediately cuts
  off a specific device (e.g. if Alice loses her phone), without touching her account's other
  access. Devices are archived, not deleted, preserving an audit trail.
* ***Posture checks*** (optional) — can require things like disk encryption before granting
  access. Likely more than needed for a personal use case, but worth knowing about.

**Setup Steps**

1. Vaultwarden is already reachable via Newt as part of Bob's Pangolin site.
2. Create a Private Resource pointing at Vaultwarden's *internal* host:port (not its
   public-facing address) — e.g. its container address on his Docker network.
3. Restrict the port policy to just what Vaultwarden needs (its HTTPS port) — nothing broader.
4. Assign the resource to the `vaultwarden-family` role only.
5. Enable device approval for that role.
6. Alice installs the Pangolin Client app (Play Store / App Store), logs in with password + TOTP,
   and Bob approves her device the first time from the dashboard. From then on, Bitwarden talks
   to the internal address through the tunnel automatically.

**Alice's Phone Experience, Step by Step**

1. Alice opens the Pangolin Client app (separate from Bitwarden).
2. It prompts her to sign in to Bob's Pangolin org — she enters her username and password.
3. She's prompted for her TOTP code — she opens her authenticator app, reads the 6-digit code,
   and enters it.
4. The Pangolin Client establishes the tunnel in the background, scoped only to the resources
   her role (`vaultwarden-family`) can reach.
5. She opens the Bitwarden app, configured with Bob's self-hosted server URL. Since the tunnel is
   up, it connects.
6. She logs into Bitwarden as normal: master password, then Vaultwarden's own 2FA (if enabled).

In practice this isn't a "re-auth every time" ritual — the Pangolin Client can stay connected or
reconnect automatically, with re-auth frequency controlled by the session length Bob configures.

**Important:** don't store the Pangolin TOTP secret inside the Bitwarden vault it's protecting —
that creates a circular dependency (locked out of the vault means locked out of the code needed
to unlock the vault). Use a separate authenticator app on Alice's phone, and keep printed/offline
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

