# Android Client <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Connecting to a self-hosted [Pangolin](../README.md) instance from the Pangolin Client (Olm)
Android app, and the DNS/routing gotchas found troubleshooting it against a live instance.

### Quick links
- [.. up dir](..)
- [Prerequisites](#prerequisites)
  - [Firefox's Own DNS-over-HTTPS Setting](#firefoxs-own-dns-over-https-setting)
  - [Android's System-Level Private DNS Setting](#androids-system-level-private-dns-setting)
- [Install and configure Pangolin VPN client](#install-and-configure-pangolin-vpn-client)
  - [Install Pangolin VPN Client](#install-pangolin-vpn-client)
  - [Configure Pangolin VPN Client](#configure-pangolin-vpn-client)
- [Install and configure Bitwarden](#install-and-configure-bitwarden)
  - [Install Bitwarden](#install-bitwarden)
  - [Using with Bitwarden app over Pangolin](#using-with-bitwarden-app-over-pangolin)

## Prerequisites
Two DNS settings — one in `Firefox`, one at the `Android OS level` — can silently break general
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

## Install and configure Pangolin VPN client

### Install Pangolin VPN Client
1. Navigate to the Google Play store
2. Search for pangolin
3. Choose `Pangolin` by `Fossorial, Inc.`

### Configure Pangolin VPN Client
1. Launch the app
2. Hit `Self-hosted or dedicated instance` button
3. Add the self-hosted server address e.g. `https://vault-vpn.example.com`
4. Login with your Email and Password and any 2FA you enabled
5. Trust your device and flip the toggle to connect
6. Allow the app to start a VPN
7. Allow the app to run in the background

## Install and configure Bitwarden

### Install Bitwarden
1. Navigate to the Google Play store
2. Search for Bitwarden
3. Choose `Bitwarden Password Manager` by `Bitwarden Inc.`

### Using with Bitwarden app over Pangolin
1. Launch the Pangolin app
2. Flip the toggle and wait for it to connect
3. Login to Pangolin if necessary
4. Now launch the Bitwarden app
5. Enter your email address that you use for Vaultwarden
6. Click the tiny `Loggin in on` option and change it to `Self-hosted`
7. Set the `Server URL` to your domain name that both Pangolin and Caddy are using e.g
   `https://vault-vpn.example.com`
8. Hit `Save` in the top right corner
9. Flip the toggle to `Rember email`
10. Hit `Continue` and enter your `Master password`
11. Finally hit `Log in with master password`
