# Synology Drive <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Synology drive is composed of three different apps:
* `Synology Drive` - used to see files in web interface
* `Synology ShareSync` - used to sync with another Synology
* `Synology Admin Console` - control configuration

Note: primary use case is to connect your phone or computer and not use the web interface except as a 
fallback emergency use.

### Quick links
* [.. up dir](..)
* [Install Synology Drive](#install-synology-drive)
* [Configure Synology Drive](#configure-synology-drive)
* [Install on android](#install-on-android)
* [Install on NixOS](#install-on-nixos)

## Install Synology Drive
1. Launch `Package Center`
2. Search for `Synology Drive Server` and click `Install`
5. Click through to finish

## Configure Synology Drive
The only folder that automatically shows up in Synology Drive is your home folder. Other folders will 
not unless you specifically add them via the admin console

Note: when enabling drive for folders it takes time for them to be indexed set up for syncying.

1. Add additional folders to Synology Drive
   1. Launch `Synology Drive Admin Console`
   2. Select `Team Folder` on the left
   3. Select the shared folders you'd like e.g. `Family` and `photo`
   4. Click `Enable` button at top
   5. Disable `Enable version control` since this is a performance hit and we have snapshots
   6. Finally click `Ok`

2. Disable versions for `My Drive (home)`
   1. Select `My Drive (home)`
   2. Click `Settings` button at top
   3. Disable `Enable version control` and click `Ok`

## Install on android
1. Search for `Synology Drive` in Google play and install
2. Enter your QuickConnect ID, Username and Password

## Install on NixOS
```nix
environment.systemPackages = [
  pkgs.synology-drive-client
];
```
