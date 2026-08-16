# Hardening <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Practical steps for hardening a `Ubuntu` system, primarily targeted at internet facing servers but
applicable to desktops as well.

### Quick links
* [.. up dir](..)
* [User Accounts](#user-accounts)
* [SSH](#ssh)
* [Firewall](#firewall)
* [Automatic Updates](#automatic-updates)
* [Fail2ban](#fail2ban)
* [AppArmor](#apparmor)
* [Auditd](#auditd)
* [Kernel Parameters](#kernel-parameters)

## User Accounts

***Disable root login*** — use `sudo` from a non-root user instead of logging in directly as root.

**Lock the root account**
```bash
$ sudo passwd -l root
```

**Enforce strong password policy** via `libpam-pwquality`
```bash
$ sudo apt install libpam-pwquality
```
Edit `/etc/security/pwquality.conf` to set minimum length, complexity and history requirements.

**Lock accounts after failed login attempts** via `faillock`
```bash
$ sudo apt install libpam-modules
```
Configure thresholds in `/etc/security/faillock.conf`.

## SSH
Harden `/etc/ssh/sshd_config`:
```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
X11Forwarding no
MaxAuthTries 3
AllowUsers alice
```

**Apply changes**
```bash
$ sudo systemctl restart sshd
```

***References***
* [Prefer key based authentication](https://help.ubuntu.com/community/SSH/OpenSSH/Keys) over
  passwords entirely.

## Firewall
`ufw` (Uncomplicated Firewall) is a friendly front end for `iptables`/`nftables`.

**Enable with default deny incoming**
```bash
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow OpenSSH
$ sudo ufw enable
```

**Check status**
```bash
$ sudo ufw status verbose
```

## Automatic Updates
Enable unattended security updates so patches are applied without manual intervention.

```bash
$ sudo apt install unattended-upgrades
$ sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Config lives at `/etc/apt/apt.conf.d/50unattended-upgrades` and
`/etc/apt/apt.conf.d/20auto-upgrades`.

## Fail2ban
`fail2ban` bans IPs that show malicious signs such as repeated failed SSH login attempts.

```bash
$ sudo apt install fail2ban
$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Edit `/etc/fail2ban/jail.local` to tune `bantime`, `findtime` and `maxretry`, then enable the `sshd`
jail.

**Check status**
```bash
$ sudo fail2ban-client status sshd
```

## AppArmor
`AppArmor` is enabled by default on Ubuntu and restricts what individual applications can access.

**List loaded profiles and their mode**
```bash
$ sudo aa-status
```

**Set a profile to enforce mode**
```bash
$ sudo aa-enforce /etc/apparmor.d/PROFILE
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
Harden network related kernel parameters via `/etc/sysctl.d/99-hardening.conf`:
```
net.ipv4.conf.all.rp_filter=1
net.ipv4.conf.all.accept_redirects=0
net.ipv4.conf.all.send_redirects=0
net.ipv4.tcp_syncookies=1
kernel.dmesg_restrict=1
```

**Apply changes**
```bash
$ sudo sysctl --system
```
