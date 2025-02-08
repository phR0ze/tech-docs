# Systemd <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Systemd manages all services and processes. It uses units to control and manage services and 
processes. A unit is a collection of configurations and settings a service or process needs to 
function.

### Quick links
* [.. up dir](..)

## systemctl

### Listing out units
You can list out different unit types by simply leaving off any command and calling out the type. To 
get the full list simply run `systemctl` and you'll see all units in the system and their active 
state.

## Targets
A target unit is a collection of units that create a particular work environment, known as a system 
state. A target consists of two parts: the target unit file and the `wants` directory which contains 
references to all unit files that need to be loaded when launching a specific target.

Targets can have dependencies on other targets, which are defined in the target unit file. The target 
unit file has no specific options. It simply inherits all common options of unit configuration.

**References**
- [systemd-target - Freedesktop](https://www.freedesktop.org/software/systemd/man/latest/systemd.target.html)

### Dependencies
Units can setup a dependency on a target using the:
* `Wants=` a weaker version of Requires that will start the indicated target but has no impact on the 
  validity of the transaction as a whole. This is recommended approach in most cases.
* `Requires=` will start the indicated targets but if the indicated targets are deactivated or fail 
  this unit will also be deactivated and marked as failed.

### List all targets

```
$ systemctl --type=target
```

## Debugging

### Not loaded
Apparently I have a bad /etc/fstab configuration which is failing to generate and run the correct 
systemd units as the NixOS rebuild and switch fails with:
```
restarting sysinit-reactivation.target
Error: Failed to get unit mnt-Backup.mount

Caused by:
    Unit mnt-Backup.mount not loaded.
```

**General status gave:**
```bash
$ systemctl
UNIT                        LOAD        ACTIVE  SUB       DESCRIPTION
● mnt-Backup.automount      not-found   active  waiting   mnt-Backup.automount
```

Alternatively I was able to see it using `systemctl --type automount --all` which gave a loaded but 
inactive dead state. Running `systemctl status mnt-Backup.automount` gave 
```
mnt-Backup.automount
     Loaded: loaded (/etc/fstab; generated)
     Active: inactive (dead)
   Triggers: ● mnt-Backup.mount
      Where: /mnt/Backup
       Docs: man:fstab(5)
             man:systemd-fstab-generator(8)

Jan 29 08:58:29 workstation systemd[1]: Set up automount mnt-Backup.automount.
Jan 29 09:39:57 workstation systemd[1]: mnt-Backup.automount: Got automount request for /mnt/Backup, triggered by 8324 (ls)
Jan 29 13:30:29 workstation systemd[1]: mnt-Backup.automount: Got automount request for /mnt/Backup, triggered by 42261 (ls)
Jan 29 15:49:46 workstation systemd[1]: mnt-Backup.automount: Deactivated successfully.
Jan 29 15:49:46 workstation systemd[1]: Unset automount mnt-Backup.automount.
```
