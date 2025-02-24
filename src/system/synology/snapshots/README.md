# Synology Snapshots <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Btrfs snapshots are a differential snapshot of changes. It only saves the delta so there is little 
wasted space. And the snapshots can be restored effortlessly and almost instantaneously.

### Quick links
* [.. up dir](..)
* [Install Snapshots](#install-snapshots)
* [Configure Snapshots](#configure-snapshots)

## Install Snapshots
1. Launch `Package Center`
2. Search for `Snapshot Replication` and click `Install`

## Configure Snapshots
The first time `Snapshot Replication` is launched it will warn about `Record File Access Time` being 
enabled and how to disable it to improve performance. Otherwise Btrfs will have to monitor sooo many 
more changes to files so just disable it.

1. Disable Access Time
   1. Launch `Storage Manager`
   2. Select your btrfs volume e.g. `Volume 1`
   3. In the volume section at bottom use the `...` menu and select `Settings`
   4. Under `Record File Access Time` change it to `Never` then click `Save`

2. Configure snapshots for each shared folder
   1. Launch `Snapshot Replication` and select the `Snapshots` section on left
   2. Shift select all supported shared folders and click `Settings`
   3. Select the `Schedule` tab
      1. Check `Enable snapshot schedule`
      4. Set `Select days` to `Daily`
      5. Set `Frequency` to `Every day`
   4. Select the `Retention` tab
      1. Check `Enable retention policy`
      2. Check `Advanced retention policy`
      3. Check `Keep all snapshots for` and set to `1 days`
      4. Check `Keep the latest snapshot of the hour for` and set to `24 hours`
      5. Check `Keep the latest snapshot of the day for` and set to `7 days`
      6. Check `Keep the latest snapshot of the week for` and set to `4 weeks`
      7. Leave `Numbers of latest snapshots to keep` at `5`
   5. Select the `Advanced` tab
      1. Check `Make snapshot visiable`

## Manual Snapshot
If you known you about to make a destructive change your worried about you can take a snapshot 
manually in preparation.

1. Launch `Snapshot Replication`
2. Select the Shared Folder you want to snapshot e.g. `photo`
3. Select `Snapshot >Take a Snapshot`
