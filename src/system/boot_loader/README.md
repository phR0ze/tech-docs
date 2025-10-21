# Boot loader <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

UEFI devices now have alternative options for boot loaders that provide options for image display,
custom fonts, and menus either on par or more advanced than the venerable GRUB2's GFXMenu in
functionality. There are a number of options to consider.

### Quick links
* [.. up dir](..)
* [BIOS firmware](#bios-firmware)
  * [BIOS boot process](#bios-boot-process)
* [UEFI firmware](#uefi-firmware)
  * [EFI boot process](#efi-boot-process)

### Linked pages
* [Grub](grub/README.md)

## BIOS firmware
BIOS was developed in the 1970s and was prevelant till the late 2000s when it began to be gradually
replaced by EFI. BIOS provides a basic text-based interface.

### BIOS boot process
BIOS looks at the first segment of the drive to find the bootloader. Traditionally this was a `MBR`
partitioning scheme; however `GPT` partitioning part of the `EFI` spec can be used instead if done
correctly. In order to accomplish this you need to create a `GPT protective MBR`. The only caveat is
the boot loader needs to be GPT aware which pretty much all Linux compatible bootloaders are. Instead
of injecting the boot loader into the MBR space in the first partition you need to create a new `BIOS
Boot Partition` code `EF02`.

By leveraging the `GPT BIOS Boot Partition EF02` on a BIOS system and instead a normal `ESP EF02`
boot partition on UEFI systems we create a configuration that is bootable by modern EFI bootloaders
like Clover in either case. This means we can create a single Clover custom UI that will boot and be
used on either BIOS or UEFI.

## UEFI firmware
From about 2010 on all computers have been using UEFI as their firmware to interface with the mother
board rather than the old BIOS firmware. 

**Note:** `UEFI` is essentially `EFI 2.0` leading to the prevasive use of just `EFI` to cover all 
flavors.

Most EFI boot loaders and boot managers reside in their own subdirectories inside the `EFI` directory
on the `EFI System Partition (ESP)` e.g. `/dev/sda1` mounted at `/boot` thus
`/boot/efi/<bootloader>`.

