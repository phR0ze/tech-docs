# Backup <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Standard best practice is the 3-2-1 method:
* 3 copies of your data
* 2 different types of storage
* 1 off-site

### Quick links
* [.. up dir](..)

## Hyper Backup
Synology offers their `Hyper Backup` application from the Package Center. Hyper Backup provides 
backup capabilities for your NAS to backup folders or the entire system to an external destination.

### Backup folder
Backups can be done per folder with their own scheduling.

1. Launch `Hyper Backup`
2. Choose `Folders and Packages`
3. Choose `Local Shared Folder or USB`
4. Choose `Multiple versions` this provides deduplication
5. Configure the Backup Destination Settings
   1. Choose the `Shared Folder` e.g.`Backup`
   2. Name the destination folder e.g. `Data`
   3. Click `Next`
6. Configure Data Backup
   1. Choose the `Volume 1 >Data` folder
   2. Choose the `Volume 1 >homes` folder
   3. Click `Next`
7. Configure Appliation Backup
   1. Choose `File Station`
   2. Choose `Hyper Backup`
   3. Click `Next`
8. Configure Backup Settings
   1. Name it `Data Backup`
   2. Check the `Enable file change detail log` or you won't have much visibility into the backup
   3. Leave the default compression and integrity check on
   4. Leave `Run at` set to `Daily 3am`
   5. Click `Next`
9. Configure Rotation Settings
   1. Set `Max number of kept versions` to something like `60`
   2. Click `Next`
10. Finally you get the option to backup now
