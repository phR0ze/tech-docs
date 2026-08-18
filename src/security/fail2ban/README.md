# Fail2ban <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Fail2ban watches log files for signs of malicious activity — repeated failed logins, brute-force
scans — and bans the offending IP via a firewall rule for a configurable duration. This page is a
general reference; see [Ubuntu Hardening](../../system/ubuntu/hardening/README.md#fail2ban) for the
walkthrough as applied to a specific internet-facing VPS, and [Alert Recon](../alert_recon/README.md)
for the triage sequence to run when an alert reports failed attempts.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
* [Deployment](#deployment)
  * [Install](#install)
  * [Configuration Layers](#configuration-layers)
* [Configuration](#configuration)
  * [Enabling a Jail](#enabling-a-jail)
  * [Custom Ports](#custom-ports)
  * [Tuning Thresholds](#tuning-thresholds)
  * [ignoreip](#ignoreip)
* [Common Gotchas](#common-gotchas)
  * [enable vs enabled](#enable-vs-enabled)
  * [Duplicate Sections](#duplicate-sections)
  * [Timing Windows Aren't Retroactive](#timing-windows-arent-retroactive)
* [Recon](#recon)
  * [Check Jail Status](#check-jail-status)
  * [Check What's Actually Banned](#check-whats-actually-banned)
  * [Show Ban Times](#show-ban-times)
  * [Validate Config Without Restarting](#validate-config-without-restarting)
  * [Watch It Work Live](#watch-it-work-live)
* [Managing Bans](#managing-bans)
  * [Ban an IP Manually](#ban-an-ip-manually)
  * [Clear a Ban](#clear-a-ban)
* [Testing Safely](#testing-safely)

## Overview
Fail2ban works by pairing a **filter** (a regex that matches "this log line means a failed attempt")
with a **jail** (which log file to watch, which filter to apply, and what action to take once a
threshold is crossed — almost always an `iptables`/`nftables`/`ufw` ban). It ships with filters for
`sshd` and dozens of other common services out of the box.

**References**
* [Fail2ban GitHub](https://github.com/fail2ban/fail2ban)
* [Fail2ban Wiki](https://github.com/fail2ban/fail2ban/wiki)

## Deployment

### Install
```bash
$ sudo apt install fail2ban
```
`jail.conf` (the package-shipped defaults) should never be edited directly — it gets overwritten on
package upgrades. Copy it to `jail.local`, which fail2ban merges on top of `jail.conf`, and edit
that instead:
```bash
$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

### Configuration Layers
Fail2ban merges config in this order, later files overriding earlier ones for the same key:
1. `/etc/fail2ban/jail.conf` — package defaults, never edit
2. `/etc/fail2ban/jail.d/*.conf` — drop-ins, loaded in filename order
3. `/etc/fail2ban/jail.local` — the main override file, loaded last

A value set in `jail.local` wins over the same key in `jail.conf` or any `jail.d/*.conf` drop-in
that sorts earlier alphabetically. If a setting doesn't seem to take effect, check for a competing
declaration in `jail.d/`:
```bash
$ grep -rn '<key>' /etc/fail2ban/jail.d/ /etc/fail2ban/jail.local
```

## Configuration

### Enabling a Jail
`jail.local` (copied from `jail.conf`) already has an `[sshd]` section — find and edit that
*existing* section rather than appending a new one:
```bash
$ grep -n '^\[sshd\]' /etc/fail2ban/jail.local
```
Set it enabled:
```
[sshd]
enabled = true
```
Every jail defaults to `enabled = false` unless explicitly turned on, even the common ones like
`sshd`.

### Custom Ports
Fail2ban doesn't read `Port` from `sshd_config` — a bare `port = ssh` resolves via `/etc/services`,
which still says `22` regardless of what your SSH daemon actually listens on. If SSH was moved to a
custom port, set it explicitly in the jail:
```
[sshd]
enabled = true
port = 2222
```
Without this, the jail's ban action targets the wrong port and blocks nothing useful even while
correctly detecting failed attempts.

### Tuning Thresholds
Set in `[DEFAULT]` and inherited by every jail unless a jail overrides them individually:
```
[DEFAULT]
bantime  = 10m
findtime = 10m
maxretry = 5
```
* **`findtime`** — the rolling window failures are counted within.
* **`maxretry`** — how many failures within `findtime` trigger a ban.
* **`bantime`** — how long the ban lasts. The `10m` default is short for an internet-facing SSH
  port — a scanner just waits it out and resumes. A jail-specific override is usually preferable to
  raising the global default:
  ```
  [sshd]
  enabled = true
  port = 2222
  bantime = 1h
  ```
  Or escalate ban duration for repeat offenders instead of a flat value:
  ```
  [DEFAULT]
  bantime.increment = true
  ```

### ignoreip
Add your own management IP(s) so fail2ban never bans your own access, even by accident during
testing:
```
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1 <your-ip>
```

## Common Gotchas

### enable vs enabled
The real directive is **`enabled`** (with a `d`) — `enable` is not recognized and is silently
dropped rather than flagged. `fail2ban-client -t` only validates syntax/schema, not whether a key
*name* is one fail2ban actually understands, so this typo passes validation cleanly while the jail
quietly stays on its `jail.conf` default (`enabled = false`). Symptom: `fail2ban-client status
<jail>` returns without error, but `Total failed`/`Total banned` stay at `0` indefinitely despite
clear brute-force activity in the watched log. Fix the spelling, then confirm the jail is genuinely
active rather than just error-free:
```bash
$ sudo fail2ban-client -t
$ sudo systemctl restart fail2ban
$ sudo fail2ban-client status sshd
```

### Duplicate Sections
Fail2ban's config parser rejects a second `[sshd]` section outright — if you need to change
something, edit the existing section, don't paste a new one below it. A duplicate section fails
config validation and the service won't start at all:
```bash
$ sudo fail2ban-client -t
```

### Timing Windows Aren't Retroactive
Fail2ban only acts on log lines it processes while running, within `findtime` of "now." A burst of
failed attempts that happened before the service was installed, or before its last restart, will
never trigger a ban after the fact — there's nothing to catch up on. Check whether the service was
even running during a given attack window before assuming a ban should have fired:
```bash
$ systemctl status fail2ban | grep -i active
$ journalctl -u fail2ban --since "<start time>" --until "<end time>"
```

## Recon

### Check Jail Status
```bash
$ sudo fail2ban-client status sshd
```
Shows `Currently failed`/`Total failed` (filter matches) and `Currently banned`/`Total banned`/
`Banned IP list` (action results) for that jail specifically.

### Check What's Actually Banned
```bash
$ sudo fail2ban-client status               # list of all active jails
$ sudo fail2ban-client status sshd           # per-jail detail, see above
$ sudo iptables -L -n | grep -i f2b          # underlying firewall rules, if using the iptables backend
```

### Show Ban Times
`status` lists which IPs are banned but not when they expire. Check the *configured* duration for a
jail first:
```bash
$ sudo fail2ban-client get sshd bantime
```
This only reports the setting new bans will use, not remaining time on an existing one. For
per-IP remaining ban time, query fail2ban's own database directly:
```bash
$ sudo sqlite3 /var/lib/fail2ban/fail2ban.sqlite3 \
  "SELECT ip, datetime(timeofban, 'unixepoch') as banned_at, \
   datetime(timeofban + bantime, 'unixepoch') as unbans_at \
   FROM bans WHERE jail='sshd';"
```
Or watch the ban/unban log lines themselves, which carry timestamps directly:
```bash
$ sudo grep 'sshd' /var/log/fail2ban.log | grep -E 'Ban|Unban'
```

### Validate Config Without Restarting
```bash
$ sudo fail2ban-client -t
```
Only checks syntax and recognized keys — it won't catch a jail that's syntactically valid but
logically wrong (custom port not set, `ignoreip` too broad, etc.). Confirm behavior with the status
command above after any change, not just a clean `-t`.

### Watch It Work Live
Fail2ban logs to its own file by default, not the systemd journal — check this directly rather than
relying on `journalctl -u fail2ban` alone, which can look empty even while fail2ban is active:
```bash
$ sudo tail -f /var/log/fail2ban.log
```

## Managing Bans

### Ban an IP Manually
```bash
$ sudo fail2ban-client set sshd banip <ip>
```

### Clear a Ban
```bash
$ sudo fail2ban-client set sshd unbanip <ip>
```

## Testing Safely
***Never trigger a test ban from the same session/IP you're currently connected with*** — a ban is
a firewall `DROP`/`REJECT` rule against the source IP, which blocks *all* traffic from it, including
already-established sessions, not just new connection attempts.

Test from a genuinely different network (phone hotspot, different location):
```bash
$ ssh -p <port> someuser@<vps-ip>   # from the OTHER network, wrong password enough times to cross maxretry
```
Then confirm and clean up from the VPS:
```bash
$ sudo fail2ban-client status sshd
$ sudo fail2ban-client set sshd unbanip <test-ip>
```
See [Alert Recon](../alert_recon/README.md) for the full recovery sequence if a test locks you out
anyway.
