# Vaultwarden Example <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

A worked case study for locking down [Vaultwarden](https://github.com/dani-garcia/vaultwarden)
behind [Pangolin](../README.md) — the kind of setup that looks simple until the Bitwarden mobile
app enters the picture. Assumes the [Access Control](../README.md#access-control) section of the
main Pangolin doc, especially [Expose a service fronted by Caddy over
Pangolin](../README.md#expose-a-service-fronted-by-caddy-over-pangolin), is already read.

### Quick links
- [.. up dir](..)
- [Overview](#overview)
  - [Background](#background)
  - [Important Nuance: Browser vs. Native App Access](#important-nuance-browser-vs-native-app-access)
  - [Private HTTP resource gap — a genuine backend limitation, not just a UI one](#private-http-resource-gap--a-genuine-backend-limitation-not-just-a-ui-one)
  - [The fix: switch to a `host`-mode resource, no override needed](#the-fix-switch-to-a-host-mode-resource-no-override-needed)
  - [Setup of the Private Resource](#setup-of-the-private-resource)
  - [Setup Steps](#setup-steps)
- [User Experience](#user-experience)
  - [Alice's Phone Experience, Step by Step](#alices-phone-experience-step-by-step)
  - [Desktop browser access, separately](#desktop-browser-access-separately)
  - [Access Flow](#access-flow)

## Overview

**Goal:** expose Vaultwarden through Pangolin so that only two named individuals — e.g. Bob and
Alice — can reach it, with independent auditing and revocation, and with Pangolin's identity layer
(not just Vaultwarden's own login) gating *all* traffic including the Bitwarden mobile app's API
calls — not merely the browser-based web vault. Also reachable on the LAN without going through
Pangolin at all (already solved locally by Caddy, untouched by anything below).

### Background
There are two different tunnel roles at play:
* ***Newt*** — runs on *Bob's* side (the site connector), establishing the outbound WireGuard
  tunnel from his home network/VPS to the Pangolin server. This is what makes Caddy reachable at
  all, without opening inbound ports on his firewall.
* ***Pangolin Client (Olm)*** — runs on the *user's* device (Alice's phone), and is what she
  connects with to reach resources she's been granted. Traffic is scoped only to the specific
  resources her role allows — not full-network VPN access.

* ***Device fingerprinting*** — identifies each device by attributes like OS version and
  hostname, so Bob can distinguish "Alice's iPhone" from any other device that might try to log in
  with her credentials.
* ***Device approval (deny-by-default for new hardware)*** — even after correct password + TOTP,
  a brand-new unrecognized device is blocked until Bob explicitly approves it from the dashboard.
  Turn this on for the `vaultwarden-family` role.
* ***Instant block on lost/compromised device*** — one click on the dashboard immediately cuts
  off a specific device (e.g. if Alice loses her phone), without touching her account's other
  access. Devices are archived, not deleted, preserving an audit trail.
* ***Posture checks*** (optional) — can require things like disk encryption before granting
  access. Likely more than needed for a personal use case, but worth knowing about.

### Important Nuance: Browser vs. Native App Access
Pangolin's SSO login wall (username/password + TOTP) is a ***browser-based redirect flow***. The
Bitwarden mobile/desktop app talks directly to the server's API — it doesn't open a browser,
follow redirects, or hold a session cookie — so the *SSO* gate specifically can't apply to it the
way it would to visiting a resource in Chrome/Safari. That doesn't mean the app's traffic can't be
identity-gated at all, though: Pangolin's Private (ZTNA) resources authenticate at the *network*
layer instead — the Pangolin Client (Olm) app on the phone does the username/password + TOTP login
to stand up a WireGuard tunnel, and nothing (not even a TCP SYN) reaches the destination until that
tunnel exists. Bitwarden itself never has to know Pangolin is involved. This is the only path that
satisfies putting the API behind Pangolin's own identity layer, not just Vaultwarden's — a Public
resource with path-exempting Resource Rules (the community's simpler pattern for cases that don't
need this) explicitly does not, since exempted paths bypass Pangolin auth entirely.

### Private HTTP resource gap — a genuine backend limitation, not just a UI one
The straightforward approach is a Private *HTTP* resource pointed at `host.containers.internal:443`,
mirroring the Caddy-fronted pattern used for the [public
resource](../README.md#expose-a-service-fronted-by-caddy-over-pangolin). It doesn't work: without a
way to override the backend TLS SNI or `Host` header, [Newt's known issue of not forwarding the
real SNI on its backend TLS connection](fosrl/pangolin#207) means a Private HTTP resource pointed
at a multi-tenant Caddy instance can't reach the right vhost — Caddy gets a TLS handshake with no
usable SNI and an HTTP request with the wrong `Host` header, and 404s or serves the wrong site.

***Checked directly against the upstream source (`fosrl/pangolin`, `main` branch, both the
open-source and Enterprise Edition code paths) — this turns out to be a real backend gap, not
merely a missing dashboard field:***
* [`server/db/pg/schema/schema.ts:183-184`](https://github.com/fosrl/pangolin/blob/main/server/db/pg/schema/schema.ts) —
  `tlsServerName`/`setHostHeader` are columns on the `resources` table, which is **Public resources
  only**. `PUT /v1/resource/{resourceId}` and its Zod schema in
  [`updateResource.ts`](https://github.com/fosrl/pangolin/blob/main/server/routers/resource/updateResource.ts)
  operate exclusively on that table.
* Private resources (every mode — Host, CIDR, HTTP, SSH) live in an entirely separate table,
  [`siteResources`](https://github.com/fosrl/pangolin/blob/main/server/db/pg/schema/schema.ts) (see
  the `siteResources = pgTable(...)` definition), managed through a completely different endpoint —
  `POST /v1/private-resource/{siteResourceId}` — with its own Zod schema in
  [`updateSiteResource.ts`](https://github.com/fosrl/pangolin/blob/main/server/routers/siteResource/updateSiteResource.ts).
  That schema has `mode`, `destination`, `destinationPort`, `scheme`, `alias`, `roleIds`, etc. —
  **no `tlsServerName` or `setHostHeader` field exists anywhere in it.**
* [`server/private/lib/traefik/getTraefikConfig.ts`](https://github.com/fosrl/pangolin/blob/main/server/private/lib/traefik/getTraefikConfig.ts) —
  the Enterprise Edition's own Traefik config generator confirms the same split: `tlsServerName`/
  `setHostHeader` are pulled from `resources.*` only. Its `siteResources` query (same file) is used
  exclusively to pre-provision TLS certs/maintenance-page placeholders for a private HTTP resource's
  domain — it never feeds SNI/Host-header data into the actual proxy config.
* [`server/lib/blueprints/privateResources.ts`](https://github.com/fosrl/pangolin/blob/main/server/lib/blueprints/privateResources.ts)
  has no `host-header`/`tls-server-name` mapping either, consistent with the field not existing on
  this resource type at all — this isn't a Blueprints-specific omission the way it first looked.

### The fix: switch to a `host`-mode resource, no override needed
A `host`-mode private resource is a raw Layer-4 tunnel — Pangolin/Newt never touch TLS or HTTP at
all, they just relay encrypted bytes end-to-end between the Pangolin Client and the destination.
The original client's TLS `ClientHello` — real SNI included — reaches Caddy completely intact,
exactly as if the client were on the LAN talking to Caddy directly. There's nothing to override
because nothing in Pangolin is parsing the TLS session in the first place.

This is also why a dedicated Caddy listener for the private path (rather than reusing the public
resource's shared, SNI-multiplexed `443` vhost) earns its keep here: point `destination`/
`destinationPort` straight at that listener's real LAN `IP:port`, and it needs zero Caddy-side
changes — the SNI reaching it is genuinely correct, not routed around a gap.

1. Navigate in the Pangolin UI to `NETWORK >Resources >Private`
2. Set the name e.g. `vault-vpn`
3. Choose the `Type` as `Host`
4. Set the Alias to the name you'd like e.g. `vault-vpn.example.com`
5. Set the site e.g. `testlab`
6. Set the destination to your internal LAN caddy service address e.g. `192.168.x.x`
7. Set the `TCP` selection to `Custom` and the actual port value to your caddy service e.g. `8433`
8. Set `UDP` to `Blocked`
9. Disable ICMP and click `Create Resource`
10. After creation switch to the `Authentication` tab and add custom roles for access

`192.168.x.x` is Caddy's real LAN address, and `8443` the dedicated listener already serving the
private vhost correctly for LAN clients — reuse whatever you already have working locally rather
than inventing a new one.

### Setup of the Private Resource
1. ***Two named user accounts — never a shared login***
   - One account per person, each with their own credentials.
   - Enables per-person audit logs and instant, independent revocation (e.g. if Alice's phone is
     lost, Bob kills *her* access only).

2. ***Enforce TOTP 2FA on both accounts***
   - Pangolin has native TOTP with backup codes.
   - This is the single highest-value control here: a leaked or guessed Pangolin password alone
     is not enough to get in.

3. ***Create a dedicated role*** — e.g. `vaultwarden-family`
   - Add only the intended users to it.
   - Pangolin supports multiple roles per user, so if Bob exposes other self-hosted services
     later, he can scope each one to its own role instead of giving blanket access.

4. ***Expose Vaultwarden as a `Host`-type Private resource, targeting Caddy's dedicated private
   listener directly***
   - `Type`: `Host`.
   - `Address`: Caddy's real LAN address (e.g. `192.168.x.x`), `Port`: the dedicated listener
     already serving the private vhost correctly on the LAN (e.g. `8443`) — **not**
     `host.containers.internal`, which is specific to the `HTTP`-type Traefik-proxied path this
     doc doesn't use here. See [The fix](#the-fix-switch-to-a-host-mode-resource-no-override-needed)
     above for why this avoids the SNI/Host-header gap entirely instead of working around it.
   - Attach it to the `vaultwarden-family` role only — no "any authenticated user," no
     public/anonymous fallback.

5. ***Add Resource Rules as a network-level filter***
   - Geo-block to the country/countries the user will realistically connect from — this cuts
     most scanner and credential-stuffing traffic before it ever reaches the login page.
   - Skip IP allowlisting when the user roams (dorm wifi, cellular, coffee shops) — it's
     impractical to maintain.

6. ***Avoid shared secrets as the primary gate***
   - Pangolin also offers resource-specific pins/passwords and email-OTP whitelisting. These are
     handy for one-off sharing but weaker than per-user identity + TOTP for something as
     sensitive as a password vault — a shared pin can't be individually revoked or audited.
   - Skip these for a resource like this.

7. ***Separate admin and resource access boundaries***
   - Non-owner accounts should be plain members of the resource's role only — never
     admin/owner on the Pangolin org itself.

### Setup Steps
1. Caddy's dedicated private listener (e.g. `192.168.x.x:8443`) is already reachable via Newt as
   part of Bob's Pangolin site, and already terminates TLS correctly for LAN clients — nothing
   changes on the Caddy side.
2. Create a Private Resource, `Type: Host`, `Address: 192.168.x.x`, `Port: 8443`, protocol TCP
   only — not Vaultwarden's own `127.0.0.1:8222`, which isn't reachable off the Caddy host at all.
3. Assign the resource to the `vaultwarden-family` role only.
4. [Enable device approval](../README.md#require-device-approval-on-private-resources) for that
   role.
5. Alice installs the Pangolin Client app (Play Store / App Store), logs in with password + TOTP,
   and Bob approves her device the first time from the dashboard. From then on, Bitwarden — pointed
   directly at the LAN `IP:port` — talks straight through the tunnel to Caddy, TLS intact.

## User Experience

### Alice's Phone Experience, Step by Step
1. Alice opens the `Pangolin Client` app (separate from Bitwarden).
2. It prompts her to sign in to Bob's Pangolin org — she enters her username and password.
3. She's prompted for her TOTP code — she opens her authenticator app, reads the 6-digit code,
   and enters it.
4. The Pangolin Client establishes the tunnel in the background, scoped only to the resources
   her role (`vaultwarden-family`) can reach.
5. She opens the `Bitwarden app`, configured with the LAN `IP:port` directly (e.g.
   `https://192.168.x.x:8443`) — a `Host`-mode resource has no Pangolin-issued domain of its own,
   it's a plain tunnel to that address. Since the tunnel is up, the connection reaches Caddy with
   the original TLS session — SNI included — completely intact.
6. She logs into Bitwarden as normal: master password, then Vaultwarden's own 2FA (if enabled).

In practice this isn't a "re-auth every time" ritual — the Pangolin Client can stay connected or
reconnect automatically, with re-auth frequency controlled by the session length Bob configures.

**Important:** don't store the Pangolin TOTP secret inside the Bitwarden vault it's protecting —
that creates a circular dependency (locked out of the vault means locked out of the code needed
to unlock the vault). Use a separate authenticator app on Alice's phone, and keep printed/offline
backup codes somewhere safe.

### Desktop browser access, separately
The browser doesn't need the Pangolin Client at all — it already goes through the [Public `vault.example.com`
resource](../README.md#expose-a-service-fronted-by-caddy-over-pangolin) and Pangolin's normal
SSO+cookie flow. Only the native mobile app, which can't do that redirect, needs the tunnel — so
the VPN client is required exactly where SSO structurally can't reach, and nowhere else.

### Access Flow

Two independent authentication layers, both scoped to named individuals, both auditable, and
both revocable independently — for *every* path the app or browser takes, not just the browser
one:

1. **Outer layer (network):** Pangolin Client login — username/password + TOTP — establishes the
   tunnel before any Vaultwarden traffic (app or otherwise) can reach the destination at all.
2. **Inner layer (application):** Vaultwarden login — master password + Vaultwarden 2FA.
