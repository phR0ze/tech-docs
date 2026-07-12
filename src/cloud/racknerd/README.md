# RackNerd <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick Links
- [.. up dir](..)
- [Overview](#overview)
  - [Specials](#specials)
  - [Upgrade Policy](#upgrade-policy)
- [Pangolin VPS](#pangolin-vps)

## Overview
[RackNerd](https://www.racknerd.com) is a budget VPS provider offering KVM-based virtual servers at
aggressively low annual pricing. They run periodic specials — particularly around holidays — that
are among the cheapest entry points for a publicly addressable VPS. Plans include full root access,
instant setup, and SolusVM for management. Each VPS is provisioned with a ***static public IPv4
address*** that persists for the life of the subscription — essential for Pangolin's WireGuard
endpoint and DNS records.

### Specials
RackNerd's specials page (`racknerd.com/specials`) lists their current promotional KVM VPS plans.
Pricing is annual-only; there is no month-to-month option for special deals.

| Plan | Price | vCPU | RAM | Storage | Transfer | Network |
|------|-------|------|-----|---------|----------|---------|
| 1 GB KVM | $1.83/mo ($21.99/yr) | 1 | 1 GB | 20 GB SSD | 3 TB/mo | 1 Gbps |
| 2 GB KVM | $3/mo ($35.99/yr) | 2 | 2 GB | 35 GB SSD | 5 TB/mo | 1 Gbps |
| 4 GB KVM | $5/mo ($59.99/yr) | 3 | 4 GB | 60 GB SSD | 7 TB/mo | 1 Gbps |
| 6 GB KVM | $7.50/mo ($89.99/yr) | 6 | 6 GB | 100 GB SSD | 12 TB/mo | 1 Gbps |
| 8 GB KVM | $10/mo ($119.99/yr) | 7 | 8 GB | 150 GB SSD | 20 TB/mo | 1 Gbps |

***Note:*** specials change frequently and sell out — verify current pricing at
[racknerd.com/specials](https://www.racknerd.com/specials/) before ordering.

### Upgrade Policy
RackNerd specials are sold as standalone annual subscriptions, not upgradable plan tiers. There is
no built-in upgrade path — to move to a larger plan you purchase a new annual deal and migrate
manually, then let the old plan expire. Some users report success opening a support ticket to pay
the price difference for an in-place upgrade, but this is not official policy.

**Practical guidance:** start on the plan you actually need. The jump from 1 GB to 2 GB is only
~$1.17/mo — not worth the migration friction.

## Pangolin VPS
The [2 GB KVM plan](https://www.racknerd.com/specials/) at $35.99/yr is the recommended choice for
running [Pangolin](../../networking/reverse_tunnel/pangolin/README.md) as a homelab reverse tunnel
relay for Jellyfin and Immich.

It meets Pangolin's recommended specs (2 vCPU, 2 GB RAM) and the 5 TB/mo transfer cap provides
~3× headroom over the [estimated 1.7 TB/mo workload](../README.md#budget-comparison). It is the
cheapest option across all surveyed providers that satisfies all requirements.

* [Setup RackNerd - DB Tech](https://youtu.be/a-a-Xk1hXBQ)

### 
