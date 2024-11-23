# Tomato Router

## Wireless Client Mode
In wireless client mode you can connect your Tomator Router to any access point and then reshare that 
connection as a local LAN for any wired devices plugging into your router. In Wireless Client mode 
your Tomato Router's LAN will be on a different subnet from the parent wireless network so you can't 
connect devices between the two.

Note: by reconfiguring your Tomato Router's Wi-Fi service to be a client connecting to another router 
you also disable the ability for your router to share out Wi-Fi as a host.

1. Navigate to `Basic >Network >WAN Settings`
2. Ensure `Type` is set to `DHCP` to allow the host router to assign your client router an IP
3. Set `Wireless Client Mode` to 
