# Alert Recon <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

A triage checklist for when a security alert lands — e.g. an `ntfy` "Daily security digest" push
reporting failed SSH password attempts — so investigating it is a repeatable sequence instead of an
improvised one each time. Written against the [Ubuntu Hardening](../../system/ubuntu/hardening/README.md)
doc's alerting setup ([Alert on Service Failures](../../system/ubuntu/hardening/README.md#alert-on-service-failures),
[Security Review Digest](../../system/ubuntu/hardening/README.md#security-review-digest)) and a
[Pangolin](../../networking/reverse_tunnel/pangolin/README.md) VPS specifically, but the sequence
applies to any internet-facing box running [Fail2ban](../fail2ban/README.md) and/or
[Crowdsec](../crowdsec/README.md).

### Quick links
* [.. up dir](..)
* [Overview](#overview)
* [1. Read the Raw Log](#1-read-the-raw-log)
  * [Checkpointing What You've Already Reviewed](#checkpointing-what-youve-already-reviewed)
  * [Sanity-Check fail2ban Is Actually Watching](#sanity-check-fail2ban-is-actually-watching)
* [2. Check What Your Defenses Already Did](#2-check-what-your-defenses-already-did)
* [3. Confirm the Port Actually Being Hit](#3-confirm-the-port-actually-being-hit)
* [4. Sanity-Check the Jail/Engine Config](#4-sanity-check-the-jailengine-config)
* [5. Look for a Pattern Worth Acting On](#5-look-for-a-pattern-worth-acting-on)
  * [Manually Banning IPs](#manually-banning-ips)
* [6. If a Ban Should Have Fired but Didn't](#6-if-a-ban-should-have-fired-but-didnt)
* [Testing a Fix Without Locking Yourself Out](#testing-a-fix-without-locking-yourself-out)

## Overview
A nonzero count in a digest isn't itself an emergency — the internet constantly scans every public
IP on port 22 (and whatever you moved SSH to). The actual question worth answering each time is
*"did my defenses handle this correctly,"* not *"is the count zero."* Work through the steps below
top to bottom; most alerts resolve at step 2.

## 1. Read the Raw Log
Start with `fail2ban.log` — it's fail2ban's own record of what it matched, more reliable than a
hand-copied regex trying to track `/etc/fail2ban/filter.d/sshd.conf`.

### Checkpointing What You've Already Reviewed
Keep a checkpoint file of your last review so already-cleared entries aren't re-triaged daily.
Capture the timestamp *before* running anything else, so entries landing mid-review aren't skipped
next time. No checkpoint yet → falls back to epoch, nothing filtered:
```bash
$ NOW=$(date +%Y-%m-%dT%H:%M:%S)
$ SINCE=$(sudo cat /var/lib/alert-recon/last-reviewed 2>/dev/null || echo 1970-01-01T00:00:00)
```
`fail2ban.log` splits its timestamp across two fields (`2026-08-20 17:57:06,781 ...`) — join them
before comparing to `$SINCE`:
```bash
$ sudo grep '\[sshd\] Found' /var/log/fail2ban.log | awk -v since="$SINCE" '($1"T"$2) > since' | tail -n 30
```
Both `fail2ban.log` and `auth.log` are written in local time (see
[System Timezone](../../system/ubuntu/hardening/README.md#system-timezone)) — keep `$NOW`/`$SINCE` in
local time too (plain `date`, not `date -u`) or the string comparison silently mis-sorts.

(One-off full-history read? Drop the `awk since` stage, or set `SINCE=1970-01-01T00:00:00`.)

Look for a single IP hammering repeatedly (should already be banned — see step 2), or many different
IPs at low volume each (botnet/credential-stuffing scan, normal background noise).

**Total attempts and top offenders**:
```bash
$ sudo grep '\[sshd\] Found' /var/log/fail2ban.log | awk -v since="$SINCE" '($1"T"$2) > since' | wc -l
$ sudo grep '\[sshd\] Found' /var/log/fail2ban.log | awk -v since="$SINCE" '($1"T"$2) > since' | \
  awk '{
    ip=""; for(i=1;i<=NF;i++) if($i=="Found") ip=$(i+1)
    d=$1; count[ip]++
    if (!(ip in first) || d<first[ip]) first[ip]=d
    if (!(ip in last) || d>last[ip]) last[ip]=d
  }
  END { for (ip in count) printf "%6d  %-16s  %s to %s\n", count[ip], ip, first[ip], last[ip] }' \
  | sort -rn | head
```
Reads as `432   203.0.113.42   2026-08-16 to 2026-08-18` — count, IP, first/last-seen date.

### Sanity-Check fail2ban Is Actually Watching
`fail2ban.log` is only as good as the service running — it's been silently down before while
`auth.log` kept filling with real failures. Confirm the service, and fall back to `auth.log` directly
if anything looks off:
```bash
$ systemctl is-active fail2ban
$ sudo grep -E "sshd\[[0-9]+\]: (Failed (password|publickey) for|[Ii]nvalid user .* from)" /var/log/auth.log | awk -v since="$SINCE" '$1 > since' | tail -n 30
```
If `auth.log` shows hits `fail2ban.log` doesn't, jump to [step 4](#4-sanity-check-the-jailengine-config).

Once done reviewing (including the rest of this checklist, in case a later step sends you back here),
advance the checkpoint:
```bash
$ sudo mkdir -p /var/lib/alert-recon
$ echo "$NOW" | sudo tee /var/lib/alert-recon/last-reviewed
```

## 2. Check What Your Defenses Already Did
Each mechanism keeps an independent blocklist — check all of them rather than assuming which one
fired, or whether any did:
```bash
$ sudo fail2ban-client status sshd
$ sudo cscli decisions list                                              # host-level Crowdsec, if installed
$ cd /opt/pangolin && sudo docker compose exec crowdsec cscli decisions list    # Pangolin-stack Crowdsec — must run from the compose file's directory, or `exec` fails with "no configuration file provided: not found"
$ sudo ipset list admin-allow                               # confirms your own IP is still trusted
```
If the offending IP already shows up banned/decided against, your defenses worked as intended — the
alert was informational, not an incident. If it doesn't show up despite a clear pattern from step 1,
continue to step 4.

## 3. Confirm the Port Actually Being Hit
If SSH was moved off `22` per the hardening doc, first confirm what `sshd` is actually listening on:
```bash
$ sudo ss -tlnp | grep ssh
```
***Don't parse `port <N>` out of the `auth.log` line itself*** — that number is the *client's*
ephemeral source port (effectively random per connection), not the port on your VPS that was hit.
Piping it through `sort | uniq -c` produces a huge, unhelpful list of near-single-digit counts that
looks like a breakdown of destination ports but isn't one — `sshd`'s log line has no destination-port
field at all. Since a single `sshd` only ever listens on one port, every `Failed password` line in
`auth.log` is inherently against whatever `ss -tlnp` showed above — there's nothing further to check
per-line.

If you *do* still see hits against the old port `22` in a firewall/connection-tracking log (not
`auth.log` — nothing is listening there to log a failed password in the first place), that traffic
never reached `sshd` at all — harmless background noise, nothing to act on.

## 4. Sanity-Check the Jail/Engine Config
The most common reason a clear brute-force pattern in the log *didn't* get banned isn't a clever
attacker — it's a config mistake that looks fine but silently does nothing. Known traps:

* **`enable = true` instead of `enabled = true`** in a fail2ban jail section — not a real directive,
  silently ignored, jail falls back to disabled. `fail2ban-client -t` validates syntax, not whether
  a key name is real, so this passes validation and still does nothing. See
  [Fail2ban's gotchas](../fail2ban/README.md#common-gotchas) for the full writeup.
* **`port` not overridden in the jail** — fail2ban resolves a bare `port = ssh` via `/etc/services`
  (`22`), not your `sshd_config`'s `Port` directive. A custom SSH port needs the jail's `port` line
  set explicitly.
* **The service started after the burst happened** — fail2ban/Crowdsec only act on log lines seen
  while running, within their configured `findtime` window. An old burst from before the service was
  even installed, or from before its last restart, won't retroactively trigger a ban:
  ```bash
  $ systemctl status fail2ban | grep -i active
  $ journalctl -u fail2ban --since "<attack start time>" --until "<attack end time>"
  ```
  Also check fail2ban's own log directly — it logs to `/var/log/fail2ban.log` by default, not the
  journal, so an empty `journalctl` window doesn't necessarily mean fail2ban was down:
  ```bash
  $ sudo tail -100 /var/log/fail2ban.log
  ```
* **Threshold not actually crossed** — `maxretry` counts within a rolling `findtime` window, not a
  running total. Attempts spread out past `findtime` between each one never accumulate to a ban.
  Check the jail's actual effective values (a jail with no explicit override inherits from
  `[DEFAULT]`):
  ```bash
  $ grep -E '^(maxretry|findtime|bantime)' /etc/fail2ban/jail.local
  ```
  Note this grep matches every section in the file — confirm which block a given value actually sits
  under before assuming it applies to the jail you're diagnosing.

Once a fix lands, always validate before restarting and confirm the jail is genuinely active
afterward, not just error-free:
```bash
$ sudo fail2ban-client -t
$ sudo systemctl restart fail2ban
$ sudo fail2ban-client status sshd
```
A jail that's truly watching shows `Currently failed`/`Total failed` counters climbing as `auth.log`
grows — `0`/`0` indefinitely despite known failed attempts means something above is still wrong.

## 5. Look for a Pattern Worth Acting On
* **Repeated attempts from one IP/subnet, still unbanned after step 4's fixes** — manually ban:
  ```bash
  $ sudo fail2ban-client set sshd banip <ip>
  $ sudo cscli decisions add --ip <ip> --duration 24h --reason manual
  ```
* **Attempts against an `AllowUsers`-restricted username specifically** (not generic scanner names)
  — verify password auth is actually disabled, since if it is, password attempts can't succeed
  regardless of volume:
  ```bash
  $ sudo sshd -T | grep -iE 'passwordauth|pubkeyauth'
  ```
* **A new, sustained daily baseline** — normal for any SSH port open to the internet. The point of
  a digest is to notice a *change* from that baseline, not to treat nonzero as alarming on its own.

### Manually Banning IPs
The `fail2ban-client set sshd banip`/`cscli decisions add` above are fine for a temporary manual ban,
but both are tied to that mechanism's own lifecycle — a fail2ban ban clears on service restart
unless persistent DB storage is configured, and a Crowdsec decision always carries an expiry. For a
permanent block on a confirmed-malicious IP, use the
[Manual IP Blocklist](../../system/ubuntu/hardening/README.md#manual-ip-blocklist) `ipset` set up in
the hardening doc — it's independent of fail2ban/Crowdsec entirely and keeps `ufw status numbered`
down to one rule no matter how many IPs accumulate. Set it up once there; the recurring step, every
time a new IP needs blocking, is just:
```bash
$ sudo /usr/local/sbin/block-ip.sh <ip>
```
Verify it took:
```bash
$ sudo ipset list manual-block
```

## 6. If a Ban Should Have Fired but Didn't
Once the config is fixed (step 4), confirm it actually works rather than trusting the fix in
isolation — this is where the tooling gets validated end to end. See
[Testing a Fix Without Locking Yourself Out](#testing-a-fix-without-locking-yourself-out) below
before running any live test.

## Testing a Fix Without Locking Yourself Out
***Never trigger a test ban from the same session/IP you're currently connected with.*** A ban is a
firewall `DROP`/`REJECT` rule against the source IP — it blocks *all* traffic from that IP,
established sessions included, not just new connection attempts. Testing from your own live session
will disconnect you mid-test.

**Test from a genuinely different network** — a phone hotspot, a different location, anything not
sharing your current session's IP:
```bash
$ ssh -p <port> someuser@<vps-ip>   # from the OTHER network; wrong password enough times to cross maxretry
```
Then back on the VPS, confirm the ban landed:
```bash
$ sudo fail2ban-client status sshd
```
Clean up so the test IP doesn't stay banned longer than needed:
```bash
$ sudo fail2ban-client set sshd unbanip <test-ip>
```

**If you get locked out anyway** (e.g. the test IP turned out to match your live session after all):
* If `bantime` is short (e.g. the fail2ban default `10m`), the simplest fix is often to just wait it
  out.
* Otherwise, use your VPS provider's out-of-band web console (RackNerd's VNC/KVM console, etc.) —
  it's outside the network path entirely, so a firewall ban against your IP doesn't affect it. From
  there:
  ```bash
  $ sudo fail2ban-client set sshd unbanip <your-ip>
  ```
* A second IP/network you have access to (if any) also works for the same unban command.

**Prevent this going forward** — add your own management IP(s) to fail2ban's `ignoreip` so this
class of self-lockout can't happen again, even by accident during future testing:
```
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1 <your-ip>
```
