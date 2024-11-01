# Wii Backup Manager <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Wii Backup Manager is used to convert backed up game images into a format that the Wii can read from 
disk.

### Quick links
* [.. up dir](..)
* [Install Wii Backup Manager](#install-wii-backup-manager)
  * [Create new 32bit Wine Prefix](#create-new-32bit-wine-prefix)

## Install Wii Backup Manager
Rated Gold on WineHQ for emulation on Linux

### Download Wii Backup Manger
1. Navigate to https://wiibackupmanager.co.uk/downloads.html 
2. Download build 78

### Create new 32bit Wine Prefix
1. Run `WINEARCH=win32 WINEPREFIX=~/.wine/prefixes/wiibm winecfg`

2. Under the `Applications` tab
   1. Set `Windows Version` to `Windows 7`

3. Under the `Graphics` tab
   1. Check `Emulate a virtual desktop`
   2. Set `Desktop size` to `1012x768`

4. Under the `Drives` tab
   1. Note the drive letter of your USB game drive

5. Click `Apply` then `Close`

### Install Wii Backup Manager into the prefix
Copy the application bits into the prefix
```bash
$ cp -r ~/Downloads/WiiBackupManager_Build78 ~/.wine/prefixes/wiibm/drive_c
```

## Run Wii Backup Manager
1. Run the app
   ```bash
   $ WINEARCH=win32 WINEPREFIX=~/.wine/prefixes/wiibm wine ~/.wine/prefixes/wiibm/drive_c/WiiBackupManager_Build78/WiiBackupManager_Win32.exe
   ```

2. Mount the USB Drive
   1. Navigate to `Options > Settings >Startup`
   2. Under `Drive 1` select 

<!-- 
vim: ts=2:sw=2:sts=2
-->
