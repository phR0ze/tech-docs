# TrueNAS <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

### Quick links
* [Overview](#overview)
  * [Install TrueNAS](#install-truenas)
* [ZFS](#zfs)

## Overview

### Install TrueNAS

1. Download the TrueNAS ISO
2. Login to Proxmox and upload it there
3. Create a new VM in Proxmox
   1. Select ISO from local storage
   2. Guest OS can stay `Linux` with default kernel
   3. Create a `32 GB` virtual hard disk for the ISO install
   4. Allocate `2` cores and `8 GB` memory minimum if using ZFS
4. Edit the Virtual Machine to pass through the physical disks
5. Boot from the ISO
6. Select the ISO install location
7. 

## ZFS


**References**
* [Understanding ZFS](https://arstechnica.com/information-technology/2020/05/zfs-101-understanding-zfs-storage-and-performance/)
* [ZFS vs RAID](https://arstechnica.com/gadgets/2020/05/zfs-versus-raid-eight-ironwolf-disks-two-filesystems-one-winner/)
