# Proxmox <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

Proxrox provides a really slick Hypervisor providing aggregation of Virtual Machine capabilities and 
community support to allow for building out almost anything super fast. It also supports LXC 
containers to reduce the overhead of the VM running.

**Final thoughts:** orginally I was fired up to use Proxmox as my virtualization service and was 
convinced it would solve all my utilization and maintenance woes. First impressions would good. Once 
I got further down the path though I realized that, although its a great virtualization system, I'm 
simply regressing back to my days prior to NixOS in which upgrading the base OS would end up being a 
a highly risky, nightmarish undertaking if a package or change slipped through Proxmox's testing 
which apparently has happened on a number of occasions. Additionally during my migration to Proxmox I 
found that some of my basic use cases were rather difficult to map into the Proxmox ecosystem. As a 
result I've decided to skip Proxmox, stay with NixOS as my base and instead look into virtualization 
managers like ***Incus*** or ***Portainer*** instead. This will provide the best of both worlds. I 
can still have my rock solid upgrade base with rollbacks if necessary while still allowing for the 
flexibility taking advantage of virtualization solutions.

### Quick link
* [Proxmox Overview](#proxmox-overview)
  * [Features](#features)
  * [Proxmox vs NAS](#proxmox-vs-nas)
  * [Potential NAS Hardware](#potential-nas-hardware)
  * [Install Proxmox](#install-proxmox)
* [Proxmox Configuration](#proxmox-configuration)
  * [First run configuration](#first-run-configuration)
  * [Disable enterprise package repository](#disable-enterprise-package-repository)
  * [Update Proxmox](#update-proxmox)
  * [Change hostname](#change-hostname)
* [Proxmox Storage](#proxmox-storage)
  * [Passthrough storage](#passthrough-storage)
  * [Add mount point](#add-mount-point)
  * [Local drive storage](#local-drive-storage)
* [Container Management](#container-management)
  * [Download container template](#download-container-template)
  * [Create container](#create-a-container)
  * [Delete container manually](#delete-container-manually)
* [Virtual Machine Management](#virtual-machine-management)
  * [Create VM](#create-vm)
* [Services on Proxmox](#services-on-proxmox)
  * [NFS Container](#nfs-container)

## Proxmox Overview
Virtual machine managers like Proxmox have spawned an alternate paradigm for homelab management. 
Traditionally you'd install a new application or Virtual Machine or even container on your 
workstation and interact directly with it on your workstation. However as your needs and 
sophistication increase you need to leave your workstation on all the time to keep your apps running.

The natural evoluation is to then shift the apps to a dedicated system in your homelab a.k.a the 
server. This is left running 24/7 to keep your apps up and you'd interact with it directly via 
keyboard, moust etc... As you manage your server you quickly realize that updating the server's OS 
can destabilize your applications. Creating and managing VMs becomes combersome. Utilization of 
resources is low. Your server has become highly complicated and difficult to manage.

This is where Proxmox, and the like, pick up. Proxmox allows you to have a rock solid base on which 
to build that can be updated as needed with little risk. It allows you to easily manage your Virtual 
Machines and Containers, Storage, Network etc... in a purpose built UI for the task that is available 
from any browser allowing for remote management. This new browser based paradigm has been picked up 
by individual applications as well allowing for containerized apps to be deployed with their 
individual web interfaces for management.

**References**
* [Replace Proxmox with Incus](https://tadeubento.com/2024/replace-proxmox-with-incus-lxd/)

### Features
* Slick GUI for organization
* Clustering
* Live Migration
* Failover
* Snapshots
* Ceph support
* Backup solution

### Virtualization competition
Proxmox caught on in the home lab community by making virtualization more approachable through their 
purpose built web interface. There are a number of competitors and new ones showing up every day it 
seems.

**Incus vs LXD**
Canonical is [now managing the LXD project](https://linuxiac.com/lxd-containers-project-goes-under-canonical-wing/).
This action generated mixed reactions within the open-source community [triggering a fork of LXD 5.16](https://linuxiac.com/incus-project-lxd-fork/) named `Incus` that is invisioned as being driven by the community rather than a commercial entity. 
Incus as of Aug. 7th, 2023 is an official member of the Linux Containers project.

[Incus has continued to mature](https://tadeubento.com/2024/replace-proxmox-with-incus-lxd/) providing a
unified experience for dealing with both LXC containers and VMs. Incus has put a lot of work into 
managing both types of virtualization in a similar fashion. It now comes with a web interface as 
well. It is purely open source and can be installed on any Debian system with little to no overhead 
and as of Debian 13 will be included in the repositories.

**Application containers vs System containers**
Docker is the defacto standard when is comes to application containers and focuses on an isolated 
container for a single application. System containers like LXC are instead more general purpose in 
that they can encapsulate manay applications even a full desktop environment like Ubuntu.

**TrueNAS**
Proxmox is a great Hypervisor that can host NAS virtual machines while NAS solutions like unRAID, 
TrueNAS or Synology are great NAS that can also do some Hypervisor work. The NAS based solutions tend 
to do a much better job of sharing out storage and offer nice container solutions but are not as 
featureful when it comes to VMs. However that isn't as much of a deterrant as it once was as more and 
more applications are now supporting the container approach. This is making Proxmox type VM first 
solutions less and less essential for home lab systems.

### Potential NAS hardware
* ASRock Rack
  * ASRock Rack X570D4U-2L2T
    * Built in internal Server management KVM via browser
    * HDMI port doesn't work
  * ICY Dock 6 Bay 2.5" Sata HDD/SSD Hot swap cage
  * Noctura NH-L9x65 SE-AM4 Premium Low-Profile CPU
  * AMD Ryzen 9 5950X 16-core, 32-thread
  * Crucial MX500 2TB 3D Nand Sata 2.5"
  * Kingston Server Premier 32 GB 3200MHz DDR4 ECC
  * Generic iStarUSA D-300-FS Chassis 3U rack
    * Power supplies don't fit
  * Samsung NVMe 970 Evo Plus
  * WD Black 2TB SN770 NVMe internal gaming ssd

* [UGREEN NASync DXP4800 Plus](https://www.ugreen.com/products/ugreen-nasync-dxp4800-plus-nas-storage) 
  * (1) NVMe drive for Proxmox
  * (2) NVMe 1TB drives as mirror pool for VM storage
  * (4) HDDs in ZFS RAID 5 for shares

### Install Proxmox
Proxmox is a full operating system that comes as a downloadable ISO

1. Download the latest ISO e.g. ***Proxmox VE 8.2 ISO Installer***
2. Burn the ~1.4GB ISO to an USB flash drive
3. Boot the server from the USB flash drive
4. Press Enter on the `Install Proxmox VE (Graphical)` option
5. Accept the EULA
6. Setup the install disk
   1. Choose from the `Target Harddisk:` drop down your drive
7. Choose your location and timezone settings
8. Enter your administrator password and email address
   * Note: Proxmox will send email notifications to that email
9. Setup your Management Network
   1. Choose your management nic
   2. Choose your hostname
   3. Choose you static IP
   4. Choose your gateway
   5. Choose your DNS server
10. You'll see a summary screen
11. Finally record the ip e.g. `https://172.16.249.75:8006`
12. Login via a browser to that address
13. Login with `root` and your chosen password
    * Note: leave the PAM authenticator option as is
14. Execute the [First run configuration](#first-run-configuration)

## Proxmox Configuration

### First run configuration
1. [Disable enterprise package repository](disable-enterprise-package-repository)
2. [Update proxmox](#update-proxmox)
3. Install support utilities
   1. ssh into proxmox e.g `ssh root@192.168.1.3`
   2. Run: `apt install vim`
4. Copy over basic utility configuration
   * `~/.bashrc`
   * `~/.dircolors`
   * `~/.vimrc`
5. Add [local drive storage](#local-drive-storage)

### Disable enterprise package repository
By default Proxmox comes preconfigured for the enterprise package repository which requires a 
subscription. Thus every time you update your system you get errors.

1. SSH into your Proxmox server e.g. `ssh root@192.168.1.3`
2. Edit `vi /etc/apt/sources.list.d/pve-enterprise.list` and comment out
   ```
   # deb https://enterprise.proxmox.com/debian/pve bookworm pve-enterprise 
   ```
3. Edit `vi /etc/apt/sources.list.d/ceph.list` and comment out
   ```
   # deb https://enterprise.proxmox.com/debian/ceph-quincy bookworm enterprise
   ```

### Update Proxmox
You can do this from the Web Interface or from the command line

**SSH command line**
1. SSH into your Proxmox server e.g. `ssh root@192.168.1.3`
2. Update package lists with: `apt-get update`
3. Upgrade with: `apt-get upgrade`
4. Reboot

**Web Interface**
1. Select your server e.g. `server3`
2. Select the `Updates` section then click the `Refresh` button to get a list of upgrades
3. Click the `>_ Upgrade` to get a shell and accept the list of upgrades
4. Finally click the `Reboot` button to reboot the server

### Enable IOMMU
This is the virtualization instruction set for your processor and motherboard to be able to work with 
virtual machines, passing PCI devices through and so forth.

1. Enable `VTx` and `VT-d` in your BIOS
   * On my HP z620 this was located under `Security >System Security`
2. Enable IOMMU support in Proxmox
   * Note: on Promox VE 8 and newer you don't need to manually do anything anymore

### Change hostname
**WARNING** I did this and it seemed to have broken my Containers

To [rename your server hostname](https://pve.proxmox.com/wiki/Renaming_a_PVE_node)

1. Login to proxmox
2. Select your server on the left
3. Select `>_ Shell` in the node options
4. Edit `vi /etc/hosts`
   ```
   192.168.1.3 proxmox.local proxmox
   ```
5. Edit `vi /etc/hostname`
   ```
   proxmox
   ```

## Proxmox Storage
Proxmox supports variety of storage solutions: ZFS, NFS, CIFS, GlusterFS, CephFS, LVM, LVM-thin and 
locally mounted storage via `/etc/fstab`. We can then make this storage available to VMs or 
containers or even pass devices directly through to the virtualized systems.

### Passthrough storage
Proxmox allows for passing through storage drives directly to the VM or container.

**References**
* [Proxmox passthrough - Wiki](https://pve.proxmox.com/wiki/Passthrough_Physical_Disk_to_Virtual_Machine_(VM))

**Find your target drive's uniqqu ID**
1. Find the drive's serial number
   ```bash
   $ lsblk -o +MODEL,SERIAL
   ```
2. Find the full drive path using the serial number
   ```bash
   $ ls /dev/disk/by-id
   ```

**Passthrough to Virtual Machine**
1. Use the VM's id to attach your physical drive to it via the console
   ```bash
   $ qm set 200 -scsi1 /dev/disk/by-id/ata-WDC_WD4000FYYZ-01UL1B2_WD-WMC130EAM0P3
   $ qm set 200 -scsi2 /dev/disk/by-id/ata-ST500DM002-1BD142_Z6EHX002
   ```
2. Refresh the UI to see the new drives added to the VM

**Passthrough to Container**
1. see [Passthrough storage](#passthrough-storage) to get its id
2. Select the container then `Resources` then click `Add >Device Passthrough`
3. Enter your device id e.g. `/dev/disk/by-id/ata-WDC_WD4000FYYZ-01UL1B2_WD-WMC130EAM0P3`

### Add mount point
Mount a arbitrary proxmox host path into your container using `pct`'s bindmount technology. These 
bindmounts are not managed by the Proxmox storage subsystem and thus don't allow for snapshots,
quotas or ACLs. Unprivileged containers won't have write priviledges to the mount by default.

**References**
* [Proxmox bind unprivileged lxc](https://www.itsembedded.com/sysadmin/proxmox_bind_unprivileged_lxc/)

In order to make your locally mount storage available via bindmounts to your containers you'll need 
to setup permissions on the storage directory to be read/write for the container user.

1. Shell into proxmox
2. Change ownership on the storage directory
   ```bash
   $ chown -R 1000:1000 /mnt/cache
   ```
3. Allow LXC daemon to use the desired uid/gid
   ```bash
   $ echo "root:1000:1" >> /etc/subuid
   $ echo "root:1000:1" >> /etc/subgid
   ```
4. Edit your container's configuration `vim /etc/pve/lxc/101.conf` and add
   ```
   mp0: /mnt/cache,mp=/mnt/cache
   lxc.idmap: u 0 100000 1000
   lxc.idmap: g 0 100000 1000
   lxc.idmap: u 1000 1000 1
   lxc.idmap: g 1000 1000 1
   lxc.idmap: u 1001 101000 64535
   lxc.idmap: g 1001 101000 64535
   ```

### Local drive storage
Proxmox supports mounting disks locally via `/etc/fstab` to then have directory stored defined for 
the given mount point. This will allow the use of any file system supported by Linux.

**References**
* [Directory Backend - Proxmox wiki](https://proxmox.local.to:8006/pve-docs/chapter-pvesm.html#_configuration)

1. SSH into your Proxmox server e.g. `ssh root@192.168.1.3`
2. Create mount point for storage disk e.g. `cache`
   ```bash
   $ mkdir /mnt/cache
   ```
3. Add your mount to the `/etc/fstab` file
   1. Find your device path
      ```bash
      $ ll /dev/disk/by-id
      ```
   2. Add it to your fstab
      ```bash
      $ echo "/dev/disk/by-id/ata-WDC_WD4000FYYZ-01UL1B2_WD-WMC130EAM0P3-part1" >> /etc/fstab
      ```
   3. Tweak in fstab
      ```
      /dev/disk/by-id/ata-WDC_WD4000FYYZ-01UL1B2_WD-WMC130EAM0P3-part1 /mnt/cache ext4 defaults 0 2
      ```
4. Mount it locally
   ```bash
   $ systemctl daemon-reload
   $ mount -a
   ```

Note: Proxmox `Directory` feature can be used to map your local drives into the UI to be used for 
various purposes. Each VM or Container can have a chunk you can allocate for their use. It literally 
creates an image in a specified size for the VM or Container to store data in. Although convenient 
for that VM or Container this is not a general purpose use case for sharing storage but a way to 
provide VMs or Containers with their own storage.
1. Login to proxmox UI
2. Navigate to `Datacenter >Storage` click `Add`
3. Set `ID:` to a descriptive name e.g. `cache`
4. Set `Directory:` to where it is mounted e.g. `/mnt/cache`
5. Set `Content:` to `Disk Image, Container` which VMs and Containers access to this directory
6. From the Container add a mount point

## Container Management
`pct` is proxmox's cli tool for working with containers

**References**
* [Proxmox Container Toolkit](https://pve.proxmox.com/pve-docs/chapter-pct.html)

* `pct list` - list containers
* `pct config ID` - list containers configuration

### Download container template
Note: anything newer than `ubuntu-22.04-standard` will fail with not supported

1. Navigate to `Storage >local >CT Templates`
2. Click the `Templates` button
3. Search for `ubuntu`
4. Select `ubuntu-22.04-standard`
5. Click `Download` then close the window when done

### Create container
Containers don't use much resource: 
* full throughput on 10 SFTP threads only consumed: 
  * `65% of 1 core`, `256 MiB RAM`, `800 MiB disk`

1. [Download container template](#download-container-template)
2. Click `Create CT`
3. Under `General` tab
   1. Set `VM ID` e.g. `101`
   2. Set `Name` to `share`
   3. Set a `Password` and `Confirm password` for root
4. Under `Template` tab
   1. Set the `Template` to `ubuntu-22.04-standard`
5. Under `Disks` tab
   1. Leave the default `local-lvm` selected
   2. Set `Disk size (GiB):` to `4` 
6. Under `CPU` tab
   1. Set `Cores` to `1`
7. Under `Memory` tab
   1. Set `Memory (MiB)` to `1024`
   1. Set `Swap (MiB)` to `512`
8. Under `Network` tab
   1. Choose `IPv4 >Static`
   2. Set `IPv4/CIDR` e.g. `192.168.1.50/24`
   3. Set `Gateway (IPv4)` e.g. `192.168.1.1`
9. Under `DNS` tab
   1. Set your preferred `DNS servers` e.g. `192.168.1.53`
10. Under `Confirm` tab
   1. Leave the `Start after created` unchecked
11. Add backports to the `sources.list`
   1. Edit `vim /etc/apt/sources.list` and add after `updates` line
      ```
      deb http://archive.ubuntu.com/ubuntu jammy-backports main restricted universe multiverse
      ```
12. Update the machine
   1. Run: `apt update && apt upgrade -y`
   2. Cleanup: `apt -y autoremove && apt -y autoclean`

### Delete container manually
1. Login to proxmox
2. Select your server on the left
3. Select `>_ Shell` in the node options
4. Run: `pct list` to list containers
5. Run: `pct destroy 100` where 100 is the container id
6. Manually delete `/etc/pve/lxc/100.conf`

## Virtual Machine Management

### Create VM
Note: not sure what went wrong but I installed my standard NixOS workstation ISO and ran a simple 
SFTP data test and found that the memory consumption would just grow until the VM crashed. I 
increased the CPU cores and RAM to insane amounts and the same test would crash the VM every time. 
Not reassuring, but my primary use case is for containers anyway. I'm hoping that my NixOS install 
must have been odd or something.

***Note:*** Setup below is using the NixOS image built from [github instructions by phR0ze](https://github.com/phR0ze/nixos-config?tab=readme-ov-file#build-the-live-iso-for-installation)
1. Click `Create VM` on the top right
2. Under `General` tab
   1. Set `VM ID` to `200` just to separate containers VM ranges
   2. Set `Name` to `share`
   3. Check the `Advanced` box at the bottom
   4. Check the `Start at boot` option that appeared
   5. Set the `Start/Shutdown order` to be `2` to allow our AdGuard instance to boot first
3. Under `OS` tab
   1. Set ISO Image to one you uploaded to the local drive
   2. Leave `Linux` and `6.x - 2.6 Kernel`
4. Under `System` tab
   1. Use all the defaults
5. Under `Disks` tab
   1. Set `Disk size (GiB):` to `40` 
6. Under `CPU` tab
   1. Set `Cores` to `2`
7. Under `Memory` tab
   1. Set `Memory (MiB)` to `16384`
8. Under `Network` tab
   1. Use all the defaults
9. Under `Confirm` tab
   1. Leave the `Start after created` unchecked
   2. Click finish
10. [Passthrough storage](#passthrough-storage)
11. Now we'll install the ISO
    1. On the VM options select `>_ Console` then `Start Now`
    2. Wait for the system to boot into the Live ISO
    3. Open a shell window to launch the installer
    4. Choose `generic >server`
    5. Walk through the wizard
    6. Set autologin when given the option
12. Once complete
    1. Power down the system
    2. Double click the `Hardware` options on the VM
    3. Double click the `CD/DVD` hardware entry
    4. Choose the `Do not use any media` option
    5. Click the `Start` button to boot back up

Note: I was able to have the memory freed up after running the kernel trigger below but I didn't test 
if this would stop it from crashing as manually running this every time the ram filled up is 
ridiculous.
```bash
sudo bash -c "echo 1 > /proc/sys/vm/drop_caches"
```

## Services on Proxmox
The community has put together a ton of helper scripts to simplify setting up services in Proxmox. 

**References**
* [Proxmox helper scripts - Github](https://github.com/community-scripts/ProxmoxVE)
* [Proxmox helper scripts - Website](https://community-scripts.github.io/ProxmoxVE/scripts?id=adguard)

### NFS Container
Purpose: provide a NFS data share for my network using Cockpit: `https://IP:9090`

**References**
* [Cockpit official plugins](https://cockpit-project.org/applications)
* [Cockpit install guide - youtube](https://www.youtube.com/watch?v=Hu3t8pcq8O0)
* [Apparmor discussion](https://forum.proxmox.com/threads/nfs-server-in-lxc.105073/)
* [Proxmox File System layout](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_file_system_layout)

Note: these directions are using LXC container ID `101`

1. Create container
   * see [Create container](#create-container)
   * ID: `101`, Name: `nfs`, Disk (GiB): `4`, Cores: `1`, RAM (MiB): `1014`, IPv4/CIDR: `192.168.1.50/24`
2. Add storage as mount point
   * see [Add mount point](#add-mount-point)
3. Allow container NFS kernel module permissions
  * Under Features check `Nesting` and `NFS`
   1. SSH into proxmox
   2. Create an apparmor profile
      1. Run: `cp /etc/apparmor.d/lxc/lxc-default-cgns /etc/apparmor.d/lxc/lxc-default-with-nfs`
      2. Edit the file and change the profile line:
         * from `profile lxc-container-default-cgns`
         * to `profile lxc-container-default-with-nfs`
      3. Edit the same file and add two more mounts inside the curly bracket
         ```
         mount fstype=nfs*,
         mount fstype=rpc_pipefs,
         ```
      4. Run: `systemctl reload apparmor`
   3. Use the new apparmor profile in your LXC container
      1. Edit `vim /etc/pve/lxc/101.conf` and add line
         ```
         lxc.apparmor.profile: lxc-container-default-with-nfs
         ```
      2. Finally ensure the container has `features: nesting=1` as well

   3. Add an appamor exclusion
      ```
      lxc.apparmor.profile: unconfined
      lxc.apparmor.profile: lxc-container-default-with-nfs
      lxc.apparmor.allow_nesting = 1
      lxc.apparmor.raw = deny mount

      lxc launch ubuntu:16.04 nfs -c security.privileged=true -c raw.apparmor="mount fstype=rpc_pipefs, mount fstype=nfsd,"

      # 22
      vim /etc/pve/nodes/NODENAME/lxc/VMID.conf

      # 3
      vim /var/lib/lxc/101/config

      lxc.apparmor.profile = lxc-container-default-with-nfs
      ```
3. Configure `ssh`
   1. Edit `vi /etc/ssh/sshd_config`
      ```
      PermitRootLogin yes
      PasswordAuthentication yes
      ```
   2. Reload system units: `systemctl daemon-reload`
   3. Start the ssh service: `systemctl start sshd`
4. Install Cockpit, as officially recommended, from most recent which is `backports`
   ```bash
   $ . /etc/os-release
   $ apt install -t ${VERSION_CODENAME}-backports cockpit --no-install-recommends -y
   ```
5. Install Cockpit plugins
   ```bash
   $ wget https://github.com/45Drives/cockpit-file-sharing/releases/download/v4.2.6/cockpit-file-sharing_4.2.6-1focal_all.deb
   $ wget https://github.com/45Drives/cockpit-identities/releases/download/v0.1.12/cockpit-identities_0.1.12-1focal_all.deb
   $ wget https://github.com/45Drives/cockpit-navigator/releases/download/v0.5.10/cockpit-navigator_0.5.10-1focal_all.deb
   $ apt install ./*.deb
   $ rm *.deb
   ```
6. Now we'll setup NFS shares
  1. Navigate to the web GUI e.g `https://192.168.1.50:9090`
  2. Login and selec the `File Sharing` option on the left
  3. Select `NFS` in the main window near the top
  4. Click the `+` button to the right to start creating a share
  5. Enter your mount point to share it e.g. `/mnt/cache/misc`
  6. Add a `Share Comment` e.g. `Misc file share`
  7. Restrict access to clients on e.g. `192.168.1.*`
  8. Set nfs settings:
    * Kodi seems to work best with `rw,sync,all_squash,insecure,no_subtree_check`
  9. Click `Apply`
7. Ensure the nfs server is up and running
   ```bash
   /proc/fs/nfsd: permission denied
   proc-fs-nfsd.mount: Mount process exited

   $ systemctl enable nfs-server
   $ systemctl start nfs-server
   ```
