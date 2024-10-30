# Storage

### Quick links
* [Add Drive](#add-drive)
* [Clone Drive](#clone-drive)
* [Backup Drive](#backup-drive)
* [Securely Wipe Drive](#securely-wipe-drive)
* [RAID Drives](#raid-drives)
* [Test Drive](#test-drive)
* [Test USB Drive](#test-usb-drive)

## Add Drive
1. Get device names
   ```bash
   $ lsblk
   ```

2. Destroy any RAID information if reusing the drive
   ```bash
   # Destroy RAID info from the begining of the drive
   $ sudo dd if=/dev/zero of=/dev/sdX bs=512 count=2048

   # Destroy RAID info from the end of the drive
   $ sudo bash -c 'dd if=/dev/zero of=/dev/sdX bs=512 count=2048 seek=$((`blockdev --getsz /dev/sdX` - 2048))'
   ```

2. Partition the drive via `gdisk`
   ```bash
   $ sudo gdisk /dev/sdb
   # n to start create a new partition wizard
   # Accept default Partition number, hit enter
   # Accept defaults for First sector and Last sector
   # Accept default Hex code 8300 for Linux filesystem
   # w to write out the changes
   ```

3. To tell kernel about changes
   ```bash
   $ sudo partprobe /dev/sdb
   ```

4. Format drive for `ext4`
   ```bash
   $ sudo mkfs.ext4 /dev/sdb1
   ```

5. Tune the drive to use its full capacity
   ```bash
   # For storage only set reserved blocks which defaults to 5% to 0 as it is unneeded.  These
   # reserved blocks are only used as a security measure on boot disks to that system functions can
   # continue to operate correctly even if a user has stuffed the drive.
   $ sudo tune2fs -m 0 /dev/sdb1
   ```

6. [Automount using FSTAB](#add-automount-using-fstab)

## Clone Drive
```bash
# Kick off clone in one terminal
# conv=sync,noerror means in the case of an error ensure length of original data is preserved and don't fail
sudo dd if=/dev/sdX of=/dev/sdY bs=1M conv=sync,noerror

# Watch clone in another with
watch -n10 'sudo kill -USR1 $(pgrep ^dd)'
```

## Backup Drive
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

## Securely Wipe Drive
To securely shred all data on a drive you can use the shred tool:
* `-v` - verbose output
* `-z` - add a final pass of zeros to hide shredding
* `-n` - number of shred passes default to 3

```bash
sudo shred -vzn 3 --random-source=/dev/urandom /dev/sdX
```

## RAID Drives
The standard URE rate of 1 in 10^14 failure in modern drives has made RAID 5 an almost 100% fail with
larger drives. RAID 6 although tolerable will also be too high a risk with larger drives.  The only
option for RAID is RAID 10 if you value your data.  Otherwise forget RAID and make regular backups.
Once configured partition and format like any other drive.

## Test Drive
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

## Test USB Drive
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

<!-- 
vim: ts=2:sw=2:sts=2
-->
