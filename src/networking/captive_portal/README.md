# Captive Portal <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](../README.md)

## NixOS
In order for Firefox to recognize and prompt you to load the captive portal login you need three 
things corrected.

1. Firefox needs the DNS over HTTPS feature disabled
2. You need to disable `resolvectl`'s DNSSEC features
3. Your WiFi link needs to have picked up the captive portal's DNS
