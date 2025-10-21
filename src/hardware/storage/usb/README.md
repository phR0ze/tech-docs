# USB <img style="margin: 6px 13px 0px 0px" align="left" src="../../data/images/logo_36x36.png" />

### Quick links
- [.. up dir](../README.md)
- [Burn ISO to USB](#burn-iso-to-usb)

## Burn ISO to USB

1. Locate your drive with `lsblk`
   ```bash
   $ lsblk
   NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
   ...
   sdd      8:64   1   7.2G  0 disk 
   └─sdd1   8:65   1   3.8G  0 part 
   ```

2. Use `dd` to burn the image to the USB device
   ```bash
   $ sudo dd if=/path/to/your.iso of=/dev/sdX bs=4M status=progress oflag=sync
   ```
