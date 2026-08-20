# Android Client <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Connecting to a self-hosted [Pangolin](../README.md) instance from the Pangolin Client (Olm)
Android app, and the DNS/routing gotchas found troubleshooting it against a live instance.

### Quick links
- [.. up dir](..)
- [Prerequisites](#prerequisites)
  - [Firefox's Own DNS-over-HTTPS Setting](#firefoxs-own-dns-over-https-setting)
  - [Android's System-Level Private DNS Setting](#androids-system-level-private-dns-setting)
- [Install and Connect](#install-and-connect)
- [Using with Bitwarden](#using-with-bitwarden)

## Prerequisites
Two DNS settings — one in Firefox, one at the Android OS level — can silently break general
browsing once the Pangolin Client's tunnel comes up. Set both of these *before* installing and
connecting the client, rather than discovering the breakage afterward.

### Firefox's Own DNS-over-HTTPS Setting
Firefox has its own resolver override (`Settings > Privacy & Security > DNS over HTTPS`),
independent of whatever Android or the VPN tunnel is doing. Set it to `Off` or else it will intercept
DNS requests and you won't see the Pangolin Client's DNS changes and the VPN won't work.

### Android's System-Level Private DNS Setting
Android has a separate, OS-wide setting at `Settings > Network & internet > Private DNS` with some
options. Choose the `Off` option to allow the Pangolin VPN to work correctly.

**If you're changing this after the client is already installed and connected**, the running
client does not re-evaluate DNS behavior for an existing connection — the setting change alone
won't take effect. Force-stop (or otherwise fully kill, not just disconnect/reconnect from within
the app) the Pangolin Client, then relaunch and reconnect.

## Install and Connect
1. Install the app and launch it
2. Hit `Self-hosted or dedicated instance` button
3. Add the self-hosted server address e.g. `https://pangolin.example.com`
4. Login with your Email and Password and any 2FA you enabled
5. Trust your device and flip the toggle to connect
6. Allow the app to start a VPN
7. Allow the app to run in the background

## Using with Bitwarden
When you need to get access to Bitwarden

1. Launch the Pangolin app
2. Flip the toggle and wait for it to connect
3. Login to Pangolin if necessary
4. Use Bitwarden as usual
