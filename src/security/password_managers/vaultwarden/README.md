# Vaultwarden <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Unofficial, lightweight Bitwarden-compatible server written in Rust. Drop-in replacement for the
official Bitwarden server, compatible with all official Bitwarden clients (browser extension,
desktop, mobile, CLI) — self-hosting the `Bitwarden` option from the parent page.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
* [Configure Server](#configure-server)
  * [First Run](#first-run)
* [Configure Android](#configure-android)
  * [Configure Bitwarden app](#configure-bitwarden-app)
* [Configure Firefox](#configure-firefox)
  * [Trust Vaultwarden CA on Firefox](#trust-vaultwarden-ca-on-firefox)
  * [Configure Bitwarden extension](#configure-bitwarden-extension)
* [Admin Panel](#admin-panel)
  * [What It Provides](#what-it-provides)
  * [Exposing Over Pangolin](#exposing-over-pangolin)

## Overview

**Pros**
* Free, fully self-hosted
* Compatible with every official Bitwarden client
* Lightweight — Rust binary, SQLite by default, runs comfortably on a small VM/container
* No official Bitwarden server license/subscription required for self-host

**Cons**
* Unofficial reimplementation — trails official server feature parity slightly
* You own patching/updates and the security posture of the instance

## Configure Server

### First Run
1. Browse to `http://<host>:<port>` (default port `8222` when deployed via this repo's
   `services.raw.vaultwarden` NixOS option) or the configured `domain`
2. Click `Create Account` and register. The *first* account created is a normal user — there's no
   separate admin login; that's the [Admin Panel](#admin-panel) token, not an account
3. Log in with the account just created

### Adding More Users
Vaultwarden auto-permits signups only until the first account exists, then locks registration unless
`SIGNUPS_ALLOWED` (`signupsAllowed` in this repo's option) is explicitly on. To add a household/team
member:
* Temporarily flip `signupsAllowed = true`, have them register, then flip it back off, or
* Invite/manage them via the [Admin Panel](#admin-panel) if it's enabled

## Configure Android

### Configure Bitwarden app
1. Search for `Bitwarden Password Manager` and install
2. Launch the app and switch to `Self-hosted`
3. Enter the server `https://<ip-address>:9222`
4. At the bottom hit `Import certificate`

## Configure Firefox

### Trust Vaultwarden CA
1. Shell into your NixOS server running caddy
2. Grab the generated CA and copy it to your target machine
   ```bash
   sudo cp /var/lib/caddy/.local/share/caddy/pki/authorities/local/root.crt ~/Downloads/
   ```
3. Launch Firefox and navigate to `Settings >Privacy and security`
4. At the bottom under `Connection and software security` click `Advanced settings`
5. Scroll to the bottom to the `Certficates` section and click `Manage certificates`
6. Click `Import` and then check the boxes and click `OK` then `OK`

### Configure Bitwarden extension
1. Navigate to Firefox's Extension and Themes
2. Search for `Bitwarden Password Manager` and click `Add to Firefox`
3. Click through loading the extension then click `Log in`
4. At the bottom of the screen switch from `bitwarden.com` to `self-hosted` 
5. Enter your Vaultwarden URL e.g. `https://<homelab-ip>:9222` and click `Save`
6. Then enter your credentials

## Admin Panel

### What It Provides

The `/admin` diagnostics page is a separate, token-gated surface — not tied to any user login — for
server-level control:

* User/org management: list all registered users/orgs, deactivate or delete accounts, reset a
  locked-out user's 2FA, force password-reset requirements, view storage usage
* Server diagnostics: version, uptime, DB connection status, SMTP test-send button, config dump
  (secrets redacted)
* Live config editing: toggle a subset of settings (signups allowed, invitations, password-hint
  display, etc.) without restarting — env-var-set values still take precedence and show as locked
* Manual backup trigger (if a backup directory is configured)

It's the "instance owner" surface — useful once more than one person uses the server (family, small
group) so a user can be deactivated or SMTP debugged without shell access. For a single-user LAN
vault it's mostly unnecessary, and enabling it is a meaningful attack-surface increase: anyone
holding the `ADMIN_TOKEN` has full control over every account on the instance, not just their own.
Treat the token like a root credential.

### Exposing Over Pangolin

If the instance is reachable over [Pangolin](../../../networking/reverse_tunnel/pangolin/README.md)
(or any public tunnel/reverse proxy), avoid exposing `/admin` on the public route:

1. **Prefer not exposing `/admin` publicly at all.** Gate `/admin*` behind Pangolin's own
   authenticated access (visitor auth / SSO on the resource) if per-path rules are supported for the
   resource, or restrict it to a private/WireGuard-only site rather than the public one. *(Needs
   verification against Pangolin's current resource-rule capabilities — at last check, Resources map
   to a subdomain/target rather than clearly documented path-level rules; confirm before relying on
   path-splitting alone.)*
2. If path-splitting isn't practical:
   * Store `ADMIN_TOKEN` as an Argon2 PHC hash (Vaultwarden supports this), not plaintext, so a
     leaked config/env file doesn't leak the usable token directly.
   * Use a long random token (32+ bytes), not something memorable.
3. Layer IP allowlisting at the Pangolin/Traefik layer in front of `/admin` if supported, in
   addition to the token.
4. Consider leaving the admin panel disabled on the internet-facing instance entirely and only
   enabling it on-demand over Tailscale/local access when admin work is actually needed — it isn't
   required for normal day-to-day vault usage.

The built-in rate limiting on the admin login form and the option to hash `ADMIN_TOKEN` help, but
`/admin` is still a single bearer-token gate with no per-user audit trail or 2FA of its own — the
proxy layer is where the stronger access control belongs.
