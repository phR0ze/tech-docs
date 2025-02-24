# Synology Photos <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Synology photos is a self hosted paid Synology product that can be installed on your Synology NAS. It 
comes with a Web interface as well as a mobile app.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [Photos Review](#photos-review)
* [Install Synology Photos](#install-synology-photos)
* [Configure Synology Photos](#configure-synology-photos)

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


## Install Synology Photos
1. Launch `Package Center`
2. Search for `Synology Photos` and click `Install`
3. Choose your main raid drive for your destination volume
4. Check the box to use this volume going forward for all apps
5. Click through to finish

## Configure Synology Photos
Creates a top level `photo` folder in `Control Panel >Shared Folder` that can be used to easily share 
photos for the family.

1. Enable `Shared Space`
   1. Launch `Synology Photos`
   2. Click through the intro you can enable those features later and click `Start Now`
   3. Top right of the page click user icon then `Settings` then `Shared Space` tab
   4. Click `Enable Shared Space` then `Save`

2. Grant `family` group access to new `photo` folder via `Synology Photos`
   1. Navigate to `Control Panel >User & Group >Group`
   2. Select `family` then click `Edit`
   3. Select the `Applications` tab
   4. Scroll down to `Synology Photos` and check `Allow`

4. Complete `Shared Space` configuration
   1. Launch `Synology Photos` then navigate to `Settings` via your icon in the top right
   2. Click `Set Access Permissions` 
   3. Select `family` from drop down and `Full Access` then click the plus icon then `Save`
   4. Check `Enable Facial Recogniction in Shared Space`
   5. Check `Enable Subject Recogniction in Shared Space`
   6. Finally click `Save`

## Enable video thumbnails
Synology Video Station will generate video thumbnails

1. 
