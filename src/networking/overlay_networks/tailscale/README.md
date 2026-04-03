# Tailscale <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

***Tailscale*** and its competitors e.g. ***Twingate***, ***netbird***, etc... provide a mesh overlay 
network that allows you to connect to your target network resources anywhere in the world as if you 
were on the same network over an encrypted secure tunnel.
* Pros
  * Zero configuration
  * Built on WireGuard
  * Bypasses firewalls
* Cons
  * Not fully opensource

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Machines](#machines)
  - [Magic DNS](#magic-dns)
  - [User invites](#user-invites)
  - [Tags](#tags)
  - [Serve](#serve)
  - [Funnel](#funnel)
  - [Exit Node](#exit-node)
- [Configure tailnet](#configure-tailnet)
  - [Registration](#registration)
  - [Name your tailnet](#name-your-tailnet)
  - [Manually approve new devices](#manually-approve-new-devices)
  - [Create machine tags](#create-machine-tags)
- [Add machines to tailnet](#add-machines-to-tailnet)
  - [Add Linux device](#add-linux-device)
  - [Add Android device](#add-android-device)
  - [Add OSX device](#add-osx-device)
  - [Add Synology device](#add-synology-device)
- [SSH](#ssh)
  - [MacOS Keychain](#macos-keychain)
- [Proper SSL certs](#proper-ssl-certs)
- [Subnet router](#subnet-router)
  - [Enable IP forwarding](#enable-ip-forwarding)

## Overview

### Machines
Every device added to a tailnet, including servers, nodes, phones, and personal computers is assigned
a unique name generated from the device's OS and hostname. This name is displayed in the `Machines`
page of the web portal, but can be renamed as desired.

### Magic DNS
Magic DNS enabled by default allows for the use of the names listed in the `Machines` page of the web
portal intead of their IP addresses to interact with those machines.

### User invites
You can invites users to join your tailnet. There are two different kinds of tailnet invites.
1. `Team member invites` are for users who will authenticate using the same identity provider you used
   when creating the tailnet. Team members with email addresses with the same domain can log in
   without needing an invite first but invites work the same and provide a notification.
2. `External invites` are for those users who are not part of your custom domain, such as
   contractors, friends or family. To invite external users use the `Users` page and select `Invite
   external users` for options.

### Tags
[Tailscale tags](https://tailscale.com/kb/1068/tags) allow you to authenticate and identify non-user
devices, such as servers and ephemeral nodes. They server two primary purposes: to provide an
identity to non-user devicdes and to let you manage access control policies based on purpose. In this
context, a purpose could be anything from hosting a web server to serving as a subnet router.

**References**
* [Tailscale ACLs](https://www.youtube.com/watch?v=08clF9srJ2k)

* Applying a tag to a device removes any user-based authentication
* Each non-user device can have as many tags as you need
* Tags are defined in the tailnet policy file in the `tagOwners` section
* Tags are free and available to all pricing plans
* Use tags for devices that are not a user desktop or phone

### Serve
Serve gives you the ability to serve out files or service to anyone on your tailnet locally.

### Funnel
Funnel gives you the ability to serve out files or service to anyone publicly

### Exit Node
A node in your tailnet can be specified as an exit node. This means that any device in your tailnet 
can then choose to route all there outgoing traffic through that node to behave as if you were 
executing requests from that machine. Potentially useful for obfuscating your location or working 
around geolocation restrictions. Most useful though is probably the ability to then get access to the 
rest of the devices in your home network through an exit node in your home network.

## Configure tailnet
Personal use accounts only allow for a single tailnet, but this is usually sufficient with some
creative tagging and ACL configurations.

### Registration
Tailscale only offers SSO based accounts. For this example I'm using Github as my IdP.

1. Choose the `Sign up with Github` option
2. Click `Authorize tailscale`
3. Choose `Personal or At-Home Use`
4. Fill out the rest and click `Next: Add your first device`
5. Choose `Skip this introduction` down at the bottom

### Name your tailnet
You can change the tailnet's name, but only to randomly generated word combination.

***WARNING*** you'll want to do this first thing as it will break connections
1. Navigate to the `DNS` page
2. Select the `Rename tailnet...` option

### Manually approve new devices
Requiring manual admin approval before devices are allowed to join your tailnet is not enabled by
default but it should be.

1. Navigate to `Settings >Device management`
2. Toggle `Manually approve new devices`

### Create machine tags
Tags can be used to organize non-user devices like servers in to function groups to use in access
control polices.

1. Navigate in the web portal to the `Access controls` page
2. Select the `Tags` tab
3. Click the `+ Create tag` button
4. Set `Tag name` to e.g. `homelab`
5. Set `Tag owner` to your user
6. Click `Save tag`
   * this creates the tag `tag:homelab`

## Add machines to tailnet

### Add Linux device
There are actually two methods for generating keys in the web portal. The first via `Machines >Add
device >Linux server` isn't useful for NixOS as it will generate a key and a install script.

1. Generate a new authkey for the machine
  1. Precreate any tags you'll be using see [Create machine tags](#create-machine-tags)
  2. Navigate to in the web portal to the `Settings` page
  3. Click `Keys` section on the bottom left
  4. In the `Auth keys` subsection click the `Generate auth key...`
  5. Add a description e.g. `homelab`
  6. Toggle the `Tags` option at the bottom
  7. Add the tag we pre-created e.g. `tag:homelab`
  8. Click `Generate key`
  9. Copy the resulting key to some place safe and save it
2. Install on NixOS
  1. [see NixOS example](https://github.com/phR0ze/nixos-config/blob/main/options/services/raw/tailscale.nix)
3. Approve machine in web portal
  1. Navigate to the `Machines` page
  2. Click the `...` to the right of your machine entry
  3. Select `Approve`
4. Configure the machine to have no expiry for convenience
  1. Click the machine's `...` and choose `Disable key expiry`

### Add Android device
Using a QR code for sign in is the easiest way to go for a device capable of reading a QR code

1. Install Tailscale from the Google Play Store
2. Launch the app and hit `OK` for the connection request to setup a VPN
3. Choose your IdP and work through the login process
4. Once connected, switch to the Tailscale portal and approve the device
5. Configure the device to have no expiry click `... >Disable key expiry`
6. Change the device name as desired
7. Finally, back on the Android device, allow the tailscale notifications

### Add OSX device
1. Install the tailscale client on your device via brew
   ```bash
   $ brew install tailscale
   ```
2. Deploy the system service and validate it works
   1. Deploy via brew
      ```bash
      $ sudo brew services start tailscale
      ```
   2. Verify that tailscaled is running
      ```bash
      $ sudo brew services list
      ```
3. Connect to the tailnet and approve access in web portal
   1. Add a new `TS_AUTHKEY` env variable
      ```bash
      $ export TS_AUTHKEY=tskey-auth-kXXXXX-VXXXXXXXXXXXXXX
      ```
   2. Connect to the tailnet
      ```bash
      $ sudo tailscale up --ssh
      ```
   3. Approve access in the tailscale web portal
      1. Navigate to the `Machines`  tab
      2. To the right of the new machine entry click the `...` options menu
      3. Click `Approve`
4. Configure SSH access in the tailscale web portal
   1. Navigate to the `Access controls` tab
   2. Click the `JSON editor`
   3. Scroll to the `ssh` section in the policy and change to
      ```json
      {
        "ssh": [
          {
            "action": "accept",
            "src":    ["tag:your-from-system-tag"],
            "dst":    ["tag:your-mac-system-tag"],
            "users":  ["autogroup:nonroot", "root"]
          }
        ]
      }
      ```

### Add Synology device
see [../../../system/synology/tailscale](../../../system/synology/tailscale/README.md)

## SSH
Tailscale supports a built in SSH server that can be enable for the machine via Tailscale's
connection. Once this is enabled you can SSH into the target system without any need for passwords or
keys or a local SSH configuration. This is super convenient for systems where you don't want to
install SSH and/or configure, hardent or maintain it.

* Tailscale SSH can have additional ACLs configured to make it safer.
* Ephemeral SSH node can be added to your tailnet so you can SSH into your system via the web portal

### MacOS Keychain
Having to login to services that you are already logged into e.g. Claude Code CLI via SSH is a
classic macOS security behavior intersecting with how Tailscale's SSH works.

#### The Root Cause: Tailscale SSH vs. macOS Keychain
Claude Code CLI securely stores its authentication token in the macOS Keychain (specifically your
login keychain).

When you sit at your MacBook and log in, macOS uses your password to decrypt and unlock this keychain
automatically. However, when you connect remotely using Tailscale SSH, Tailscale authenticates you
using its control plane and cryptographic keys—not your macOS password.

Because macOS never received your password during the SSH handshake, your login keychain remains
locked for that specific terminal session. When Claude Code asks the system for your stored
Anthropic/Okta token, macOS denies access (or throws a silent UI prompt on your MacBook's physical
screen), causing the CLI to think you aren't logged in.

#### The Solution: Manually Unlock the Keychain
You simply need to unlock your keychain in your SSH session before running Claude.

 1. SSH into your MacBook via Tailscale as you normally do.
 2. Run the following command:
    ```bash
    $ security unlock-keychain ~/Library/Keychains/login.keychain-db
    ```
 3. It will prompt you for your password. Enter the password you use to log into your MacBook's lock screen.
 4. Once unlocked, run claude. It should now instantly recognize your existing enterprise login.

(Note: You will need to do this once per SSH session).

#### Reducing friction with TMUX
If you want to avoid typing your password every time you SSH in, you can piggyback off your MacBook's
local session using a terminal multiplexer:

 1. On the MacBook locally: Open your terminal and start a new tmux session by typing tmux.
 2. On your secondary machine: SSH into the MacBook via Tailscale and type tmux attach.

Because the tmux server was started within your local UI session, it inherits the permanently
unlocked keychain state. You can disconnect and reconnect via SSH all day, and Claude will always
remain authenticated inside that tmux session.

## Proper SSL certs
Most homelab users are going to want to access their self-hosted services when they are away from
their home network. Tailscale is the perfect solution for this personal use case.

**References**
* [Tailscale self-hosting guide](https://www.youtube.com/watch?v=guHoZ68N3XM&t=2s)

1. Install Tailscale

### Tailscale Certs via Lets Encrypt
Tailscale when using the `serve` option will automatically create and manage a TLS certificate for
your device. It takes a min the first time because its configuring all of this free throught Lets
Encrypt. After that when you connect to the device via the tailnet the TLS is being handled by
Tailscale transparently and gets rid of the HTTPS warnings.

## Configure a Subnet router
Subnet routers let you extend your tailnet to allow connections from participating tailnet machines to
other devices that are visible to tailnet machines on their local networks that are not part of the
tailnet. This can be really convenient when you simply want to establish a secure tunnel from the
outside world back into your home network to get access to your home devices.
* Pros
  * Easy access to your home network
  * Devices behind a subnet router don't count toward your pricing plan limit
* Cons
  * Relays home network traffic through the designated subnet router which might be a bottleneck
  depending on the amount of traffic.
  * Don't have end to end encryption and loose some configurability by not including edge devices

**References**
* [Tailscale Subnet routers](https://tailscale.com/kb/1019/subnets)

1. Install tailscale on the target machine
2. Enable IP forwarding and advertising of the subnet routes
3. 

### Enable IP forwarding

