# Tailscale <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](..)
* [Secure access from anywhere](#secure-access-from-anywhere)
  * [Install Tailscale](#install-tailscale)
  * [Configure auto-update](#configure-auto-update)
  * [Configure subnet routing](#configure-subnet-routing)
  * [Configure TLS certificate](#configure-tls-certificate)

## Secure access from anywhere
Tailscale has a synology compatible app in the Synology app store

**References**
* [Tailscale guide for Synology](https://youtu.be/0o2EhK-QvmY)

### Install Tailscale
1. Navigate to the `Package Center` 
2. Search for `Tailscale` and click `Install`
3. Launch `Tailscale` from the app menu
   * This will open a browser tab and ask you to `Reauthenticate`
4. Click the `Reauthenticate` button and work through the wizard
5. Verify the generatged information is acceptable and click `Connect`
6. Click the `Open in admin console` then click `Approve`
7. Navigate in the admin console to `... >Disable key expiry`

### Configure auto-update
The tailscale version in the app store can be woefully out of date. So its a good idea to set up an
auto update script to sync with what is available upstream.

1. Navigate to `Control Panel >Task Scheduler`
2. Then `Create >Scheduled Task >User-defined script`
3. Set the task name e.g. `Tailscale update`
4. Change the user to `root`
5. Switch to the `Schedule` tab
6. Set it to once a week on a Sunday
7. Switch to the `Task Settings` tab
8. Add in the command `tailscale update --yes`
9. Click `OK` and click through
10. Finally run the action by clicking the `Run` button

### Configure subnet routing
Tailscale can be configured for your synology to use it as a subnet router into your local network.
This allows for connecting to any of your internal machines without them having to be on the
tailnet. This is super useful for IP cameras, routers or printers that can't join the tailnet
directly.

**References**
* [Tailscale subnet routers](https://tailscale.com/kb/1019/subnets)

1. Create a Task to set IP forwarding on boot
   1. Navigate to `Control Panel >Task Scheduler >Create >Triggered Task >User-defined script`
   2. Set the task name e.g. `Tailscale IP forward`
   3. Change the user to `root`
   4. Ensure `Event` is set to `Boot-up`
   5. Switch to the `Task Settings` tab
   6. Add in the command `sysctl -w net.ipv4.ip_forward=1`
   7. Click `OK` and click through
   8. Finally select the script and click the `Run` button

2. Enable tailscale subnet router function on your Synology
   1. SSH into your Synology
   2. Check that ip forward worked:
      ```bash
      $ sysctl net.ipv4.ip_forward
      ```
   3. Configure tailscale to serve as a subnet router for e.g. 192.168.1.0/24
     ```bash
     $ sudo tailscale up --advertise-routes=192.168.1.0/24 --reset
     ```

3. Configure your Synology node in the Tailscale admin portal
   1. Login to the Tailscale admin portal
   2. Switch the `Machines` and then select `... >Edit route settings...` for the synology entry
   3. Check the box to enable e.g. `192.168.1.0/24`
   3. Click `Save`

### Configure TLS certificate
Tailscale allows for configuring a free Let's Encrypt backed TLS certificate with the `ts.net`
domain.

1. Configure on the Tailscale portal side
   1. Login and navigate to the `DNS` tab
   2. Ensure that `MagicDNS` is enabled
   3. Enable `HTTPS Certificates`
      * ***WARNING*** the caveat here is that your machine names will get [published on the public
      ledger](https://tailscale.com/kb/1153/enabling-https#machine-names-in-the-public-ledger) for
      TLS certificates.
2. For each machine that needs a TLS cert run
   ```bash
   $ tailscale cert
   ```
3. Create a new scheduled task to renew the Let's Encrypt certificate
   1. Navigate to `Control Panel >Task Scheduler`
   2. Then `Create >Scheduled Task >User-defined script`
   3. Set the task name e.g. `Tailscale cert update`
   4. Change the user to `root`
   5. Switch to the `Schedule` tab
   6. Set it to `Monthly` then first e.g. `Sunday` of month
   7. Switch to the `Task Settings` tab
   8. Add in the command `tailscale configure synology-cert`
   9. Click `OK` and click through
   10. Finally run the action by clicking the `Run` button
      * Note: might take a few minutes to complete
4. Install the new certificate
   1. Navigate to `Control Panel >Security >Certificate`
   2. Select the new cert
   3. Click `Action >Edit`
   4. Check the `Set as default certificate`
   5. Click `OK`
     * Note: might have to delete the default self-signed Synology cert to make it work

### Node Sharing

### Access control policies
### Exit node
 
