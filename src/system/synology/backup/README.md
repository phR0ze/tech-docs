# Backup <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Standard best practice is the 3-2-1 method:
* 3 copies of your data
* 2 different types of storage
* 1 off-site

### Quick links
* [.. up dir](..)
* [Synology Drive Client](#synology-drive-client)
  * [Create Backup Folder](#create-backup-target)
  * [Backup target files](#backup-target-files)
* [Hyper Backup](#hyper-backup)
  * [Backup folder](#backup-folder)
  * [External Backup Drive](#external-backup-drive)
* [Restoring](#restoring)
  * [Restoring a file](#restoring-a-file)

## Synology Drive Client
The Synology Drive Client can be used to create scheduled backup tasks to backup files on your 
computer to the Synology Drive server, as well as create versions of these files that can be restored 
or downloaded when needed.

### Create Backup Folder
In order to setup backups you'll first have to create a Shared folder on your main storage and then 
add it for management for Synology Drive.

1. First create a new shared folder named e.g. `Apps`
   1. Follow the directions here [Create shared folder](../README.md#create-shared-family-folder)

2. Create a sub directory to store your app data in
   1. From the Synology web app open `File Station`
   2. Navigate to `Apps` and create e.g. `OneUp`

3. Enable Synology Drive for the `Apps` folder
   1. From the Synology web app open `Synology Drive Admin Console`
   2. Select `Team Folder` on the left
   3. Select `Apps` on the right and click `Enable`
   4. Keep `Enable version control` and set `Maximum Versions` to `30`
   5. Click `OK`

### Install and configure drive client
[Install and configure the drive client](../drive/README.md#client-side)

### Backup target files
1. Launch the client as root `sudo synology-drive`
2. Login to the server
   1. Set `Synology NAS` to your server e.g. `192.168.1.5`
   2. Set `NAS login username` e.g. `Frodo`
   3. And set the corresponding password then click `Next`
3. Create the Backup task
   1. Choose `Backup Task`
   2. Select the desired folders to backup e.g. `/var/lib/oneup`
   3. Click the `Select` button and select the backup destination e.g. `/Apps`
    * Note: your machine name will be appended e.g. `/Apps/homelab` 
   4. Click 
4. Configure folder to sync
   1. Sync `/home` from server to your local machine. By default this will skip your `/home/Photos` 
      directory on the server but will select anything else in `/home`
   2. Choose a local directory on your machine as the other end of the sync

## Hyper Backup
Focus: Synology and its storage drives.

Synology offers their `Hyper Backup` application from the Package Center. Hyper Backup provides 
backup capabilities for your NAS to backup folders or the entire system to an external destination.

### Backup folder
Backups can be done per folder with their own scheduling.

1. Launch `Hyper Backup`
2. Choose `Folders and Packages`
3. Choose `Local Shared Folder or USB`
4. Choose `Multiple versions` this provides deduplication
5. Backup Destination Settings page
   1. Choose the `Shared Folder` destination e.g.`Backup`
   2. Name the destination `Directory` e.g. `Data`
   3. Click `Next`
6. Data Backup page
   1. Choose target data to backup e.g.`Volume 1 >Data`
   2. Choose target data to backup e.g.`Volume 1 >photo`
   3. Click `Next`
7. Appliation Backup page
   * Note: application backups duplicate a lot of other data, might be best to skip them
   1. Choose `File Station`
   2. Choose `Hyper Backup`
   3. Click `Next`
8. Backup Settings page
   1. Name it `Data Backup`
   2. Check the `Enable file change detail log` or you won't have much visibility into the backup
   3. Leave the default compression and integrity check on
   4. Leave `Run at` set to `Daily 3am`
   5. Click `Next`
9. Rotation Settings page
   1. Set `Max number of kept versions` to something like `60`
   2. Click `Next`
10. Finally you get the option to backup now

### External Backup Drive
Synology NAS supports external devices formatted with `Btrfs`, `FAT32` and `ext4` among others.

1. Plug your external backup device into the Synology NAS
2. Navigate to `Control Panel >External Devices >External Devices`
3. A folder will be created automatically named `usbshare[number]` for USB drives
4. The new folder will show up in the `File Station` app
5. Right clicking on the `usbshare1` folder will allow you to `Eject` it safely

## Restoring

### Restoring a file
1. Launch Hyper Backup
2. Find your file
3. Right click on it and select Restore
