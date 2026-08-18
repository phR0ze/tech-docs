# Crowdsec <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Crowdsec is a behavior-based intrusion prevention system: an **agent** parses logs against
scenarios to detect attack patterns and issue decisions, and a separate **bouncer** enforces those
decisions (usually as `iptables`/`nftables` bans). It shares anonymized threat intel across its
community, so a pattern seen elsewhere can inform decisions here too. This page is a general
reference; see [Ubuntu Hardening](../../system/ubuntu/hardening/README.md#crowdsec) for the
walkthrough as applied to a specific internet-facing VPS, [Pangolin](../../networking/reverse_tunnel/pangolin/README.md#crowdsec-two-separate-engines-by-design)
for running a second, separate Dockerized instance alongside the host one, and
[Alert Recon](../alert_recon/README.md) for the triage sequence to run when an alert fires.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Crowdsec vs Fail2ban](#crowdsec-vs-fail2ban)
* [Deployment](#deployment)
  * [Install the Agent](#install-the-agent)
  * [Install a Bouncer](#install-a-bouncer)
  * [Dockerized Deployment](#dockerized-deployment)
* [Configuration](#configuration)
  * [Collections](#collections)
  * [Acquisition](#acquisition)
  * [Custom Port Services](#custom-port-services)
* [Recon](#recon)
  * [Check Metrics](#check-metrics)
  * [Check Active Decisions](#check-active-decisions)
  * [Check Bouncer Registration](#check-bouncer-registration)
  * [Diagnosing a Ban That Fired Unexpectedly](#diagnosing-a-ban-that-fired-unexpectedly)
* [Managing Decisions](#managing-decisions)
  * [Ban an IP Manually](#ban-an-ip-manually)
  * [Clear a Ban](#clear-a-ban)
* [Two Separate Engines](#two-separate-engines)
* [Testing Safely](#testing-safely)

## Overview
Crowdsec splits detection from enforcement: the agent (`crowdsec`) reads logs, matches them against
installed **scenarios** (e.g. `crowdsecurity/sshd-bf` for SSH brute force), and records a
**decision** (ban, captcha, etc.) — but does nothing to actually block traffic on its own. A
**bouncer** is a separate component that watches the agent's decision stream and applies it, most
commonly as a firewall rule.

**References**
* [Crowdsec Hub](https://hub.crowdsec.net) — available collections, parsers and bouncers.
* [Crowdsec Docs](https://docs.crowdsec.net/)

### Crowdsec vs Fail2ban
[Fail2ban](../fail2ban/README.md) and Crowdsec solve the same core problem — ban IPs showing
malicious signs — but differ in scope: fail2ban is filter-per-jail and mostly self-contained,
Crowdsec is scenario-based, decouples detection from enforcement via separate bouncers, and can
consult community threat intelligence in addition to local log patterns. They aren't mutually
exclusive — running both, watching different services, is common.

## Deployment

### Install the Agent
```bash
$ curl -s https://install.crowdsec.net | sudo sh
$ sudo apt install crowdsec
```
The installer runs `cscli setup` interactively, auto-detecting running services (e.g. `sshd`) and
enabling matching collections for you. Check what actually got enabled before installing more:
```bash
$ sudo cscli collections list
```
It also writes acquisition config for whatever it detected (e.g.
`/etc/crowdsec/acquis.d/setup.sshd.yaml`) but doesn't load it into the running agent automatically —
reload to pick it up:
```bash
$ sudo systemctl reload crowdsec
```

### Install a Bouncer
The agent only detects and scores threats — it enforces nothing on its own. A bouncer is what turns
a decision into an actual firewall ban:
```bash
$ sudo apt install crowdsec-firewall-bouncer-iptables
```

### Dockerized Deployment
Crowdsec also ships an official container image, useful when it needs to sit alongside a
Docker-based stack (e.g. Pangolin's Traefik-facing instance):
```yaml
crowdsec:
  image: docker.io/crowdsecurity/crowdsec:latest
  environment:
    GID: "1000"
    COLLECTIONS: crowdsecurity/traefik crowdsecurity/appsec-virtual-patching crowdsecurity/appsec-generic-rules
    PARSERS: crowdsecurity/whitelists
  volumes:
    - ./config/crowdsec:/etc/crowdsec
    - ./config/crowdsec/db:/var/lib/crowdsec/data
```
`COLLECTIONS`/`PARSERS` set as environment variables install those collections on container start —
no manual `cscli collections install` step needed for a fresh container. See
[Pangolin's Manual Install](../../networking/reverse_tunnel/pangolin/README.md#manual-install-docker-compose)
for the full compose file this is drawn from, including the matching Traefik plugin config that
turns agent decisions into actual blocks for HTTP traffic.

## Configuration

### Collections
A collection bundles a parser + scenarios for a given service. Install anything the auto-setup
missed:
```bash
$ sudo cscli collections install crowdsecurity/sshd
$ sudo cscli collections install crowdsecurity/linux
```
List what's currently enabled:
```bash
$ sudo cscli collections list
```

### Acquisition
Acquisition config (`/etc/crowdsec/acquis.d/*.yaml`) tells the agent which log files to read and
which parser to apply. Auto-generated entries from `cscli setup` are a reasonable starting point —
confirm the `filenames`/`labels` in a given file actually match what's installed before assuming a
service is being watched:
```bash
$ cat /etc/crowdsec/acquis.d/*.yaml
```

### Custom Port Services
Unlike fail2ban, Crowdsec's `sshd` scenarios key off log line content (`Failed password ...`), not
the port number in isolation — so a custom SSH port doesn't need a separate config change the way a
fail2ban jail's `port` line does. The bouncer still needs to be watching the correct chain to
actually block that port's traffic; see the `DOCKER-USER` chain note below if the service is
containerized.

## Recon

### Check Metrics
```bash
$ sudo cscli metrics
```
Shows parser hit counts and scenario trigger counts — useful for confirming the agent is actually
processing the log volume you expect, separate from whether any ban resulted.

### Check Active Decisions
```bash
$ sudo cscli decisions list
```
Shows the matched scenario, event count, and time remaining until auto-expiry for each active ban.

### Check Bouncer Registration
```bash
$ sudo cscli bouncers list
```
A registered bouncer with a stale/blank "last pull" time means decisions exist but aren't being
enforced — the agent is detecting, the bouncer isn't acting on it.

### Diagnosing a Ban That Fired Unexpectedly
Repeated pre-auth connections from one IP in a short window — even with zero failed
*authentications* — can trip the `ssh-bf` scenario on its own. Recon tooling is a common, entirely
legitimate trigger: `ssh-keyscan` opens one connection per host-key algorithm and disconnects before
ever authenticating, and running it more than once or twice in quick succession looks identical to a
scan from Crowdsec's point of view. Fail2ban's default `sshd` filter, by contrast, only keys off
actual `Failed password` log lines, so it won't fire from this pattern alone — don't assume it's the
culprit without checking both:
```bash
$ sudo cscli decisions list
$ sudo fail2ban-client status sshd
```

## Managing Decisions

### Ban an IP Manually
```bash
$ sudo cscli decisions add --ip <ip> --duration 24h --reason manual
```

### Clear a Ban
```bash
$ sudo cscli decisions delete --ip <ip>
```

## Two Separate Engines
A host-installed agent and a Dockerized one (e.g. Pangolin's `--crowdsec` stack) are entirely
independent — separate databases, separate `cscli` contexts, separate decision lists. Querying the
host's `cscli` tells you nothing about the container's state and vice versa:
```bash
$ sudo cscli decisions list                              # host engine
$ sudo docker compose exec crowdsec cscli decisions list # containerized engine
```
If a periodic digest script only queries one, it's blind to the other — see
[Extend Monitoring & Alerts for Pangolin](../../networking/reverse_tunnel/pangolin/README.md#extend-monitoring--alerts-for-pangolin)
for wiring both into a single alert.

***The `iptables` bouncer only protects traffic reaching the host's own chains.*** Docker manages
`iptables`/`nftables` directly and inserts its own rules ahead of the host's, so a container
publishing ports (as Traefik/Gerbil do) is reachable regardless of the host bouncer's rules. Point
the bouncer at the `DOCKER-USER` chain instead of relying on the default `INPUT` chain alone if it
needs to cover those ports too.

## Testing Safely
Same rule as fail2ban: ***never trigger a test ban from the same session/IP you're currently
connected with*** — a ban blocks all traffic from that IP, established sessions included. Test from
a different network, confirm with `cscli decisions list`, then clear it:
```bash
$ sudo cscli decisions delete --ip <test-ip>
```
See [Alert Recon](../alert_recon/README.md) for the full recovery sequence if a test locks you
out anyway.
