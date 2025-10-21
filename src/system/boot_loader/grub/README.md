# Grub <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

[GRUB2](https://www.gnu.org/software/grub) offers the ability to easily create a bootable USB drive
for both BIOS and UEFI systems as well as a customizable menu for arbitrary payloads. This
combination is ideal for a customizable initramfs based installer. Using GRUB2 we can boot on any
system with a custom splash screen and menus and then launch our initramfs installer which will
contain the tooling necessary to install the system. After which the initramfs installer will reboot
the system into the newly installed OS.

### Quick links
* [.. up dir](..)
* [Overview](#overview)
  * [grub-mkimage](#grub-mkimage)
  * [BIOS grub-mkimage](#bios-grub-mkimage)
  * [UEFI grub-mkimage](#uefi-grub-mkimage)
  * [GFXMenu module](#gfxmenu-module)
* [Trouble-shooting](#trouble-shooting)
  * [Keyboard not working](#keyboard-not-working)
  * [Incompatible license](#incompatible-license)

## Overview
GRUB is composed of a `kernel` which contains the fundamental features from memory allocation to
basic commands, the module loader and a simplistic rescue shell. The `modules` can be loaded by the
kernel to add functionality such as additional commands or support for various filesystems. The `core`
image which is constructed via `grub-mkimage` consists of the `kernel` the `specified modules` the
`prefix string` put together in a platform specific format.

Once GRUB is running the first thing it will do is try to load modules from the `prefix string`
location post fixed with the architecture e.g. `/boot/grub/x86_64-efi`. The modules included in the
core image are just enough to be able to load additional modules from the real filesystem usually
bios and filesystem modules.

**References**:
* [GRUB lower level](http://www.dolda2000.com/~fredrik/doc/grub2)
* [GRUB Documentation](https://www.gnu.org/software/grub/manual/grub/html_node/index.html)
* [GRUB Developers Manual](https://www.gnu.org/software/grub/manual/grub-dev/html_node/index.html)
* [GFXMenu Components](https://www.gnu.org/software/grub/manual/grub-dev/html_node/GUI-Components.html#GUI-Components)

### grub-mkimage
***grub-mkimage*** is the key to building GRUB bootable systems. All of GRUB's higher level utilities
like `grub-[install,mkstandalone,mkresuce]` all use `grub-mkimage` to do their work.

Resources:
* [GRUB on ISO](https://sites.google.com/site/grubefikiss/grub-on-iso-image)
* [GRUB image descriptions](https://www.gnu.org/software/grub/manual/grub/html_node/Images.html)
* [grub-mkstandalone](https://willhaley.com/blog/custom-debian-live-environment-grub-only/#create-bootable-isocd)
* [GRUB2 bootable ISO with xorriso](http://lukeluo.blogspot.com/2013/06/grub-how-to-2-make-boot-able-iso-with.html)

Essential Options:
* `-c, --config=FILE` is only required if your not using the default `/boot/grub/grub.cfg`
* `-O, --format=FORMAT` calls out the platform format e.g. `i386-pc` or `x86_64-efi`
* `-o DESTINATION` output destination for the core image being built e.g. `/efi/boot/bootx64.efi`
* `-d MODULES_PATH` location to modules during construction defaults to `/usr/lib/grub/<platform>`
* `-p /boot/grub` directory to find grub once booted i.e. prefix directory
* `space delimeted modules` list of modules to embedded in the core image
  * `i386-pc` minimal are `biosdisk part_msdos fat`

### BIOS grub-mkimage

```bash
local shared_modules="iso9660 part_gpt ext2"

# Stage the grub modules
# GRUB doesn't have a stable binary ABI so modules from one version can't be used with another one
# and will cause failures so we need to remove then all in advance
cp -r /usr/lib/grub/i386-pc iso/boot/grub
rm -f iso/boot/grub/i386-pc/*.img

# We need to create our core image i.e bios.img that contains just enough code to find the grub
# configuration and grub modules in /boot/grub/i386-pc directory
# -p /boot/grub                 Directory to find grub once booted, default is /boot/grub
# -c /boot/grub/grub.cfg        Location of the config to use, default is /boot/grub/grub.cfg
# -d /usr/lib/grub/i386-pc      Use resources from this location when building the boot image
# -o temp/bios.img              Output destination
grub-mkimage --format i386-pc -d /usr/lib/grub/i386-pc -p /boot/grub \
  -o temp/bios.img biosdisk ${shared_modules}

echo -e ":: Concatenate cdboot.img to bios.img to create CD-ROM bootable eltorito.img..."
cat /usr/lib/grub/i386-pc/cdboot.img temp/bios.img" > iso/boot/grub/i386-pc/eltorito.img
```

#### UEFI grub-mkimage
The key to making a bootable UEFI USB is to embedded the grub `BOOTX64.EFI` boot image inside an
official `ESP`, i.e. FAT32 formatted file, at `/EFI/BOOT/BOOTX64.EFI` then pass the resulting
`efi.img` to xorriso using the `-isohybrid-gpt-basdat` flag.

Resources:
* [UEFI only bootable USB](https://askubuntu.com/questions/1110651/how-to-produce-an-iso-image-that-boots-only-on-uefi)

**xorriso UEFI bootable settings**
```bash
-eltorito-alt-boot               `# Separates BIOS settings from UEFI settings` \
-e boot/grub/efi.img             `# EFI boot image on the iso filesystem` \
-no-emul-boot                    `# Image is not emulating floppy mode` \
-isohybrid-gpt-basdat            `# Announces efi.img is FAT GPT i.e. ESP` \
```

**ESP creation including grub-mkimage bootable image**
```bash
mkdir -p iso/EFI/BOOT

# Stage the grub modules
# GRUB doesn't have a stble binary ABI so modules from one version can't be used with another one
# and will cause failures so we need to remove then all in advance
cp -r /usr/lib/grub/x86_64-efi iso/boot/grub
rm -f iso/grub/x86_64-efi/*.img

# We need to create our core image i.e. BOOTx64.EFI that contains just enough code to find the grub
# configuration and grub modules in /boot/grub/x86_64-efi directory.
# -p /boot/grub                   Directory to find grub once booted, default is /boot/grub
# -c /boot/grub/grub.cfg          Location of the config to use, default is /boot/grub/grub.cfg
# -d /usr/lib/grub/x86_64-efi     Use resources from this location when building the boot image
# -o iso/EFI/BOOT/BOOTX64.EFI     Using wellknown EFI location for fallback compatibility
grub-mkimage --format x86_64-efi -d /usr/lib/grub/x86_64-efi -p /boot/grub \
  -o iso/EFI/BOOT/BOOTX64.EFI fat efi_gop efi_uga ${shared_modules}

echo -e ":: Creating ESP with the BOOTX64.EFI binary"
truncate -s 8M iso/boot/grub/efi.img
mkfs.vfat iso/boot/grub/efi.img
mkdir -p temp/esp
sudo mount iso/boot/grub/efi.img temp/esp
sudo mkdir -p temp/esp/EFI/BOOT
sudo cp iso/EFI/BOOT/BOOTX64.EFI temp/esp/EFI/BOOT
sudo umount temp/esp
```

### GFXMenu module
The [gfxmenu](https://www.gnu.org/software/grub/manual/grub-dev/html_node/Introduction_005f2.html#Introduction_005f2)
module provides a graphical menu interface for GRUB 2. It functions as an alternative to the menu
interface provided by the `normal` module. The graphical menu uses the GRUB video API supporting
VESA BIOS extensions (VBE) 2.0+ and supports a number of GUI components.

`gfxmenu` supports a container-based layout system. Components can be added to containers, and
containers (which are a type of component) can then be added to other containers, to form a tree of
components. The root component of the tree is a `canvas` component, which allows manual layout of its
child components.

**Non-container components**:
* `label`
* `image`
* `progress_bar`
* `circular_progress`
* `list`

**Container components**:
* `canvas`
* `hbox`
* `vbox`

The GUI component instances are created by the theme loader in `gfxmenu/theme_loader.c` when a them
is loaded.

## Trouble-shooting

### Keyboard not working
Booting into the ACEPC AK1 I found that GRUB had no keyboard support. After some research I found
that newer systems use `XHCI` mode for USB which is a combination of USB 1.0, USB 2.0 and USB 3.0 and
is newer. On newer Intel motherboards XHCI is the only option meaning that there is no way to fall
back on EHCI.

**Research:**
* XHCI GRUB module?
  * I see a `uhci.mod ehci.mod usb_keyboard.mod` in `/usr/lib/grub/i386-pc`
  * `uhci.mod` supports USB 1 devices
  * `ehci.mod` supports USB 2 devices
  * `xhci.mod` supports all USB devices including USB 3
* Use Auto rather than Smart Auto or Legacy USB in BIOS
  * Doesn't seem to help

Turns out that GRUB2 doesn't have any plans to support xHCI. After digging into this futher it
appears that the UEFI community is pulling away from GRUB2 as the boot manager of choice. There are 
other options out there for newer systems like [rEFInd](#rEFInd)

#### Incompatible license
If you get an ugly GRUB license error as follows upon boot you'll need to re-examine the GRUB modules
you've included on your EFI boot.
```
GRUB loading...
Welcome to GRUB!

incompatible license
Aborted. Press any key to exit.
```
During boot `GRUB` will check for licenses embedded in the EFI boot modules. You can test ahead of
time to see of any are flagged. Acceptable licenses are `GPLv2+`, `GPLv3` and `GPLv3+`.

Following the instructions in [Test USB in VirtualBox](#test-usb-in-virtualbox) I was able to
determine that the problem existed in VirtualBox as well so its not unique to the target machine I
was attempting to test my USB on. This means that either:

* A non-compliant licensed module being included could cause this
  * Looking through all modules wih `for x in *.mod; do strings $x | grep LICENS; done` revealed
  everything is kosher.
* GRUB doesn't have a stable binary ABI so mixing modules versions could cause this
  * Ensuring that all modules were removed before recreating didn't help
* Copying the ISO to the USB incorrectly may cause this
  * Validated the same process with the Arch Linux ISO and it works 
* The construction of the GRUB boot images isn't accurate
  * I dropped it down to the minimal modules, possible this helped but unlikely
* The construction of the ISO isn't accurate
  * I believe the issue was the xorriso properties I had used. After switching back to the original
  xorriso settings from cyberlinux 1.0 I fixed it.

