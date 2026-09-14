# Quickemu <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Given the historical layers and mystery that surrounds the venerable QEMU its no wonder that tools 
like `Quickemu` have cropped up.  Quickemu is a wrapper around QEMU that claims that it ***does the 
right thing*** when creating virtual machines. If that is true this project is awesome as QEMU is 
shrouded in outdated guides and broken projects too numerous to count.

### Quick links
- [.. up dir](../README.md)
- [Getting Started](#getting-started)
  - [Install on NixOS](#install-on-nixos)
  - [Create a VM](#create-a-vm)
  - [Use a Local ISO](#use-a-local-iso)
  - [Create NixOS VM](#create-nixos-vm)
- [Usage](#usage)
  - [Display Options](#display-options)
  - [Screen Resolution](#screen-resolution)
  - [Reconnect Remote Viewer](#reconnect-remote-viewer)
  - [Disable Auto Viewer](#disable-auto-viewer)
  - [Shared Folders](#shared-folders)
  - [SSH Access](#ssh-access)
  - [Snapshots](#snapshots)
  - [Remove Installation Media Prompt](#remove-installation-media-prompt)
  - [BIOS support](#bios-support)
  - [Kill the VM](#kill-the-vm)
- [Review](#review)
  - [Example outputs](#example-outputs)
  - [Arco linux example](#arco-linux-example)

## Getting Started
Quickemu does one thing correct upfront and that is it stores configuration about your VM in a 
configuration file to then be referenced or overriden.

### Install on NixOS
Quickemu is packaged in nixpkgs as `quickemu`, which provides both the `quickemu` and `quickget`
binaries. It depends on QEMU being available, so on a NixOS host it's simplest to enable libvirt
or QEMU first (see [libvirt](../libvirt/README.md)) and then add `quickemu` on top.

**Installation is simple, just include the package**

```nix
environment.systemPackages = [ pkgs.quickemu ];
```

**or try it out ad-hoc without installing**

```bash
$ nix shell nixpkgs#quickemu
```

KVM acceleration requires your user be part of the `kvm` group, which is typically already the case
when `virtualisation.libvirtd.enable` or `programs.virt-manager.enable` is set.

### Create a VM
`quickget` allows you to choose an `operating system`, `release` and an `edition` as below. It will
download the ISO and create a configuration file to launch the VM for the ISO and prompt you with
what you should run to start.

1. Prepare a directory for you vm
   1. Choose a location for your vms
      ```bash
      cd ~/Projects/vms
      ```
   2. Create a vm dir
      ```bash
      mkdir ubuntu-server1
      ```
2. Determine the OS you want to build from
   * You can get a list of distros at the bottom of help
     ```bash
     quickget -h
     ```
   * or you can get a list all supported distros and versions (takes a min)
    ```bash
     quickget --list
     ```
3. Download the iso and generate the config
   1. Download your iso
      ```bash
      quickget ubuntu-server 24.04
      ```
   2. I find quickget's organization odd:
      * Remove the subdirectoy in the config
      * Move the iso to the top level of your directory
      * Rename the config to something generic that you'll run i like just `vm`
4. Now modify the config to customize the display and disable shared folder.
   ```conf
   #!/run/current-system/sw/bin/quickemu --vm
   guest_os="linux"
   disk_img="disk.qcow2"
   iso="ubuntu-24.04.5-live-server-amd64.iso"
   disk_size="10G"
   ram="2G"
   ssh_port="2222"
   display="spice"
   viewer="remote-viewer"
   public_dir="none"
   ```
2. The config has a shebang for `quickemu` and is executable so execute
   ```bash
   ./vm
   ```

### Use a Local ISO
`quickget` always downloads its own ISO, there's no flag to point it at one you already have.
`--create-config` gets you a working VM config from a local ISO without any of that, but be aware
of what it does and doesn't give you.

**References**
* [quickemu wiki - Manually create Linux guests](https://github.com/quickemu-project/quickemu/wiki/02-Create-Linux-virtual-machines)

1. Point `--create-config` at your ISO - it moves the file into a new `<name>/` directory and
   writes `<name>.conf` next to it
   ```bash
   quickget --create-config my-custom-ubuntu ~/Downloads/ubuntu-24.04.iso
   ```
2. Launch it like any other quickemu VM
   ```bash
   quickemu --vm my-custom-ubuntu.conf
   ```

**What you don't get**: the name you pass (`my-custom-ubuntu` above) is only used to name the
directory/config file - `--create-config` never looks it up against quickget's OS database. It
guesses `guest_os` from the filename alone (`windows`/`freebsd`/`reactos`/`kolibrios`, else plain
`linux`) and writes a bare `disk_size="16G"` with no `ram=`, `tpm=`, or `boot=` lines. Any per-distro
tuning quickget would normally apply - e.g. `disk_size="10G"`/`ram="4G"`/`tpm="on"` for
`ubuntu-server`, `disk_size="64G"`/`tpm="on"` for Windows 11 - is skipped entirely.

To recover it, run a throwaway `quickget <os> <release>` once, diff its `.conf` against yours, and
copy over any extra `disk_size`/`ram`/`tpm`/`boot` lines it added.

If you'd rather skip `--create-config` altogether, hand-write the same file yourself:
```bash
$ mkdir my-custom-ubuntu
$ mv ~/Downloads/ubuntu-24.04.iso my-custom-ubuntu/my-custom-ubuntu.iso
```
```bash
# my-custom-ubuntu.conf
guest_os="linux"
disk_img="my-custom-ubuntu/disk.qcow2"
iso="my-custom-ubuntu/my-custom-ubuntu.iso"
```

### Create NixOS VM
Quickemu can be used to quickly create a completely independent VM setup by booting the NixOS
minimal ISO and running a normal `nixos-install` inside the guest.

**References**
* [NixOS Server test with Quickemu](https://guekka.github.io/nixos-server-1/)

1. Fetch the NixOS minimal ISO and create the VM config
   ```bash
   $ quickget nixos 24.11 minimal
   ```
2. Boot the VM from the ISO
   ```bash
   $ quickemu --vm nixos-24.11-minimal.conf
   ```
3. Inside the guest, partition the virtual disk (`/dev/vda`) - a GPT layout with an EFI system
   partition, swap, and a single `btrfs` root
   ```bash
   $ parted /dev/vda -- mklabel gpt
   $ parted /dev/vda -- mkpart ESP fat32 1MiB 1GiB
   $ parted /dev/vda -- set 1 boot on
   $ parted /dev/vda -- mkpart Swap linux-swap 1GiB 9GiB
   $ parted /dev/vda -- mkpart primary btrfs 9GiB 100%
   ```
4. Format and mount the new partitions
   ```bash
   $ mkfs.vfat -n BOOT /dev/vda1
   $ mkswap -L swap /dev/vda2 && swapon /dev/vda2
   $ mkfs.btrfs -L nixos /dev/vda3

   $ mount /dev/vda3 /mnt
   $ mkdir /mnt/boot
   $ mount /dev/vda1 /mnt/boot
   ```
5. Generate the base configuration
   ```bash
   $ nixos-generate-config --root /mnt
   ```
6. Edit `/mnt/etc/nixos/configuration.nix` to set at minimum a hostname, a user account with a
   password/SSH key, and `services.openssh.enable = true;` if you want remote access to the VM.
7. Install and reboot into the new system
   ```bash
   $ nixos-install
   $ reboot
   ```

## Usage

### Display Options
Set with `display="..."` in the config. Default is `gtk` if unset.

| Value        | Where it renders                               | Notes |
| ------------ | ---------------------------------------------- | ----- |
| `gtk`        | Local window, embedded in the quickemu process | Default. Uses VirGL/GL acceleration when available. |
| `sdl`        | Local window, embedded, lighter than gtk       | No menu bar, still local/GL-capable. |
| `spice`      | Headless QEMU + SPICE server                   | Connect separately with `spicy`/`remote-viewer`. Enables clipboard, USB redirection, folder sharing. Required for [reconnecting](#reconnect-remote-viewer) later or [disabling the viewer](#disable-auto-viewer). |
| `spice-app`  | Same SPICE backend, viewer auto-launched       | Same as `spice` but quickemu starts the viewer for you. |
| `none`       | Fully headless, no display device at all       | Interact only via serial/SSH/monitor socket. |
| `cocoa`      | macOS native display                           | Not applicable on Linux hosts. |

Only `spice`/`spice-app` give you a detachable viewer - `gtk`/`sdl`/`none` are tied to the QEMU
process's own window (or lack of one).

### Screen Resolution
`width`/`height` set the guest's initial framebuffer resolution (default `1280x800` if unset):
```bash
width="1280"
height="1024"
```
This only affects the resolution QEMU hands the guest - it does **not** control the on-screen size
of the `spicy`/`remote-viewer` window itself, those are two separate things. To get a bigger local
window, either pass `--fullscreen` at launch (CLI only, not persistable in the config), or switch to
`viewer="remote-viewer"` and resize its window once - with a guest agent installed the guest
resolution then follows the client window size dynamically.

### Reconnect Remote Viewer
Closing the SPICE viewer window does **not** shut down the VM - with `display="spice"` the guest
runs headless behind a SPICE socket, and the viewer is just a client attached to it, same as closing
an RDP window doesn't power off the remote machine. Reconnect any time the VM is still running with:
```bash
cd ~/Projects/vms/ubuntu-server1
remote-viewer spice+unix://vm.sock
```

### Disable Auto Viewer
To have quickemu start the SPICE server without auto-launching a viewer window (useful if you
mostly SSH in and only occasionally want the graphical console), set:
```bash
viewer="none"
```
Then connect on demand with the same `remote-viewer spice+unix://vm.sock` command above.

### Shared Folders
`public_dir` exposes a single host directory to the guest over three transports simultaneously, this
isn't configurable per-transport - it's all or nothing:
```bash
public_dir=~/Downloads/temp
```
* **9P** (`virtio-9p-pci`) - works out of the box on modern Linux guests, mount manually:
  ```bash
  sudo mount -t 9p -o trans=virtio,version=9p2000.L,msize=104857600 Public-<user> ~/shared
  ```
* **WebDAV** (SPICE channel) - needs `spice-webdavd` installed in the guest to actually mount it.
* **Samba/CIFS** (usermode network `smb=`) - works from any guest with an SMB client, including
  Windows/macOS, no extra guest package needed for basic mounting.

For a Linux guest, 9P is normally the one to actually use - the other two just sit unused and cost
nothing if you never mount them. Set `public_dir="none"` to disable sharing entirely.

Tilde expansion only works if the value is unquoted (`public_dir=~/Downloads/temp`)

### SSH Access
Quickemu's default usermode networking forwards a host port to the guest's port 22 automatically,
no bridge or static IP needed:
```bash
ssh user@localhost -p 22220
```
The active port is recorded in `<name>.ports` in the VM directory. To pick your own port, set
`ssh_port` in the config:
```bash
ssh_port="2222"
```
The forward is baked into the QEMU command line at launch, so a reboot **inside** the guest won't
pick up the change - stop the whole VM at the host level and relaunch:
```bash
./vm --kill
./vm
```
OpenSSH server also needs to actually be installed/running in the guest (Ubuntu Server offers this
during setup; otherwise `sudo apt install openssh-server`).

### Snapshots
Quickemu wraps `qemu-img snapshot` directly on the disk image:
```bash
./vm --snapshot create <tag>   # take a snapshot
./vm --snapshot apply <tag>    # restore to a snapshot
./vm --snapshot delete <tag>   # remove a snapshot
./vm --snapshot info           # list existing snapshots
```
**The VM must be stopped first** (`./vm --kill`) - these operate directly on the qcow2 file rather
than through QEMU's live monitor, so running them against a running VM risks corrupting the disk.
These are disk-only snapshots (no RAM/CPU state, unlike a full VM suspend) - restoring one reverts
the disk contents, but the VM still boots fresh each time rather than resuming mid-session.

### Remove Installation Media Prompt
At the end of an OS install you'll often be told to "remove the installation medium, then press
ENTER" before it reboots. This is the installer's own prompt, not a bug - but since the ISO is
virtual there's no way to actually eject it mid-session. Pressing ENTER just soft-resets the *same*
running QEMU process with the ISO still attached exactly as it was at launch, which can loop back
into the installer/GRUB instead of booting your new install.

Quickemu decides whether to attach the ISO at all based on the disk image's file size at launch
time - once it's past a small "looks unused" threshold, quickemu drops the ISO from the command line
automatically on the *next* launch. So instead of trusting the in-guest reboot, stop the VM fully at
the host level and relaunch fresh:
```bash
./vm --kill
./vm
```

### BIOS support
Use `boot="legacy"` in your config to get BIOS support instead of the default UEFI. Be aware of a
gotcha though: Linux guests using `display="spice"` render through `virtio-gpu`, which has no legacy
VGA BIOS ROM - it only gets a framebuffer once either UEFI/OVMF hands it one via GOP, or the guest
kernel's own virtio-gpu driver loads. Combine `boot="legacy"` with `display="spice"` and you'll see
nothing at all - no BIOS POST, no GRUB - until deep into guest boot, which shows up in
`remote-viewer` as "Display output is not active".

If you need legacy BIOS boot, pair it with `display="sdl"` or `display="gtk"` instead - those use
`virtio-vga`, which is VGA-compatible and shows output from the very first frame. Note this also
means giving up the SPICE viewer, see [Display Options](#display-options).

### Kill the VM
If you just want to kill the VM and move on, run this from the VM directory:
```bash
./vm --kill
```
This is a hard stop (equivalent to pulling the power). If the guest OS is actually installed and
running, prefer a graceful shutdown from inside the guest first (e.g. `sudo poweroff` over SSH, or
`shutdown` from the console) so the filesystem unmounts cleanly.

## Review
The project claims the ability to:
* Easily create Windows 10 and 11 VMs with TPM 2.0
* Full SPICE support including host/guest clipboard sharing
* VirtIO-webdavd file sharing for Linux and Windows guests
* VirtIO-9p files sharing for Linux and macOS guests
* QEMU guest agent support
* VirGL acceleration
* USB device pass-through
* Network port forwarding
* Full duplex audio
* BIOS and UEFI support


### Example outputs
Gave a really nice dump of the VM confguration when running with `--display spice`
```
 - Host:     NixOS 25.05 (Warbler) running Linux 6.6.64 workstation
 - CPU:      Intel(R) Xeon(R) CPU E5-2637 v2 @ 3.50GHz
 - CPU VM:   host, 2 Socket(s), 4 Core(s), 2 Thread(s)
 - RAM VM:   16G RAM
 - BOOT:     EFI (Linux), OVMF (/nix/store/38hgw2w1sprb2vk81b22x0x0jx2cyf87-OVMF-202411-fd/FV/OVMF_CODE.fd), SecureBoot (off).
 - Disk:     arcolinux-v24.12.02-plasma/disk.qcow2 (16G)
             Just created, booting from arcolinux-v24.12.02-plasma/arcoplasma-v24.12.02-x86_64.iso
 - Boot ISO: arcolinux-v24.12.02-plasma/arcoplasma-v24.12.02-x86_64.iso
 - Display:  SPICE, virtio-gpu, GL (on), VirGL (off) @ (1280 x 800)
 - Sound:    intel-hda (hda-micro)
 - ssh:      On host:  ssh user@localhost -p 22220
 - SPICE:    On host:  spicy --title "arcolinux-v24.12.02-plasma" --port 5930 --spice-shared-dir /home/USER
 - WebDAV:   On guest: dav://localhost:9843/
 - 9P:       On guest: sudo mount -t 9p -o trans=virtio,version=9p2000.L,msize=104857600 Public-USER ~/USER
 - smbd:     On guest: smb://10.0.2.4/qemu
 - Network:  User (virtio-net)
 - Monitor:  On host:  socat -,echo=0,icanon=0 unix-connect:arcolinux-v24.12.02-plasma/arcolinux-v24.12.02-plasma-monitor.socket
 - Serial:   On host:  socat -,echo=0,icanon=0 unix-connect:arcolinux-v24.12.02-plasma/arcolinux-v24.12.02-plasma-serial.socket
 - Process:  Started arcolinux-v24.12.02-plasma.conf as arcolinux-v24.12.02-plasma (16514)
 - Viewer:   spicy --title "arcolinux-v24.12.02-plasma" --port "5930" --spice-shared-dir "/home/USER" "" >/dev/null 2>&1 &
```

### Arco linux example
Quickemu just worked for Arcolinux. It booted up with zero effort. Even if I don't use this for long 
running VMs it would definitely be nice for testing out other distros.

The process arguments ended up looking like:
```
  -global kvm-pit.lost_tick_policy=discard \
  -rtc base=localtime,clock=host,driftfix=slew \
  -pidfile arcolinux-v24.12.02-plasma/arcolinux-v24.12.02-plasma.pid \

  -vga none -device virtio-gpu,xres=1280,yres=800 -display none \
  -spice disable-ticketing=on,port=5930,addr=127.0.0.1 \
  -device virtio-serial-pci \
  -chardev socket,id=agent0,path=arcolinux-v24.12.02-plasma/arcolinux-v24.12.02-plasma-agent.sock,server=on,wait=off \
  -device virtserialport,chardev=agent0,name=org.qemu.guest_agent.0 \
  -chardev spicevmc,id=vdagent0,name=vdagent \
  -device virtserialport,chardev=vdagent0,name=com.redhat.spice.0 \
  -chardev spiceport,id=webdav0,name=org.spice-space.webdav.0 \
  -device virtserialport,chardev=webdav0,name=org.spice-space.webdav.0 \
  -device virtio-rng-pci,rng=rng0 \
  -object rng-random,id=rng0,filename=/dev/urandom \
  -device qemu-xhci,id=spicepass \
  -chardev spicevmc,id=usbredirchardev1,name=usbredir \
  -device usb-redir,chardev=usbredirchardev1,id=usbredirdev1 \
  -chardev spicevmc,id=usbredirchardev2,name=usbredir \
  -device usb-redir,chardev=usbredirchardev2,id=usbredirdev2 \
  -chardev spicevmc,id=usbredirchardev3,name=usbredir \
  -device usb-redir,chardev=usbredirchardev3,id=usbredirdev3 \
  -device pci-ohci,id=smartpass \
  -device usb-ccid \
  -chardev spicevmc,id=ccid,name=smartcard \
  -device ccid-card-passthru,chardev=ccid \
  -device usb-ehci,id=input -device usb-kbd,bus=input.0 \
  -k en-us \
  -device usb-tablet,bus=input.0 \
  -audiodev spice,id=audio0 \
  -device intel-hda \
  -device hda-micro,audiodev=audio0 \
  -device virtio-net,netdev=nic \
  -netdev user,hostname=arcolinux-v24.12.02-plasma,hostfwd=tcp::22220-:22,smb=/home/USER,id=nic \
  -global driver=cfi.pflash01,property=secure,value=on \
  -drive if=pflash,format=raw,unit=0,file=/nix/store/...-OVMF-202411-fd/FV/OVMF_CODE.fd,readonly=on \
  -drive if=pflash,format=raw,unit=1,file=arcolinux-v24.12.02-plasma/OVMF_VARS.fd \
  -drive media=cdrom,index=0,file=arcolinux-v24.12.02-plasma/arcoplasma-v24.12.02-x86_64.iso \
  -device virtio-blk-pci,drive=SystemDisk \
  -drive id=SystemDisk,if=none,format=qcow2,file=arcolinux-v24.12.02-plasma/disk.qcow2 \
  -fsdev local,id=fsdev0,path=/home/USER,security_model=mapped-xattr \
  -device virtio-9p-pci,fsdev=fsdev0,mount_tag=Public-USER-monitor \
  unix:arcolinux-v24.12.02-plasma/arcolinux-v24.12.02-plasma-monitor.socket,server,nowait \
  -serial unix:arcolinux-v24.12.02-plasma/arcolinux-v24.12.02-plasma-serial.socket,server,nowait
```

```
qemu-system-x86_64 -name ubuntu-24.10,process=ubuntu-24.10 \
  -machine q35,smm=off,vmport=off,accel=kvm \
  -global kvm-pit.lost_tick_policy=discard \
  -cpu host -smp cores=4,threads=2,sockets=2 \
  -m 16G -device virtio-balloon \
  -rtc base=localtime,clock=host,driftfix=slew \
  -pidfile ubuntu-24.10/ubuntu-24.10.pid \
  -vga none -device virtio-vga-gl,xres=1280,yres=800 \
  -display sdl,gl=on \
  -device virtio-rng-pci,rng=rng0 \
  -object rng-random,id=rng0,filename=/dev/urandom \
  -device qemu-xhci,id=spicepass \
  -chardev spicevmc,id=usbredirchardev1,name=usbredir \
  -device usb-redir,chardev=usbredirchardev1,id=usbredirdev1 \
  -chardev spicevmc,id=usbredirchardev2,name=usbredir \
  -device usb-redir,chardev=usbredirchardev2,id=usbredirdev2 \
  -chardev spicevmc,id=usbredirchardev3,name=usbredir \
  -device usb-redir,chardev=usbredirchardev3,id=usbredirdev3 \
  -device pci-ohci,id=smartpass \
  -device usb-ccid \
  -chardev spicevmc,id=ccid,name=smartcard \
  -device ccid-card-passthru,chardev=ccid \
  -device usb-ehci,id=input \
  -device usb-kbd,bus=input.0 -k en-us -device usb-tablet,bus=input.0 -audiodev pa,id=audio0 -device intel-hda -device hda-micro,audiodev=audio0 -device virtio-net,netdev=nic -netdev user,hostname=ubuntu-24.10,hostfwd=tcp::22220-:22,smb=/home/USER,id=nic -global driver=cfi.pflash01,property=secure,value=on -drive if=pflash,format=raw,unit=0,file=/nix/store/38hgw2w1sprb2vk81b22x0x0jx2cyf87-OVMF-202411-fd/FV/OVMF_CODE.fd,readonly=on -drive if=pflash,format=raw,unit=1,file=ubuntu-24.10/OVMF_VARS.fd -drive media=cdrom,index=0,file=ubuntu-24.10/ubuntu-24.10-desktop-amd64.iso -device virtio-blk-pci,drive=SystemDisk -drive id=SystemDisk,if=none,format=qcow2,file=ubuntu-24.10/disk.qcow2 -fsdev local,id=fsdev0,path=/home/USER,security_model=mapped-xattr -device virtio-9p-pci,fsdev=fsdev0,mount_tag=Public-USER -monitor unix:ubuntu-24.10/ubuntu-24.10-monitor.socket,server,nowait -serial unix:ubuntu-24.10/ubuntu-24.10-serial.socket,server,nowait
```

