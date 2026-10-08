# Pangolin <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Pangolin is the closest approximation to Cloudflare Tunnels without the limitations. It has a
polished dashboard, auto SSL, and access control built in. Being able to hand out a custom domain
name instead of random IPs makes it easier for less technical family members - they just go to the
URL, log in with Google SSO, and use the service. This is the strongest self-hosted option.

Cloudflare Tunnel isn't an option for me because of the `100mb` upload limit and the ToS that blocks
the use of Jellyfin.

Everything here is deployed declaratively from
[nixos-config](https://github.com/phR0ze/nixos-config) on NixOS - both the VPS running Pangolin and
the homelab running Newt. There's no hand-written compose file, no `apt`, and no hand-edited config:
the modules render every file, own every unit, and pull secrets from sops at activation.

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Security Advantage](#security-advantage)
  - [Concepts](#concepts)
  - [How It Maps onto nixos-config](#how-it-maps-onto-nixos-config)
- [Configure Domain Name](#configure-domain-name)
  - [Purchase Domain Name](#purchase-domain-name)
  - [Configure DNS](#configure-dns)
  - [Verify the Wildcard Record](#verify-the-wildcard-record)
  - [Create the Cloudflare API Token](#create-the-cloudflare-api-token)
- [Configure the VPS Host](#configure-the-vps-host)
  - [Recommended Resources](#recommended-resources)
  - [Create the Isolated Host](#create-the-isolated-host)
  - [Host Args](#host-args)
  - [Host Secrets](#host-secrets)
  - [Host Configuration](#host-configuration)
  - [Build and Deploy](#build-and-deploy)
- [What the Pangolin Module Deploys](#what-the-pangolin-module-deploys)
  - [Units and Files](#units-and-files)
  - [Deviations from Upstream's Installer](#deviations-from-upstreams-installer)
  - [Ports and Firewall](#ports-and-firewall)
  - [Kernel Settings](#kernel-settings)
  - [Subnet Conflict Check](#subnet-conflict-check)
  - [CrowdSec: Two Separate Engines](#crowdsec-two-separate-engines)
  - [Geo-Allowlist](#geo-allowlist)
  - [Enterprise Edition](#enterprise-edition)
- [Configure the Homelab Newt](#configure-the-homelab-newt)
  - [Newt Host Args and Secrets](#newt-host-args-and-secrets)
  - [Egress Containment](#egress-containment)
  - [Endpoint Pinned to pangolin.ip](#endpoint-pinned-to-pangolinip)
  - [Why NO_CLOUD Is Not Set](#why-no_cloud-is-not-set)
- [Local Test Pair (vm-vps1 + vm-homelab)](#local-test-pair-vm-vps1--vm-homelab)
- [Operations](#operations)
  - [Health Check](#health-check)
  - [Alerts](#alerts)
  - [Updating Images](#updating-images)
  - [Logs](#logs)
  - [Back Up Pangolin's State](#back-up-pangolins-state)
- [Troubleshooting](#troubleshooting)
- [Configure Pangolin](#configure-pangolin)
  - [Create Admin Account](#create-admin-account)
  - [First Run Experience](#first-run-experience)
  - [Create a Site for the Homelab](#create-a-site-for-the-homelab)
  - [Verify the Tunnel End-to-End](#verify-the-tunnel-end-to-end)
  - [Access Control](#access-control)
  - [Using the Pangolin API](#using-the-pangolin-api)
  - [Configure Pangolin Client](#configure-pangolin-client)

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

### How It Maps onto nixos-config
| Piece                               | Where it lives                                 | Runs on |
| ----------------------------------- | ---------------------------------------------- | ------- |
| Pangolin + Gerbil + Traefik + CrowdSec | `services.oci.pangolin` (`modules/services/oci/pangolin.nix`) | VPS |
| Host hardening, geo-block, sshd     | `layers.console.server.harden`                 | VPS (and any hardened server) |
| Host-level CrowdSec + firewall bouncer | `services.native.crowdsec`                   | VPS |
| Alerts (ntfy)                       | `services.native.alerts`                       | Both |
| Newt site connector                 | `services.oci.newt` (`modules/services/oci/newt.nix`) | Homelab |
| Caddy (what Newt actually targets)  | `services.native.caddy`                        | Homelab |

The Pangolin stack runs through `podman-compose` rather than one systemd unit per container: its
inter-container wiring (`network_mode: service:gerbil`, health-gated `depends_on`) is upstream's own
and already correct, so the module renders upstream's compose file (with the deviations listed
[below](#deviations-from-upstreams-installer)) instead of reimplementing it.

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
homelab's AdGuard (`services.native.adguardhome`), which rewrites `*.<domain>` to the homelab's own
Caddy, so LAN traffic never hairpins through the VPS. Anything that should reach the VPS from the
LAN - at minimum the dashboard, `pangolin.<domain>` - needs its own exact rewrite to the VPS IP
(exact matches beat the wildcard), set through `services.native.adguardhome.dnsRewrites` in the
homelab's args.

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
secrets as `pangolin/cloudflareApiToken` (see [Host Secrets](#host-secrets)).

## Configure the VPS Host
A RackNerd 2GB KVM is the budget option this was sized on: Pangolin is lightweight and the VPS does
no transcoding or storage, so the binding constraint for media-heavy use is monthly transfer, not
compute.

### Recommended Resources
**Minimum (per official docs)**: 1 vCPU, 1.5 GB RAM, 8 GB SSD

**Recommended**: 2 vCPU, 2 GB RAM, 20 GB SSD, transfer sized to your workload (see
[Cloud Budget Comparison](../../../../cloud/README.md#budget-comparison)). For Jellyfin (3× 1080p
movies/day) plus Immich browsing, estimated transfer is ~0.85 TB/month (~1.7 TB with 2× headroom).

Measured on the 2 GB staging VM with the full stack up: ~1 GB used by the four containers (pangolin
~500 MB of its 1 GB limit, crowdsec ~230 MB, traefik ~220 MB, gerbil ~30 MB), with ~1.1 GB still
available - workable, but don't plan on running much else there.

### Create the Isolated Host
The VPS is the one public-facing box, so its config is deliberately cut off from the rest of the
fleet: a compromise of the fleet's shared secrets mustn't expose it, and vice versa.

1. Create `hosts/vps1/` (production) - adding a host is just creating its directory
2. Add an empty `hosts/vps1/.isolated` marker. The flake then skips both root layers (`args.nix` and
   `args.dec.yaml`), so the host is fully self-contained
3. Give it a dedicated age key in `.sops.yaml`, matched before the fleet-wide fallback rule. The
   existing rule `hosts/(vm-)?vps[0-9]+/.*\.(dec|enc)\.(pem|yaml)$` already covers `vps<N>` and its
   local staging twin `vm-vps<N>` with the `*vps` key

### Host Args
Build-time values go in `hosts/vps1/args.enc.yaml` (edit with `sops`). Because the host is isolated,
everything it needs from args lives here - nothing is inherited from the root args:
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
`host.network.domain` and `host.network.allowList` are forwarded by `modules/default.nix` into
`services.oci.pangolin.baseDomain`/`geoblockAllowList`, the host geo-block, and the host CrowdSec
whitelist. Keep the allowlist to addresses you actually control - never shared/CGNAT ranges, which
would exempt strangers too. Single addresses and CIDRs can be mixed; the CrowdSec whitelist splits
them into its separate `ip`/`cidr` fields automatically.

### Host Secrets
Runtime secrets go in `hosts/vps1/secrets.enc.yaml`. sops-nix decrypts them at activation into
`/run/secrets` - they never touch the Nix store or git:

| Key                              | What                                                    |
| -------------------------------- | ------------------------------------------------------- |
| `pangolin/serverSecret`          | Pangolin's session/token signing key: `openssl rand -base64 32` |
| `pangolin/cloudflareApiToken`    | The scoped DNS-01 token from above                      |
| `crowdsec/capiCredentials`       | Host CrowdSec Central API credentials, from a one-time `cscli capi register` |
| `alerts/ntfyTopic`               | The ntfy.sh topic all alerts push to                    |
| `users/admin/...`                | The host's admin user, same as any fleet host           |

`server.secret` signs every existing session; rotating it logs everyone out. The module restarts the
stack automatically when either Pangolin secret changes.

### Host Configuration
`hosts/vps1/configuration.nix`:
```nix
{ ... }:
{
  imports = [ ./hardware-configuration.nix ];

  config = {
    layers.console.server = {
      enable = true;
      harden = true;      # boot/kernel/network/sshd/systemd hardening, geo-block, CrowdSec, alerts
      lowMemory = true;
    };
    services.oci.pangolin = {
      enable = true;
      pangolinTag = "ee-1.21.1";         # see Enterprise Edition below; plain "1.21.1" for CE
      gerbilTag = "1.5.1";
      traefikTag = "v3.7";               # floating minor tag, republished upstream on every patch
      crowdsecTag = "v1.7.8";
      badgerPluginVersion = "v1.5.0";
      crowdsecPluginVersion = "v1.4.4";
    };
  };
}
```
Every image tag is required and pinned on purpose: nothing changes under you on a rebuild, and the
[image update alert](#alerts) tells you when upstream ships something newer.

Other `services.oci.pangolin` options worth knowing (defaults shown):

| Option                          | Default            | Purpose |
| ------------------------------- | ------------------ | ------- |
| `dashboardDomain`               | `pangolin.<domain>` | Dashboard hostname |
| `memoryLimit` / `memoryReservation` | `1g` / `512m`  | Pangolin container memory (upstream's `2g` doesn't fit a 2 GB VPS) |
| `rateLimitWindowMinutes` / `rateLimitMaxRequests` | `1` / `100` | Global API rate limit (Pangolin's own default is 500/min) |
| `dashboardSessionLengthHours`   | `24`               | Dashboard login lifetime (Pangolin default 720) |
| `resourceSessionLengthHours`    | `168`              | Resource login lifetime (Pangolin default 720) |
| `disableUserCreateOrg`          | `true`             | Only admins create organizations |
| `crowdsecCollections`           | traefik, http-cve, appsec-virtual-patching, appsec-generic-rules | Container CrowdSec collections |

### Build and Deploy
Getting NixOS onto the VPS itself depends on the provider (custom ISO + `sudo ./clu install`, or an
in-place conversion); once it's running, every change after that is the normal flow on the host:
```bash
$ ./clu build
```
That decrypts only this host's args, evaluates purely, and switches. On a switch the stack restarts
whenever any rendered config changes (the compose/Traefik/CrowdSec text is hashed into the unit), and
every stack start begins with `podman-compose down`, so containers are always recreated from the
current compose file - see [Troubleshooting](#troubleshooting) for why that's not optional.

## What the Pangolin Module Deploys

### Units and Files
All state lives under `/var/lib/pangolin`:

| Path | What |
| ---- | ---- |
| `docker-compose.yml`, `.env` | Rendered compose file and the Cloudflare token env file (symlinked from the store / `/run/secrets-rendered`) |
| `config/config.yml` | Pangolin's config, including `server.secret` (copied, `0400`) |
| `config/db/` | Pangolin's SQLite database - users, orgs, sites, resources |
| `config/letsencrypt/acme.json` | Certificates + ACME account key |
| `config/traefik/` | Static config, `dynamic/` (file-provider directory), `logs/access.log` |
| `config/crowdsec/` | Container CrowdSec config, acquisitions, profiles, and its `db/` |
| `config/GeoLite2-*.mmdb` | MaxMind country/ASN databases for Pangolin's resource rules |
| `state/` | Bouncer key and the cached US CIDR list |

| Unit | What |
| ---- | ---- |
| `pangolin-stack.service` | `podman-compose down` → network check → `up -d`; `down` on stop |
| `pangolin-crowdsec-bouncer.service` | Registers Traefik's CrowdSec bouncer once and writes its LAPI key into `config/traefik/dynamic/crowdsec.yml` |
| `pangolin-geolite-refresh.timer` | Weekly MaxMind refresh (no account/license key needed) |
| `pangolin-geoblock-refresh.timer` | Daily refresh of Traefik's US-only allowlist |
| `logrotate` (`pangolin-traefik`) | Daily rotation of Traefik's access log, signalling Traefik to reopen it |

Config files bind-mounted into containers are real copies, not store symlinks - a container can't
resolve a `/nix/store` path - and each is removed and re-copied on every activation so it always
reflects the current module source.

**Retrieve the initial setup token** after the first start:
```bash
$ sudo podman logs pangolin 2>&1 | grep -A 2 -B 2 'SETUP TOKEN'
```

### Deviations from Upstream's Installer
The stack starts from upstream's `--crowdsec` installer templates. Everything that differs, and why:

* **DNS-01 + wildcard certificate from the start** - `dnsChallenge: cloudflare`, with
  `prefer_wildcard_cert` for resources and an explicit `*.<domain>` `domains:` override on all three
  dashboard routers (`next-router`, `api-router`, `ws-router`). Leaving even one router without the
  override makes Traefik hold a second, exact-match cert for the dashboard and prefer it.
* **No port 80 at all** - not published, no `web` entrypoint, no `ping`, no HTTP→HTTPS redirect
  router. Nothing listens on 80 even inside the container. Any router Pangolin generates on its
  default `web` entrypoint is skipped with an "entryPoint web doesn't exist" log line - only plain
  HTTP routes are lost, which is the point.
* **No HTTP/3** - no `http3` block and no `443/udp` publish. Only `443/tcp` is exposed.
* **Traefik's API/dashboard off** - upstream ships `api.insecure: true`; nothing here uses it.
* **No telemetry** - Pangolin `anonymous_usage: false`, Traefik `checkNewVersion`/`sendAnonymousUsage`
  off.
* **`aliasHeadersStrategy: delete`** on the HTTPS entrypoint (Traefik 3.7+) - drops headers like
  `X_Forwarded_For` that backends deriving variable names from headers would confuse with the real
  ones Traefik manages.
* **US-only geo-allowlist middleware** first on the entrypoint, ahead of CrowdSec - see
  [Geo-Allowlist](#geo-allowlist).
* **CrowdSec bouncer trusts nothing implicitly** - `forwardedHeadersTrustedIPs` is empty (upstream:
  `0.0.0.0/0`), and the RFC1918 ranges are gone from `clientTrustedIPs` (upstream: 10/8, 172.16/12,
  192.168/16). Traefik is the edge and sees real client addresses, so the bouncer never needs
  `X-Forwarded-For` - trusting it from everyone means one spoofed header can skip CrowdSec and
  AppSec the moment anything loosens the entrypoint. What remains trusted is Gerbil's tunnel range
  and the host's `allowList`.
* **`security-headers@file` on every Pangolin-generated router** (`traefik.additional_middlewares`) -
  otherwise only the dashboard routers get it.
* **Hardened Pangolin config** - `trust_proxy: 1` (Traefik is the only hop), global rate limit,
  session lengths, `save_logs`, `log_failed_attempts`. `rate_limits.auth` and `traefik.rate_limit`
  are deliberately absent: tracing Pangolin's source found nothing that reads either.
* **Pinned images everywhere**, including CrowdSec (upstream floats `:latest`), and upstream's
  copy-paste `command: -t` (validate-and-exit) left off CrowdSec.
* **CrowdSec's Prometheus port (6060) unpublished**, and **IPv6 off** (the fleet disables it).
* **Pangolin memory limit `1g`** instead of `2g`.

### Ports and Firewall
| Port    | Protocol | Purpose                                 |
| ------- | -------- | --------------------------------------- |
| `443`   | TCP      | Dashboard + HTTPS resources             |
| `51820` | UDP      | Site tunnels - Newt → Gerbil            |
| `21820` | UDP      | Pangolin Client (Olm) → Gerbil relay    |
| `2222`  | TCP      | sshd (hardened)                         |

***`networking.firewall` doesn't gate container ports*** - podman/netavark DNATs published ports in
its own nftables rules, which traffic reaches regardless of the NixOS firewall. The module still
lists the ports in `allowedTCPPorts`/`allowedUDPPorts` for consistency, but the real control is what
the compose file publishes. Likewise the host's nftables geo-block only hooks `input`, which
forwarded container traffic never traverses - hence the separate Traefik-level geo-allowlist.

Check what's actually listening:
```bash
$ sudo ss -tulpn
```
Expect `tcp 443`, `udp 51820`, `udp 21820` (owned by `conmon`), sshd, and the host CrowdSec on
`127.0.0.1:8080`/`6060` only.

### Kernel Settings
Pangolin doesn't require any. Its [DNS & Networking](https://docs.pangolin.net/self-host/dns-and-networking)
docs don't mention any, and nothing in the `fosrl/gerbil`, `fosrl/pangolin` or `fosrl/newt` source
reads or sets `rp_filter` or `ip_forward` (checked against all three, October 2026).

* **`ip_forward`** - needed only for podman to forward published ports to Gerbil's container; the
  fleet's kernel module already sets it for container hosts.
* **`rp_filter`** - the hardened kernel sets strict mode (`1`), and that's fine for Gerbil, despite
  older advice that it needs loose mode (`2`):
  * WireGuard packets arrive on the container's `eth0` from public addresses; the route back is the
    default route out that same `eth0`.
  * Tunnel traffic arrives on `wg0` from site addresses inside the subnet Gerbil assigns to `wg0`
    itself, so the route back is `wg0` too.
  * Nothing is routed *between* interfaces in the kernel: Traefik shares Gerbil's network namespace,
    so tunnel traffic terminates locally, and Gerbil's own firewall drops everything inbound from the
    tunnel except established connections, ping and 80/443 to its own address. The `21820` relay
    forwards in userspace over ordinary UDP sockets, which `rp_filter` never sees.

  Measured on vm-vps1 under strict mode with a Newt site connected and passing traffic: the kernel's
  reverse-path drop counter in Gerbil's namespace stayed at `0`. Check it on your own box:
  ```bash
  $ sudo podman exec gerbil awk '/^TcpExt:/{if(!h){split($0,k);h=1}else{split($0,v);for(i in k)if(k[i]=="IPReversePathFilter")print k[i],v[i]}}' /proc/net/netstat
  ```
  If it ever climbs while a site is connected, loosen `rp_filter` for Gerbil's container alone
  (`sysctls:` in the compose file), never the whole host.

### Subnet Conflict Check
Gerbil addresses site tunnels from `100.89.128.0/20` (`wg0` gets `100.89.128.1/24` for the first
site); Pangolin also reserves `100.90.128.0/20` (`orgs.subnet_group`) and `100.96.128.0/20`
(`orgs.utility_subnet_group`). None of them may overlap anything already in use - the homelab LAN
or any other WireGuard/VPN in CGNAT space. Check before creating the first site, since resubnetting
an active site means reconfiguring every Newt:
```bash
$ ip route show | grep -E '100\.(6[4-9]|[7-9][0-9]|1[01][0-9]|12[0-7])\.'
```
The CrowdSec bouncer's trusted range (`100.89.137.0/20` in the module, i.e. `100.89.128.0/20`) must
be kept in sync with any override.

### CrowdSec: Two Separate Engines
Two independent CrowdSec engines run on the VPS, each with its own Local API and decisions:

|             | Host CrowdSec (`services.native.crowdsec`)    | Container CrowdSec (Pangolin stack) |
| ----------- | --------------------------------------------- | ----------------------------------- |
| Watches     | sshd logs, kernel firewall drops (port scans) | Traefik's access log + AppSec (WAF) |
| Enforces    | nftables firewall bouncer, permanent bans     | Traefik bouncer plugin, 4h bans / captcha for HTTP scenarios |
| Inspect     | `sudo cscli ...`                              | `sudo podman exec crowdsec cscli ...` |

Neither sees the other's traffic, so keep both. Useful commands:
```bash
$ sudo cscli decisions list
$ sudo cscli bouncers list
$ sudo podman exec crowdsec cscli decisions list
$ sudo podman exec crowdsec cscli metrics show acquisition   # confirms access.log is being read
```
`traefik-bouncer` shows an empty `last_pull` - expected: the plugin runs in `live` mode, querying
the LAPI per request rather than pulling. Removing a ban on yourself:
```bash
$ sudo podman exec crowdsec cscli decisions delete --ip <your-ip>
```
Addresses in `host.network.allowList` are never banned by either engine.

### Geo-Allowlist
Traefik's `us-allowlist@file` middleware runs first on the HTTPS entrypoint and rejects non-US
clients before CrowdSec's synchronous LAPI/AppSec round-trip. Its source is the same
`ipverse/country-ip-blocks` US list the host geo-block uses, refreshed daily into
`config/traefik/dynamic/geo-allowlist.yml`. A failed fetch falls back to the last cached list, and
the `allowList` entries are always included, so a trusted address can't be locked out even before
the first fetch. The file holds thousands of entries when healthy - only a handful means the US list
hasn't been fetched yet.

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

***EE behaves differently from CE in one place this setup hit*** - see
[Why NO_CLOUD Is Not Set](#why-no_cloud-is-not-set).

## Configure the Homelab Newt
Newt runs on the homelab as an OCI container (`services.oci.newt`): fully userspace WireGuard, so it
needs no capabilities and no `/dev/net/tun`, runs non-root with `--cap-drop=ALL`, `no-new-privileges`
and a read-only rootfs. Its client tunnels (`DISABLE_CLIENTS`) and SSH auth daemon (`DISABLE_SSH`)
are off, so the Pangolin server can't open more paths into the homelab than the Resources defined
for the site.

```nix
services.oci.newt = { enable = true; user.uid = 2005; tag = "1.16.0"; };
```

### Newt Host Args and Secrets
In the homelab host's `args.enc.yaml`:
```yaml
host:
  services:
    oci:
      newt:
        pangolin:
          url: https://pangolin.example.com   # the site's "Endpoint"
          ip: 203.0.113.10                    # the VPS's IPv4 - see the next two sections
        id: <newt id>                         # the site's "ID"
        subnet: 10.89.110.0/24                # Newt's own isolated podman network (/24, .0)
        ip: 10.89.110.2                       # Newt's fixed address in it
```
and in its `secrets.enc.yaml`:
```yaml
newt:
    clientSecret: <newt secret>               # the site's "Secret"
```
The three values come from the dashboard when [creating the site](#create-a-site-for-the-homelab).

### Egress Containment
Pangolin decides which targets Newt proxies to, so without a limit whoever controls the Pangolin
server could reach any LAN host:port or use the homelab as a relay to the internet. The
`newt-egress` nftables table only lets Newt's bridge reach:
* its own gateway on `tcp/443` (Caddy - i.e. only Caddy-fronted services) and `udp/53` (podman DNS)
* `pangolin.ip` on `tcp/443` (API + websocket) and `udp/51820,21820` (Gerbil)

Everything else is ***rejected*** (TCP reset / ICMP admin-prohibited) rather than dropped, so a
blocked dial fails immediately instead of waiting out its timeout - Newt's startup update check to
`api.fossorial.io`, which has no off switch, would otherwise stall every start by 10 seconds. The hook
is prerouting at mangle priority, ahead of netavark's DNAT, so it judges the address Newt actually
dialed.

***Consequence: only Caddy-fronted services can be Resources.*** A service on some other host:port
is unreachable through Newt by design - front it with Caddy. See
[Expose a Caddy-fronted service](#expose-a-caddy-fronted-service).

### Endpoint Pinned to pangolin.ip
The endpoint's hostname is pinned to `pangolin.ip` in Newt's `/etc/hosts` (`--add-host`), which Newt
checks before DNS. That makes the name and the egress rule agree by construction:
* otherwise the name resolves through podman's DNS to the host's upstream resolver. On a host that
  doesn't use the LAN AdGuard the name may not resolve at all (no public record for a test VPS), and
  where AdGuard is used its `*.<domain>` split-horizon wildcard would answer with the homelab's own
  Caddy - either way Newt silently never connects
* a spoofed DNS answer can't steer Newt's dials
* TLS still validates against the name, and Gerbil's `base_endpoint` (the WireGuard dial) is the same
  hostname, so it's covered too

The URL is still required even with the IP pinned: Traefik's certificate doesn't cover an IP, its
routers match `Host(pangolin.<domain>)`, and Newt builds its API/websocket URLs from it. `pangolin.ip`
decides *where* packets go; `pangolin.url` provides the name TLS and Traefik check.

### Why NO_CLOUD Is Not Set
Despite the name, Pangolin's **Enterprise** build answers a Newt that reports `noCloud: true` with no
`gerbil`-type exit nodes at all - including a self-hosted Gerbil
(`server/private/lib/exitNodes/exitNodes.ts`):
```ts
node.type === "gerbil" && (!filterOnline || node.online) && !noCloud
```
Newt then logs `No exit nodes provided` and never brings its tunnel up. The CE build ignores the flag,
which is why it looks harmless. Cloud failover is impossible regardless - the egress rule only
allows `pangolin.ip`.

## Local Test Pair (vm-vps1 + vm-homelab)
`hosts/vm-vps1` is the local-VM staging twin of `hosts/vps1` (isolated, same `*vps` sops key, same
module config) and `hosts/vm-homelab` the twin of the homelab. Test changes there before production.

```bash
$ ./clu deploy vm vps1      # on the VM host: copies the repo to /var/lib/vms/vm-vps1 and builds it
```
Inside a running VM, changes are applied the normal way (`./clu build`).

What's different from production:
* **`pangolin.ip` is vm-vps1's LAN IP** in vm-homelab's args, so Newt's egress rule and `/etc/hosts`
  pin point at the VM rather than a public address.
* **LAN browsers need an AdGuard rewrite** `pangolin.<domain>` → vm-vps1's LAN IP to reach the
  dashboard (via `services.native.adguardhome.dnsRewrites`). Newt itself doesn't need it - its pin
  bypasses DNS.
* **vm-vps1's `allowList` includes the LAN CIDR**, so LAN clients pass the US geo-allowlist and skip
  CrowdSec.
* **Shut it down with `sudo poweroff`.** Closing the QEMU SDL window quits QEMU on the spot - a
  power cut: no unit stops, the journal tail is lost, containers survive into the next boot, and
  SQLite/`acme.json` writes can be interrupted.

## Operations

### Health Check
A read-only pass over the whole VPS deployment (no secrets printed):
```bash
$ cat > /tmp/pangolin-check.sh <<'EOF'
s() { printf '\n===== %s =====\n' "$1"; }
s "systemd";     systemctl is-system-running; systemctl --failed --no-legend --plain
s "containers";  podman ps -a --format '{{.Names}}\t{{.State}}\t{{.Status}}' </dev/null
s "ports";       ss -tulpnH | awk '{print $1, $5, $7}' | sort -u
s "tunnel";      podman exec gerbil ip -s link show wg0 </dev/null | grep -A1 -E 'RX|TX'
s "crowdsec";    cscli bouncers list -o raw; podman exec crowdsec cscli bouncers list -o raw </dev/null
s "geo";         grep -c '^ *- ' /var/lib/pangolin/config/traefik/dynamic/geo-allowlist.yml
s "cert";        D=$(grep -oP 'dashboard_url: "https://\K[^"]+' /var/lib/pangolin/config/config.yml)
                 echo | openssl s_client -connect 127.0.0.1:443 -servername "$D" 2>/dev/null \
                   | openssl x509 -noout -enddate -ext subjectAltName
s "alerts";      for f in /var/lib/alerts/*.state; do printf '%s: ' "${f##*/}"; [ -s "$f" ] && cat "$f" || echo ok; done
s "resources";   free -h | head -2; podman stats --no-stream --format '{{.Name}}\t{{.MemUsage}}' </dev/null
EOF
$ sudo bash /tmp/pangolin-check.sh
```
Writing it to a file matters: piping a script into `bash` through a heredoc lets the first command
that reads stdin (several `podman` subcommands do) swallow the rest of it silently.

Healthy means: no failed units; all four containers running (pangolin and crowdsec `(healthy)`);
only the [expected ports](#ports-and-firewall); `wg0` RX/TX non-zero and climbing once a site is
connected; both bouncers registered; thousands of geo entries; SANs `DNS:<domain>, DNS:*.<domain>`
with expiry more than ~30 days out; all alert states `ok`.

### Alerts
`services.native.alerts` pushes to ntfy, and the Pangolin module registers itself with it - nothing
to configure per host beyond the `alerts/ntfyTopic` secret:

* **Failed units** - every 5 minutes, on change.
* **Containers** - every 5 minutes: alerts when one of the stack's containers is missing, stopped or
  unhealthy while `pangolin-stack` is active. A oneshot compose unit stays `active` however its
  containers fare, so the failed-unit check alone never sees a crash-looping container. Every
  `oci-containers` service on any host (e.g. the homelab's Newt) is registered automatically too,
  plus a restart-count check that catches containers crash-looping too slowly to ever mark their
  unit failed.
* **Daily security digest** - rejected SSH logins plus active decisions from *both* CrowdSec
  engines.
* **Image updates** - daily, compares every pinned tag against the project's latest GitHub release
  and alerts when the set of outdated images changes. The `ee-` prefix is stripped for the comparison
  and re-added in the alert (so it names the exact tag to pull); Traefik's floating `v3.7`-style tag
  is compared by minor version only. A notice is never an update - bumping stays deliberate.

### Updating Images
1. Read the release notes - especially CrowdSec and Traefik
2. Bump the tag in the host's `configuration.nix` and `./clu build`

The changed compose file restarts `pangolin-stack`, whose start always runs `podman-compose down`
first, so all four containers are recreated together. That also sidesteps the classic
[gerbil-alone update](#troubleshooting) breakage, since Traefik lives in Gerbil's network namespace.

### Logs
```bash
$ sudo journalctl -u pangolin-stack -b         # stack start/stop + all container output
$ sudo podman logs --since 1h traefik          # Traefik's general log (stdout)
$ sudo tail -f /var/lib/pangolin/config/traefik/logs/access.log
```
Container output is attributed to `pangolin-stack` in the journal (the log processes run in its
cgroup) and logged at error priority regardless of content, so `journalctl -p err` is mostly noise -
CrowdSec's healthcheck alone logs a `POST /v1/watchers/login` every 10 seconds.

Traefik's *access* log is a file rather than stdout because the container CrowdSec reads it through
a shared mount. logrotate rotates it daily and then sends Traefik `USR1`, Traefik's documented hook to
reopen its log files. Without that, Traefik keeps writing to the renamed `access.log.1` - outside
CrowdSec's `*.log` acquisition glob - blinding it from the first rotation on.

### Back Up Pangolin's State
***Not automated yet*** - there's no `backup-pangolin` unit. What has to survive losing the VPS:
* **`config/db/db.sqlite`** - users, orgs, sites, resources. Copy with `sqlite3 .backup`, not `cp`,
  for a consistent snapshot.
* **`config/letsencrypt/acme.json`** - certificates and the ACME account; re-issuing from scratch is
  rate-limited by Let's Encrypt.
* **`config/crowdsec/db/` and `state/`** - CrowdSec history, the bouncer key, the cached US list.

Everything else (`config.yml`, compose, Traefik, CrowdSec config) is regenerated from Nix and the
secrets. The fleet pattern to follow is the one in `services.native.vaultwarden`: a `backupDir`
forwarded from `host.backupDir`, a nightly `backup-<name>` unit, and registration in
`services.native.alerts.backup.services`. A local copy on the same VPS doesn't protect against
losing the VPS, so pull it to the homelab over the tunnel.

## Troubleshooting
| Symptom | Cause / fix |
| ------- | ----------- |
| A removed port (e.g. `443/udp`) still listening after a rebuild | A stale container survived. podman-compose 1.6.0 does detect a changed service's config hash, but only recreates *running* dependents with it - at boot everything is exited, so changing gerbil skips traefik (which shares gerbil's netns), podman refuses to remove gerbil, the create fails on the name in use, and podman-compose starts the stale container and exits 0. The stack now always runs `down` before `up`; `sudo systemctl restart pangolin-stack` forces the same. Check with `sudo podman inspect gerbil --format '{{.Created}}'` |
| Newt: `lookup pangolin.<domain> ... no such host` | The endpoint doesn't resolve where Newt is. The `--add-host` pin fixes this - check `sudo podman exec newt cat /etc/hosts` |
| Newt: `get token ... status code: 400`, `No newt found with that newtId` | The site doesn't exist on *this* Pangolin (different instance, deleted, DB reset). Create it and update `id` + `newt/clientSecret` |
| Newt: `Secret is incorrect` | ID exists, secret doesn't match - secrets are only shown at creation, so recreate the site |
| Newt: `Websocket connected` then `No exit nodes provided` | `NO_CLOUD` set against an EE server - see [Why NO_CLOUD Is Not Set](#why-no_cloud-is-not-set) |
| Newt healthy, `wg0` RX/TX at 0 on the VPS | UDP 51820 not getting through - provider firewall, or `pangolin.ip` wrong in the egress rule |
| No `/api/v1/auth/newt` requests in the access log at all | Newt never reached Traefik: DNS, egress `pangolin.ip`, or a TLS failure (TLS failures never reach the access log) |
| Dashboard `403` from a LAN/trusted client | Geo-allowlist or a CrowdSec decision - check the address is in `allowList`, then `cscli decisions list` in the container |
| Host `crowdsec.service` failing: `ParseAddr("x.x.x.x/24")` | A CIDR under a whitelist's `ip:` field. The module splits `ip`/`cidr` now; a stale generated whitelist is pruned by `crowdsec-prune-stale-links` |
| Traefik logs DNS errors to `127.0.0.11` after only gerbil was recreated | Traefik is orphaned in gerbil's old network namespace - recreate both together; the stack's `down`-first start always does |

## Configure Pangolin

### Create Admin Account
1. Visit `https://pangolin.<your-domain>`
2. Enter the setup token from [the pangolin container's log](#units-and-files)
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

### Create a Site for the Homelab
A site is one Newt connection - one homelab host exposing any number of services.

1. Navigate to `NETWORK >Sites` in the left hand navigation then click `+ Add Site`
2. Choose `Newt Site (Recommended)` and set the `Name` e.g. `homelab`
3. Save the three credentials - `Endpoint`, `ID` and `Secret`. ***The secret is only ever shown
   here*** - there's no way to view it again, only to recreate the site
4. Click `Create Site` and disable `Enable Docker Blueprint`
5. Put them into the homelab's config ([Newt Host Args and Secrets](#newt-host-args-and-secrets)):
   ```bash
   $ sops hosts/homelab/args.enc.yaml       # services.oci.newt.id (+ pangolin.url/ip)
   $ sops hosts/homelab/secrets.enc.yaml    # newt.clientSecret
   ```
6. `./clu build` on the homelab

Dashboard-created sites always get generated credentials. (Pangolin's integration API's
`PUT /org/{orgId}/site` accepts your own `newtId`/`secret`, but that API is off in this setup;
Blueprints can't create sites at all - their `sites:` section only renames/configures existing ones.)

**Confirm it connected** on the homelab:
```bash
$ sudo podman ps --filter name=newt --format '{{.Status}}'       # (healthy) after the 60s start period
$ sudo podman logs --tail 10 newt 2>&1                           # "Tunnel connection to server established successfully!"
```
and on the VPS `wg0` RX/TX should be non-zero and climbing, and the site **Online** in the
dashboard.

### Verify the Tunnel End-to-End
A connected site only proves Newt registered. Prove traffic flows by exposing one existing
Caddy-fronted homelab service (e.g. Homarr) as a public resource, then from outside the LAN (or from a
LAN client resolving the name to the VPS):
```bash
$ curl -I https://home.example.com
```
That service's response means VPS → tunnel → Newt → Caddy works. Because of
[egress containment](#egress-containment), a throwaway container on an arbitrary port (the classic
`traefik/whoami` test) is *not* reachable through Newt - only Caddy is.

### Access Control
Pangolin resources come in two flavors:

|                 | Public Resource                       | Private Resource (ZTNA)                                 |
|-----------------|---------------------------------------|---------------------------------------------------------|
| Access method   | Browser, no client needed             | Requires the Pangolin Client app                        |
| What's exposed  | An HTTP(S) endpoint via reverse proxy | A specific host/IP or CIDR range, at the network layer  |
| Auth            | SSO redirect + cookie session         | Login to the client app itself                          |
| Best for        | Web apps used in a browser            | Native apps, SSH, databases, anything not browser-based |

All Pangolin resources are ***deny-by-default***.

***Private resources need Newt's client support, which this setup turns off*** (`DISABLE_CLIENTS`)
- with it off, Newt never sets up client connections, so private resources can't route through the
homelab site. Using them means dropping that flag from `services.oci.newt` *and* opening the egress
rule to whatever the private resources target - a deliberate widening of what the Pangolin server can
reach.

#### Expose a Caddy-fronted service
Every homelab service reachable through Newt sits behind Caddy, which terminates TLS and multiplexes
apps on 443 by hostname. The target is therefore always the same:

1. `NETWORK >Resources >Public` → `+ Add Resource`, `Name` e.g. `homarr`, `Type` `HTTP`,
   `Subdomain` e.g. `home.example.com`
2. `+ Add Target`: `Site` = the homelab, `Scheme` = `https`, `Address` = `host.containers.internal`,
   `Port` = `443`. `host.containers.internal` is pinned to Newt's own network gateway - reachable only
   from inside Newt's namespace, and exactly what the egress rule allows
3. If the public subdomain differs from the Caddy vhost, under
   `HTTP Settings >Additional Proxy Settings` set **TLS Server Name** and **Custom Host Header** to
   the vhost (e.g. `home.example.com`). Caddy routes on `Host`; a mismatch 502s
4. Click `Create Resource`

Newt's HTTPS proxying doesn't send SNI on the backend connection (fosrl/pangolin#207), so Caddy's
`default_sni` is what keeps the handshake alive - `TLS Server Name` only controls verification on
Pangolin's side. Caddy sees all of this traffic from Newt's fixed container IP, which
`modules/default.nix` adds to Caddy's `trustedProxies`, so backends (e.g. Vaultwarden's login rate
limiting) see real client IPs from Traefik's `X-Forwarded-For`.

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
See [Vaultwarden Example](vault_example/README.md). It relies on a Private resource - see the
[`DISABLE_CLIENTS` note](#access-control) above before applying it to this setup.

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
