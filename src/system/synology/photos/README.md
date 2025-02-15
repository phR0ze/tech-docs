# Synology Photos <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Installing Synology Photos creates two folders:
* `photo` which is a shared space
* `home/Photos` which is your personal photo space

### Quick links
* [.. up dir](..)
* [Install Synology Photos](#install-synology-photos)
* [Configure Synology Photos](#configure-synology-photos)

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
