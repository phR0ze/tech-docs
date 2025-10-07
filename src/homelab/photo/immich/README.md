# Immich <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Immich is a self-hosted, open-source photo and video management solution. They aim to make backing 
up, organizing and managing photos on your own server easy and private.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
  * [Immich Review](#immich-review)
* [Setup Immich](#setup-immich)
  * [Install Immich on NixOS](#install-immich-on-nixos)
  * [First run experience](#first-run-experience)
  * [Configure Immich](#configure-immich)
  * [Wipe Immich](#wipe-immich)
* [Features](#features)
  * [External Libraries](#external-libraries)
  * [Folder View](#folder-view)
* [Correct Meta Data](#correct-meta-data)
  * [Missing Exif Date](#missing-exif-date)
* [Mobile App](#mobile-app)
* [Backup](#backup)
* [Upgrade](#upgrade)

## Overview

**References**
* [Immich Options](https://github.com/NixOS/nixpkgs/blob/nixos-unstable/nixos/modules/services/web-apps/immich.nix)
* [Immich on NixOS discussion](https://github.com/immich-app/immich/discussions/12920)

### Immich Review

**Notes**
* Longevity
  - fully open source
  - working on a non-interferring paid model
  - has modern flutter frontend but TypeScript backend, yikes
* Community
  - thriving community, largest i've seen
* Deployability
  - Portainer support out of box
  - Fully integrated with NixOS out of box
  - Documentation is fantastic
* Web UI
  - `` Drag and drop upload
  - Facial Recognition
    - `` Shows selected faces
    - `` Allows for hiding/ignoring people
    - `` Nice merging and updating features
* Mobile
  - Backup goes straight into the library without external option
    - Synology allows for custom backup location, while Immich does not
  - Very snappy, loads and scrolls quickly without lag
    *   Synology photos lags on initial load while the NAS comes out of sleep mode
    *   Synology has some lag when scrolling through viewing full sized images as well
  * `` Video thumbnails show
    *   Synology does not show thumbnails in mobile app for videos
  *   Backups are only allowed to `/var/lib/immich/upload`
* Server
  *   Original assets are uploaded to `/var/lib/immich/upload`
  *   Original assets are renamed to confusing machine only readable hashes

* Pros
  * Nice features for sharing and requesting photos?
  * Ability to backup from mobile app to server?
  * Facial and object recognition is ok?
  * Object search e.g. `cat on chair`, `koi fish pond`
  * Scrolls and views super fast on mobile phones
  * Loads faster on old hardware than NextCloud loads anything
  * Live transcoding streams of your videos as needed

## Setup Immich

**References**
* [Immich - NixOS](https://wiki.nixos.org/wiki/Immich)

### Install Immich on NixOS
Note: Immich depends on and will deploy as part of the service a `Postgres DB` and a `Redis Cache`.

```nix
services.immich = {
  enable = true;
  host = "0.0.0.0";
  openFirewall = true;
};

# Enable hardware accelerated video transcoding
users.users.immich.extraGroups = [ "video" "render" ];
```

### First run experience
1. Open your browser to `localhost:2283`
2. Setup you new account
3. Login with the new account
4. Walk through the setup wizard
   * Note: the `Storage Template` can be used to auto-organizing files based on filename patterns but 
   will make changes to your files for you

### Create a family user
My main purpose in experimenting with Immich is to simply expose the family photo collection locally 
in my home network to the family.

1. Navigate to the user icon and select `Administration`
2. Then click `Create user` near the top right
3. Fill out the shared account ***without concerns for security***
   1. Email e.g. `family@local`
   2. Set the `Password` something simple e.g. `family`
   3. Disable the `Require user to change password on first login`
   4. Set the `Name` to e.g. `Family`
   5. Click `Create`


### Configure Immich
Immich can consume a ton of space with its default settings approximately 12.5% extra for thumbnails 
and encoded video.

Launch the `Administration` options
1. Navigate to `Settings >Image Settings`
   1. Change `Thumbnail Settings >Resolution` to `200p`
   2. Change `Preview Settings >Format` to `WebP`
   3. Change `Preview Settings >Resolution` to `720p`
2. Navigate to `Settings >Job Settings`
   1. Change `Generate Thumbnails concurrancy` to `20`
3. Navigate to `Settings >Video Transcoding Settings`
   1. Change `Accepted video codecs` to check all
   2. Change `Encoding Options >Video Codec` to `av1`
   3. Change `Encoding Options >Audio Codec` to `mp3`
   4. Change `Transcode policy` to `Don't transcode any videos, may break playback on some clients`
   5. Change `Hardware Acceleration >Acceleration API` to `Quick Sync`
   6. Change `Hardware Acceleration >Hardware decoding` to `on`

### Wipe Immich
Follow the steps below to wipe immich

1. Stop the services
   ```bash
   $ sudo systemctl stop immich-server
   $ sudo systemctl stop immich-machine-learning
   ```

2. Drop the PostgreSQL database
   ```bash
   $ sudo -u postgres psql
   $ DROP DATABASE immich;
   $ \q
   $ CREATE DATABASE immich OWNER immich;
   $ \q
   $ sudo systemctl stop postgresql
   ```

3. Clear the Redis cache
   ```bash
   $ sudo redis-cli -s /var/run/redis-immich/redis.sock FLUSHALL
   $ sudo systemctl stop redis-immich
   ```

4. Remove all uploaded media
   ```bash
   $ sudo rm -rf /var/lib/immich/*
   ```

5. Start the services
   ```bash
   $ sudo systemctl start postgresql
   $ sudo systemctl start redis-immich
   $ sudo systemctl start immich-server
   $ sudo systemctl start immich-machine-learning
   ```

## Features

### External Libraries
External libraries track assets stored in the filesystem outside of Immich. When an external library 
is scanned, Immich will load videos and photos from disk and create the corresponding assets. These 
assets will then be shown in the main timeline, and they will look and behave like any other asset, 
including viewing on the map, adding to albums, etc... Later if a file is modified outside of Immich, 
you need to scan the library for the changes to show up. If an external asset is deleted from disk, 
Immich will move it to the trash on rescan. To restore the asset, you need to restore the original 
file. After 30 days the file will be removed from the trash and any metadata within Immich willb be 
lost.

**References**
* [Immich docs - external library](https://immich.app/docs/guides/external-library/)

Note: any changes to metadata for external libraries will only be available in Immich and will not be 
persisted in the asset file. If you move the asset to another location all custom metadata will be 
lost during the rescan and you will need to re-add it.

* By default all files in the import path will be added to the library.
* Exclusion glob patterns can be used to skip some files

**Configure new External Library**
1. First mount your photos as a read only share
2. Navigate to `Administration >External Libraries`
3. Click `Create Library`
4. Select the library owner user
5. Set import paths
   1. Select from the library menu `Edit Import Paths`
   2. Enter the path e.g. `/mnt/Pictures`
6. Set a library name
   1. Select from the library menu `Rename`
7. Click `Scan All Libraries`

### Folder View
An optional folder view can be enabled to add folder browing as a top level view. This is useful for 
highly organized libraries.

1. Navigate to `Account Settings >Features >Folders`
2. Toggle `Enable` and `Sidebar`
3. Click `Save` at the bottom of the section

## Storage Template
Immich allows the admin user to set the uploaded filename pattern at the directory and file level as 
well as the storage label for a user. When using a storage template all assets will be stored in
`/var/lib/immich/library`. When it is off they will be stored instead at `/var/lib/immich/upload`.

**References**
* [Storage Template - Immich docs](https://immich.app/docs/administration/storage-template/)

* Each user has a unique string id by default but can be overridden with a custom storage label
  * This value is used to store in `/var/lib/immich/library/<userID>`
  * You can find your ID and Storage Label in `Account Settings >Account > User ID`
* Behavior
  * Immich provides a set of variables that can be used in constructing the template along with 
  custom text
  * Files with the same name won't be overwritten as a sequence will be appended to them
* Default template
  * `Year/Year-Month-Day/Filename.Extension`

### Configure Storage Template
During the setup process or later in settings you can enable the `Storage Template` feature


## Correct Meta Data

### Missing Exif Date
Correcting missing Exif dates can be done with:

1. Using `exiftool` to find missing creation dates
   ```
   $ exiftool -r -if 'not $DateTimeOriginal' -filename /path/to/your/folder
   ```
2. Correct missing Exif creation date with `xnviewmp`
   1. Select the photos you want to change
   2. Right click and select `Metadata >Change timestamp...`

## Mobile App
The mobile app can be configured to sync 

### Mobile backup
* Original assets are uploaded to `/var/lib/immich/upload` and renamed to hashes
  * 

### General Experience
* Very snappy, loads and scroll quickly without lag
  * Synology photos lags on initial load while the NAS comes out of sleep mode
  * Synology has some lag when scrolling through viewing full sized images as well
* Video thumbnails show
  * Synology does not show thumbnails in mobile app for videos

## Backup
Immich provides a [Borg](https://www.borgbackup.org/) template to be used as a cron job to backup 
your database and photo/video library to a separate hard drive connected to your server.

**References**
* [Immich Backup and Restore - docs](https://immich.app/docs/administration/backup-and-restore)
* [Backup Borg template - docs](ohttps://immich.app/docs/guides/template-backup-script)

The NixOS deployment of Immich uses the following paths:
* `/run/secrets/immich` - file for extra environment variables to be loaded from disk
* `/var/lib/immich/backups` - stores daily automatic database backups
* `/var/lib/immich/encoded-video`
* `/var/lib/immich/library` - ?
* `/var/lib/immich/profile` - ?
* `/var/lib/immich/thumbs` - thumbnails and previews are generated and stored here for each upload
* `/var/lib/immich/upload` - original assets end up here unless using the storage template

### Disable Automatic Daily Backup
When testing or if you are using the Borg backup script you'll want to disable the automatic built-in 
database backups to save space as they won't be needed.

1. Navigate to `Administration >Settings >Backup Settings`
2. Toggle the `Enable database backups`
3. Click `Save`

## Upgrade

[Immich postgresql upgrade](https://wiki.nixos.org/wiki/Immich#Upgrading_pgvecto.rs)
