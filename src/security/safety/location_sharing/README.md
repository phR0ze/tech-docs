# Location Sharing <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Family location sharing. The requirements are always on in the background with a free tier for
Android. Ideally it would also be less reputation tarnished than life360.

### Quick links
- [.. up dir](..)
- [Overview](#overview)
  - [Life360 (baseline)](#life360-baseline)
  - [Life360 vs GeoZilla](#life360-vs-geozilla)
- [Disqualified](#disqualified)
  - [No Always Sharing](#no-always-sharing)
    - [Microsoft Family Safety](#microsoft-family-safety)
    - [Google Maps location sharing](#google-maps-location-sharing)
    - [Glympse](#glympse)
  - [No Free Tier](#no-free-tier)
    - [HeyPolo](#heypolo)
    - [FamiSafe](#famisafe)
  - [No Android Option](#no-android-option)
    - [Find My Friends](#find-my-friends)
  - [Too Invasive](#too-invasive)
    - [Google Family Link](#google-family-link)
  - [Poor Reputation](#poor-reputation)
    - [iSharing](#isharing)

### Linked pages
- [Dawarich](dawarich/README.md)
- [OwnTracks](owntracks/README.md)

## Overview

### Life360 (baseline)
[Life360](https://www.life360.com/) is the app every alternative in this page is measured against —
it meets both hard requirements (free tier, purpose-built always-on background tracking) but
carries the reputation problems the other options are being evaluated to avoid.

**Free tier** — a full circle can share live location for free, but it's capped at **2 Places
(geofences) and 2 days of location history**; paid tiers raise those caps and add features:
Silver ($7.99/mo) gives 5 Places and 7 days, Gold ($14.99/mo) gives unlimited Places, 30 days, and
driver reports, Platinum ($24.99/mo) adds towing and stolen-funds reimbursement.

**Battery** — Life360 claims ~10% extra drain over 24 hours via an adaptive wake algorithm, but
battery drain is consistently one of the most common complaints in the category; some Android
users report 20%+ drain after OS updates broke the optimization.

**Reducing background battery drain** — a lot of "Life360 battery tips" advice online conflates two
different things, and one of them breaks always-on tracking entirely:
* **Don't** restrict Life360's Android battery access to "Restricted" or set location permission to
  "Only while using the app" — this lets Android suspend the background service, so location stops
  updating when the app isn't open, defeating the point of always-on tracking.
* **Do** set Life360's Android battery access to **Unrestricted** and location permission to
  **Allow all the time** — this keeps Android from killing the service, which is what makes
  tracking reliable in the first place.
* The actual lever for reducing drain is inside Life360's *own* settings, not Android's: switch
  Location Settings from the default/"High Accuracy" to **Battery Saving** or **Low** (trades GPS
  precision for less frequent polling, similar to the Significant Location Change trade-off used by
  [OwnTracks](owntracks/README.md)), turn off **Drive Detection** if driving reports aren't needed
  (it keeps motion sensors and GPS active more aggressively to detect trip start/stop), and disable
  **Motion Detection**/**Analytics Sharing** in Advanced Settings if present.
* There's a real ceiling here — this reduces drain, it doesn't eliminate the always-on background
  GPS cost. Life360 doesn't expose the underlying polling thresholds it uses, so unlike OwnTracks
  this trade-off isn't fully transparent or tunable.

**Data monetization and legal history:**
* **FTC enforcement (Jan 2025)** — the FTC formally ordered Life360 to stop selling sensitive
  location data collected from users.
* **Texas AG lawsuit** — Texas AG Ken Paxton sued Allstate/Arity over unlawful collection and sale
  of location/driving data from 45M+ consumers via embedded SDKs, naming Life360 as an example of
  how the data pipeline worked (insurers using it to justify premium increases).
* **Data breach (Mar–Jul 2024)** — a hacker exploited a Life360 login API vulnerability; by
  July 2024 a database of 442,519 users' names, phone numbers, and emails was posted publicly.
* **Original class-action** over data sales without consent was dismissed in Nov 2023 without a
  settlement — no compensation fund exists, but the underlying practice is what triggered the FTC
  action above.

**Teen resentment** — the most distinctive reputation problem, separate from the data/privacy
issues above: Life360 became a TikTok meme for teens to mock and evade, with widely shared
tutorials on spoofing or disabling it without a parent noticing. Researchers cited in coverage warn
constant tracking can foster resentment, anxiety, and reduced autonomy in teens specifically.

**Takeaway** — Life360's core problem isn't just "sells data" (now under an FTC stop-order), it's
a pattern: data monetization + a data breach + documented teen-culture rejection. Every alternative
compared on this page is being weighed against that bar, not just against Life360's feature set.

**References**
* [Life360](https://www.life360.com/)
* [Life360 plans and pricing](https://www.life360.com/plans-pricing)
* [Life360 Sued for Selling Location Data — The Markup](https://themarkup.org/privacy/2023/06/01/life360-sued-for-selling-location-data)
* [Life360 and Battery Usage — official support article](https://support.life360.com/hc/en-us/articles/23053716563223-Life360-and-Battery-Usage)
* [Life360: Turn Off Battery Optimization](https://poweruptips.com/turn-off-battery-optimization-life360/)

### Life360 vs GeoZilla
[GeoZilla](https://geozilla.com/) most closely mirrors Life360's own feature set among the free-tier
alternatives, with basic location sharing, geofencing, and alerts genuinely free — not just
"free to join" like [HeyPolo](#heypolo). It requests "Always"/background location permission
specifically to enable continuous GPS, geofencing, and crash detection while closed, the same
purpose-built model Life360 uses.

**References**
* [GeoZilla](https://geozilla.com/)
* [GeoZilla Privacy Policy](https://geozilla.com/privacy-policy.html)
* [GeoZilla review — battery and billing complaints](https://umobix.com/blog/geozilla-review/)

| Feature              | Life360                        | GeoZilla                   |
| -------------------- | ------------------------------ | -------------------------- |
| Location sharing     | ✓ Free                         | ✓ Free                     |
| Sharing model        | Always-on                      | Always-on                  |
| Driving reports      | ✓ Free summary, paid per-member| ✓ Paid (Driver Protection) |
| Crash detection      | ✓ Free (paid dispatch)         | ✓ Paid                     |
| SOS / emergency      | ✓ Free                         | ✓ Paid (unlimited alerts)  |
| Shares data with ad networks | Criticized for selling location data | ✓ Confirmed in its own privacy policy |
| Battery drain complaints | ✓ Common complaint         | ✓ Common complaint, reported worsening over time |
| Billing complaints   | Paywalled features, teen resentment | Unauthorized charges, hard to cancel, unresponsive support |
| Focus                | Location + driving             | Location + driving         |
| Pricing              | Free tier + paid Gold/Platinum | Free tier + paid Premium   |

**Takeaway** — matches Life360 feature-for-feature, but not on reputation. GeoZilla covers the
same driving/crash/SOS feature set as Life360, just gated behind its own paid tier — but it isn't
the cleaner alternative it first appears to be: its own privacy policy confirms sharing device and
usage data with third-party ad networks (a softer version of the data-sale criticism leveled at
Life360), and it carries its own cluster of battery-drain and billing complaints (unauthorized
charges, difficult cancellation, unresponsive support). For someone specifically trying to avoid
Life360's reputation problems, GeoZilla doesn't clearly deliver that — it's cheaper feature-for-feature,
not obviously better on privacy or reliability. If owning the data outright matters more than
either of these vendor features, see [Dawarich](dawarich/README.md) (full history/trip stats) or
[OwnTracks](owntracks/README.md) (lighter, current-location-only) for a self-hosted alternative.
For an app that also bundles screen time and content filtering alongside location, see
[FamiSafe](#no-free-tier) — but note it has no free tier.

## Disqualified

### No Always Sharing
The following are commonly suggested Life360 alternatives that don't make it into the comparisons
above because they can't provide continuous, unattended background location — they fail the base
"works in the background" requirement outright:

#### Microsoft Family Safety
[Microsoft Family Safety](https://www.microsoft.com/en-us/microsoft-365/family-safety) is a
parental control and family organization app bundled with Microsoft 365. It originally offered
its own location sharing and driving-safety features, but Microsoft discontinued both on
November 29, 2024, leaving screen time and content filtering as its core focus.

| Feature            | Life360                              | Microsoft Family Safety   |
| ------------------ | ------------------------------------- | ------------------------- |
| Location sharing   | ✓ Free                                | ✗ Discontinued Nov 2024   |
| Place alerts       | ✓ Free (2), unlimited on paid         | ✗ Discontinued Nov 2024   |
| Driving reports    | ✓ Free summary, paid per-member       | ✗ Discontinued Nov 2024   |
| Crash detection    | ✓ Free (paid dispatch)                | ✗                         |
| SOS / emergency    | ✓ Free                                | ✗                         |
| Screen time        | ✗                                     | ✓ Free                    |
| Content filtering  | ✗                                     | ✓ Free, via Edge          |
| Spending controls  | ✗                                     | ✓ Free                    |
| Focus              | Location + driving                    | Device usage + content    |
| Pricing            | Free tier + paid Gold/Platinum        | Free, no paid tier        |

**Takeaway** — no longer a location-sharing option at all: Microsoft discontinued location
sharing and driving-safety features in November 2024, leaving only screen time and content
filtering. Families wanting location tracking need Life360 (or a similar app) regardless of
whether they also use Family Safety for screen time management.

**References**
* [Microsoft Family Safety](https://www.microsoft.com/en-us/microsoft-365/family-safety)
* [Life360 driving safety](https://www.life360.com/driving-safety)
* [Life360 lesser known features](https://www.life360.com/learn/life360-lesser-known-features)

#### Google Maps location sharing
[Google Maps](https://maps.google.com/) has a built-in, free "Share location" feature and is
often suggested as the simplest free Life360 alternative since it requires no extra app install —
every Android phone already has it.

Google also surfaces the same feature through a **People tab** in the
[Find My Device](https://support.google.com/accounts/answer/9363497?hl=en) app (rolled out in the
March 2025 Pixel Drop, still marked beta), showing a map of everyone sharing with you along with
their battery level and last-updated time. It looks like a separate, more purpose-built option
since it lives next to the "Devices" tracker tab, but it isn't independent infrastructure — shares
made in Find My Device are the same "Google Location Sharing" service as Maps and
[mirror directly to/from the Maps app](https://www.androidcentral.com/phones/google/google-find-my-device-app-people-tab-update-rollout).
It's a different UI on top of the identical backend, so it inherits the exact same background
reliability problems described below rather than fixing them.

| Feature              | Life360                        | Google Maps location sharing |
| --------------------- | ------------------------------ | ----------------------------- |
| Cost                  | Free tier + paid Gold/Platinum | ✓ Free                        |
| Extra app to install  | ✓ Yes                          | ✗ None (pre-installed)        |
| Reliable background tracking | ✓ Purpose-built for it  | ✗ Not reliable                |
| Geofencing / place alerts | ✓ Free (2), paid unlimited | ✗                              |
| Driving reports       | ✓ Free summary, paid per-member| ✗                              |
| Crash detection       | ✓ Free (paid dispatch)         | ✗                              |
| SOS / emergency       | ✓ Free                         | ✗                              |

**Takeaway — does not meet the "runs in the background" requirement.** Google's own support
threads document that background sharing routinely stops updating once the Maps app is swiped
away or the OS applies battery optimization, and several users report it silently falls back to
only updating while Maps is open in the foreground. Getting it to behave reliably requires
manually exempting Maps from battery optimization, granting "Allow all the time" location
permission, and disabling any background-app-refresh restriction — and even then it's not
guaranteed on all devices/Android versions. For a concerned parent who needs a teen's location to
keep updating unattended, this makes Google Maps a poor fit compared to Life360 or the other
purpose-built apps compared above, which are designed and tested specifically for persistent
background location reporting.

**References**
* [Google Maps location sharing](https://maps.google.com/)
* [Google Maps Community — location sharing only working when Maps app open](https://support.google.com/maps/thread/334241195/location-sharing-only-working-when-maps-app-open?hl=en)
* [Google Maps Community — location sharing just stopped](https://support.google.com/maps/thread/446013611/google-maps-location-sharing-just-stopped-anyone-else?hl=en)
* [Find My Device People tab rollout](https://www.androidcentral.com/phones/google/google-find-my-device-app-people-tab-update-rollout)
* [Android Authority — reverting from the new Find My People tab back to Maps sharing](https://www.androidauthority.com/google-find-my-device-people-tested-3538619/)

#### Glympse
[Glympse](https://app.glympse.com/) is a free, real-time GPS-sharing app that's been around since
2008, built around temporary, ETA-style shares — recipients don't even need the app installed,
since a share opens as a link in any browser.

| Feature              | Life360                        | Glympse |
| --------------------- | ------------------------------ | -------- |
| Cost                  | Free tier + paid Gold/Platinum | ✓ Free    |
| Sharing model         | Always-on circle                | Timer-based, 5 min – 12 hrs, then auto-expires |
| Private Groups        | ✓ Always-on family circle      | ✓ Invite-only, but members only visible while actively sharing |
| Reliable background tracking | ✓ Purpose-built for it  | ✗ Requires manual battery-optimization exemption; reviews note lag/stalls |
| Geofencing / place alerts | ✓ Free (2), paid unlimited | ✗ |
| Driving reports       | ✓ Free summary, paid per-member| ✗ |
| Crash detection       | ✓ Free (paid dispatch)         | ✗ |
| SOS / emergency       | ✓ Free                         | ✗ |

**Takeaway — built for the opposite use case, doesn't meet the base requirement.** Glympse's
entire model is opt-in, timer-bound sharing: every share (even inside a Private Group) has to be
manually started and expires on its own after a few minutes to a max of 12 hours, at which point
location data is wiped from Glympse's servers. There's no persistent "always show my teen's
location" mode — a parent would need the teen to repeatedly restart sharing to stay visible. On
top of that it has the same background-reliability weakness as Google Maps above: Android battery
optimization has to be manually disabled for Glympse, and it can still lag or stall mid-session.
For a concerned parent who wants continuous, unattended background tracking, Glympse is a weaker
fit than Life360 or the other purpose-built apps compared above.

**References**
* [Glympse](https://app.glympse.com/)
* [What is the Glympse app?](https://app.glympse.com/faq/what-is-the-glympse-app-and-what-does-it-do/)
* [Glympse Android background sharing configuration](https://glympse.zendesk.com/hc/en-us/articles/20212069131547-Glympse-Configuration-Settings-Android)

### No Free Tier
The following are commonly suggested Life360 alternatives that don't make it into the comparisons
above because they have no free tier — they fail the base "free" requirement outright, so their
background-reliability and feature trade-offs are moot:

#### HeyPolo
[HeyPolo](https://heypolo.com/) (built by the team behind the Surfshark VPN) is often pitched as
free, privacy-first location sharing — and joining someone else's circle is genuinely free — but
setting up your own family circle isn't. Per HeyPolo's own pricing page, only one member needs to
subscribe ($3.99-5.99/mo) to host a circle and invite unlimited members for free. For a parent
setting up a new circle to track their own teen, that means someone in the family has to be the
paying host; there's a 7-day trial, but no way to actually run a circle for free long-term.

It's also built around consent-based, largely time-limited sharing — scheduled/repeating windows
or one-time shares rather than Life360's default always-on model — which edges toward the same
"No Always Sharing" problem as [Glympse](#glympse), on top of the free-tier gap.

**References**
* [HeyPolo](https://heypolo.com/)
* [HeyPolo pricing](https://heypolo.com/pricing)
* [Surfshark launches HeyPolo](https://surfshark.com/blog/surfshark-launches-heypolo)

#### FamiSafe
[FamiSafe](https://www.famisafe.com/) (Wondershare) leans further into parental-control territory
than a pure location-sharing app, with screen time, content filtering, driving monitoring,
call/message monitoring, and monitoring across 30+ social platforms alongside GPS tracking. It also
offers **reverse location tracking**, letting the monitored teen see the parent's location back.

There's no free tier — only a 3-day trial capped to 1 device and 1 day of history, after which it's
$4.99/mo minimum (5 devices), scaling to $9.99+/mo for 10-30 devices on quarterly/annual plans.
Reviewers report background location tracking works reasonably well (one measurement put battery
impact around 0.94%, lower than apps like Instagram), but note real-time updates depend on a
stable internet connection and degrade in poor-coverage areas — and Wondershare's own in-app
guidance warns to only use continuous location tracking for short periods.

**References**
* [FamiSafe](https://www.famisafe.com/)
* [FamiSafe review — pricing and trial terms](https://cybernews.com/best-parental-control-apps/famisafe-review/)

### No Android Option
The following are commonly suggested Life360 alternatives that don't make it into the comparisons
above because they're exclusive to Apple's ecosystem — no Android app exists, so they fail the
base "installs on an Android phone" requirement outright, before any of the background-reliability
or feature trade-offs even apply:

#### Find My Friends
[Find My Friends](https://support.apple.com/guide/iphone/share-your-location-iph01954dc44/ios)
(now merged into Apple's **Find My** app) is iOS-only, proprietary to Apple. Apple points users
wanting to share between an iPhone and an Android device toward Google Maps or a third-party app
like Life360 instead; there's no way to install it on Android.

**References**
* [Apple Find My — Share your location](https://support.apple.com/guide/iphone/share-your-location-iph01954dc44/ios)

### Too Invasive
The following are commonly suggested Life360 alternatives that don't make it into the comparisons
above because they're built around full device supervision rather than lightweight location
sharing — a poor fit for an older, independent teen even though the location-sharing piece works:

#### Google Family Link
[Google Family Link](https://families.google/familylink/) is Google's free parental-control app,
bundling app approval, screen time limits, content filtering, and location tracking for a child's
Android device.

| Feature              | Life360                        | Google Family Link |
| --------------------- | ------------------------------ | -------------------- |
| Cost                  | Free tier + paid Gold/Platinum | ✓ Free                |
| Location sharing      | ✓ Free                         | ✓ Free                |
| Designed age range     | All ages, including independent teens | ✓ Best for under-13 |
| Teen can self-remove supervision | ✗ No               | ✓ Yes, with parent consent flow (parent notified) |
| Screen time / app approval | ✗                         | ✓ Free                 |
| Content filtering      | ✗                              | ✓ Free                 |
| Perceived by teen as  | Location-sharing peer group ("circle") | Parental control / device lockdown |

**Takeaway — fits younger children well, but doesn't meet the base requirement for older
teens.** Google itself positions Family Link as best suited for children under 13; for teens, both
the parent and teen accounts have to consent to supervision, and the teen can disable it from
their own device (the parent is only notified after the fact, not asked first). Combined with its
core identity as a full device-lockdown tool — app approval, content filtering, screen time — it
reads to an older teen as parental control rather than a lightweight "let my family see where I
am" circle, and it invites the same removal/pushback problems documented in Google's 2026
age-13 supervision-removal controversy. For a concerned parent who wants ongoing, hard-to-disable
background location on an older teen's phone, Family Link is a weaker fit than Life360 or the
other purpose-built location-sharing apps compared above.

**References**
* [Google Family Link](https://families.google/familylink/)
* [Family Link FAQ](https://families.google/familylink/faq/)
* [Google Mandates Parental Approval for Teens to End Family Link Oversight](https://www.webpronews.com/google-mandates-parental-approval-for-teens-to-end-family-link-oversight/)

### Poor Reputation
The following meet the base free/always-on/Android requirements but don't make it into the
comparisons above because their reputation is worse than [Life360's](#life360-baseline) — the
thing this whole comparison is trying to avoid:

#### iSharing
[iSharing](https://isharingsoft.com/) looks like the strongest free option on paper: its free tier
includes **unlimited real-time location sharing, unlimited place alerts, and 30-day location
history** — more generous than Life360's free tier (2 Places, 2-day history) — with continuous
background tracking and reportedly lower battery impact (~10-15%/day) than Life360.

That's outweighed by its security history. A 2024 vulnerability exposed **35 million users**: bugs
in the app's group-creation and authorization logic let *any* user access *anyone else's*
real-time location, name, email, phone number, and profile photo — without consent — by exploiting
sequential user IDs and a hardcoded authorization key baked into the app itself. That's a worse
failure mode than Life360's 2024 breach (which leaked contact info, not live location access to
strangers). Reviewers also report aggressive ads/popups and refund refusals.

**Takeaway — fails the reputation requirement, arguably worse than Life360.** iSharing beats
Life360 and GeoZilla on the free-tier/background criteria alone, but a vulnerability that let
strangers pull anyone's real-time location without consent is a harder disqualifier than anything
Life360 has been criticized for. Meeting the hard requirements isn't enough if the app itself is
the bigger risk.

**References**
* [iSharing](https://isharingsoft.com/)
* [iSharing App Security Lapse Put 35 Million Users at Risk — CyberInsider](https://cyberinsider.com/isharing-app-security-lapse-put-35-million-users-at-risk/)
* [Phone tracking app with millions of users has a major security flaw — TechRadar](https://www.techradar.com/pro/security/phone-tracking-app-with-millions-of-users-has-a-major-security-flaw-that-can-expose-precise-locations)
