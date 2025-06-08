# Synology Photos <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Synology photos is a self hosted paid Synology product that can be installed on your Synology NAS. It 
comes with a Web interface as well as a mobile app.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Photos Review](#photos-review)
* [Configure Synology Photos](#configure-synology-photos)
  * [Install Synology Photos](#install-synology-photos)
  * [Enable Shared Space](#enable-shared-space)
  * [Grant family group access](#grant-family-group-access)
  * [Enable video thumbnails](#enable-video-thumbnails)
* [Mobile Configuration](#mobile-configuration)
  * [Mobile Backup](#mobile-backup)
* [Photo curation](#photo-curation)
  * [Questions and Answers](#questions-and-answers)
  * [Curation process](#curation-process)

## Overview
Installing Synology Photos creates two folders:
* `photo` which is a shared space
* `home/Photos` which is your personal photo space

### Photos Review
* Pros
  * Mobile app and web app for viewing photos
  * Nice features for sharing and requesting photos
  * Ability to backup from mobile app to server
  * Facial and object recognition is ok
* Cons
  * `Show stopper` - Video thumnails are missing in both mobile app and web app
  * `Annoying` - Facial recognition seems to confuse my children
  * `Unfortunate` - No live video previews of videos in mobile app or web app

* Mobile App
  *   Some noticable lag on occasion but usable
    * `` Immich is way faster for load and scrolling
  *    Video thumbnails don't show
    * `` Immich provides video thumbnails
  * `` Backups are allowed to the shared photos space
    *   Immich only provides the user space `/var/lib/immich/upload`
* Web App
  * `` Drag and drop upload
  * Facial Recognition
    * `` Shows selected faces
      * Synology does not provide this
    * `` Allows for hiding/ignoring people
    * `` Nice merging and updating features
      * Synology allows for reassigning image but only useful for single person images
* Server
  *   Original assets are uploaded to `/var/lib/immich/upload`
  *   Original assets are renamed to confusing machine only readable hashes

## Configure Synology Photos
Creates a top level `photo` folder in `Control Panel >Shared Folder` that can be used to easily share 
photos for the family.

### Install Synology Photos
1. Launch `Package Center`
2. Search for `Synology Photos` and click `Install`
3. Choose your main raid drive for your destination volume
4. Check the box to use this volume going forward for all apps
5. Click through to finish

### Enable Shared Space
1. Launch `Synology Photos`
2. Click through the intro you can enable those features later and click `Start Now`
3. Top right of the page click user icon then `Settings` then `Shared Space` tab
4. Click `Enable Shared Space` then `Save`

### Grant family group access
Grant `family` group access to new `photo` folder via `Synology Photos`.

**WARNING** `file level permissions` for mounted shares over the network and `Photos app permissions` 
are two separate permissioning systems. You need to configure both to make sure `Read Only` access is 
configured in both cases if that is your desire.

**NOTE** by removing the `Manager` and `Uploader` permissions below you also remove the ability to 
use the timeline view for the shared space :( which is super lame.

**File level permissions and app access**
1. Navigate to `Control Panel >User & Group >Group`
2. Select `family` then click `Edit`
3. Select the `Applications` tab
4. Scroll down to `Synology Photos` and check `Allow`

**Photos App permissions**
1. Launch `Synology Photos` then navigate to `Settings` via your icon in the top right
2. Click the `Shared Space` tab then click `Set Access Permissions` in the middle
3. Select `family` from drop down and `Custom`
   1. Uncheck `Manager` and `Uploader` options
   2. Click `Done`
4. Check `Enable Facial Recogniction in Shared Space`
5. Check `Enable Subject Recogniction in Shared Space`
6. Finally click `Save`

### Enable video thumbnails
Synology Photos doesn't generate video thumbnails or previews by default. You need to install 
Synology Video Station which will then generate video thumbnails.

## Mobile Configuration

### Mobile Install

1. Download and Install `Synology Photos`
2. Punch in your local IP address and port e.g. `192.168.1.5:5000`
3. Enter your user name and password
4. Uncheck `HTTPS` for now
5. Hit `Sign In`

### Mobile Backup
Backing up to your personal space and later curating the photos to then be migrated to the Shared 
space seems to be the best approach.

1. Navigate to the `More` menu
2. Select `Photo Backup Not Enabled`
2. Flip the switch to `Enable`
3. Under `Backup Rule`
   1. Select `Scan and back up all photos`
4. Under `Backup Path` select `Backup Destination`
   1. Select `Personal Space`
   2. Select `Default Folder`
   3. Ensure `Auto-create year/month folder` is set
5. Under `Backup Path` select `Backup Source`
   1. Select `Custom`
   2. Select `DCIM >Camera`
   2. Select `Screenshots`
6. Select `Upload Settings >Wi-Fi Only`

## Photo curation
I've reviewed a number of photo solutions and none of them seem to provide the exact photo 
organization that I want. However Synology's solution is the closest I've seen and can be useable 
with some careful planning. Namely I want to organize my photos by year with nested months and keep 
the original photo naming un-touched such that I can simply access them via file browser if I desire.

By backing up to the personal space for each user you can then manually curate them and migrate them 
to the shared space if so desired. This allows for a family collection without duplicating photos in 
both the personal and shared space while still allowing personal photos to remain personal as 
desired.

### Question and Answers
For a family its often desirable to move most photos from the Personal space to the Shared space. 
This allows for easy management and sharing in a central space while not duplicating photos in 
multiple locations.

* ***Operations performed by file manager*** on mounted share of Shared space:
  * What happens when they are deleted?
    * They are simply trimmed out of the Synology time line, no fuss
  * What happens when you copy photos to the share?
    * They are automatically indexed and show up in the time, no fuss
  * What happens when a photo is deleted from the Personal space?
    * It is removed from the Personal web view
    * It is removed from the mobile app view
    * It is not removed from mobile device
    * It is marked in the mobile app as being out of sync with an option to delete to sync with NAS

* ***Operations performed via mobile app***:
  * What happens when a photo is deleted from the Personal space?
    * Depending on app settings, it is deleted from device and from the Personal space on NAS

* ***Operations performed via the web app***:
  * What happens when a photo is deleted from the Personal space?
    * It is removed from the Personal space web view
    * The shared space copy is not affected
    * It is removed from the mobile app view, but is marked as out of sync in the more options which 
      then gives an option to delete it locally as well.
    * It is not removed from mobile device

### Curation process
1. Allow mobile backup to run as per usual which will dump it into the Personal user space on the NAS

2. Using a file manager to browse the mounted share locally:
   1. Move photos from the Personal space to the Shared space

3. Manually trigger NAS photos backup to ensure you have two copies
   1. Login to NAS
   2. Launch `Hyper Backup`
   3. Select `Photo Backup`
   4. Click `Back up now`

3. Sync mobile app with NAS to remove photos from mobile device
   1. Launch Synology Photos mobile app
   2. Hit the `More` hamburger menu at the bottom right
   3. Hit the `Out-of-sync changes` and review
   4. Hit the vertical elipse menu top right and select `Delete all`
