# ChromeOS <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

General configuration reference for ChromeOS devices.

### Quick links
- [.. up dir](..)
- [System settings](#system-settings)
  - [Place holder](#place-holder)
- [Apps](#apps)
  - [Installing Android apps](#installing-android-apps)
    - [From the Play Store](#from-the-play-store)
    - [Sideloading an APK](#sideloading-an-apk)
- [Remote Desktop](#remote-desktop)
  - [Chrome Remote Desktop](#chrome-remote-desktop)
    - [Install Chrome Remote Desktop](#install-chrome-remote-desktop)
    - [Enable remote access on ChromeOS](#enable-remote-access-on-chromeos)
    - [Connect from a client](#connect-from-a-client)
- [Password Manager](#password-manager)
  - [Proton Authenticator](#proton-authenticator)
- [Troubleshooting](#troubleshooting)
  - [Powerwash](#powerwash)
  - [Hardware diagnostics](#hardware-diagnostics)

## System settings

### Place holder
?

## Apps

### Installing Android apps

#### From the Play Store
1. Open `Apps > Google Play Store`
2. Enable the Play Store if not already active
3. Install apps from the Play Store as on any Android device

#### Sideloading an APK
For apps not published on the Play Store, ChromeOS supports installing APKs directly without
enabling full developer mode.

1. Open `Settings > Apps > Google Play Store`
2. Click `Manage Android preferences`
3. Navigate to `Security > Unknown sources` (or `Special app access > Install unknown apps`)
4. Enable installs from the app you'll use to open the APK, e.g. `Files`
5. Download or copy the `.apk` file into the `Files` app
6. Open the `.apk` from `Files` and confirm the install prompt

## Remote Desktop

### Chrome Remote Desktop
Chrome Remote Desktop (CRD) uses a proprietary Google protocol over WebRTC. Once remote access is
enabled on a device, it can only be reached via a Chromium-based browser or the official CRD
mobile app; there is no third-party or open protocol client (e.g. RDP/VNC) that will connect to it.

#### Install Chrome Remote Desktop
1. Open Chrome and navigate to `remotedesktop.google.com/access`
2. Click the `Install` option at the top

#### Enable remote access on ChromeOS
1. Launch `Chrome Remote Desktop`
2. Click `Remote support` on the left
3. Click the download option on the large `Share this screen` section on the right
4. Add the `Chrome Remote Desktop` extension
5. Click `+ Generate Code`
6. Securely share the code with the remote support individual
7. Once they connect click the `Share` button to accept the remote support help

#### Connect from a client
Any device with a Chromium-based browser can act as a client.

1. Launch `Chromium`
2. Navigate to `remotedesktop.google.com/access`
3. Under the large `Connect to another computer` on the right enter the given pin
4. Wait for the remote client to click `Share` on their end
   ***Don't click on Remote Access*** as that will terminate your session

## Password Manager

### Duo Mobile
[Duo Mobile](https://duo.com/product/multi-factor-authentication-mfa/duo-mobile-app) is a free, open
source 2FA app from `Cisco Systems`
that syncs codes across devices via end-to-end encryption. No account required and fully
offline-capable.

1. Install `Proton Authenticator` from the Play Store, see [Installing Android apps](#installing-android-apps)
2. Launch the app and sign in with your Proton account to enable sync, or skip sign-in for local-only storage
3. Add a code
   1. Tap `+`
   2. Scan the QR code with the camera, or choose `Enter key manually` to paste a secret
4. Codes appear on the main screen and refresh automatically

## Troubleshooting

### Powerwash
Powerwash resets the device to factory state, removing all local data.

1. Open `Settings > Advanced > Reset settings`
2. Click `Reset` under `Powerwash`
3. Confirm and let the device reboot and reset

### Hardware diagnostics
1. Open `Settings > About ChromeOS > Diagnostics`
2. Review `CPU`, `Memory`, and `Battery` test results
3. Run individual tests under each tab as needed
