# Captive Portal <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](../README.md)

## NixOS
In order for Firefox to recognize and prompt you to load the captive portal login you need three 
things corrected.

1. Firefox needs the DNS over HTTPS feature disabled
2. You need to disable `resolvectl`'s DNSSEC features
3. Your WiFi link needs to have picked up the captive portal's DNS - if a global nameserver (e.g. a 
   pinned home resolver or `1.1.1.1`) is configured as the default route, it'll shadow the portal's 
   own local DNS server and the login page domain won't resolve at all. See 
   [Wildcard domain override](../dns/README.md#wildcard-domain-override) for the `resolvectl domain 
   <iface> "~."` fix and the permanent NixOS-side fix (don't force a global nameserver on roaming 
   devices, use `FallbackDNS=` instead).
