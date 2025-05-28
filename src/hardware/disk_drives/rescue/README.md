# Rescue <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [TestDisk vs DDRescue](#testdisk-vs-ddrescue)
- [TestDisk](#testdisk)
  - [](#test-disk)

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

### 

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
etc) to another, trying to rescue the good parts first in case of read errors. 

DD treats block devices like a file which allows it to bypass certain issues.

