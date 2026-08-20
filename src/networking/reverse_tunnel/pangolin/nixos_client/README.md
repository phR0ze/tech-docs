# NixOS Client <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Connecting to a self-hosted [Pangolin](../README.md) instance from a NixOS (or any Linux) box using
`Pangolin CLI`, and how to run it without letting it silently take over the machine's routing or DNS.

### Quick links
- [.. up dir](..)
- [Install](#install)
- [Login and Bring Up a Connection](#login-and-bring-up-a-connection)
- [Routing Scope Is Set Server-Side, Not by a Client Flag](#routing-scope-is-set-server-side-not-by-a-client-flag)
- [The DNS Override Is What Actually Jacks Your Network](#the-dns-override-is-what-actually-jacks-your-network)
- [Recovering From a Stuck or Crashed Client](#recovering-from-a-stuck-or-crashed-client)
- [Tear Down Deliberately](#tear-down-deliberately)

## Install
Olm (the older WireGuard-only client is being phased out in favor of `Pangolin CLI` — a combined auth
+ tunnel client available in `nixpkgs` as `pangolin-cli`. Run it ad hoc without installing anything
system-wide:
```bash
$ nix run nixpkgs#pangolin-cli -- --help
```
Or add it permanently via `environment.systemPackages = [ pkgs.pangolin-cli ];` in your NixOS
configuration if you'll use it regularly.

## Login and Bring Up a Connection
Login and bringing up the connection both need `sudo`, and for the same reason: creating the
tunnel interface requires `CAP_NET_ADMIN`/`/dev/net/tun` access, and the CLI stores its login state
per-user (`~/.config/olm-client/config.json`-style path). Logging in as your normal user and then
running `up` under `sudo` looks at *root's* home directory instead, finds no saved session, and
silently behaves as if you were never logged in — so do both under `sudo` from the start:
```bash
$ sudo nix run nixpkgs#pangolin-cli -- login https://pangolin.example.com
$ sudo nix run nixpkgs#pangolin-cli -- up --attach
```
**On a headless machine or VM, ignore the `Running Firefox as root ... is not supported` error** —
`login` is a device-code flow, the same pattern as `gh auth login`/`docker login`. It always prints a
one-time code and a URL (`First copy your one-time code: XXXX-XXXX` /
`https://pangolin.example.com/auth/login/device`) *before* attempting to auto-launch a local browser,
and that auto-launch is only a convenience — it isn't required for login to succeed. When it's run
under `sudo` in a session it doesn't own (a `vm-test`-style VM logged in as one user with `sudo` to
root, or any box with no GUI at all), the browser launch fails exactly like the screenshot above,
but the CLI keeps polling regardless. Open the printed URL in *any* browser on *any* device — your
host machine, your phone, doesn't need to be the VM itself — log in there as your normal
(non-`sudo`) Pangolin identity, and enter the one-time code. The terminal session picks up the
completed login as soon as you authorize it, with no further action needed there.

**Use `--attach` for your first connection on any new machine.** `up` runs detached (background) by
default — if something goes wrong (see the DNS warning below), there's no foreground process to
`Ctrl-C`, and you're left hunting for the right `down`/`reset-dns` incantation instead of just
stopping it. Once you've confirmed a clean connect/disconnect cycle with `--attach`, detached mode
is fine for routine use.

## The DNS Override can Cause confusion
`--override-dns` (on by default) is the setting most likely to break connectivity — not routing. It
replaces your system's DNS resolution wholesale, for every query, not just Pangolin's own resource
names: anything it doesn't recognize as a Pangolin resource gets forwarded to `--upstream-dns`,
which defaults to a bare `1.1.1.1`. On a machine that depends on split-horizon DNS —
`systemd-resolved` per-link resolvers, Tailscale MagicDNS (`100.100.100.100`), an internal recursive
resolver, LAN `.local`/mDNS names — that silently stops working the moment the tunnel comes up,
since none of it matches `1.1.1.1`'s view of the world. Both Pangolin CLI and Tailscale also
independently try to own the global resolver config through the same `systemd-resolved` D-Bus API,
so running both is a real conflict, not just a hypothetical one. Two ways to avoid it:
* **Don't override DNS at all**, if you don't need to reach private resources by hostname —
  explicitly disable it rather than relying on remembering to pass nothing:
  ```bash
  $ sudo nix run nixpkgs#pangolin-cli -- up --attach --override-dns=false
  ```
* **If you do need it**, point `--upstream-dns` at your machine's *actual* resolver instead of the
  `1.1.1.1` default, so unmatched queries still land somewhere that knows about your split-DNS setup
  (e.g. `systemd-resolved`'s stub, or Tailscale's MagicDNS address):
  ```bash
  $ sudo nix run nixpkgs#pangolin-cli -- up --attach --override-dns --upstream-dns 127.0.0.53
  ```

## Recovering From a Stuck or Crashed Client
If the process was killed uncleanly (OOM, `kill -9`, a `nix run` wrapper getting reaped) before it
could tear down its own DNS override, `reset-dns` restores your original resolver config without
needing the client itself to be running:
```bash
$ sudo nix run nixpkgs#pangolin-cli -- reset-dns --force
$ resolvectl status   # confirm your normal DNS servers are back
```

## Tear Down Deliberately
Don't assume a background connection already stopped:
```bash
$ sudo nix run nixpkgs#pangolin-cli -- down
$ sudo nix run nixpkgs#pangolin-cli -- status
```
