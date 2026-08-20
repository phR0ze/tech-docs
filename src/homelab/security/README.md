# Security <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Research and practices for securely exposing homelab services to the public internet without
putting the rest of the home network at risk. The core principle is ***making exposure
deliberate and recovery easy*** — assume anything reachable from the internet will eventually be
probed or attacked, and design so a compromise of one exposed service can't cascade into the rest
of the network.

### Quick links
- [.. up dir](..)
- [Threat Model](#threat-model)
- [Network Segmentation](#network-segmentation)
- [Exposure Methods](#exposure-methods)
  - [Pangolin Reverse Tunnel](#pangolin-reverse-tunnel)
    - [VPS as the DMZ](#vps-as-the-dmz)
    - [Trust Boundary Tradeoffs](#trust-boundary-tradeoffs)
- [Reverse Proxy Hardening](#reverse-proxy-hardening)
- [Intrusion Prevention](#intrusion-prevention)
  - [Fail2ban](#fail2ban)
  - [CrowdSec](#crowdsec)
- [Authentication](#authentication)
- [Container Isolation](#container-isolation)
- [Host Hardening](#host-hardening)
- [Secrets Management](#secrets-management)
- [Monitoring and Logging](#monitoring-and-logging)
- [Backups and Recovery](#backups-and-recovery)

### Linked pages

## Threat Model
Before exposing anything, decide *what* is being protected and from *whom*:
* ***Internal network*** — a compromised exposed service must not be able to pivot to trusted
  devices, NAS shares, or credentials on the LAN
* ***The exposed service itself*** — data loss, defacement, or being used as a relay/botnet node
* ***The exposed host*** — container escape or host compromise via a vulnerable service

**References**
* [Homelab Security Best Practices](https://homelabaddiction.com/homelab-security/)
* [How to Secure a Homelab Network](https://readthemanual.co.uk/secure-your-homelab-2025/)
* [Exposing your Homelab](https://theorangeone.net/posts/exposing-your-homelab/)

## Network Segmentation
Without segmentation every device on the network can freely talk to every other device, so a
compromised IoT device or exposed service can reach a NAS or workstation directly. Segment with
VLANs and enforce the boundaries at the firewall, not just at the switch:

* ***Trusted*** — personal devices and workstations
* ***IoT*** — smart home devices, cameras, isolated from everything else
* ***DMZ*** — the reverse proxy, VPN endpoint and any other internet facing host; treated as
  ***semi-trusted*** with no direct bridge back to the trusted VLAN
* ***Lab/Services*** — internal-only self-hosted services that are never exposed directly

Only the [Reverse Proxy](#reverse-proxy-hardening) (or VPN endpoint) should live in the DMZ VLAN.
Backend services stay on an internal VLAN the DMZ can reach only on the specific ports required,
never the other direction.

**References**
* [Hardening homelab with VLAN and Zero Trust](https://informatecdigital.com/en/Homelab-hardening-with-VLAN%3A-a-complete-home-security-guide/)
* [VLAN Segmentation at Home](https://www.grandmasterj.com/blog/homelab-vlan-segmentation)
* [Segmented Homelab Network with VLANs, VPN, and Reverse Proxy](https://tmbyers.com/projects/arceus-lan/)
* [Secure Homelab: Public Plex with Cloudflare + DMZ](https://willgrana.com/posts/homelab2025/)

## Exposure Methods
Best practice is to NOT port fowarding straight from the home router entirely. This avoids an entire
category of threats and configuration hardening. Instead use a `Reverse Tunnel`. The best of these
right now is the self-hosted [Pangolin](../../networking/reverse_tunnel/pangolin/README.md) which
gives you outbound-only tunnel benefits similar to Cloudflare Tunnel without its `100mb` upload
cap or ToS restrictions (e.g. Cloudflare doesn't allow Jellyfin).

### Pangolin Reverse Tunnel
A cheap VPS ($4-6/month) runs Pangolin — `Traefik` as the reverse proxy and `Gerbil` for
`WireGuard` tunnel management — as the sole publicly reachable endpoint, accepting traffic on
80/443. The homelab itself only runs a lightweight `newt` client that opens an ***outbound-only***
WireGuard connection to the VPS. From that point the VPS bridges internet traffic to the homelab
without the home router ever opening an inbound port or exposing the home IP. See
[Pangolin](../../networking/reverse_tunnel/pangolin/README.md) for the full VPS provisioning, DNS,
and deployment walkthrough — this section covers where it fits in the overall security model.

#### VPS as the DMZ
The VPS takes the place of the [DMZ VLAN](#network-segmentation) described above — it's the only
host with a public IP and it terminates TLS via Traefik before anything reaches the homelab over
the tunnel. Model it exactly as a DMZ host:
* Treat it as semi-trusted, not trusted, even though you own and control it
* Give it no credentials or access beyond what's needed to route traffic
* Harden it the same as any [internet facing host](../../system/ubuntu/hardening/README.md) —
  it's now a target you maintain
* Scope Pangolin ***sites*** narrowly — a site maps to a subnet/host reachable over its tunnel, so
  don't collapse every exposed service behind a single site if it can be scoped tighter
* Scope Pangolin ***resources*** (the individual exposed services) to only what actually needs to
  be public

#### Trust Boundary Tradeoffs
Moving the public endpoint off the home network onto a VPS changes *what* needs defending, it
doesn't remove the need for defense.

**Gains**
* Home IP and router are never exposed; no inbound firewall rules needed at home at all
* Single choke point for access control, TLS termination, and auth
* A compromised exposed service can't directly reach the home network — it has to cross the
  WireGuard tunnel from the VPS first
* Blast radius containment: losing the VPS doesn't hand over anything on the home LAN

**New responsibilities**
* The VPS is now an internet facing host you must patch and monitor — see
  [Host Hardening](#host-hardening)
* The VPS provider itself becomes a trust boundary
* Pangolin's SSO (e.g. Google OAuth2) is a single point of failure for access — pair it with
  [MFA](../../security/iam/mfa/README.md)
* Pangolin ships with no built-in WAF or bot protection — layer
  [CrowdSec](../../system/ubuntu/hardening/README.md#crowdsec) (or Fail2ban) in front of Traefik
  on the VPS; Pangolin's own Dockerized instance runs as a
  [second, separate engine](../../networking/reverse_tunnel/pangolin/README.md#crowdsec-two-separate-engines-by-design)
  alongside the host-level one
* Misconfiguring a resource/site is now the primary way to accidentally over-expose something

## Reverse Proxy Hardening
Expose the reverse proxy — never the underlying service port directly (e.g. don't forward Plex's
`32400` to the world). The proxy becomes the single choke point where TLS termination, security
headers, rate limiting and access control are enforced.

* Terminate TLS at the proxy with a real certificate (e.g. via Let's Encrypt/ACME)
* Add security headers (`X-Frame-Options`, `Content-Security-Policy`, `Strict-Transport-Security`)
* Rate limit and cap request body sizes to blunt brute force and abuse
* Only proxy the exact routes/hosts required; deny by default

See [Caddy](../../networking/reverse_proxy/caddy/README.md) and
[Traefik](../../networking/reverse_proxy/traefik/README.md) for specific configuration.

**References**
* [What is a Reverse Proxy? Why Every Homelab Needs One](https://corelab.tech/what-is-a-reverse-proxy/)
* [Configuring a homelab reverse proxy](https://brandonio21.com/configuring-a-homelab-reverse-proxy/)

## Intrusion Prevention

### Fail2ban
Watches local logs (SSH, proxy access logs, etc.) and bans IPs after repeated failures. It only
sees what hits your own server, so it reacts after the fact rather than pre-emptively.

### CrowdSec
A modern alternative that separates detection from blocking: a local agent parses logs against a
library of community-maintained detection scenarios, and a ***bouncer*** enforces the ban at the
firewall, reverse proxy, or Cloudflare. Because bans are shared across CrowdSec's user base, known
malicious IPs can be blocked before they ever show up in local logs. It can also run as WAF-style
middleware in front of Caddy/Traefik, inspecting requests for SQLi/XSS/command-injection patterns
via its AppSec component.

For internet facing homelab services, prefer CrowdSec over (or alongside) Fail2ban for the shared
threat intelligence and its broader library of pre-built detections.

**References**
* [Fail2ban vs CrowdSec](https://edywerder.ch/fail2ban-vs-crowdsec/)
* [Configuring CrowdSec with Traefik](https://blog.lrvt.de/configuring-crowdsec-with-traefik/)
* [Setting Up CrowdSec in Your Homelab](https://homelabstarter.com/homelab-crowdsec-setup/)

## Authentication
Put an identity-aware proxy or SSO layer in front of anything that doesn't already enforce strong
auth on its own, and require MFA wherever possible. See [IAM](../../security/iam/README.md),
[MFA](../../security/iam/mfa/README.md) and [OAuth](../../security/iam/oauth/README.md).

## Container Isolation
Most homelab services run in Docker/Podman — treat every exposed container as hostile territory
if it's ever compromised:

* Run as a non-root user (`--user`), or use rootless Docker/Podman entirely
* Drop all capabilities and add back only what's required
  (`--cap-drop=ALL --cap-add=NET_BIND_SERVICE`)
* Mount the filesystem read-only where possible (`--read-only`, with `--tmpfs /tmp` for scratch
  space)
* Set `--security-opt=no-new-privileges`
* Avoid `:latest` tags; pin versions so upgrades are deliberate
* One Docker network per stack — don't let an exposed stack share a network with internal-only
  services
* Never mount the Docker socket into an internet facing container

**References**
* [10 Docker Security Best Practices for Self-Hosters](https://blog.byte-guard.net/docker-security-best-practices/)
* [Hardening Docker: Rootless Mode and User Namespaces](https://dohost.us/index.php/2026/03/26/hardening-docker-using-rootless-mode-and-user-namespaces-for-security/)
* [Docker Engine security](https://docs.docker.com/engine/security/)

## Host Hardening
The host running exposed services needs the same baseline hardening as any internet facing
server — key-only SSH, automatic security updates, a default-deny firewall, and unused services
removed. See [Ubuntu Hardening](../../system/ubuntu/hardening/README.md) for a concrete checklist
and [Firewall](../../networking/firewall/README.md) for NixOS firewall configuration.

## Secrets Management
Never bake credentials into container images or commit them to configuration repos. Use a secrets
manager or encrypted-at-rest store such as [sops-nix](../../system/nixos/secrets/sops_nix/README.md)
for NixOS, or a password manager like [Vaultwarden](../../security/password_managers/vaultwarden/README.md)
for day to day credentials.

## Monitoring and Logging
Centralize logs from the reverse proxy, host firewall, and intrusion prevention tooling so an
incident can actually be investigated after the fact. At minimum, ship reverse proxy access logs
and auth logs somewhere outside the exposed host itself, so an attacker who gains access can't also
erase the evidence.

## Backups and Recovery
Assume an exposed service will eventually be compromised or need to be rebuilt. Keep configuration
declarative/version controlled and data backed up off the exposed host, so recovery is a redeploy
rather than an investigation under pressure. See [Backup](../../hardware/storage/backup/README.md).
