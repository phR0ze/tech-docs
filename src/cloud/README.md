# Cloud <img style="margin: 6px 13px 0px 0px" align="left" src="../data/images/logo_36x36.png" />

### Quick Links
- [.. up dir](..)
- [Overview](#overview)
- [Budget Comparison](#budget-comparison)
  - [Oracle Cloud](#oracle-cloud)

### Linked pages
- [Google Cloud Platform](google_cloud_platform/README.md)

## Overview
Hyperscalers are large cloud service providers that can provide services such as computing and 
storage at enterprise scale. While there is no universal standard for what should be classified as a 
hyperscaler, major cloud providers such as Amazon Web Services, Google Cloud, Microsoft Azure, IBM 
Cloud and Alibaba Cloud fit the description.

**References**
* [RedHat Hyberscaler](https://www.redhat.com/en/topics/cloud/what-is-a-hyperscaler)

## Budget Comparison
The VPS runs [Pangolin](../networking/reverse_tunnel/pangolin/README.md) as a reverse tunnel relay
for homelab services. Pangolin itself is lightweight (recommended: 2 vCPU, 2 GB RAM, 20 GB SSD) —
the VPS does no transcoding or storage. The binding constraint is **monthly transfer**.

**Workload estimate**

| Service | Usage | Bitrate | Daily transfer | Monthly transfer |
|---------|-------|---------|---------------|-----------------|
| Jellyfin | 3 × 1080p movies/day (~2 hr each) | ~10 Mbps | ~27 GB | ~810 GB |
| Immich | Photo browsing (~50 photos/day) | ~2–5 MB/photo | ~150 MB | ~5 GB |
| Immich | Video playback (~10 min/day) | ~20 Mbps | ~1.5 GB | ~45 GB |
| **Total** | | | **~29 GB** | **~860 GB (~0.85 TB)** |

With 2× headroom for overhead, peaks, and growth: **~1.7 TB/month minimum required.**

**Minimum VPS requirements**
- vCPU: 2
- RAM: 2 GB
- Storage: 20 GB
- Transfer: ≥ 2 TB/month

Note this table could be wildly inaccurate. Check the latest for yourself.

| Provider | Plan | Price | vCPU | RAM | Storage | Transfer | Network | Notes |
|----------|------|-------|------|-----|---------|----------|---------|-------|
| [RackNerd](https://www.racknerd.com/specials/) | 1 GB KVM | $1.83/mo ($21.99/yr) | 1 | 1 GB | 20 GB SSD | 3 TB/mo | 1 Gbps | Annual billing only; KVM; instant setup |
| [RackNerd](https://www.racknerd.com/specials/) | 2 GB KVM | $3/mo ($35.99/yr) | 2 | 2 GB | 35 GB SSD | 5 TB/mo | 1 Gbps | Most popular; annual billing only |
| [RackNerd](https://www.racknerd.com/specials/) | 4 GB KVM | $5/mo ($59.99/yr) | 3 | 4 GB | 60 GB SSD | 7 TB/mo | 1 Gbps | Annual billing only |
| [IONOS](https://www.ionos.com/servers/vps#plans) | VPS S+ | $2/mo → $5/mo | 2 | 2 GB | 90 GB NVMe | Unlimited | 1 Gbps | $2 promo for first 3 mo, then $5/mo; no transfer cap per ToS |
| [IONOS](https://www.ionos.com/servers/vps#plans) | VPS M+ | $4/mo → $11/mo | 4 | 4 GB | 120 GB NVMe | Unlimited | 1 Gbps | $4 promo for first 3 mo, then $11/mo; no transfer cap per ToS |
| [Vultr](https://www.vultr.com/pricing/) | Regular 2 GB | $10/mo | 1 | 2 GB | 55 GB SSD | 2 TB/mo | 1 Gbps | Only 1 vCPU; older Intel CPU; transfer barely meets 2 TB floor |
| [Vultr](https://www.vultr.com/pricing/) | Regular 2 vCPU | $15/mo | 2 | 2 GB | 65 GB SSD | 3 TB/mo | 1 Gbps | Meets all requirements; older Intel CPU |
| [Vultr](https://www.vultr.com/pricing/) | High Perf 2 vCPU | $18/mo | 2 | 2 GB | 60 GB NVMe | 4 TB/mo | 1 Gbps | Meets all requirements; AMD EPYC/Intel Xeon + NVMe |
| [DigitalOcean](https://www.digitalocean.com/pricing/droplets) | Basic 2 GB | $12/mo | 1 | 2 GB | 50 GB SSD | 2 TB/mo | 1 Gbps | Only 1 vCPU; transfer barely meets 2 TB floor |
| [DigitalOcean](https://www.digitalocean.com/pricing/droplets) | Basic 2 GB Dual | $18/mo | 2 | 2 GB | 60 GB SSD | 3 TB/mo | 1 Gbps | Meets all requirements; month-to-month billing |
| [DigitalOcean](https://www.digitalocean.com/pricing/droplets) | Basic 4 GB | $24/mo | 2 | 4 GB | 80 GB SSD | 4 TB/mo | 1 Gbps | Comfortable headroom; good if adding more services |

### Oracle Cloud
Oracle Cloud's ***Always Free*** option is awesome if you can get it; however its next to impossible
to actually get access as host availability is always consumed.

The option comes with ARM-based 2 OCPUs and 12GB RAM, 200GB of SSD and 10TB of monthly outbound data
transfer.

1. Login to http://oracle.com/cloud after creating an account
2. Create a new VCN
   1. Navigate to `Networking >Virtual cloud networks`
   2. Hit `Create VCN`
   3. Name the VCN e.g. `apps`
   4. Use the IPv4 CIDR Block e.g. `10.0.0.0/16`
   5. Hit `Create VCN`
3. Create new Subnet
   1. Navigate to the VCN's `Subnets` tab
   2. Hit `Create`
   3. Name it `apps-subnet`
   4. Set the IPv4 CIDR Block e.g. `10.0.0.0/24`
   5. Ensure `Public Subnet` is selected
   6. Set DNS Label to `dns`
   7. Hit `Create Subnet`
4. Create VM
   1. Navigate to `Compute >Instances`
   2. Hit `Create Instance`; you'll need at least 1 OCPU and 6GB RAM
   3. Name the new VM e.g. `apps`
   4. Click `Change Image` to `Canonical Ubuntu 24.04` and hit `Select Image`
   5. Ensure the `Shape` is the `VM.Standard.A1.Flex` with the `Always Free-elgible` label
   6. Click `Next` and `Next`
   7. Set the `VNIC` name e.g. `apps-vnic`
   8. Choose `Select existing virtual cloud network` and `Select existing subnet`
   9. Keep the `Automatically assign private IPv4 address`
   10. Click through next to `Create`
