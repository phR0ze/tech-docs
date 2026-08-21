# Hardening <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Practical steps for hardening a `Ubuntu` system, written with a [Pangolin](../../../networking/reverse_tunnel/pangolin/README.md)
VPS in mind — the sole internet-facing ingress point into a homelab — but applicable to any
internet facing server or desktop. Sections are ordered to match the sequence a fresh server
should be worked through: patch first, lock down identity and access, then layer on intrusion
prevention, attack-surface reduction, ongoing maintenance and auditing. Follow it top to bottom.
Steps marked **If this VPS runs `Pangolin`** call out where its Docker-based deployment (Gerbil,
Traefik, Newt) changes or adds to the generic advice.

### Quick links
* [.. up dir](..)
* [System Updates](#system-updates)
* [System Timezone](#system-timezone)
  * [Scheduled Job Times](#scheduled-job-times)
* [User Accounts](#user-accounts)
  * [Disable root login](#disable-root-login)
  * [Use passwordless access](#use-passwordless-access)
* [Harden sshd](#harden-sshd)
* [Firewall](#firewall)
* [GeoIP Blocking](#geoip-blocking)
  * [Install `ipset`](#install-ipset)
  * [Add an admin allow-list](#add-an-admin-allow-list)
  * [Keep ipset populated across reboots](#keep-ipset-populated-across-reboots)
  * [Persisting DOCKER-USER Rules](#persisting-docker-user-rules)
* [Manual IP Blocklist](#manual-ip-blocklist)
* [Fail2ban](#fail2ban)
* [Crowdsec](#crowdsec)
  * [Diagnosing and clearing a ban](#diagnosing-and-clearing-a-ban)
* [Automatic Updates](#automatic-updates)
* [Service Minimization](#service-minimization)
  * [Remove services](#remove-services)
  * [Disable services](#disable-services)
  * [To think about later?](#to-think-about-later)
  * [Never disable](#never-disable)
* [File System Hardening](#file-system-hardening)
* [AppArmor](#apparmor)
* [Auditd](#auditd)
* [Kernel Parameters](#kernel-parameters)
  * [Already hardened by RackNerd's image](#already-hardened-by-racknerds-image)
* [Monitoring & Alerts](#monitoring--alerts)
  * [Ship Logs Off-Box](#ship-logs-off-box)
  * [Alert on Service Failures](#alert-on-service-failures)
  * [Security Review Digest](#security-review-digest)
* [Auditing with Lynis](#auditing-with-lynis)

## System Updates
Patch known vulnerabilities before anything else — this is the first action on a fresh server.

```bash
$ sudo apt update && sudo apt full-upgrade -y
$ sudo reboot
```

## System Timezone
A fresh VPS image typically defaults to `UTC`. Set it to your own timezone early — before any of
the scheduled jobs later in this doc are configured — since `unattended-upgrades`' automatic reboot
window, every `systemd` timer's `OnCalendar`, and `cron` all interpret a bare `HH:MM`/schedule
against the system's local timezone by default. Setting it once here means none of those need a
timezone-aware variant or manual recalculation later, and it stays correct across DST transitions
automatically — the timezone database, not a fixed offset, is what's doing the work.

```bash
$ sudo timedatectl set-timezone America/Boise
```

**Verify**
```bash
$ timedatectl
```
`Time zone` should read `America/Boise (MDT, -0600)` or `(MST, -0700)` depending on time of year.

**Pick your own zone** if you're not in Mountain Time — list valid values with:
```bash
$ timedatectl list-timezones | grep -i boise
```
Mountain Time has two IANA entries with identical `MST`/`MDT` offsets — `America/Denver` and
`America/Boise` — because `tzdata` splits zones by *historical* DST-observance differences, not
current ones; the two have followed the same rules for decades now, so either resolves to the same
wall-clock time today. Use whichever matches your actual location.

***Side effect worth knowing***: log timestamps shift too. `journalctl`, `auth.log`, and container
logs (`docker compose logs`) all switch to local time once this is set, not `UTC` — `rsyslog` writes
`auth.log`'s ISO 8601 timestamps in whatever the system's local timezone is, not a fixed `UTC`.
[Alert Recon](../../../security/alert_recon/README.md#checkpointing-what-youve-already-reviewed)'s
checkpoint file depends on matching that exactly for its plain string-comparison filtering to stay
correct — see that doc's own note on why its checkpoint uses local `date`, not `-u`/UTC.

### Scheduled Job Times
Every cron/timer job across this doc and the [Pangolin doc](../../../networking/reverse_tunnel/pangolin/README.md)
that touches this VPS is deliberately spread across the early-morning hours (local time, once the
timezone above is set) rather than left at whatever hour each section happened to introduce it —
avoids two jobs landing on the same minute and fighting over disk/CPU, or a reboot cutting off a
backup mid-write:

| Local time | Job | Frequency | Section |
|------------|-----|-----------|---------|
| `01:00` Sun | GeoIP blocklist refresh | Weekly | [GeoIP Blocking](#geoip-blocking) |
| `03:00` | `unattended-upgrades` reboot window | Daily, conditional — only if a reboot is actually pending | [Automatic Updates](#automatic-updates) |
| `04:00` | Pangolin config backup | Daily | [Back Up Pangolin's State](../../../networking/reverse_tunnel/pangolin/README.md#back-up-pangolins-state) |
| `05:00` | Security review digest | Daily | [Security Review Digest](#security-review-digest) |
| `09:00` | Pangolin image-version check | Daily | [Alert on New Upstream Image Versions](../../../networking/reverse_tunnel/pangolin/README.md#alert-on-new-upstream-image-versions) |
| `06:00`–`07:00`* | `apt-daily-upgrade.timer` (`unattended-upgrades` run itself, distinct from the `03:00` reboot window above) | Daily | *Ubuntu system default, not configured by this doc* |
| `06:00`/`18:00`, ±12h*| `apt-daily.timer` (package index/download only) | Twice daily | *Ubuntu system default, not configured by this doc* |

`04:00` is deliberately placed, not just spaced — the Pangolin backup runs *after* the `03:00` reboot
window plus a buffer, so it captures the post-reboot state and doesn't risk running mid-reboot. The
`09:00` image-version check is the one deliberate exception to
the "early morning" pattern — it pushes an `ntfy` notification meant to actually be seen, so it stays
in normal waking hours rather than joining the night-time spread. The two `apt-daily*` rows are listed
for completeness, not because this doc sets them — they ship as Ubuntu system defaults
(`systemctl cat apt-daily-upgrade.timer` / `apt-daily.timer`) with their own `RandomizedDelaySec`
(`60m` / `12h` respectively), so unlike every other row here their actual fire time drifts within that
window rather than landing on the minute.

***If you change the system timezone after these timers already exist*** (i.e. running
[System Timezone](#system-timezone) on a VPS that's been up for a while, rather than fresh), expect
one extra out-of-window run from any `Persistent=true` timer shortly after the change — recomputing
each timer's next elapse against the new local time can make systemd think a scheduled run was missed
and fire an immediate catch-up. This is a one-time transitional artifact of the zone change, not a
sign the timer is misconfigured; `systemctl list-timers` settling back into this table's normal
windows on the next cycle confirms it self-corrected.

## User Accounts

### Disable root login
Use `sudo` from a non-root user instead of logging in directly as root.

#### Create a sudo user
```bash
$ sudo adduser alice
$ sudo usermod -aG sudo alice
```

#### Lock the root account
```bash
$ sudo passwd -l root
```

### Use passwordless access

#### Generate your ssh key
**Generate a key pair** on your local machine, not the server (`ed25519` is preferred over `rsa`)
```bash
$ ssh-keygen -t ed25519 -C "alice@example"
$ ssh-copy-id alice@server
```

## Harden sshd

1. **Edit `/etc/ssh/sshd_config`**
   ```
   PermitRootLogin no
   PasswordAuthentication no
   PubkeyAuthentication yes
   X11Forwarding no
   MaxAuthTries 3
   AllowUsers alice
   Port 2222
   ```
2. **Apply changes**
   ```bash
   $ sudo systemctl daemon-reload
   $ sudo systemctl restart ssh.socket
   $ sudo systemctl restart ssh
   ```

***Before closing your current session***, open a second terminal and confirm you can still log in
with the new settings — locking yourself out of a VPS with no console access is a real risk.

## Firewall
`ufw` (Uncomplicated Firewall) is a friendly front end for `iptables`/`nftables`.

**Enable with default deny incoming**
```bash
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2222/tcp
$ sudo ufw enable
```
***Don't use `ufw allow OpenSSH`*** if you changed the SSH port above — it's a static application
profile hardcoded to `22/tcp` and won't follow the `Port` change in `sshd_config`. Allow the actual
port number instead, or you'll open port 22 (nothing listening) while your real port stays
blocked.

**If this VPS runs `Pangolin`**, also allow its required ports (Gerbil's site and client
tunnels, plus HTTP/S for Traefik and ACME):
```bash
$ sudo ufw allow 80/tcp
$ sudo ufw allow 443/tcp
$ sudo ufw allow 443/udp
$ sudo ufw allow 51820/udp
$ sudo ufw allow 21820/udp
```
`443/tcp` and `51820/udp` are always required. The other three are conditional, not blanket
musts — see [Port Requirements](../../../networking/reverse_tunnel/pangolin/README.md#port-requirements)
in the Pangolin doc for when `80/tcp` (skippable with a DNS-01 cert resolver), `443/udp`
(skippable if you disable HTTP/3, on by default in Pangolin's own installer template), and
`21820/udp` (skippable if you never expose Private/ZTNA resources) can be left closed instead.

**Check status**
```bash
$ sudo ufw status verbose
```

***Docker bypasses `ufw`*** — Docker manages `iptables`/`nftables` directly and inserts its rules
ahead of `ufw`'s, so a container started with `ports:` (as Pangolin/Gerbil/Newt are) is reachable
from the internet regardless of `ufw deny` rules. Since Pangolin's components run in Docker, treat
`ufw` as informational for those ports and lock them down with
[`ufw-docker`](https://github.com/chaifeng/ufw-docker) or explicit `DOCKER-USER` chain rules
instead — see [Persisting DOCKER-USER Rules](#persisting-docker-user-rules) below for how this doc
sets up and persists the latter. This doesn't matter for 80/443/51820/21820 since those are meant to
be public, but it matters for anything else you containerize later that shouldn't be.

## GeoIP Blocking
Restrict inbound connections to a single country (e.g. `US`) using `ipset` fed by
[ipdeny.com](https://www.ipdeny.com/ipblocks/)'s per-country CIDR lists, wired into `ufw` via its
`before.rules` hook. This blocks a lot of scanner/credential-stuffing noise outright, before it
ever reaches `fail2ban`/`Crowdsec` or a service's own auth.

### Install `ipset`
```bash
$ sudo apt install ipset
```

**Create a refresh script** that builds the allow-list. The US zone file has well over 65,536 CIDR
entries — `ipset`'s default `maxelem` for a `hash:net` set — so the set is created with a much
higher `maxelem`, or loading fails partway through with `Hash is full, cannot add more elements`.
This also loads entries in one bulk `ipset restore` pass rather than forking `ipset add` per line —
the per-line version can take several minutes (or longer on a small VPS) and will make
`systemctl start`/`enable --now` appear to hang, since it blocks until the oneshot service
finishes. The temp set is destroyed unconditionally before each run rather than reused with
`-exist` — `-exist` only suppresses ipset's "already exists" error when the create parameters
match *exactly*, so a leftover set from an interrupted or older run (different `maxelem`, say)
would otherwise fail the next run instead of just getting rebuilt. `geo-allow`'s own `create` is
allowed to fail silently instead, since `ipset swap` doesn't require the two sets' `maxelem` to
match — the create only ever needs to succeed once, the very first time the set doesn't exist yet.
`curl -f` plus `pipefail` make sure a failed download (e.g. a 404/5xx error page from ipdeny.com)
aborts the script instead of silently feeding garbage lines into `ipset restore` — without
`pipefail`, `set -e` only checks the exit status of the *last* command in a pipeline, so a failed
`curl` piped into `sed` would otherwise go unnoticed. It also saves the result to disk — that
on-disk copy is what a fast, network-free restore reads at boot (see below), so `ufw` doesn't end
up waiting on a network download before it can start.
```bash
$ sudo tee /usr/local/sbin/update-geoipset.sh > /dev/null <<'EOF'
#!/bin/bash
set -euo pipefail
ipset destroy geo-allow-tmp 2>/dev/null || true
{
  echo "create geo-allow-tmp hash:net maxelem 262144"
  curl -sf https://www.ipdeny.com/ipblocks/data/countries/us.zone \
    | sed 's/^/add geo-allow-tmp /'
} | ipset restore
ipset create geo-allow hash:net maxelem 262144 2>/dev/null || true
ipset swap geo-allow-tmp geo-allow
ipset destroy geo-allow-tmp
mkdir -p /etc/ipset
ipset save geo-allow > /etc/ipset/geo-allow.conf
EOF
$ sudo chmod +x /usr/local/sbin/update-geoipset.sh
$ sudo /usr/local/sbin/update-geoipset.sh
```

### Add an admin allow-list
Add an admin allow-list before enabling the drop rule — do not skip this.*** Static country
lists like ipdeny.com's have false negatives: a VPN, mobile carrier CGNAT, or an imperfectly
attributed ISP block can put *your own* connecting IP outside the `US` set, geo-blocking the
person managing the box with no way back in except console access. Carve out your own IP(s)
independently of the country list:
```bash
$ sudo ipset create admin-allow hash:ip -exist
$ sudo ipset add admin-allow <your-public-ip> -exist
```
Add every IP you might connect from (home, phone hotspot, etc.) — check your current one with
`curl ifconfig.me` from the connecting device, not the VPS.

**Add both rules** to `/etc/ufw/before.rules`, on their own lines right before the final `COMMIT`
— ***the `admin-allow` accept must come before the `geo-allow` drop***, since `iptables`/`ufw`
processes this chain top-to-bottom and the first match wins:
```
-A ufw-before-input -m set --match-set admin-allow src -j ACCEPT
-A ufw-before-input -m set ! --match-set geo-allow src -j DROP
```

**Apply and verify**
```bash
$ sudo ufw reload
$ sudo ipset test admin-allow <your-public-ip>
$ sudo iptables -L ufw-before-input -v -n | grep -E 'admin-allow|geo-allow'
```
Confirm `ipset test` reports your IP is in the set *before* trusting the drop rule — test this
from a second, still-open session, exactly like recovering from a lockout above.

**Persist `admin-allow` across reboots** — unlike `geo-allow`, it isn't rebuilt from a download, so
save it to a file the boot service can restore:
```bash
$ sudo mkdir -p /etc/ipset
$ sudo ipset save admin-allow | sudo tee /etc/ipset/admin-allow.conf > /dev/null
```
The `tee` is required, not just style — `sudo cmd > file` redirects in your own shell *before*
`sudo` runs, so it's your user (not root) trying to write into `/etc/ipset`, which fails with
`Permission denied`. Re-run this `save` any time you add or remove an IP from `admin-allow`.

### Keep ipset populated across reboots
We need to persist the ipsets. IP sets are in-memory only by default, and `ufw` will
fail to (re)apply its rules if either set doesn't exist yet when it starts. `ufw.service` itself is
deliberately early-boot (`DefaultDependencies=no`, `Before=network-pre.target`) so the firewall is
up before networking is. That means the restore step ***must not*** depend on `network-online.target`
— doing so would put it after `ufw.service` in the dependency graph while also being required
*before* it, which is an ordering cycle systemd will detect and break for you, silently leaving
`ufw` to start without waiting on the sets after all. The restore below only reads the on-disk
copies (no network involved), so it can sit in the same early-boot tier as `ufw` itself:
```bash
$ sudo tee /etc/systemd/system/geoipset.service > /dev/null <<'EOF'
[Unit]
Description=Restore persisted ipsets before ufw starts
DefaultDependencies=no
Before=ufw.service
After=local-fs.target
Requires=local-fs.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c '/sbin/ipset restore -exist < /etc/ipset/admin-allow.conf'
ExecStart=/bin/sh -c '/sbin/ipset restore -exist < /etc/ipset/geo-allow.conf'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now geoipset.service
$ sudo systemctl edit ufw.service
```
Add, so `ufw` waits for the sets to exist on boot:
```
[Unit]
After=geoipset.service
Requires=geoipset.service
```
The network-dependent refresh (the `update-geoipset.sh` script itself) stays out of the boot path
entirely — it only needs to run periodically via the cron job below, updating both the live set and
its on-disk copy for next time.

**Refresh the list periodically** — country CIDR allocations do change
```bash
$ (sudo crontab -l 2>/dev/null; echo "0 1 * * 0 /usr/local/sbin/update-geoipset.sh") | sudo crontab -
```

### Persisting DOCKER-USER Rules
Docker bypasses this the same way it bypasses `ufw` — a rule in `ufw-before-input` doesn't see
traffic Docker DNATs straight to a published container port. If this VPS runs `Pangolin`, the
`admin-allow`/`geo-allow` rules above (and `manual-block`, set up in
[Manual IP Blocklist](#manual-ip-blocklist) below) all need a mirror in the `DOCKER-USER` chain to
actually cover 80/443/51820/21820 — one unified mechanism here covers all three rather than
repeating the setup per section. Two things make a bare `iptables -A DOCKER-USER ...` insufficient
on its own:

* **Rule order** — `dockerd` creates `DOCKER-USER` with a trailing `-j RETURN` rule the moment the
  daemon starts, so any traffic that reaches the chain without matching an earlier rule immediately
  jumps back to Docker's own chain instead of falling through to anything appended after it. `-A`
  (append) adds a rule *after* that existing `RETURN`, where it's never evaluated. Insert with
  `-I DOCKER-USER 1` instead, so the new rule lands ahead of it.
* **Nothing restores it automatically** — unlike `ufw-before-input` (rebuilt by `ufw reload`/on boot
  from `/etc/ufw/before.rules`), a rule added straight to `DOCKER-USER` lives only in the running
  kernel table. It doesn't survive a reboot, and `dockerd` re-creates the chain fresh each time it
  starts, silently dropping anything added by hand.

#### Create an idempotent apply script
Create an idempotent apply script — checks each rule before inserting so re-running it (e.g. on
every `docker.service` start) doesn't stack duplicate rules. Rules are inserted in *reverse* priority
order deliberately: each `-I DOCKER-USER 1` pushes to the very top of the chain, so the last one
inserted ends up evaluated first — the sequence below leaves `admin-allow`'s ACCEPT on top, `geo-allow`'s
DROP below it, `manual-block`'s DROP below that, and Docker's own trailing `RETURN` last:
```bash
$ sudo tee /usr/local/sbin/apply-docker-user-rules.sh > /dev/null <<'EOF'
#!/bin/bash
set -euo pipefail
insert_at_top() {
  if ! iptables -C DOCKER-USER "$@" 2>/dev/null; then
    iptables -I DOCKER-USER 1 "$@"
  fi
}
insert_at_top -m set --match-set manual-block src -j DROP
insert_at_top -m set ! --match-set geo-allow src -j DROP
insert_at_top -m set --match-set admin-allow src -j ACCEPT
EOF
$ sudo chmod +x /usr/local/sbin/apply-docker-user-rules.sh
```
Adding, removing, or reordering a mirror rule later means editing this one script — it's the single
source of truth for all three `DOCKER-USER` mirrors, not three copy-pasted `iptables` commands scattered
across sections.

#### Hook it into `docker.service` itself
Hook it into the `docker.service` itself, rather than a standalone systemd unit — `ExecStartPost` runs
every time the daemon (re)starts, which is the actual moment `DOCKER-USER` gets reset, including after
an `unattended-upgrades`-triggered Docker package update:
```bash
$ sudo systemctl edit docker.service
```

Add the following:
```
[Unit]
After=geoipset.service

[Service]
ExecStartPost=/usr/local/sbin/apply-docker-user-rules.sh
```
`After=geoipset.service` ensures the `admin-allow`/`geo-allow`/`manual-block` ipsets themselves already
exist by the time the script runs — matching against a nonexistent set makes `iptables` error out
rather than silently skip it.

#### Apply now without restarting Docker
Apply now without restarting Docker — a full `docker.service` restart would briefly take down every
Pangolin container, so run the script directly this first time instead of restarting the daemon just to
trigger `ExecStartPost`:
```bash
$ sudo systemctl daemon-reload
$ sudo /usr/local/sbin/apply-docker-user-rules.sh
```

**Verify**
```bash
$ sudo iptables -L DOCKER-USER -v -n --line-numbers
```
Should show, top to bottom: the `admin-allow` ACCEPT, `geo-allow` DROP, `manual-block` DROP, then
Docker's own `RETURN`. Repeat traffic that should be blocked (e.g. a confirmed-malicious IP already in
`manual-block`) climbing that rule's packet counter is the real confirmation it's active — not just
that the rule exists.

***Consider scope before blocking `Pangolin`'s public ports (80/443) via the `geo-allow` mirror***
— Pangolin already has per-resource
[Resource Rules](../../../networking/reverse_tunnel/pangolin/README.md#how-access-control-works)
that can geo-filter individual resources. A blanket host-level block on 80/443 is coarser: it
affects every resource the same way, including ones you might want reachable from outside the US
(e.g. a family member traveling). Applying it to SSH and any admin-only surface is close to
risk-free; applying it to 80/443 is a real trade-off, not just a stronger version of the same
control. This caveat doesn't apply to `manual-block` — a confirmed-malicious IP has no legitimate
traffic to preserve.

## Manual IP Blocklist
Fail2ban and Crowdsec bans below are both tied to that mechanism's own lifecycle — a fail2ban ban
clears on service restart unless persistent DB storage is configured, and a Crowdsec decision always
carries an expiry. For a small set of confirmed-malicious IPs you want blocked permanently, an
`ipset`-based blocklist is the right tool: independent of either mechanism, persists across reboots,
and — critically at scale — keeps `sudo ufw status numbered` down to a single rule no matter how many
IPs accumulate, instead of one `ufw` rule per IP. Same swap-based pattern as `admin-allow`/`geo-allow`
above, just a plain deny list instead of a country allow-list.

**1. Create the source-of-truth file** — one IP per line, `#` for comments. This file is never read
directly by `ipset` or `ufw` — it only takes effect once the rebuild script below loads it:
```bash
$ sudo mkdir -p /etc/ipset
$ sudo tee /etc/ipset/manual-block-list.txt > /dev/null <<'EOF'
# known malicious IPs — one per line
91.92.42.126
91.92.40.36
EOF
```

**2. Create a rebuild script** — same swap-based pattern as `update-geoipset.sh`, so a bad edit never
leaves the set half-populated. This is what actually parses `manual-block-list.txt` and loads it into
the live `manual-block` set — it also writes `/etc/ipset/manual-block.conf`, a separate,
machine-generated file in `ipset save` format used only for boot persistence (step 4), not something
you edit by hand:
```bash
$ sudo tee /usr/local/sbin/update-manual-block.sh > /dev/null <<'EOF'
#!/bin/bash
set -euo pipefail
ipset destroy manual-block-tmp 2>/dev/null || true
{
  echo "create manual-block-tmp hash:ip maxelem 65536"
  grep -vE '^\s*(#|$)' /etc/ipset/manual-block-list.txt | sed 's/^/add manual-block-tmp /'
} | ipset restore
ipset create manual-block hash:ip -exist
ipset swap manual-block-tmp manual-block
ipset destroy manual-block-tmp
ipset save manual-block > /etc/ipset/manual-block.conf
EOF
$ sudo chmod +x /usr/local/sbin/update-manual-block.sh
$ sudo /usr/local/sbin/update-manual-block.sh
```

**3. Add one `ufw` rule matching the whole set**, in `/etc/ufw/before.rules` right before `COMMIT`
(same spot as `admin-allow`/`geo-allow`):
```
-A ufw-before-input -m set --match-set manual-block src -j DROP
```
```bash
$ sudo ufw reload
```

**4. Persist across reboots** by adding a third restore line to the `geoipset.service` unit set up
above, alongside its existing `admin-allow`/`geo-allow` restores:
```bash
$ sudo systemctl edit geoipset.service
```
Add:
```
[Service]
ExecStart=/bin/sh -c '/sbin/ipset restore -exist < /etc/ipset/manual-block.conf'
```

**If this VPS runs Pangolin**, this set also needs a `DOCKER-USER` mirror to cover 80/443/etc. — same
Docker-bypasses-`ufw` caveat as `geo-allow` above. The `manual-block` DROP is already included in
[Persisting DOCKER-USER Rules](#persisting-docker-user-rules)'s apply script — if that section is set
up (do it once, alongside `geo-allow`'s own mirror), nothing further is needed here. If it isn't yet, go
set it up now: a `manual-block` entry with no `DOCKER-USER` mirror in place silently doesn't cover ports
80/443/51820/21820 at all, even though `ipset list manual-block` shows the IP present.

**5. Add a helper script for the recurring workflow** — wraps the append-then-rebuild sequence into
one command instead of hand-editing the list file each time. `grep -qxF` does an exact, whole-line,
fixed-string match rather than a substring search, so adding `1.2.3.4` doesn't false-positive against
an existing `11.2.3.4` entry:
```bash
$ sudo tee /usr/local/sbin/block-ip.sh > /dev/null <<'EOF'
#!/bin/bash
set -euo pipefail
IP="${1:?Usage: block-ip.sh <ip>}"
LIST=/etc/ipset/manual-block-list.txt
if grep -qxF "$IP" "$LIST"; then
  echo "$IP is already in $LIST"
  exit 0
fi
echo "$IP" | tee -a "$LIST" > /dev/null
/usr/local/sbin/update-manual-block.sh
echo "Added $IP to $LIST"
ipset list manual-block | grep -qxF "$IP" && echo "Verified: $IP is active in the manual-block ipset"
EOF
$ sudo chmod +x /usr/local/sbin/block-ip.sh
```

**Usage**
```bash
$ sudo /usr/local/sbin/block-ip.sh <ip>
```

See [Alert Recon's Manually Banning IPs](../../../security/alert_recon/README.md#manually-banning-ips)
for the day-to-day workflow of adding a newly discovered malicious IP once this is set up.

## Fail2ban
`fail2ban` bans IPs that show malicious signs such as repeated failed SSH login attempts.

```bash
$ sudo apt install fail2ban
$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Edit `/etc/fail2ban/jail.local` to tune `bantime`, `findtime` and `maxretry`, then enable the `sshd`
jail. `jail.local` has an `[sshd]` section (copied from `jail.conf`) — find and edit that *existing*
section, don't paste a new one:
```bash
$ grep -n '^\[sshd\]' /etc/fail2ban/jail.local
```
Within it, set `enabled = true`, and change the existing `port = ssh` line (resolves to 22) to `port
= 2222`. Also set `bantime = 1h` as the default `10m` is too short for an internet-facing SSH port
(scanners will just wait and resume).
```
enabled = true
port = 2222
bantime = 1h
```
Confirm syntax of configuration file
```bash
$ sudo fail2ban-client -t
```
Without the `port` line, the jail's ban action targets the wrong port and won't actually block
anything useful.

***WARNING: `enable = true` is not a real directive — the key is `enabled`.*** This is an easy typo to make
and `fail2ban-client -t` won't catch it: `-t` only validates syntax, not whether a directive *name*
is one fail2ban actually recognizes. A misspelled key is silently dropped, and the jail falls back
to `jail.conf`'s shipped default for `[sshd]`, which is `enabled = false` — so the jail never
watches anything, `Total failed`/`Total banned` sit at `0` indefinitely, and repeated brute-force
attempts in `auth.log` go completely unactioned even though the config "looks" fine and passes
validation. Double-check the spelling, restart, and confirm the jail is actually live before
trusting it:
```bash
$ sudo systemctl restart fail2ban
$ sudo fail2ban-client status sshd
```
A genuinely active jail shows `Currently failed`/`Total failed` counters that climb as `auth.log`
grows. If they stay at `0` while you know failed attempts are happening, re-check this exact typo
before assuming something else is wrong — see [Alert Recon](../../../security/alert_recon/README.md)
for the fuller diagnostic sequence.

**Check status**
```bash
$ sudo fail2ban-client status sshd
```

**Test a ban safely** — from a *different* network than your active session (phone hotspot, not the
machine you're SSH'd in from). A ban is a firewall `DROP`/`REJECT` rule against the source IP, which
blocks *all* traffic from that IP — established sessions included, not just new login attempts — so
testing from your own session's IP will disconnect you:
```bash
$ ssh -p 2222 someuser@<vps-ip>   # from the OTHER network, wrong password 5x
```
Then back on the VPS:
```bash
$ sudo fail2ban-client status sshd
```
Should show `Currently banned: 1` with the test IP listed. Unban it once confirmed:
```bash
$ sudo fail2ban-client set sshd unbanip <test-ip>
```
**Add your own IP(s) to `ignoreip`** in `[DEFAULT]` so fail2ban never bans your own management
access, even by accident during testing:
```
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1 <your-ip>
```

See [Fail2ban](../../../security/fail2ban/README.md) for a deeper reference on configuration layers,
common gotchas, and tuning beyond this VPS-specific walkthrough.

## Crowdsec
`Crowdsec` is a behavior-based IPS that parses logs to detect attack patterns and shares threat
intel across its community. It's a more capable alternative (or complement) to `fail2ban`,
particularly for internet facing servers behind a reverse proxy like `Pangolin`.

**Install the agent**
```bash
$ curl -s https://install.crowdsec.net | sudo sh
$ sudo apt install crowdsec
```
The installer runs `cscli setup` interactively during install, auto-detecting running services
(e.g. `sshd`) and enabling matching collections for you — check what's already enabled before
manually installing more:
```bash
$ sudo cscli collections list
```
It also writes the acquisition config for whatever it detected (e.g. `/etc/crowdsec/acquis.d/setup.sshd.yaml`)
but doesn't load it into the running agent — reload to pick it up:
```bash
$ sudo systemctl reload crowdsec
```

**Install a bouncer** — the agent above only detects and scores threats, it doesn't block
anything on its own. As `crowdsec` itself will tell you post-install, a bouncer is what turns a
decision into an actual `iptables`/`nftables` ban:
```bash
$ sudo apt install crowdsec-firewall-bouncer-iptables
```

**If any collection you need wasn't auto-enabled above**, install it explicitly, e.g.:
```bash
$ sudo cscli collections install crowdsecurity/sshd
$ sudo cscli collections install crowdsecurity/linux
```

**Check status**
```bash
$ sudo cscli metrics
$ sudo cscli decisions list
```

**If this VPS runs `Pangolin`**, the `iptables` bouncer only protects traffic reaching the host's
own chains — same caveat as the `ufw`/Docker note above. To have it also cover ports Docker
publishes, point the bouncer at the `DOCKER-USER` chain (see the bouncer's config) rather than
relying on the default `INPUT` chain alone.

### Diagnosing and clearing a ban
Repeated pre-auth connections from one IP in a short window — even with zero failed
*authentications* — can trip Crowdsec's `ssh-bf` scenario on its own. Recon tooling is a common,
entirely legitimate trigger: `ssh-keyscan` opens one connection per host-key algorithm and
disconnects before ever authenticating, and running it more than once or twice in quick succession
looks identical to a scan from Crowdsec's point of view. `fail2ban`'s default `sshd` filter, by
contrast, only keys off actual `Failed password` log lines, so it won't fire from this pattern
alone — don't assume it's the culprit without checking both.

See [Crowdsec](../../../security/crowdsec/README.md) for a deeper reference on scenarios, the
agent/bouncer split, and running two separate engines (host-level and Pangolin's own Dockerized
instance) side by side.

***References***
* [Crowdsec Hub](https://hub.crowdsec.net) for available collections, parsers and bouncers.

**Check what's actually blocking a given IP** — each mechanism keeps an independent blocklist, so
check all three rather than assuming which one fired:
```bash
$ sudo cscli decisions list
$ sudo fail2ban-client status sshd
$ sudo ipset list admin-allow
```
`cscli decisions list` shows the scenario that matched, event count, and time remaining until the
ban auto-expires. `fail2ban-client status sshd` shows currently banned IPs for the `sshd` jail
specifically. `ipset list admin-allow` confirms which IP(s) are actually trusted right now — verify
this directly rather than relying on memory of what you added and when, especially if you're
troubleshooting on the assumption "my IP should already be allowed."

#### Clear a Crowdsec ban
You can clear a crowdsec ban manually if you so choose

```bash
$ sudo cscli decisions delete --ip <ip>
```

#### Clear a fail2ban ban
You can clear a fail2ban ban manually if you so choose

```bash
$ sudo fail2ban-client set sshd unbanip <ip>
```

## Automatic Updates
Enable unattended security updates so patches keep applying without manual intervention.

```bash
$ sudo apt install unattended-upgrades
$ sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Config lives at `/etc/apt/apt.conf.d/50unattended-upgrades` and
`/etc/apt/apt.conf.d/20auto-upgrades`.

**Enable automatic reboot** so kernel security patches actually take effect, in
`/etc/apt/apt.conf.d/50unattended-upgrades`:
```
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "03:00";
```

**If this VPS runs `Pangolin`**, the scheduled reboot is safe as long as its containers use
`restart: unless-stopped` (the default in Pangolin/Gerbil/Newt's docker-compose) — Docker brings
them back up automatically after the host restarts, no manual intervention needed.

***Known annoyance: a `wall` broadcast lands on your terminal the moment a reboot gets
scheduled*** — `unattended-upgrades` schedules the reboot by calling `shutdown -r 03:00`, and
`shutdown` immediately broadcasts a notice to every logged-in terminal. If it interrupts you
mid-command, the line just looks garbled — press `Enter` or `Ctrl-L` to clear it, nothing actually
hung. `mesg n` on your own session does *not* suppress this: it only blocks broadcasts from other
regular users, not from root (which is what `shutdown`/systemd is acting as here). To avoid the
interruption entirely while you're actively connected, set:
```
Unattended-Upgrade::Automatic-Reboot-WithUsers "false";
```
in `/etc/apt/apt.conf.d/50unattended-upgrades` — the reboot then only gets scheduled when no one
has an active session, instead of firing (and broadcasting) while you're working.

## Service Minimization
Every running service is attack surface. Disable anything not actually needed on a headless
server.

### Remove services
```bash
$ sudo apt purge snapd
$ sudo apt purge modemmanager
$ sudo apt autoremove
```

### Disable services
```bash
$ sudo systemctl disable --now gpu-manager.service ModemManager.service \
  open-vm-tools.service vgauth.service \
  multipathd.service multipathd.socket \
  open-iscsi.service iscsid.socket \
  lxd-installer.socket thermald.service udisks2.service \
  apport.service apport-autoreport.path apport-autoreport.timer apport-forward.socket \
  motd-news.timer update-notifier-download.timer update-notifier-motd.timer \
  fwupd-refresh.timer \
  mdcheck_continue.timer mdcheck_start.timer mdmonitor-oneshot.timer \
  ua-reboot-cmds.service ubuntu-advantage.service ua-timer.timer
```

**List enabled services**
```bash
$ systemctl list-unit-files --state=enabled --no-pager
```

**Verify before touching LVM-related units** — disabling these could break boot if your root
filesystem actually depends on them:
```bash
$ lsblk
$ mount | grep -i lvm
```
If nothing is LVM-backed, `lvm2-monitor.service`, `lvm2-lvmpolld.socket`, `dm-event.socket`, and
`blk-availability.service` are also safe to disable. If it is, leave all four alone.
```bash
$ sudo systemctl disable --now \
  lvm2-monitor.service lvm2-lvmpolld.socket \
  dm-event.socket blk-availability.service
```

**Verify nothing broke** after any batch of service disables — an empty result confirms no unit
failed to stop or is stuck in a bad state:
```bash
$ systemctl --failed
```

### Never disable
**Never disable**, on any VPS, regardless of role: `ssh.socket`, `cron.service`, `rsyslog.service`
(fail2ban/CrowdSec read from it), `ufw.service`, `fail2ban.service`, `crowdsec.service`,
`unattended-upgrades.service`, `apt-daily*.timer`, `systemd-networkd*`, `systemd-resolved.service`,
`systemd-timesyncd.service` (accurate clock matters for TOTP and TLS cert validation),
`logrotate.timer`, `apparmor.service`, `fstrim.timer`, and `getty@.service` (your provider's
console fallback if SSH ever breaks).

**If this VPS runs `Pangolin`**, leave `docker.service` (and `containerd.service`) enabled — it's
what runs Gerbil, Traefik and Newt. Leave `crowdsec-firewall-bouncer.service` and `geoipset.service`
enabled too if you've set up [Crowdsec](#crowdsec)/[GeoIP Blocking](#geoip-blocking) above.

## File System Hardening

**Edit `/etc/fstab`** to prevent executing binaries or set-uid programs from `/tmp` and
`/dev/shm`:
```bash
$ sudo vim /etc/fstab
```
Add these two lines at the bottom of the file — they're config, not commands to run directly:
```
tmpfs   /tmp   tmpfs   defaults,noexec,nosuid,nodev   0 0
tmpfs   /dev/shm   tmpfs   defaults,noexec,nosuid,nodev   0 0
```
Save and exit (`Esc`, then `:wq`, `Enter`).

**Validate before trusting it** — a bad `/etc/fstab` line is one of the few things that can break
boot, dropping you into an emergency shell that needs console access. Check it now, with the live
session intact, rather than finding out on next reboot:
```bash
$ sudo findmnt --verify --verbose
$ sudo systemctl daemon-reload
$ sudo mount -a
```
`findmnt --verify` checks syntax without mounting anything; `mount -a` actually applies every
`fstab` entry and errors immediately on a typo instead of silently waiting for next boot.

**Apply changes**
```bash
$ sudo mount -o remount /tmp
$ sudo mount -o remount /dev/shm
```

**Verify it took**
```bash
$ mount | grep -E ' /tmp | /dev/shm '
$ sudo findmnt --verify --verbose
```
`noexec`, `nosuid` and `nodev` should all appear in the options for both lines, and the second
command should report `0 parse errors, 0 errors, 0 warnings`.

***Two things worth knowing, not urgent but real:***
* **`tmpfs` size is RAM-bound** — it defaults to 50% of RAM if you don't set a `size=` option. On
  a small VPS this is a real ceiling if anything ever writes a large file to `/tmp`. Check current
  usage isn't already close: `df -h /tmp /dev/shm`.
* **Existing `/tmp` contents get shadowed**, not deleted, if `/tmp` was previously just a directory
  on the root filesystem — don't be surprised if disk usage on `/` doesn't drop even though `/tmp`
  now looks empty.

***Caution:*** a small number of `.deb` post-install scripts write and execute helper scripts from
`/tmp`, and `Automatic Updates` keeps running indefinitely after this point. If a future unattended
upgrade fails with a permission error pointing at `/tmp`, that's why — a separate `exec`-allowed
scratch mount (e.g. `/var/tmp`) is the usual workaround rather than reverting this section.

**If this VPS runs `Pangolin`**, don't apply these mount options to Docker's own storage
directory (`/var/lib/docker`) — leave it on the root filesystem as-is. `noexec`/`nosuid` on `/tmp`
and `/dev/shm` don't affect Gerbil, Newt or Traefik, which run entirely from their container
images.

## AppArmor
`AppArmor` is enabled by default on Ubuntu and restricts what individual applications can access.

**List loaded profiles and their mode**
```bash
$ sudo aa-status
```

## Auditd
`auditd` records security relevant events for later review.

```bash
$ sudo apt install auditd audispd-plugins
```

**Search the audit log**
```bash
$ sudo ausearch -m avc,user_avc,auditd -ts recent
```

## Kernel Parameters

**Check current values first** — Ubuntu's defaults (and any cloud-image-specific overrides your
provider ships) may already set some of these, and comparing before/after is the only way to
confirm the config below actually took effect:
```bash
$ sysctl net.ipv4.conf.all.rp_filter net.ipv4.conf.all.accept_redirects \
  net.ipv4.conf.all.send_redirects net.ipv4.tcp_syncookies kernel.dmesg_restrict
```

**On a RackNerd VPS**, the stock image already ships with hardened values
```
net.ipv4.conf.all.rp_filter=2
net.ipv4.conf.all.accept_redirects=0
net.ipv4.tcp_syncookies=1
kernel.dmesg_restrict=1
```

Only `send_redirects` (`1` by default) still needs hardening to `0`. `rp_filter=2` (loose mode) being
the default here happens to line up with the Pangolin loose-mode override noted below anyway, so
leave it as-is rather than tightening it to `1`

**Create `/etc/sysctl.d/99-hardening.conf`**
```
net.ipv4.conf.all.send_redirects=0
```

* **`rp_filter=1`** — enables strict reverse-path filtering: the kernel drops any inbound packet
  whose source address wouldn't be routed back out the interface it arrived on. This blocks IP
  spoofing, where an attacker forges a source address to hide their origin or reflect traffic at a
  third party.
* **`accept_redirects=0`** — ignores ICMP redirect messages. A host on the local network could
  otherwise send a forged redirect to silently repoint your routing table, enabling
  man-in-the-middle traffic interception.
* **`send_redirects=0`** — stops this host from emitting ICMP redirects itself. Only relevant if
  it's acting as a router; a server that's just an endpoint has no legitimate reason to send them,
  and leaving it on gives an attacker one more way to manipulate neighboring hosts' routes.
* **`tcp_syncookies=1`** — lets the kernel handle SYN floods (a common denial-of-service attack)
  by encoding connection state into the SYN-ACK sequence number instead of queuing a half-open
  connection, so the backlog can't be exhausted by forged SYNs that never complete the handshake.
* **`dmesg_restrict=1`** — restricts `dmesg` (kernel ring buffer) access to root. Kernel logs can
  leak memory addresses and driver versions useful for crafting a local privilege-escalation
  exploit; an unprivileged user or compromised low-privilege process shouldn't be able to read them.

**Apply changes**
```bash
$ sudo sysctl --system
```

**If a value doesn't reflect what you set**, `sysctl --system` applies files in a fixed order —
`/run/sysctl.d/`, `/etc/sysctl.d/`, `/usr/lib/sysctl.d/` (each glob sorted by filename), then
`/etc/sysctl.d/99-sysctl.conf` and `/etc/sysctl.conf` last — so a later file can silently override
an earlier one for the same key. Check for a competing declaration before assuming the value
didn't apply:
```bash
$ grep -rn <key> /etc/sysctl.d/ /usr/lib/sysctl.d/ /etc/sysctl.conf
```

### Already hardened by RackNerd's image
This VPS's stock RackNerd image ships several other security-relevant sysctl values out of the box
via drop-ins under `/etc/sysctl.d/` (e.g. `10-kernel-hardening.conf`, `10-network-security.conf`,
`10-ptrace.conf`) — no action needed, just worth knowing they're already there before assuming
this section is the only source of hardening on the box:

* **`kernel.apparmor_restrict_unprivileged_userns=1`** — blocks unprivileged users from creating
  user namespaces with AppArmor confinement, closing off a common container-escape/privilege-
  escalation technique.
* **`kernel.kptr_restrict=1`** — hides kernel pointer addresses from `/proc` for unprivileged
  users, same rationale as `dmesg_restrict` above: fewer addresses leaked, fewer memory-corruption
  exploits made easier.
* **`kernel.yama.ptrace_scope=1`** — restricts `ptrace` to a process's own children, preventing
  one user process from attaching to and inspecting or injecting into another unrelated process
  (e.g. reading another user's process memory for secrets).
* **`vm.mmap_min_addr=65536`** — blocks mapping memory at low virtual addresses, closing off a
  class of NULL-pointer-dereference kernel exploits that rely on mapping attacker-controlled data
  at address 0.
* **`kernel.sysrq=176`** — limits the kernel's SysRq "magic key" to a safe subset (sync, remount
  read-only, etc.) instead of the full set, which includes commands that can dump kernel memory or
  force an immediate reboot if triggered from an unprivileged context with console access.
* **`fs.protected_fifos=1`, `fs.protected_hardlinks=1`, `fs.protected_regular=2`,
  `fs.protected_symlinks=1`** — close a family of `/tmp`-style symlink/hardlink race-condition
  attacks, where a lower-privileged user pre-creates a symlink or hardlink to trick a
  higher-privileged process into writing to or through it.
* **`net.ipv6.conf.all.use_tempaddr=2`** (and `.default`) — uses temporary, randomized IPv6
  addresses for outbound connections instead of ones derived from the NIC's MAC address, so
  outbound traffic can't be used to track or fingerprint this host across networks.

Not security-relevant but also present from the same drop-ins: `net.core.default_qdisc=fq_codel`
(bufferbloat mitigation), `kernel.printk` (console log verbosity), `vm.max_map_count` (per-process
memory-map limit, relevant to apps like Elasticsearch), and `kernel.pid_max` (max PID value).

**If this VPS runs `Pangolin`**, `rp_filter=1` (strict mode) can drop legitimate return traffic
through Gerbil's WireGuard tunnels once traffic starts arriving/leaving over multiple interfaces
(the public NIC vs. the `wg`/tunnel interface). Use loose mode instead:
```
net.ipv4.conf.all.rp_filter=2
net.ipv4.conf.default.rp_filter=2
```
Gerbil also needs IP forwarding enabled to route tunneled traffic — this isn't on by default:
```
net.ipv4.ip_forward=1
```

## Monitoring & Alerts
After the changes above land, put something in place to notice anomalies, resource spikes and
service failures — hardening reduces risk, it doesn't eliminate the need to watch the box. If this
VPS runs `Pangolin`, it already has an encrypted WireGuard tunnel back to your homelab — reuse it
for log shipping instead of sending logs over the open internet or trusting a third-party SaaS.

### Ship Logs Off-Box
If an attacker gains root, the first thing they'll do is clear `journalctl`/`auth.log` to erase
evidence. A copy that already left the box before that happens is the only one you can trust.

**Run a receiver on your homelab** — a small `rsyslog` container listening only on the private
tunnel network Newt/Gerbil already established:
```yaml
# docker-compose.yml on your homelab host
services:
  rsyslog:
    image: rsyslog/syslog_appliance_alpine
    container_name: rsyslog
    restart: unless-stopped
    ports:
      - "<homelab-tunnel-ip>:514:514/udp"
    volumes:
      - ./rsyslog-data:/var/log
```
Bind it to the tunnel-side address specifically (not `0.0.0.0`) so it's unreachable from your LAN
or the internet, only from the VPS across the WireGuard link.

**Point the VPS at it** — add a forwarding rule so `rsyslog` mirrors everything to the homelab
receiver in addition to the local log, in a new drop-in rather than editing the stock config:
```bash
$ sudo tee /etc/rsyslog.d/90-forward.conf > /dev/null <<'EOF'
*.* @<homelab-tunnel-ip>:514
EOF
$ sudo systemctl restart rsyslog
```
The single `@` uses UDP — fine here since the link is already encrypted end-to-end by WireGuard,
so there's no plaintext-on-the-wire concern the way there would be forwarding over the open
internet. Use `@@` (TCP) instead if occasional dropped log lines during a network hiccup matter
more to you than simplicity.

**Verify delivery**
```bash
$ logger "rsyslog forwarding test $(date)"
$ ssh <homelab-host> "tail -n 5 ./rsyslog-data/*.log"
```

### Alert on Service Failures
Poll for failed `systemd` units and push a notification the moment something changes state,
rather than relying on catching it during a manual check.

**Get a notification channel** — [ntfy.sh](https://ntfy.sh) is free, requires no signup, and has
mobile apps for push notifications; pick a private, hard-to-guess topic name (it's the only
"auth" a public topic has):
```bash
$ topic=$(openssl rand -hex 16)
$ curl -d "test" ntfy.sh/$topic
```
Subscribe to that topic in the ntfy mobile app to confirm the test message arrives. Self-host it
later if you want the alerting channel itself off a third party.

**Create the check script** — only alerts when the failed-unit set actually *changes*, so a
still-broken unit doesn't re-page you every 5 minutes:
```bash
$ sudo tee /usr/local/sbin/check-failed-units.sh > /dev/null <<'EOF'
#!/bin/bash
set -euo pipefail
STATE_FILE=/var/lib/check-failed-units.state
FAILED=$(systemctl --failed --no-legend --plain)
PREV=$(cat "$STATE_FILE" 2>/dev/null || true)
if [ "$FAILED" != "$PREV" ]; then
  if [ -n "$FAILED" ]; then
    curl -sf -H "Title: systemd unit failure on $(hostname)" -H "Priority: high" \
      -d "$FAILED" https://ntfy.sh/<your-private-topic-name>
  else
    curl -sf -H "Title: systemd units recovered on $(hostname)" \
      -d "All previously failed units are healthy again" https://ntfy.sh/<your-private-topic-name>
  fi
fi
echo "$FAILED" > "$STATE_FILE"
EOF
$ sudo chmod +x /usr/local/sbin/check-failed-units.sh
```

**Schedule it** to run every 5 minutes:
```bash
$ sudo tee /etc/systemd/system/check-failed-units.timer > /dev/null <<'EOF'
[Unit]
Description=Check for failed systemd units every 5 minutes

[Timer]
OnBootSec=2min
OnUnitActiveSec=5min

[Install]
WantedBy=timers.target
EOF
$ sudo tee /etc/systemd/system/check-failed-units.service > /dev/null <<'EOF'
[Unit]
Description=Check for failed systemd units

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/check-failed-units.sh
EOF
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now check-failed-units.timer
```

**Test it** by intentionally failing a disposable unit and confirming the push notification
arrives. `systemctl start` on a unit name that doesn't exist errors client-side before systemd ever
loads it — it never reaches a `failed` state, so it won't show up in `systemctl --failed` and won't
trigger the alert. Use `systemd-run` instead to create a real transient unit that actually runs and
fails:
```bash
$ sudo systemd-run --unit=test-failure --no-block /bin/false
$ sudo systemctl start check-failed-units.service
```
Confirm the `ntfy` push arrives, then clean up so the test unit doesn't linger in `--failed` output
indefinitely:
```bash
$ sudo systemctl reset-failed test-failure.service
```

### Security Review Digest
Instead of remembering to periodically SSH in and manually check `auth.log`, `fail2ban`, and
`Crowdsec`, push a daily summary so a pattern of failed logins or bans shows up automatically.

```bash
$ sudo tee /usr/local/sbin/security-digest.sh > /dev/null <<'EOF'
#!/bin/bash
set -euo pipefail
AUTH_FAILS=$(grep -c "Failed password" /var/log/auth.log 2>/dev/null || echo 0)
F2B_STATUS=$(fail2ban-client status sshd 2>/dev/null || echo "fail2ban not running")
CS_DECISIONS=$(cscli decisions list -o raw 2>/dev/null | tail -n +2 | wc -l || echo 0)
curl -sf -H "Title: Daily security digest — $(hostname)" -d "$(cat <<MSG
Failed SSH password attempts (today's log): $AUTH_FAILS
$F2B_STATUS
Active Crowdsec bans: $CS_DECISIONS
MSG
)" https://ntfy.sh/<your-private-topic-name>
EOF
$ sudo chmod +x /usr/local/sbin/security-digest.sh
```

**Schedule it** daily via `cron`:
```bash
$ (sudo crontab -l 2>/dev/null; echo "0 5 * * * /usr/local/sbin/security-digest.sh") | sudo crontab -
```
Auth log rotation means `grep`-ing `/var/log/auth.log` only ever covers roughly the last day or
two by default (see `/etc/logrotate.d/rsyslog`) — that's a feature here, not a bug, since it keeps
each digest scoped to recent activity rather than re-counting old attempts every run.

**When a digest reports failed attempts**, don't stop at the count — confirm your defenses actually
acted on them rather than assuming a nonzero number means something is wrong. See
[Alert Recon](../../../security/alert_recon/README.md) for the full triage sequence (raw log review,
per-mechanism ban status, port/config sanity checks, safe ban testing).

## Auditing with Lynis
`Lynis` scans the system and scores it against the CIS benchmark, surfacing hardening gaps the
above steps missed. Run it last, after applying everything above, then periodically thereafter.

```bash
$ sudo apt install lynis
$ sudo lynis audit system
```

A hardening index of 70+ is a reasonable baseline for an internet facing server; 85+ tracks with
CIS Level 1/2 compliance. Suggestions are logged to `/var/log/lynis-report.dat`.

***References***
* [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks) for the underlying standard Lynis
  scores against.
