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
* [Automated Recon Script](#automated-recon-script)
  * [Passwordless sudo for the recon script](#passwordless-sudo-for-the-recon-script)
* [1. Read the Raw Log](#1-read-the-raw-log)
  * [Checkpointing What You've Already Reviewed](#checkpointing-what-youve-already-reviewed)
  * [Sanity-Check fail2ban Is Actually Watching](#sanity-check-fail2ban-is-actually-watching)
* [2. Check What Your Defenses Already Did](#2-check-what-your-defenses-already-did)
* [3. Confirm the Port Actually Being Hit](#3-confirm-the-port-actually-being-hit)
* [4. Sanity-Check the Jail/Engine Config](#4-sanity-check-the-jailengine-config)
* [5. Look for a Pattern Worth Acting On](#5-look-for-a-pattern-worth-acting-on)
  * [Manually Banning IPs](#manually-banning-ips)
    * [Automatically Promoting Confirmed Exploit Bans](#automatically-promoting-confirmed-exploit-bans)
* [6. If a Ban Should Have Fired but Didn't](#6-if-a-ban-should-have-fired-but-didnt)
* [Testing a Fix Without Locking Yourself Out](#testing-a-fix-without-locking-yourself-out)

## Overview
A nonzero count in a digest isn't itself an emergency — the internet constantly scans every public
IP on port 22 (and whatever you moved SSH to). The actual question worth answering each time is
*"did my defenses handle this correctly,"* not *"is the count zero."* Work through the steps below
top to bottom; most alerts resolve at step 2. Steps 1–3 are read-only and don't change alert to
alert — [Automated Recon Script](#automated-recon-script) below wraps them into one command instead
of re-typing the same pipelines by hand each time; the manual walk-throughs stay in place as the
reference for what it's doing and why, useful when something looks off and needs digging into by
hand.

## Automated Recon Script
Steps 1–3 below — read the raw log, check what your defenses already did, confirm the port actually
being hit — are read-only end to end and identical every time an alert lands. Wrapping them into one
script means an alert resolves in one command instead of hand-typing the same `grep`/`awk` pipelines,
while leaving those sections in place below as the reference for what each piece is doing.

**Create the script** — same checkpointing logic as
[Checkpointing What You've Already Reviewed](#checkpointing-what-youve-already-reviewed), the raw-log
read, the fail2ban-is-actually-watching sanity check from
[Sanity-Check fail2ban Is Actually Watching](#sanity-check-fail2ban-is-actually-watching), what your
defenses already did ([step 2](#2-check-what-your-defenses-already-did)), the SSH port sanity check
([step 3](#3-confirm-the-port-actually-being-hit)), and a closing list of suspect IPs worth a manual
block ([step 5](#5-look-for-a-pattern-worth-acting-on)), in one pass. Unlike the manual walk-through,
the script never advances the checkpoint itself — it only prints the command to do so, so re-running
it mid-investigation always shows the same window instead of silently narrowing it:
```bash
$ sudo tee /usr/local/sbin/alert-recon.sh > /dev/null <<'EOF'
#!/bin/bash
set -uo pipefail
CHECKPOINT_DIR=/var/lib/alert-recon
CHECKPOINT=$CHECKPOINT_DIR/last-reviewed
mkdir -p "$CHECKPOINT_DIR"
NOW=$(date +%Y-%m-%dT%H:%M:%S)
SINCE=$(cat "$CHECKPOINT" 2>/dev/null || echo 1970-01-01T00:00:00)
THRESHOLD=5 # attempts within the reviewed window before an IP is flagged as a block suspect

echo "=== Reviewing since $SINCE ==="

echo
echo "=== [sshd] Found entries in fail2ban.log ==="
grep '\[sshd\] Found' /var/log/fail2ban.log | awk -v since="$SINCE" '($1"T"$2) > since' | tail -n 30

echo
echo "=== Total attempts and top offenders ==="
OFFENDERS=$(grep '\[sshd\] Found' /var/log/fail2ban.log | awk -v since="$SINCE" '($1"T"$2) > since' | \
  awk '{
    ip=""; for(i=1;i<=NF;i++) if($i=="Found") ip=$(i+1)
    d=$1; count[ip]++
    if (!(ip in first) || d<first[ip]) first[ip]=d
    if (!(ip in last) || d>last[ip]) last[ip]=d
  }
  END { for (ip in count) printf "%6d  %-16s  %s to %s\n", count[ip], ip, first[ip], last[ip] }' \
  | sort -rn)
echo "$OFFENDERS" | head

echo
echo "=== fail2ban service sanity check ==="
systemctl is-active fail2ban || echo "WARNING: fail2ban is not active"

echo
echo "=== auth.log direct check (cross-check against fail2ban.log above) ==="
grep -E "sshd\[[0-9]+\]: (Failed (password|publickey) for|[Ii]nvalid user .* from)" /var/log/auth.log \
  | awk -v since="$SINCE" '$1 > since' | tail -n 30

echo
echo "=== What your defenses already did ==="
fail2ban-client status sshd
cscli decisions list
if [ -f /opt/pangolin/docker-compose.yml ]; then
  (cd /opt/pangolin && docker compose exec -T crowdsec cscli decisions list)
fi
ipset list admin-allow

echo
echo "=== Port actually being hit ==="
ss -tlnp | grep ssh

echo
echo "=== Suspect IPs (>= $THRESHOLD attempts, not already trusted or blocked) ==="
FOUND_SUSPECT=0
while read -r count ip _rest; do
  [ -z "${ip:-}" ] && continue
  [ "$count" -lt "$THRESHOLD" ] && continue
  if ipset test admin-allow "$ip" &>/dev/null; then
    echo "  $ip ($count attempts) — SKIPPED, present in admin-allow (your own IP?)"
    continue
  elif ipset test manual-block "$ip" &>/dev/null; then
    echo "  $ip ($count attempts) — already in manual-block, nothing to do"
  else
    echo "  $ip ($count attempts) — sudo /usr/local/sbin/block-ip.sh $ip"
    FOUND_SUSPECT=1
  fi
done <<< "$OFFENDERS"
[ "$FOUND_SUSPECT" -eq 0 ] && echo "  none"

echo
echo "=== Checkpoint NOT advanced — this run is safe to repeat ==="
echo "Once you're done reviewing (including any manual blocks above), advance it with:"
echo "  echo \"$NOW\" | sudo tee $CHECKPOINT"
EOF
$ sudo chmod +x /usr/local/sbin/alert-recon.sh
```

**Run it**
```bash
$ sudo /usr/local/sbin/alert-recon.sh
```
Review the output, run `sudo /usr/local/sbin/block-ip.sh <ip>` for any suspect worth blocking, then
copy the `echo ... | sudo tee ...` command the script prints at the end to advance the checkpoint —
it already has that run's exact timestamp baked in, so nothing gets skipped on the next alert.

### Passwordless sudo for the recon script
The script is read-only end to end (the only write is the checkpoint file itself), so it's safe to
grant `NOPASSWD` for it specifically. ***Don't grant `NOPASSWD` for the individual commands it
wraps*** (`grep`, `cat`, etc.) — those would open a path to reading arbitrary files as root (e.g.
`/etc/shadow`) the moment any of them gets invoked with different arguments elsewhere. Scoping to
one fixed script keeps the exposure to exactly what's written above:
```bash
$ sudo visudo -f /etc/sudoers.d/alert-recon
```
Contents:
```
alice ALL=(root) NOPASSWD: /usr/local/sbin/alert-recon.sh
```
**Verify**
```bash
$ sudo -l
$ sudo /usr/local/sbin/alert-recon.sh
```
`sudo -l` should list only this one script under `NOPASSWD`; everything else run with `sudo` should
still prompt for a password as before.

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

### Automatically Promoting Confirmed Exploit Bans
Manually eyeballing `cscli decisions list` after every alert doesn't scale once scanner traffic becomes
a daily occurrence, and a lot of it does resolve on its own — `captcha` on a soft signal like
`http-bad-user-agent` doesn't need a permanent block the way an actual exploit-attempt `ban` does. This
promotes the strong-signal subset automatically, leaving weaker signals to Crowdsec's own temporary
ban/captcha:

***Promotion policy*** — only a decision with `type == ban` **and** a scenario name matching an
exploit-signature pattern (`cve|rce|exploit|backdoor|vpatch|traversal`) gets permanently blocked.
Softer scenarios (`http-probing`, `http-bad-user-agent`) stay on Crowdsec's own temporary
ban/captcha, even when they fire against the same IP — a single low-signal hit isn't reason enough for
a permanent block, but a matched CVE/RCE scenario is.

***Polling interval matters here*** — Crowdsec ban durations can be as short as an hour or two (see the
[Diagnosing and clearing a ban](../../system/ubuntu/hardening/README.md#diagnosing-and-clearing-a-ban)
example durations), so a once-daily check (matching the [Security Review
Digest](../../system/ubuntu/hardening/README.md#security-review-digest)'s cadence) would miss most
decisions before they naturally expire. This runs on a 15-minute timer instead, the same interval as
[Alert on Service Failures](../../system/ubuntu/hardening/README.md#alert-on-service-failures)'s
`check-failed-units.timer`.

**Install `jq`** — needed to parse `cscli`'s JSON output:
```bash
$ sudo apt install jq
```

**Create the script** — checks both Crowdsec engines (host-level and, if this VPS runs Pangolin, the
dockerized stack engine — see
[CrowdSec: Two Separate Engines by Design](../../networking/reverse_tunnel/pangolin/README.md#crowdsec-two-separate-engines-by-design)),
skips anything already in `admin-allow` as a safety net against a false-positive match on your own IP,
and always prints what it found before acting on it — silent success and a silently-broken pipeline
should never look the same. `set -e` is deliberately left off here (unlike this doc's other scripts):
each `cscli`/`jq` call below has its own `|| echo '[]'` fallback, so a transient failure on one engine
degrades to "no matches from that engine" instead of aborting the whole run — with `-e` in place, an
unreachable `docker compose exec` (e.g. Pangolin stack mid-restart) would silently kill the script
before it ever reached the host-level check or printed anything, which is the exact failure mode that
looks like unexplained silence:
```bash
$ sudo tee /usr/local/sbin/auto-promote-crowdsec-bans.sh > /dev/null <<'EOF'
#!/bin/bash
set -uo pipefail
PATTERN='(cve|rce|exploit|backdoor|vpatch|traversal)'

find_matches() {
  # $1: raw `cscli decisions list -o json` output (or '[]' on failure)
  # `-o json` returns an array of *alerts*, each holding a nested `decisions`
  # array — the `type`/`scenario`/`value` fields being filtered on live inside
  # that nested array, not on the alert object itself.
  echo "$1" | jq -r --arg pat "$PATTERN" \
    '(. // [])[] | (.decisions // [])[]? | select(.type=="ban" and (.scenario | test($pat; "i"))) | "\(.value)\t\(.scenario)"' 2>/dev/null
}

HOST_JSON=$(cscli decisions list -o json 2>/dev/null || echo '[]')
PANGOLIN_JSON=$(cd /opt/pangolin 2>/dev/null && docker compose exec -T crowdsec cscli decisions list -o json 2>/dev/null || echo '[]')

MATCHES=$(
  { find_matches "$HOST_JSON"; find_matches "$PANGOLIN_JSON"; } | sort -u
)

if [ -z "$MATCHES" ]; then
  echo "No decisions currently match the promotion policy (type=ban, scenario~=$PATTERN)."
  exit 0
fi

echo "Decisions matching promotion policy:"
while IFS=$'\t' read -r ip reason; do
  printf '  %-16s %s\n' "$ip" "$reason"
done <<< "$MATCHES"
echo

NEWLY_ADDED=""
while IFS=$'\t' read -r ip reason; do
  if ipset test admin-allow "$ip" &>/dev/null; then
    echo "  SKIP             $ip ($reason) — present in admin-allow"
  elif ipset test manual-block "$ip" &>/dev/null; then
    echo "  ALREADY BLOCKED  $ip ($reason)"
  else
    echo "  ADDING           $ip ($reason)"
    /usr/local/sbin/block-ip.sh "$ip" > /dev/null
    NEWLY_ADDED+="$ip ($reason)"$'\n'
  fi
done <<< "$MATCHES"

if [ -n "$NEWLY_ADDED" ]; then
  curl -sf -H "Title: New permanent IP block — $(hostname)" \
    -d "$NEWLY_ADDED" https://ntfy.sh/<your-private-topic-name>
fi
EOF
$ sudo chmod +x /usr/local/sbin/auto-promote-crowdsec-bans.sh
```
***Both loops feed from `<<< "$MATCHES"` (a here-string) rather than `echo "$MATCHES" | while ...`***
— piping into the loop runs its body in a subshell, so `NEWLY_ADDED` set inside it would vanish the
moment the loop ends, and the `curl` push afterward would never see anything to send. A here-string
runs the loop in the current shell instead, so the variable actually persists past `done`.

**Use the same `ntfy` topic already set up** in
[Alert on Service Failures](../../system/ubuntu/hardening/README.md#alert-on-service-failures) —
replace `<your-private-topic-name>` with that same topic rather than generating a new one, so a new
permanent block lands in the same channel you're already watching instead of a second, easy-to-forget
subscription.

Drop the `PANGOLIN_JSON` line (and its `find_matches` call below) if this VPS doesn't run Pangolin —
there's no dockerized Crowdsec engine to query in that case; `HOST_JSON` alone is sufficient.

**Test it manually before scheduling** — confirms the script actually matches and promotes against
whatever's currently in `cscli decisions list`, rather than trusting it blind the first time a timer
fires:
```bash
$ sudo /usr/local/sbin/auto-promote-crowdsec-bans.sh
$ sudo ipset list manual-block
```

**Schedule it** on a 15-minute timer:
```bash
$ sudo tee /etc/systemd/system/auto-promote-crowdsec-bans.timer > /dev/null <<'EOF'
[Unit]
Description=Promote confirmed-exploit CrowdSec decisions to permanent manual-block

[Timer]
OnBootSec=5min
OnUnitActiveSec=15min

[Install]
WantedBy=timers.target
EOF
$ sudo tee /etc/systemd/system/auto-promote-crowdsec-bans.service > /dev/null <<'EOF'
[Unit]
Description=Promote confirmed-exploit CrowdSec decisions to manual-block

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/auto-promote-crowdsec-bans.sh
EOF
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now auto-promote-crowdsec-bans.timer
```

**Verify it's actually running on schedule**, not just enabled:
```bash
$ sudo systemctl list-timers auto-promote-crowdsec-bans.timer
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
