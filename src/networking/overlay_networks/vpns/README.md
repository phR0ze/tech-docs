# VPNs <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Virtual Private Networks are just another type of overlay network. They connect your machine to 
another network through an encrypted tunnel. The purpose is to get access to the destination 
network's resources. Common use cases:
1. Get access to work servers and equipment
2. Get access to home network when away from home
3. Work around censorship or geolocation blocking

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
  * [Kape](#kape)
  * [Five Eyes](#five-eyes)
  * [Fourteen eyes](#fourteen-eyes)
* [IVPN](#ivpn)
* [Mulllvad](#mullvad)
* [Proton VPN](#proton-vpn)

## Overview
The security influencer community has consistently recommended only 2 VPNs as being safe and privacy 
focused. The wash of other options are all either corporate garbage, in Five Eyes countries, don't 
have anonymous registration or payments or are openly known as nefarious and untrusted.

| Service       | Owner       | Jurisdiction    | Logs | Devices | Price    | Monero  | Reg
| ------------- | ----------- | --------------- | ---- | ------- | -------- | ------- | -----
| Mullvad       | Independent | Sweden          | None | 10      | $5.90/m  | Yes     | Anon
| IVPN          | Independent | Gilbralter      | None | 2       | $6/m     | Yes     | Anon

* Prices are reflective of end of year sales for 2025.
* Monero is the defacto standard for anonymous crypto payments
* Anonymous registration is offered by the best
* Eliminating those without anonymouse registration and anonymous payments
  * Proton VPN
* Eliminating Kape Technology owned VPNs for their nefarious or untrusted reputation
  - Express VPN, Cyber Ghost, Private Internet Access, Intego Privacy Protection
* Eliminating corporate garbage i.e. multiple owned by the same company
  * Ziff Davis
    - IPVanish, StrongVPN, Encrypt.me, SaferVPN, Perimeter 81, Fast VPN, Internet Shield, SpeedtestVPN, netDNA
  * Nord Security
    - SurfShark, NordVPN, Atlas
* Eliminating Five Eyes countries (i.e. USA, United Kingdom, Canada, Australia, New Zealand)
  * Tunnel Bear, TorGuard, Windscribe

### Kape
Kape Technologies, formerly known as Crossdrider, is a holding company that owns many VPN servicdes 
and review affiliate websites. The majority shareholder is Israli businessman Teddy Sagi via his firm 
Unikmind. Crossrider originally dealt in Malware and their specialty was injecting ads. Due to 
increasingly negative attention their antics were causing they rebranded as Kape Technologies.

### Five Eyes
All traffic flowing through a Five Eyes country should be considered to be scrutinized by the 
government and added to your profile.

### Fourteen Eyes
Similar to Five Eyes

## IVPN
* `󰄬` allows for the creation of completely anonymous accounts
* `󰄬` recent no logs audit

## Mullvad
Mullvad is the industry leader for innovation in VPN stealth and protection.

* `󰄬` Sweden has good privacy respecting laws
* `󰄬` allows for the creation of completely anonymous accounts
    * you could use a pre-paid VISA card you paid for with cash for added protection
* `󰄬` constantly innovating 
    * WireGuard Stealth
    * AI analysis protection with distortion
* `󰄬` recent no logs audit
* `󰄬` do not pay content creators to promote them or do affiliate links
* slower??

### Mullvad Configuration
* DAITA - enhances privacy but slows it down
* Multihop - enhances privacy but slows it down
* VPN Settings
  * Launch app on start-up
  * Auto-connect
  * Local network sharing
  * DNS content blockers
    * Ads
    * Trackers
    * Malware
    * Gambling
    * Adult content
    * Social media

### WireGuard Stealth
WireGuard never included anything for stealth in the protocol, but Mullvad has added a layer above 
WireGuard to provide obfuscation by wrapping it inside a HTTPS QUIC session.

## Proton VPN
* `󰄬` Switzerland has good privacy respecting laws
* `󰄬` has a large corporate office that `All Things Secured` visited
* `󰄬` recent no logs audit

### Handed over identity to Spanish law enforcement
"Guardia Civil sent legal requests through Swiss police to Wire and Proton, which are both based in 
Switzerland. The Guardia Civil requested any identifying information related to accounts on the two 
companies’ respective platforms. Wire responded providing the email address used to register the Wire 
account, which was a Protonmail address. Proton responded providing the recovery email for that 
Protonmail account, which was an iCloud email address, according to the documents."
from [TechCrunch Article](https://techcrunch.com/2024/05/08/encrypted-services-apple-proton-and-wire-helped-spanish-police-identify-activist/)

* Didn't hand over name of user because they don't have it
* Didn't hand over any email as its encrypted
* Did give recovery email address for account which was an Apple address and Apple did hand over their 
  identity
