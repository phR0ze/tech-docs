# Nextcloud <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Nextcloud is a open-source self-hosted private cloud solution. The project aims to provide services 
on par with the public cloud conglomerates.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)
  * [Quick Portainer Deploy](#quick-portainer-deploy)
  * [First run setup](#first-run-setup)
  * [First impressions](#first-impressions)
* [NextCloud Apps](#nextcloud-apps)
  * [Dashboard](#dashboard)
  * [Files](#files)
  * [Nextcloud Office](#nextcloud-office)
  * [Whiteboard](#whiteboard)
  * [Cospend](#cospend)
  * [Cookbook](#cookbook)
  * [Memories](#memories)
  * [Recognize](#recognize)

## Overview
Nextcloud Hub is the core and provides multiple Nextcloud products: Files, Talk, Groupware, Office 
and Assistant into a single platform, optimizing the flow for collaboration.

**References**
* [linuxserver/nextcloud](https://docs.linuxserver.io/images/docker-nextcloud/)
* [Timezone list](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

### Quick Portainer Deploy
1. Select `Nextcloud` from your `Templates >Application` list
2. Ensure that `TZ` is set to a value e.g. `America/Boise`
3. Set a `DATABASE_PASSWORD` e.g. `insecure`
4. Set a `MYSQL_ROOT_PASSWORD` e.g. `insecure`
5. Set a `PORT` e.g. `443`
6. Click `Deploy the stack`

### First run setup
Navigate to your new Nextcloud instance e.g. `https://192.168.1.61:443`

1. Set your admin credentials
2. Setup the default DB connection
   1. Set `Database account` to `nextcloud`
   2. Set `Database password` to `insecure`
   3. Set `Database name` to `nextcloud_db`
   4. Set `Database host` to `localhost:3306` ???
      * Failed, but then worked with vm ip `192.168.1.61:3306` maybe, not sure
3. Had to skip default apps as didn't work

### First impressions
* Has `Dashboard`, `Files`, `Photos` and `Activity` apps installed by default

## Deploy Nextcloud
There are multiple ways of deploying Nextcloud. Deploying with Docker gives, I think, the most 
ability to control where the persistent data goes and the specific version to use independent of OS 
upgrades. Additionally it opens up newer versions that are not available in OS repos.

**References**
* [Nextcloud Admin docs](https://docs.nextcloud.com/server/latest/admin_manual/)
* [Nextcloud Installation docs](https://docs.nextcloud.com/server/latest/admin_manual/installation/index.html)
* [Deploy Nextcloud with Docker - Chris Grime](https://chrisgrime.medium.com/deploy-nextcloud-with-docker-compose-935a76a5eb78)

```yaml
services:
  nextcloud:
    image: nextcloud:32.0.0-apache
    container_name: nextcloud
    restart: unless-stopped
    networks: 
      - cloud
    depends_on:
      - nextclouddb
      - redis
    ports:
      - 8081:80
    volumes:
      - ./html:/var/www/html
      - ./custom_apps:/var/www/html/custom_apps
      - ./config:/var/www/html/config
      - ./data:/var/www/html/data
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=dbpassword
      - MYSQL_HOST=nextclouddb
      - REDIS_HOST=redis

  nextclouddb:
    # See compatibility matrix for Nextcloud 31
    # https://docs.nextcloud.com/server/31/admin_manual/installation/system_requirements.html
    image: mariadb:10.11.14
    image: mariadb
    container_name: nextcloud-db
    restart: unless-stopped
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    networks: 
      - cloud
    volumes:
      - ./nextclouddb:/var/lib/mysql
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles
      - MYSQL_RANDOM_ROOT_PASSWORD=true
      - MYSQL_PASSWORD=dbpassword
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      
  collabora:
    image: collabora/code
    container_name: collabora
    restart: unless-stopped
    networks: 
      - cloud
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles
      - password=password
      - username=nextcloud
      - domain=example.com
      - extra_params=--o:ssl.enable=true
    ports:
      - 9980:9980

  redis:
    image: redis:alpine
    container_name: redis
    volumes:
      - ./redis:/data  
    networks: 
      - cloud
```

### Domain Name
* Forward ports 80, 81, 443

### Volume Configuration
There are two separate volumes needed
* Data
* DB


## Nextcloud Apps
Under the user options there is a `+ Apps` where you can add additional apps

### Dashboard
Dashboard is a default application. You can download additional widgets for the dashboard
* I installed `Analytics` to see what they are like

### Files
Easy and secure document management and sharing with other people. It is deepling integrated with all 
the other applications.

* Teams are groups of users that common properties can be applied to
* Team folders allow for sharing files with the team
* Users can manage team folders directly not just administrators similar to Google
* Sharing can be done easily on the same server
* Sharing can be done easily between other Nextcloud servers as well via built in Federation

### Notes

### Imaginary
Provides previews for `heic`, `heif`, `illustrator`, `pdf`, `svg`, `tiff`, `webp` and more

### Chat and Talk

### Nextcloud Office

### Whiteboard

### Cospend

### Cookbook

### Memories
The project claims to be a fast, modern and advanced photo management suite. Memories works best when 
you also install a few helper apps: `Preview Generator`, `Recognize`, `Photos` the default app needed 
for albums support, and `Face Recognition`

Memories stores all its metadata in the DB and the files and configs on the mount points:
* `/portainer/Files/AppData/Config/Nextcloud/Data:/data`
* `/portainer/Files/AppData/Config/Nextcloud/Config:/config`
* Webdave is accessible `davs://192.168.1.61/remote.php/dav/files/$USER`

**References**
* [Memories docs](https://memories.gallery/install/)
* [Configuration for Memories](https://memories.gallery/config/)
* [Preview service](https://docs.nextcloud.com/server/latest/admin_manual/installation/server_tuning.html#previews)

1. I clicked the button to install and it seemed to give me an icon at the top but never did show 
   done installing or any progress
2. Clicking the icon start the app even though it didn't say it was installed
3. You need to then choose the root of your photos location
4. Configuration navigate to `Administration settings >Memories`
   1. Scroll down and enable file support for `Images (JPEG, PNG, GIF, BMP)`, `HEIC (Imagick)`, `TIFF 
      (Imagick)`, `Videos (ffmpeg)`
   2. Download the Geo Planet database (it will take 30min), use the console to run:
      ```bash
      $ occ memories:places-setup`
      ```

* Features
  * Timeline View
  * Public share URL
  * Auto / Manual tagging including Faces/Places/Objects
  * Map View
  * No Lock-In with Metadata
    * Updates EXIF headers so they will be available in other applications
    * Including captions, titles and descriptions
    * Bulk updates to dates and times
  * Simple editing, rotation, croping
  * Commenting using
    * Permissions are based on file groups
    * Create shared nextcloud folder
    * Share the folder with registered NC users
    * Create Albums in Memories
    * Add NC users as Collaborators in Album
    * Upload photos in Memories to NC Album and Memories Album
    * Collaborator then views Photos
  * Permissions
    * Based on Folders and Albums can span folders so gives a virtual view
    * If you then share the Album user will only see the photos in the Album that are in the folder they can see
* General UI observations
  * Clicking on a photos provides a nice modern transition zoom
  * Escape will close the current view
  * Default view is the timeline
  * Very slick modern feel to how settings are handled
  * Videos automatically play when clicked
  * Video player has speed options
* Why Not Photos app?
  * Photos provides slide show looping
  * Uploads don't prompt for tags or albums
    * Memories does
  * No time line view like memories
  * Map View opens separately while memories has is integrated
  * Showws all media from /root
  * No titles or descriptions at all
  * A little bit slower than Memories
* Mobile Apps
  * Android auto upload

### Recognize
Recognize needs to be configured in the administration settings before it will do anything

1. Navigate to `Administration settings >Recognize`
2. Flip the different enablement sliders
   1. `Enable face recognition`
   2. `Enable object recognition`
   3. `Enable landmark recognition`
3. Shell into the container in Portainer and run the following. It will look like it hung but will 
   complete eventually.
   ```bash
   $ occ recognize:download-models
   ```
