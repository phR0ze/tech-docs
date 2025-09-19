# Synology Drive <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Synology drive is composed of three different apps:
* `Synology Drive` - used to see files in web interface
* `Synology ShareSync` - used to sync with another Synology
* `Synology Admin Console` - control configuration

Note: the primary use case for Synology Drive is to keep a copy of your files on your local device 
such as your phone or laptop that you can work on while offline and then have them sync to the server 
once back online. This does require though a full copy on both your local device as well as the 
server so if you are using the server as a means of providing more storage than is available on your 
local device then direct SMB would be a better option. That said if you have the storage then having 
additional local copy increases the safety factor of your data.

### Quick links
* [.. up dir](..)
* [Client Side](#client-side)
  * [Install on android](#install-on-android)
  * [Install on NixOS](#install-on-nixos)
* [Server Side](#server-side)
  * [Install](#install)
  * [Configure](#configure)

## Client Side

### Install on android
1. Search for `Synology Drive` in Google play and install
2. Enter your QuickConnect ID, Username and Password

### Configure on NixOS

1. Install
   ```nix
   environment.systemPackages = [
     pkgs.synology-drive-client
   ];
   ```
2. Launch the client `Network >Synology Drive Client`
3. Login to the server
   1. Set `Synology NAS` to your server e.g. `192.168.1.5`
   2. Set `NAS login username` e.g. `Frodo`
   3. And set the corresponding password then click `Next`
4. Choose how you want drive to behave
   1. `Sync Task` keeps your files in sync between you machine and the NAS, think Google Drive
   2. `Backup Task` is really meant to be scheduled to backup your files not for daily use
   3. In most cases choose `Sync Task` for daily use
5. Configure folder to sync
   1. Sync `/home` from server to your local machine. By default this will skip your `/home/Photos` 
      directory on the server but will select anything else in `/home`
   2. Choose a local directory on your machine as the other end of the sync

## Server Side
In order for Synology Drive to work you first have to install and configure it on your Synology 
device.

### Install Synology Drive
1. Launch `Package Center`
2. Search for `Synology Drive Server` and click `Install`
5. Click through to finish

### Configure Synology Drive
The only folder that automatically shows up in Synology Drive is your home folder. Other folders will 
not unless you specifically add them via the admin console

Note: when enabling drive for folders it takes time for them to be indexed and set up for syncying.

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


