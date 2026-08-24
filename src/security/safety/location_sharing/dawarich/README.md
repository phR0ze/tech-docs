# Dawarich <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

[Dawarich](https://dawarich.app/) is an open-source, self-hosted, Google-Timeline-style location
history app — the self-hosted alternative to commercial family-tracking apps like Life360 when
full data ownership matters more than a polished family-safety feature set.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
* [Getting started](#getting-started)
  * [Docker Compose](#docker-compose)
  * [Configuring the Dawarich mobile app](#configuring-the-dawarich-mobile-app)
* [Alternative clients](#alternative-clients)
  * [OwnTracks](#owntracks)
* [Alternative backends](#alternative-backends)
  * [Traccar](#traccar)
  * [OwnTracks Recorder](#owntracks-recorder)
* [Comparing the three mobile apps](#comparing-the-three-mobile-apps)
* [Comparison with commercial apps](#comparison-with-commercial-apps)

## Overview
Dawarich is the actual product in this space: a server plus a web UI plus its own official mobile
app (Android and iOS). Point the app at your self-hosted instance and it renders your location
history on an interactive map, with statistics like distance traveled and places visited — the
self-hosted equivalent of Google Timeline, built specifically to replace vendor apps like Life360
when the priority is owning the data outright.

Dawarich doesn't only accept its own app's pings — it also ingests several other tracking clients
(OwnTracks, Traccar Client, GPSLogger, Overland, PhoneTrack), which is useful if you already have
one of those set up, or want a client with capabilities Dawarich's own app doesn't have (see
[Alternative clients](#alternative-clients) below). Entirely separate self-hosted systems also
exist as alternatives to running Dawarich at all (see
[Alternative backends](#alternative-backends)).

## Getting started

### Docker Compose
Dawarich ships an official Docker Compose file (Rails + PostGIS + Redis + Sidekiq). Copy it from
the [self-hosting docs](https://dawarich.app/docs/self-hosting/installation/docker-compose/),
start it with `docker compose up -d`, then visit `http://<your-server-ip>:3000` to finish account
setup. On the Account page, Dawarich exposes a per-user **API key** (a 32-byte token) for the
mobile app, plus an OwnTracks-compatible ingestion URL for alternative clients.

A hosted option, Dawarich Cloud, is also available for those who want the app without managing
the server themselves.

### Configuring the Dawarich mobile app
1. Install the Dawarich app (Android: Play Store, or the community FOSS build on F-Droid; iOS:
   App Store)
2. In Settings, enter your **Dawarich instance URL** (`https://<your-domain>` — put it behind a
   reverse proxy with TLS; see [Comparison with commercial apps](#comparison-with-commercial-apps)
   for why a VPN isn't required)
3. Enter the per-user **API key** from the Account page
4. Tap **Test connection** to confirm the app can reach the server and authenticate
5. The app records locally first and queues points on-device, uploading automatically once back
   online — a dropped connection doesn't lose data in the meantime

This is the simplest path if Dawarich is the only backend: one app, one server, native HTTPS +
API-key auth, no separate publisher/backend split to configure.

## Alternative clients
Dawarich also accepts pings from other tracking apps instead of its own — useful if one of them
offers something Dawarich's own app doesn't.

### OwnTracks
[OwnTracks](https://owntracks.org/) is only a location *publisher* — the mobile app reports
position to a server over MQTT or HTTP, with no map UI or history of its own. Point it at
Dawarich's OwnTracks-compatible ingestion URL (HTTP mode) and Dawarich renders the history exactly
as if its own app had sent it.

Configure it:
1. Install OwnTracks (F-Droid, Play Store, or App Store)
2. In app settings, set the mode to `HTTP`
3. Point the `Host`/`URL` field at Dawarich's ingestion URL, **using `https://`** — OwnTracks
   strongly recommends never sending credentials over plain HTTP
4. Under **Identification**, set a username/password — this is **HTTP Basic Auth**, checked by
   the reverse proxy in front of Dawarich, not the API-key mechanism Dawarich's own app uses
5. Set a `Device ID` and `Tracker ID` per family member so points are attributed correctly
6. Choose a tracking mode — `Move` (reports on a configurable distance/interval, default 100m /
   300s), `Significant`, or `Manual` — to balance accuracy against battery usage

**References**
* [OwnTracks](https://owntracks.org/)
* [OwnTracks HTTP mode](https://owntracks.org/booklet/tech/http/)

## Alternative backends
Entirely separate self-hosted systems, for when Dawarich itself isn't the right fit.

### Traccar
[Traccar](https://www.traccar.org/) is a free, open-source GPS tracking platform aimed more at
real-time device/fleet tracking than personal history and analytics — it supports 2,000+ GPS
tracker protocols on top of its own mobile clients, with live map viewing, event notifications,
geofencing, and reporting built into its web interface. Unlike Dawarich, it's oriented around
live tracking and device management rather than a Google-Timeline-style history view, and it can
track non-phone GPS hardware (cars, bags, pets) alongside family members.

It ships an official Docker image; a minimal stack (using the bundled H2 database, fine for
personal/family use — swap in PostgreSQL for anything larger) looks like:

```yaml
services:
  traccar:
    image: traccar/traccar:latest
    volumes:
      - ./traccar/logs:/opt/traccar/logs
      - ./traccar/data:/opt/traccar/data
    ports:
      - "8082:8082"   # web interface
      - "5055:5055"   # OsmAnd/Traccar Client protocol
```

Browse the web interface at `http://<your-server-ip>:8082` (default login `admin`/`admin` — change
it immediately) and add a device per family member to get their unique identifier.

Configure the **Traccar Client** app (Android/iOS) per device:
1. Install Traccar Client (F-Droid, Play Store, or App Store)
2. Set **Server URL** to `http://<your-server-ip>:5055`
3. Set the **Device Identifier** to match the device you created in the web UI
4. Choose an **Accuracy** level (`High` uses GPS continuously; lower levels trade precision for
   battery)
5. Enable **Wake lock** (Android only) so tracking continues reliably while moving, and grant the
   app unrestricted background usage — otherwise Android's deep sleep can create large gaps in
   reported location

Traccar Client's pings can also be pointed at Dawarich instead of a Traccar server, since Dawarich
ingests that format too — useful if the goal is Dawarich's history view but Traccar Client's
tracking behavior specifically.

### OwnTracks Recorder
[OwnTracks Recorder](https://github.com/owntracks/recorder) is a much more minimal alternative to
Dawarich — it subscribes to an MQTT broker, ingests location publishes, and stores points as plain
files with no external database required. It has a live map (current location, updated over
WebSocket) built in, but no history stats, geofencing dashboard, or family/circle concept.

Because it pairs with the OwnTracks app the same way as the [Alternative clients](#alternative-clients)
setup above, it has its own dedicated peer doc with full setup steps and a detailed comparison
against Dawarich: see [OwnTracks](../owntracks/README.md). Recorder is worth it specifically when
history/trip stats aren't needed at all — just a live "where is everyone right now" map with a
much lighter server footprint than Dawarich's four-service stack. Existing Recorder deployments
aren't wasted if outgrown later: Dawarich can import Recorder's stored history when migrating.

Other backends exist too:
* [PhoneTrack](https://apps.nextcloud.com/apps/phonetrack) — a Nextcloud app, useful if
  location tracking should live alongside an existing Nextcloud instance

## Comparing the three mobile apps
Three purpose-built client apps show up across this stack, one per backend — [Dawarich](#configuring-the-dawarich-mobile-app),
[OwnTracks](#owntracks), and [Traccar Client](#traccar) — each a first-party publisher for the
server it's paired with, not a repurposed third-party app.

| | Dawarich app | OwnTracks app | Traccar Client |
| --- | --- | --- | --- |
| Built for | Dawarich specifically | Any MQTT/HTTP backend (Dawarich, Recorder, Home Assistant, others) | Traccar specifically (though Dawarich also ingests its format) |
| Auth | Native API key (revocable per user) | HTTP Basic Auth (shared username/password, enforced by a reverse proxy) | Device Identifier only — no separate password/token; treat the identifier itself as a secret |
| HTTPS | ✓ Native | ✓ Native, strongly recommended over plain HTTP | ✓, but requires a reverse proxy in front — port 5055 itself is plain HTTP |
| Offline handling | Queues points on-device, uploads when back online | Depends on tracking mode/backend | Not a strong point historically — designed for continuous connectivity |
| Home Assistant / automations | ✗ Not built for it | ✓ MQTT is pub/sub — the same location stream can also drive HA zone automations | ✗ Not built for it |
| Distribution | Play Store/App Store; official app needs Google Play Services, separate community FOSS build on F-Droid | F-Droid, Play Store, App Store — cleanly open source everywhere | F-Droid, Play Store, App Store — cleanly open source everywhere, no ads/tracking |
| Setup complexity | Lower — one app, one auth model | Higher — client + backend + reverse-proxy auth to configure separately | Moderate — client + backend, but auth is just an identifier (simpler, but weaker) |

**Takeaway** — the auth model is the real differentiator. Dawarich's API key is the cleanest
(revocable per user, no shared secret). OwnTracks' Basic Auth is solid as long as it's always
proxied over HTTPS, and its MQTT pub/sub model is the only one of the three that plays well with
Home Assistant automations. Traccar Client's Device Identifier is the weakest scheme by design —
it's not a password, so anyone who learns or guesses it (and can reach the tracking port) can post
fake locations under that identity unless the endpoint is locked down behind a reverse proxy with
its own auth layer. For a family setup, that makes Dawarich's app the safest default, OwnTracks the
right call specifically for Home Assistant integration, and Traccar Client worth the extra
proxy-side care only when its live tracking/device-management features are actually needed.

**References**
* [Dawarich for Android](https://dawarich.app/docs/dawarich-for-android/)
* [OwnTracks HTTP mode](https://owntracks.org/booklet/tech/http/)
* [Home Assistant OwnTracks integration](https://www.home-assistant.io/integrations/owntracks/)
* [Traccar Client configuration](https://www.traccar.org/client-configuration/)
* [Traccar secure connection guide](https://www.traccar.org/secure-connection/)

## Comparison with commercial apps
Self-hosting trades the polished safety features of commercial apps for full control over the
data and no subscription cost — but it also puts setup, uptime, and battery-tuning on you.

| Feature                   | Dawarich                    | Life360                       | HeyPolo                     | GeoZilla                   |
| ------------------------- | --------------------------- | ------------------------------- | ------------------------------ | ----------------------------- |
| Data ownership            | Full, self-hosted           | Vendor-hosted                    | Vendor-hosted                   | Vendor-hosted                  |
| Real-time location        | ✓ Configurable interval     | ✓ Free                           | ✓ Free to join                  | ✓ Free                          |
| Location history          | ✓ Unlimited, self-hosted    | Limited on free, paid for more    | Time-limited by design           | Limited free, paid for more      |
| Geofencing / place alerts | Possible via Home Assistant | ✓ Free (2), paid unlimited          | ✓ Arrival alerts                  | ✓ Paid                            |
| Driving reports           | ✗ Not built in              | ✓ Free summary, paid per-member       | ✓ Paid                             | ✓ Paid                             |
| Crash detection           | ✗                           | ✓ Free (paid dispatch)                  | ✗                                    | ✓ Paid                               |
| SOS / emergency dispatch  | ✗ Not built in              | ✓ Free (paid dispatch)                    | ✓ Paid                                | ✓ Paid                                 |
| Ongoing cost              | Server hosting only (or Dawarich Cloud) | Free tier + paid Gold/Platinum            | Free to join, paid to host             | Free tier + paid Premium                 |
| Maintenance burden        | High — you run the server/updates          | None                                        | None                                    | None                                      |

**Takeaway** — Dawarich makes sense when the priority is owning your location history outright
(no vendor, no data sale risk, no subscription) and you're already comfortable running self-hosted
services. For emergency-response features — crash detection with dispatch, SOS with notified
contacts — Dawarich doesn't replicate that out of the box; those still require a commercial app or
a custom Home Assistant automation layer on top.

**Reaching the server from cellular data** — a teen's phone off the home Wi-Fi still needs to reach
the server. The Dawarich app supports this over plain HTTPS with its API key, and the OwnTracks app
supports it via HTTP Basic Auth behind a reverse proxy — so a VPN/tunnel isn't required, just a
domain, a TLS certificate, and a reverse proxy (or a tunnel service like Cloudflare Tunnel if
avoiding port-forwarding). That's still more setup than installing a commercial app, but it's
normal self-hosting infrastructure, not an extra always-on VPN client running alongside the
tracker.

**References**
* [Dawarich](https://dawarich.app/)
* [Dawarich self-hosting docs](https://dawarich.app/docs/self-hosting/introduction/)
* [Dawarich for Android](https://dawarich.app/docs/dawarich-for-android/)
* [Dawarich vs Traccar](https://dawarich.app/docs/comparisons/vs-traccar/)
* [Dawarich vs OwnTracks](https://dawarich.app/docs/comparisons/vs-owntracks/)
* [Traccar](https://www.traccar.org/)
* [Traccar Docker installation](https://www.traccar.org/docker/)
* [Traccar Client configuration](https://www.traccar.org/client-configuration/)
* [OwnTracks Recorder](https://github.com/owntracks/recorder)
