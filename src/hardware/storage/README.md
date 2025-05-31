# Storage <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
- [.. up dir](../README.md)
* [Drives](#drives)
  * [Add Drive](#add-drive)
  * [Automount drive](#automount-drive)
  * [Clone Drive](#clone-drive)
  * [Backup Drive](#backup-drive)
  * [Securely Wipe Drive](#securely-wipe-drive)
  * [Test Drive](#test-drive)
  * [Test USB Drive](#test-usb-drive)
* [RAID Drives](#raid-drives)
  * [Destroy RAID info](#destroy-raid-info)
  * [Software RAID](#software-raid)

### Linked pages
- [Rescue](rescue/README.md)
- [Truenas](truenas/README.md)

## Drives

### Add Drive
1. Get device names
   ```bash
   $ lsblk
   ```
2. [Destroy any RAID information](#destroy-raid-info) if reusing the drive
3. Partition the drive via `gdisk`
   ```bash
   $ sudo gdisk /dev/sdb
   # n to start create a new partition wizard
   # Accept default Partition number, hit enter
   # Accept defaults for First sector and Last sector
   # Accept default Hex code 8300 for Linux filesystem
   # w to write out the changes
   ```
4. To tell kernel about changes
   ```bash
   $ sudo partprobe /dev/sdb
   ```
5. Format drive for `ext4`
   ```bash
   $ sudo mkfs.ext4 /dev/sdb1
   ```
6. Tune the drive to use its full capacity
   ```bash
   # For storage only set reserved blocks which defaults to 5% to 0 as it is unneeded.  These
   # reserved blocks are only used as a security measure on boot disks to that system functions can
   # continue to operate correctly even if a user has stuffed the drive.
   $ sudo tune2fs -m 0 /dev/sdb1
   ```
7. see [Automount drive](#automount-drive)

### Automount drive
Updated for NixOS

1. Find the ids of your storage drives
   1. List their details with
      ```bash
      $ lsblk -o +MODEL,SERIAL
      NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS MODEL         SERIAL
      sda      8:0    0  3.6T  0 disk             QEMU HARDDISK drive-scsi1
      └─sda1   8:1    0  3.6T  0 part
      ```
   2. List out disks by id `ll /dev/disk/by-uuid/`
      ```bash
      $ ll /dev/disk/by-uuid/
      lrwxrwxrwx 1 root root  10 Nov 14 14:32 17b2d3ba-7db5-4707-a346-dffbc3e27f97 -> ../../sdb3
      lrwxrwxrwx 1 root root  10 Nov 14 14:32 4f524d6b-c988-44ac-8179-4d12442e9404 -> ../../sda1
      lrwxrwxrwx 1 root root  10 Nov 14 14:32 4fc61204-a3d2-44ab-a93c-91ee61738b15 -> ../../sdb2
      ```
   3. Note the id linked to the target dev partition to mount e.g. `sda1`
      ```
      lrwxrwxrwx 1 root root  10 Nov 14 14:32 4f524d6b-c988-44ac-8179-4d12442e9404 -> ../../sda1
      ```
2. Edit your `/etc/nixos/hardware-configuration.nix` and add that id as a mount
   ```nix
   fileSystems."/mnt/cache" = {
     device = "/dev/disk/by-uuid/4f524d6b-c988-44ac-8179-4d12442e9404";
     fsType = "ext4";
   };
   ```
3. Apply the new configuration
   ```bash
   $ git add hardware-configuration.nix
   $ ./clu update system
   ```

### Clone Drive
conv=sync,noerror means in the case of an error ensure length of original data is preserved and don't fail

**Kick off clone in one terminal**
```bash
$ sudo dd if=/dev/sdX of=/dev/sdY bs=1M conv=sync,noerror
```

**Watch clone in another with**
```bash
$ watch -n10 'sudo kill -USR1 $(pgrep ^dd)'
```

### Backup Drive
One of the simplest and least expensive ways to backup your data is to buy a single hot swappable 
bay and a couple of large drives. Then just load one in and copy your data as desired then pull it 
out and store it in an anti-static/shock proof caddy.

1. Load your drive into the hot swappable bay
2. Determine the device path with `lsblk`
3. [Format the storage drive if needed](#add-drive)
4. Mount your device using `noatime` to speed up transfers a bit
   ```bash
   $ sudo mount -o defaults,noatime /dev/sdd1 /mnt/backup
   ```
5. Ensure the drive and all its data is owned by your user
   ```bash
   $ sudo chown USER: -R /mnt/backup
   ```
6. Launch `FileZilla` or your transfer app of choice and copy your data over

### Securely Wipe Drive
To securely shred all data on a drive you can use the shred tool:
* `-v` - verbose output
* `-z` - add a final pass of zeros to hide shredding
* `-n` - number of shred passes default to 3

```bash
sudo shred -vzn 3 --random-source=/dev/urandom /dev/sdX
```

### Test Drive
Using the SMART monitor tools and the built in diagnostics in drives we can determine their health.
SMART offers two different tests, according to specification type. Each of these tests can be
performed in two modes:

* ***Background Mode*** - the priority of the test is low, which means the normal instructions can
continue to be processed. If the drive is doing real work the test gets paused and compelted when
it's no longer busy.
* ***Forground Mode*** - all commands will be answered during the test with a `CHECK_CONDITION`
status i.e. you only want to use this option if the hard disk is unmounted.

The `Short Test` provides rapid identification of a defective hard drive. It only takes a max of
`2min` and checks the disk by looking at `Electrical Properties`, `Mechanical Properties` and
`Read/Verify`. The read verify spot check is usually a specific area as called out by the
manufacturer.

The `Long Test` is similar to the short test but also dos a `Read/Verify` on the entire disk, not
just the spot check done in the short test.

```bash
# Install smart monitor tools
$ sudo pacman -S smartmontools

# Check that drive is smart capable - check the last couple lines of output
$ sudo smartctl -i /dev/sda
...
SMART support is: Available - device has SMART capability.
SMART support is: Enabled

# Check the approximate time required to run the tests - look near bottom
$ sudo smartctl -c /dev/sda
...
Extended self-test routine
recommended polling time: 	 ( 333) minutes.
...

# Run short drive test in foreground
$ sudo smartctl -t short -C /dev/sda

# Run long test
# Note: captive forground doesn't seem to work
$ sudo smartctl -t long /dev/sda

# Show overall health check from tests once all tests have been run
$ sudo smartctl -H /dev/sda
...
=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED

# Show specific test results, the latest test is #1 and so on
$ sudo smartctl -l selftest /dev/sda
...
=== START OF READ SMART DATA SECTION ===
SMART Self-test log structure revision number 1
Num  Test_Description    Status                  Remaining  LifeTime(hours)  LBA_of_first_error
# 1  Short captive       Interrupted (host reset)      50%     30363         -
# 2  Extended offline    Completed without error       00%     30356         -
# 3  Extended offline    Aborted by host               90%         2         -
```

### Test USB Drive
1. Locate your drive with `lsblk`
   ```bash
   $ lsblk
   NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
   ...
   sdb      8:64   1   7.2G  0 disk 
   └─sdb1   8:65   1   3.8G  0 part 
   ```
2. Run `badblocks` against it
   ```bash
   $ sudo badblocks -w -s /dev/sdb
   ```
3. Test with f3
   ```bash
   # Install the f3 tools
   $ yay -Ga f3; cd f3; makepkg -s; sudo pacman -U f3-8.0-2-x86_64.pkg.tar.zst

   # Partition and format FAT32
   $ sudo wipefs --all --force /dev/sdd
   $ sudo sgdisk -n 0:0 -t 0:0700 -c 0:"flash" /dev/sdd
   $ sudo mkfs.vfat /dev/sdd1

   # Execute the tests on the automount location
   $ sudo f3write /run/media/USER/AB4C-86A3
   $ sudo f3read /run/media/USER/AB4C-86A3
   ```

## RAID Drives
The standard URE rate of 1 in 10^14 failure in modern drives has made RAID 5 an almost 100% fail with
larger drives. RAID 6 although tolerable will also be too high a risk with larger drives.  The only
option for traditional RAID is RAID 1 or 10.  Otherwise forget RAID and make regular backups. Beyond 
traditional means though there is ZFS and other NAS type solutions that overcome this issue.

### Destroy RAID info
RAID systems store RAID information at the begining and end of a drive. If you are reusing the drive 
for another purpose you'll need to destroy this information to avoid miss-identifying the drives.

```bash
# Destroy RAID info from the begining of the drive
$ sudo dd if=/dev/zero of=/dev/sdX bs=512 count=2048

# Destroy RAID info from the end of the drive
$ sudo bash -c 'dd if=/dev/zero of=/dev/sdX bs=512 count=2048 seek=$((`blockdev --getsz /dev/sdX` - 2048))'
```

### Build Software RAID
Linux provides support for Software RAID with the `mdadm` package.

### Migrate Software RAID
To move your software RAID to another server i.e. physically moving the drives then setting back up 
the RAID on the new server while keeping your data intact do the following:

**References**
* [NixOS mdadm](https://discourse.nixos.org/t/i-want-to-create-a-raid0-for-var-but-im-unable-to-figure-how-to-load-mdamd-on-boot/30381/4)

1. Store your current mdadm raid info on the original system
   ```bash
   $ sudo mdadm --detail --scan >> /etc/mdadm.conf
   ```

2. Migrate to the new system
   1. Physically install the drives in the new system
   2. Copy over the `/etc/mdadm.conf` to a temp dir

3. Assemble the RAID from the previously created array
   ```bash
   $ sudo mdadm --assemble
   ```

   ```nix
   boot.swraid.enable = true;
   boot.swraid.mdadmConf
   ```

<!-- 
vim: ts=2:sw=2:sts=2
-->
