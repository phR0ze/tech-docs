# Quickemu <img style="margin: 6px 13px 0px 0px" align="left" src="../../../../data/images/logo_36x36.png" />

Given the historical layers and mystery that surrounds the venerable QEMU its no wonder that tools 
like `Quickemu` have cropped up.  Quickemu is a wrapper around QEMU that claims that it ***does the 
right thing*** when creating virtual machines. If that is true this project is awesome as QEMU is 
shrouded in outdated guides and broken projects too numerous to count.

### Quick links
- [.. up dir](../README.md)

## Overview
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

### Getting Started
`quickget` allows you to choose an `operating system`, a `release` and an `edition` as below. It will 
download the ISO and create a configuration file to launch the VM for the ISO and prompt you with 
what you should run to start.

1. Use quickget to download a target OS ISO
   ```bash
   $ quickget arcolinux v24.12.02 plasma
   ```
2. Using the outputted start command run
   ```bash
   $ quickemu --vm arcolinux-v24.12.02-plasma.conf
   ```

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

## Configuration
Quickemu does one thing correct upfront and that is it stores configuration about your VM in a 
configuration file to then be referenced or overriden.

