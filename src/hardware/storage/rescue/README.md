# Rescue <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [TestDisk vs DDRescue](#testdisk-vs-ddrescue)
- [TestDisk](#testdisk)
  - [Restore Partition](#restore-partition)
- [Photorec](#photorec)
- [DDRescue](#ddrescue)
  - [Command line flags](#command-line-flags)
  - [Copy drive to image](#copy-drive-to-image)
  - [Mount raw image](#mount-raw-image)

## Overview

### TestDisk vs DDRescue
TestDisk was designed to work at the logical level i.e. repair partition issues while DDRescue was 
designed to work at the raw block level for damaged disks.

## TestDisk
[TestDisk](https://www.cgsecurity.org/wiki/Main_Page) is a command line utility for data recovery. If 
you have lost a partition or have strange problems with your hard disk partitions run `testdisk` to 
recovery your data.

Supports all common disk formats including: `NTFS`, `FAT32`, and `ext4`

**References**
* [TestDisk docs](https://www.cgsecurity.org/testdisk_doc/)

### Restore partition

1. Launch `testdisk` in interactive mode
   ```bash
   $ sudo testdisk
   ```
2. Hit enter to `Create`
3. Use the arrow keys to select the target drive e.g. `Disk /dev/sdb`
4. Hit enter to use the detected `EFI GPT` partition map
5. Hit enter on `Analyse`
6. Hit enter to `Quick Search`
6. Hit enter to `Continue`
8. `Write` partition structure to disk

## Photorec
PhotoRec is file data recovery software designed to recover lost files including video, documents and 
archives from hard disks (Mechanical Hard drives, Solid State Drives...), CD-ROMs, and lost pictures 
(thus the Photo Recovery name) from digital camera memory. PhotoRec ignores the file system and goes 
after the underlying data, so it will still work even if your media's file system has been severely 
damaged or reformatted. 

## DDRescue
GNU ddrescue is a data recovery tool. It copies data from one file or block device (hard disc, cdrom, 
etc) to another, trying to rescue the good parts first in case of read errors. You can run multiple 
times to read or fill in additional data.

### Command line flags
* `-d` - use direct disc access to bypass kernel and read drive directly avoids caching issues
* `-r1` - retry the data 1 time on failures

### Copy drive to image
Using DDRescue we can copy a failing drive to a device image that we can then work with. The log file 
can be used to try to recover missed data that it logs in the log file. This can be done in the case 
where the initial copy fails and you try again or in the case where it was successful but missed some 
of the data.

1. Determine your bad drive and umount if mounted
   ```bash
   $ lsblk
   $ sudo umount /run/media/$USER/714500e1-0b9c-419d-9dee-865ef98f168c
   ```
2. Mount your backup drive to copy to
   ```bash
   $ sudo mount /dev/sdb1 /mnt/storage1
   ```
3. Perform copy from bad drive to image on backup drive
   ```bash
   $ cd /mnt/storage1
   $ ddrescue -d -r1 /dev/sdd failed_drive.raw failed_drive.log
   ```

### Mount raw image
Raw images have more than one partition in the image. In order to mount the correct partition from 
the image we first have to discover where the image starts and mount with an offset to that location 
to get a working filesystem.

1. Determine where parition starts
   1. Start parted
      ```bash
      $ parted failed_drive.raw
      ```
   2. Enter `unit`
   3. Enter `B`
   4. Enter `print`
   5. Enter `quit`
      ```
      lksjdflkjsdlfk
      ```
2. Mount the partition offset in image
   ```bash
   $ mount -o loop,offset12345678 failed_drive.raw /mnt/temp
   ```

