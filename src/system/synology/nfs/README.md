# NFS <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

WARNING: NFS works best when your uid and gid match those on the server and if they do all is well. 
If they don't you have to do all kinds of work arounds and kludges and its not worth it end of story.

### Quick links
* [.. up dir](..)

## Synology Configuration

### Enable NFS
Enable NFS fist as you'll need it for setting folder access

1. Navigate to `Control Panel > File Services > NFS`
2. Check the `Enable NFS service`

### Create folders
First you'll need to create the folders you'd like to expose over NFS.

1. Navigate to `Control Panel > Shared Folder`
2. Click `Create > Create Shared Folder`
3. Name your folder e.g. `Backup` 
4. Choose the volume location e.g. `Volume2: ext4`
5. Enable `Hide sub-folders and files from users without permissions`
6. Enable `Enable Recycle bin`
   1. Enable `Restrict access to administrators only`
7. Choose your `additional security measure` e.g. `Skip` and click `Next`
8. Choose user permissions for the folder

### Configre NFS Permissions
Once your folders are created and NFS is enabled you'll want to set NFS Permissions for your folders.

1. Navigate to `Control Panel > Shared Folder`
2. Select your folder then click `Edit` and the `NFS Permissions` tab
3. Click `Create`
4. Specify a network segment that has access e.g. `192.168.1.0/24`
5. Set `Privilege` e.g. `Read/Write`
6. Set `Squash` e.g. `No mapping`
7. Leave `Security` as `AUTH_SYS`
8. Leave `Enable asynchronous` enabled
9. Check `Allows users to access mounted subfolders`

* Note the `Mount path:` at the bottom e.g. `/volume2/Backup`

## Client Configuration
On the various NixOS systems you'd like to connect to your new NFS share add the following 
configuration where `server` is something like `192.168.1.5` and the `path` is the one from above 
e.g. `/volume2/Backup`.

```nix
{
  fileSystems."/mnt/Backup" = {
    device = "${nfs_host}:${nfs_path}";
    fsType = "nfs";
  };
}
```
