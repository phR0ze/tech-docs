# Life360 <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

[Life360](https://www.life360.com/) is the app every alternative in [Location Sharing](../README.md)
is measured against — it meets both hard requirements (free tier, purpose-built always-on
background tracking) but carries the reputation problems the other options are being evaluated to
avoid.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
* [Free tier and pricing](#free-tier-and-pricing)
* [Battery](#battery)
  * [Reducing background battery drain](#reducing-background-battery-drain)
* [Data monetization and legal history](#data-monetization-and-legal-history)
* [Teen resentment](#teen-resentment)

## Overview
Life360 is a commercial, vendor-hosted family location-sharing app — a "circle" of family members
share live location with each other, with driving safety and emergency-response features layered
on top. It's the default most people think of first, and the baseline every other app in
[Location Sharing](../README.md) gets compared against on three axes: does it stay free, does it
actually run in the background, and is it less reputation-tarnished than Life360 itself.

## Free tier and pricing
A full circle can share live location for free, but it's capped at **2 Places (geofences) and
2 days of location history**; paid tiers raise those caps and add features:
* **Silver** ($7.99/mo) — 5 Places, 7 days of history
* **Gold** ($14.99/mo) — unlimited Places, 30 days of history, driver reports
* **Platinum** ($24.99/mo) — adds towing and stolen-funds reimbursement

## Battery
Life360 claims ~10% extra drain over 24 hours via an adaptive wake algorithm, but battery drain is
consistently one of the most common complaints in the category; some Android users report 20%+
drain after OS updates broke the optimization.

### Reducing background battery drain
A lot of "Life360 battery tips" advice online conflates two different things, and one of them
breaks always-on tracking entirely:

#### Settings
* Tap `Notification Center`
  * Tap `Place Notifications` and delete them all
  * Tap `Low Battery Notifications` and switch `OFF`
  * Tap `Safe Drive Notifications` and switch `OFF`
  * Tap `Safety Incident Alerts` and `Disable All`
  * Tap `Severe Weather Alerts` and `Disable All`
  * Tap `Subscription Tracker notifications` and switch `OFF`
  * Tap `Pet routine reminders` and switch `OFF`
  * Tap `Place Ads` and switch `OFF`
  * Tap `Real-time Ads` and switch `OFF`
* Tap `Flight Detection` and switch `OFF`
* Tap `Pet Finder Network` and switch `OFF`
* Tap `Place suggestions` and switch `OFF`
* Tap `Privacy Center` then:
  * Tap `Privacy Choices` then switch `Include my data` to `OFF`
  * Tap `Traffic and retial insights` then switch `Include my data` to `OFF`
  * Tap `Arity` then switch `Connect to Arity` to `OFF` and `Disconnect Arity`
* Tap `Digital Safety` then `Digital Safety` and switch `Data Breach Alerts` to `OFF` and confirm

#### Location Permissions

##### Parent settings
* Set to `Allow only while using the app` - not going to work unless you're purely in an observer
  capacity and don't need to share your location
* Disable `Use precise location` - instead of GPS-level accuracy (within a few meters), the phone
  reports location using cell towers and Wi-Fi triangulation, which is far less power-hungry, but can
  be off by a few meters to up to a mile.

##### Child settings
* Set to `Allow all the time`
* Enable `Use precise location` - uses GPS tracking rather than cell tower and Wi-Fi triangulation

## Data monetization and legal history
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

## Teen resentment
The most distinctive reputation problem, separate from the data/privacy issues above: Life360
became a TikTok meme for teens to mock and evade, with widely shared tutorials on spoofing or
disabling it without a parent noticing. Researchers cited in coverage warn constant tracking can
foster resentment, anxiety, and reduced autonomy in teens specifically.

**Takeaway** — Life360's core problem isn't just "sells data" (now under an FTC stop-order), it's
a pattern: data monetization + a data breach + documented teen-culture rejection. Every alternative
compared in [Location Sharing](../README.md) is being weighed against that bar, not just against
Life360's feature set.

**References**
* [Life360](https://www.life360.com/)
* [Life360 plans and pricing](https://www.life360.com/plans-pricing)
* [Life360 Sued for Selling Location Data — The Markup](https://themarkup.org/privacy/2023/06/01/life360-sued-for-selling-location-data)
* [Life360 and Battery Usage — official support article](https://support.life360.com/hc/en-us/articles/23053716563223-Life360-and-Battery-Usage)
* [Life360: Turn Off Battery Optimization](https://poweruptips.com/turn-off-battery-optimization-life360/)
