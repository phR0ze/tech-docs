# Extension Client <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Connecting to a self-hosted [Pangolin](../README.md) instance from the Bitwarden Firefox extension —
a browser-based flow, unlike the [Android](../android_client/README.md) and
[NixOS](../nixos_client/README.md) clients, which both install the Pangolin VPN client (Olm/CLI) to
reach a Private resource instead.

### Quick links
- [.. up dir](..)
- [Overview](#overview)
  - [Browser Access](#browser-access)
- [Install and configure Bitwarden extension](#install-and-configure-bitwarden-extension)
  - [Install the Extension](#install-the-extension)
  - [Configure the Extension](#configure-the-extension)

## Overview

### Browser Access
The Bitwarden extension is baked into the Browser and thus can leverage typical browser flows thereby
working correctly with Pangolin's public resources. The browser hits the ***Public HTTP resource*** (e.g.
`vault.example.com`) and is prompted for authentication with the Pangolin SSO layer before it's
granted access to the resource — the same redirect flow as opening the resource in a tab, since the
extension runs inside the browser's own process and shares its cookies/session for the domain.

## Install and configure Bitwarden extension

### Install the Extension
1. Open Firefox and navigate to `addons.mozilla.org`
2. Search for `Bitwarden Password Manager`
3. Click `Add to Firefox` and confirm the permissions prompt

### Configure the Extension
1. Click the Bitwarden icon in the Firefox toolbar
2. On the login screen, click the `Logging in at bitwarden.com` link near the bottom
3. Choose `Self-hosted` and set `Server URL` to your public resource domain e.g.
   `https://vault.example.com`
4. Click `Save`
5. Enter the email address you use for Vaultwarden
6. Click `Continue`
7. When redirected, complete Pangolin's browser-based SSO login (username/password + TOTP) — this is
   a one-time gate per browser session; the extension itself never talks to Pangolin, only to
   Vaultwarden once the session is established
8. Enter your Vaultwarden `Master password`
9. Click `Log in with master password`
