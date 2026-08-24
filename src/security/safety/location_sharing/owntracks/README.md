# OwnTracks <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

[OwnTracks](https://owntracks.org/) (the mobile app) paired with
[OwnTracks Recorder](https://github.com/owntracks/recorder) (the backend) is a light, self-hosted
"where is everyone right now" stack — the peer to [Dawarich](../dawarich/README.md), but for when
Google-Timeline-style history and trip stats aren't needed, just a live map of current location.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
* [Getting started](#getting-started)
  * [Docker Compose](#docker-compose)
  * [Configuring the mobile app](#configuring-the-mobile-app)
* [Comparison with Dawarich](#comparison-with-dawarich)
* [Comparison with commercial apps](#comparison-with-commercial-apps)

## Overview
OwnTracks the mobile app is only a location *publisher* — it reports position to a server over
MQTT or HTTP and has no map UI or history of its own. OwnTracks Recorder is the minimal backend
that gives it one: it subscribes to an MQTT broker, ingests location publishes, and stores points
as plain files, with no external database required.

What Recorder provides out of the box: a **live map** of current location, updated in real time
over WebSocket as new publishes arrive, plus a table of each device's last-known position. What it
doesn't provide: history/trip stats, geofencing, a family/circle concept, or a mobile app of its
own beyond OwnTracks itself. If those are needed later, [Dawarich](../dawarich/README.md) is the
fuller-featured peer built around browsing history rather than just current position — and it can
import Recorder's stored history if migrating (see
[Comparison with Dawarich](#comparison-with-dawarich) below).

## Getting started

### Docker Compose
A minimal stack pairs Recorder with [Eclipse Mosquitto](https://mosquitto.org/) as the MQTT
broker:

```yaml
services:
  mosquitto:
    image: eclipse-mosquitto
    volumes:
      - ./mosquitto/config:/mosquitto/config
      - ./mosquitto/data:/mosquitto/data
    ports:
      - "1883:1883"

  recorder:
    image: owntracks/recorder
    environment:
      OTR_HOST: mosquitto
      OTR_PORT: "1883"
    volumes:
      - ./recorder/store:/store
    ports:
      - "8083:8083"
    depends_on:
      - mosquitto
```

Browse the live map at `http://<your-server-ip>:8083`.

Recorder has no encryption or authentication of its own — by itself it's meant to sit behind a
reverse proxy (nginx, Caddy, Traefik) that terminates TLS (e.g. via Let's Encrypt) and adds HTTP
Basic Auth in front of it, rather than being exposed directly. See
[Configuring the mobile app](#configuring-the-mobile-app) below for how the client authenticates
against that setup.

### Configuring the mobile app
1. Install OwnTracks (F-Droid, Play Store, or App Store)
2. In app settings, set the mode to `HTTP`
3. Point the `Host`/`URL` field at your server, **using an `https://` URL**
   (`https://<your-domain>/pub`) — Recorder should sit behind a reverse proxy that terminates TLS;
   OwnTracks strongly recommends never sending credentials over plain HTTP
4. Under **Identification**, set a username and password to match the HTTP Basic Auth credentials
   configured on the reverse proxy
5. Set a `Device ID` and `Tracker ID` per family member so points are attributed correctly
6. Choose a tracking mode — `Move` (reports on a configurable distance/interval, default 100m /
   300s), `Significant`, or `Manual` — to balance accuracy against battery usage

## Comparison with Dawarich
| | OwnTracks + Recorder | Dawarich |
| --- | --- | --- |
| What it is | A minimal MQTT/HTTP subscriber that stores points as plain files | A full Rails app (PostGIS + Redis + Sidekiq) built around browsing history |
| Live current location | ✓ Live map, updates over WebSocket as publishes arrive, plus a last-locations table | ✓ |
| Map view | Live position + daily tracks only | Live position, polyline, heatmap, and "fog of war" (unexplored-area shading) |
| History / stats | ✗ None — raw point storage only, bring your own analysis | ✓ Distance traveled, places visited, trip breakdown |
| Areas / geofencing | ✗ Not built in | Built-in "Areas" concept (place-based, not real-time alerts) |
| Own mobile app | ✗ None — OwnTracks app only, HTTP Basic Auth via reverse proxy | ✓ Native app (Android/iOS) with per-user API-key auth |
| Home Assistant / automations | ✓ MQTT is pub/sub — the same location stream can also drive HA zone automations | ✗ Not built for it |
| Storage | Plain files, no database to run | Requires PostgreSQL/PostGIS + Redis (heavier ops footprint) |
| Resource footprint | Very light — one small container | Heavier — four services (app, Postgres, Redis, Sidekiq) |
| Active development | Limited in recent years | Active (single maintainer, frequent releases) |

**Takeaway** — pick this stack when the requirement really is just "where is my teen right now,"
with nothing more: lighter to run (one small container vs. four services), and OwnTracks' MQTT
pub/sub model is a genuine advantage if Home Assistant automations are also a goal. Pick Dawarich
instead as soon as history, trip stats, or a purpose-built mobile app (rather than OwnTracks'
Basic-Auth setup) start to matter — migrating later isn't wasted work, since Dawarich can import
Recorder's stored history. For how the OwnTracks app's auth model stacks up against Dawarich's own
app and Traccar Client, see
[Comparing the three mobile apps](../dawarich/README.md#comparing-the-three-mobile-apps) in the
Dawarich doc.

**References**
* [Dawarich](https://dawarich.app/)
* [Dawarich vs OwnTracks](https://dawarich.app/docs/comparisons/vs-owntracks/)
* [Home Assistant OwnTracks integration](https://www.home-assistant.io/integrations/owntracks/)

## Comparison with commercial apps
Self-hosting trades the polished safety features of commercial apps for full control over the
data and no subscription cost — but it also puts setup, uptime, and battery-tuning on you.

| Feature                   | OwnTracks + Recorder        | Life360                       | HeyPolo                     | GeoZilla                   |
| ------------------------- | --------------------------- | ------------------------------- | ------------------------------ | ----------------------------- |
| Data ownership            | Full, self-hosted           | Vendor-hosted                    | Vendor-hosted                   | Vendor-hosted                  |
| Real-time location        | ✓ Configurable interval     | ✓ Free                           | ✓ Free to join                  | ✓ Free                          |
| Location history          | ✗ Not built in (current position only) | Limited on free, paid for more    | Time-limited by design           | Limited free, paid for more      |
| Geofencing / place alerts | Possible via Home Assistant | ✓ Free (2), paid unlimited          | ✓ Arrival alerts                  | ✓ Paid                            |
| Driving reports           | ✗ Not built in              | ✓ Free summary, paid per-member       | ✓ Paid                             | ✓ Paid                             |
| Crash detection           | ✗                           | ✓ Free (paid dispatch)                  | ✗                                    | ✓ Paid                               |
| SOS / emergency dispatch  | ✗ Not built in              | ✓ Free (paid dispatch)                    | ✓ Paid                                | ✓ Paid                                 |
| Ongoing cost              | Server hosting only         | Free tier + paid Gold/Platinum            | Free to join, paid to host             | Free tier + paid Premium                 |
| Maintenance burden        | Moderate — lighter than Dawarich, but you still run the server/updates | None | None | None |

**Takeaway** — this stack matches the exact scope of "always on in the background, free, current
location on a map" without paying for the history/trip features Dawarich adds on top. As with
Dawarich, there's no built-in emergency-response layer (crash detection with dispatch, SOS with
notified contacts) — those still require a commercial app or a custom Home Assistant automation
layer.

**Reaching the server from cellular data** — a teen's phone off the home Wi-Fi still needs to reach
the server. The OwnTracks app supports this over plain HTTPS with HTTP Basic Auth behind a reverse
proxy, so a VPN/tunnel isn't required — just a domain, a TLS certificate, and a reverse proxy (or a
tunnel service like Cloudflare Tunnel if avoiding port-forwarding). That's still more setup than
installing a commercial app, but it's normal self-hosting infrastructure, not an extra always-on
VPN client running alongside the tracker.

**References**
* [OwnTracks](https://owntracks.org/)
* [OwnTracks Recorder](https://github.com/owntracks/recorder)
* [OwnTracks HTTP mode](https://owntracks.org/booklet/tech/http/)
