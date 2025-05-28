# Synology <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
* [.. up dir](..)
* [Getting started](#getting-started)
  * [Configure groups and users](#configure-groups-and-users)
  * [Enable user home service](#enable-user-home-service)
* [Folder structure](#folder-structure)
  * [Create shared family folder](#create-shared-family-folder)
* [External Access](#external-access)
  * [QuickConnect](#quickconnect)

### Linked pages
* [Backup](backup/README.md)
* [NFS](nfs/README.md)
* [SMB](smb/README.md)
* [Synology Drive](drive/README.md)
* [Synology Photos](photos/README.md)
* [Synology Snapshots](snapshots/README.md)

## Getting started

### Configure groups and users
The first thing you'll want to do is configure groups and users.

1. Ensure `guest` and `admin` are disabled
   1. Navigate to `Control Panel >User & Group >User`
   2. Ensure that `guest` and `admin` are both set to `Deactivated`

2. Create a new `family` group which can in the future be used to add new members to
   1. Navigate to `Control Panel >User & Group >Group`
   2. Click `Create` and set the `Group Name` to `family`
   3. Add a desription and click `Next`
   4. Add any existing users you want to be part of it and click `Next`
   5. Skip through the rest

### Enable user home service
By enabling user homes each user will get space on the system to store their data, backup their 
computers or anything else they need.

1. Enable user home service
   1. Navigate to `Control Panel >User & Group`
   2. Select the `Advanced` tab
   3. At the bottom check `Enable user home service`
   4. Click `Apply`
2. Hide homes folder from local network
   1. Navigate to `Control Panel >Shared Folder`
   2. Select `homes` and click `Edit`
   3. Check `Hide this shared folder in "My Network Places"`
   4. Check `Hide sub-folders and files from users without permissions`
   5. Check `Enable Recycle Bin`

## Configure Volumes
The first thing you'll want to do is determine your volume storage layout. I have the `DS1522+` which 
is a 5 bay model. I've loaded (4) 14TB drives into bays 1,2,3 and 5. Bay 4 I'm leaving open for 
future expansion.

### Create Volumes
1. Launch `Storage Manager`
2. Build a `raid5` storage volume with `btrfs`
3. Build a `basic` backup volume with `ext4`

### Change RAID type
Synology NAS has a nice feature where you can start with a `Basic (Without data protection)` RAID 
type and later convert it into a different RAID type without data loss.

1. Backup anything critical, which is always a good practice
2. Ensure your Storage Pool Status is healthy and insert your new drives into the NAS
3. Launch `Storage Manager` and select your storage pool e.g. `Storage Pool 1`
4. Click the drop down `...` options button on your pool and select `Change RAID Type`
5. Select the new target type `RAID 5`, note the requirements and click `Next`
6. Select the additional drives needed e.g. `Drive 2` and `Drive 3` and click `Next`
7. Continue and check the `Expand` box then click `Next` and finally `Apply` and `OK`

Note: that your storage pool will transition through RAID 1 to get to RAID 5

### Expand RAID Storage
SHR, RAID 1, 5, 6, 10 and F1 can all be expanded.

**References**
* [Storage Pool Expansion](https://kb.synology.com/en-us/DSM/help/DSM/StorageManager/storage_pool_expand_replace_disk?version=6)

1. Backup anything critical. Always a good practice
2. Ensure your Storage Pool Status is healthy
3. Insert the new drives into your Synology NAS
4. Launch `Storage Manager`
5. Navigate to the target storage pool e.g. `Storage Pool 1` on the left

## Folder structure
Creating and sharing out a number of folders is the primary purpose of a NAS.

### Create shared family folder
It's nice to have a shared folder where the family can share documents easily

1. Navigate to `Control Panel >Shared Folder`
2. Click `Create >Create Shared Folder`
3. Name it `Family`
4. Check `Enable Recycle Bin` and sub option `Restrict access to administrators only`
5. Click through `Next` and check `Enable data checksum for advanced data integrity`
6. Finally add the `family` group with `Read/Write` and finish

### Create media folders
1. Navigate to `Control Panel >Shared Folder`
2. Click `Create >Create Shared Folder`
3. Name it `Movies`
4. Check `Enable Recycle Bin` and sub option `Restrict access to administrators only`
5. Click through `Next` and check `Enable data checksum for advanced data integrity`
6. Finally add the `family` group with `Read/Write` and finish

## External Access
Is this secure?

### QuickConnect
By default this will use a Synology Relay Server

1. Enable QuickConnect
   1. Navigate to `Control Panel >External Access >QuickConnect`
   2. Check the `Enable QuickConnect` option
   3. Choose a `QuickConnect ID` that is globally unique
   4. Only enable for photos access
      1. Click the `Advanced >Advanced Settings` button
      2. Remove permissions from the `DSM` option
      3. Click `Apply`
   5. Click `Save`
