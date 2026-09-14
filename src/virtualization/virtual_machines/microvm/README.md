# MicroVM <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

The MicroVM project provides a path for managing production NixOS virtual machines via:
* Isolated /nix/store for the guest NixOS
* Individual VM configuration updates
* Modern Rust Hypervisor support designed for `virtio`

### Quick links
- [.. up dir](../README.md)
- [Overview](#overview)
  - [Backends compared](#backends-compared)
  - [QEMU vs MicroVM](#qemu-vs-microvm)
- [Getting Started](#getting-started)
  - [Launch your own vm](#launch-your-own-vm)

## Overview
Virtual Machines can be a better fit for some use cases than containers. My primary use case is for a 
full remote (i.e. via SPICE) environment in which to test system configuration or to use as a node in 
other containerization solutions such as Kubernetes, Docker swarm, Portainer or others. This provides 
an excellent decoupling of the host system from guest systems for simpler management.

**References**
- [microvm.nix](https://github.com/microvm-nix/microvm.nix)
- [Virtual Networking](https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking#macvlan)

NixOS is the ideal OS for this scenario with declarative syntax and robust rollback. It is also 
excellently integrated for one off QEMU virtual machine tests of your configuration. Unfortunately 
the project hasn't extended this management into production Virtual Machines with separate isolated 
configurations. The `microvm.nix` project was built to solve this.

`microvm.nix` creates virtual machine disk images and runner script packages for the entries of the 
`nixosConfigurations` section of a `flake.nix` file.

### Backends compared

| Backend           | Language          | Origin                 | Boot time         | Device model                                          | Notes                                                                                    |
| ----------------- | ----------------- | ---------------------- | ----------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| qemu              | C                 | QEMU project           | Fast (~125-200ms) | Broadest — virtio + legacy devices, PCI               | Most mature, best fallback for anything exotic (GPU passthrough, unusual guests) |
| cloud-hypervisor  | Rust (rust-vmm)   | Intel/rust-vmm         | Fast (~150-300ms) | virtio-only, but broader than Firecracker — supports vhost-user, CPU/memory hotplug | "Middle ground" — more features than Firecracker, still much lighter than QEMU |
| firecracker       | Rust (rust-vmm)   | AWS                    | Fastest (~125ms)  | Deliberately minimal — 5 emulated devices only        | Built for ephemeral, high-density serverless workloads (Lambda) |
| crosvm            | Rust              | Google/ChromeOS        | Fast              | virtio-only, strong virtiofs/GPU-forwarding support   | Most mature for desktop/ChromeOS VM use, less battle-tested for long-running server workloads |
| stratovirt        | Rust              | Huawei                 | Fast              | virtio-only, lightweight                              | Similar niche to Firecracker/Cloud Hypervisor, less common outside China/Huawei cloud |
| alioth            | Rust              | newer/community        | Fast              | Minimal, still early-stage                            | Newest of the bunch, least mature |
| kvmtool           | C                 | Linux kernel project   | Very fast         | Extremely minimal                                     | Originally a kernel-testing tool, lowest overhead but fewest features |
| vfkit             | Go                | Apple/community        | Fast              | virtio-only                                           | macOS-focused (Virtualization.framework), not relevant on NixOS/Linux hosts |

### QEMU vs MicroVM
Both sit on the same KVM acceleration underneath, so neither one is "faster" at the CPU/memory-virtualization
level. The difference is in what's built around that core:

| Aspect             | QEMU                                                                       | MicroVM                                                                                             |
| ------------------ | -------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Config surface     | Full device model — sound, USB, GPU passthrough, dozens of bus types       | Deliberately minimal — `virtio` devices only                                                          |
| Hypervisor choice  | QEMU only                                                                  | Choice of 8 backends: `qemu`, `cloud-hypervisor`, `firecracker`, `crosvm`, `kvmtool`, `stratovirt`, etc. |
| Boot time          | Seconds, full BIOS/UEFI + kernel boot path                                 | Sub-second with `firecracker`/`cloud-hypervisor` — no BIOS, direct kernel boot                        |
| Memory footprint   | Tens of MB of emulated device overhead per VM                              | A few MB overhead per VM — designed for high VM density                                               |
| Use case fit       | Desktop VMs, Windows guests, GPU/USB passthrough, general purpose testing  | NixOS-only guests, microservices, CI runners, isolated one-service-per-VM workloads                   |
| Declarative config | Imperative flags or `libvirt` XML                                          | Native NixOS module — a `microvm` block in your `flake.nix`/`configuration.nix`                       |
| Guest OS support   | Any OS                                                                     | NixOS guest only |

In short: reach for QEMU when you need a general-purpose VM — a desktop, a Windows guest, GPU
passthrough. Reach for MicroVM when the guest is NixOS itself and you want many small, fast-booting,
tightly isolated VMs rather than one heavyweight general-purpose one.

## Getting Started

### Launch your own vm
1. Download the project template
   ```bash
   $ nix flake init -t github:astro/microvm.nix
   ```
2. Build and launch the vm
   ```bash
   $ nix run .#my-microvm
   ```
3. Login with `root` and no password

```
-name my-microvm 
-M microvm,accel=kvm:tcg,acpi=on,mem-merge=on,pcie=on,pic=off,pit=off,usb=off
-m 512
-smp 1
-nodefaults -no-user-config -no-reboot
-kernel /nix/store/jzl52vx9j42jgn92nynihpniamzwd31p-linux-6.6.64/bzImage
-initrd /nix/store/mdw5d0j85h530c47hjxj2ims8pj9ir9j-initrd-linux-6.6.64/initrd
-chardev stdio,id=stdio,signal=off
-device virtio-rng-pci
-serial chardev:stdio
-enable-kvm
-cpu host,+x2apic,-sgx 
-device i8042
-append earlyprintk=ttyS0 console=ttyS0 reboot=t panic=-1 root=fstab loglevel=4
 init=/nix/store/blr4mm3bjv7anp9spfjfj05iqqhp513b-nixos-system-my-microvm-25.05.20241213.3566ab7/init
 regInfo=/nix/store/sglj78lb7g516wxassar23v5srkkmm70-closure-info/registration -nographic -sandbox on 
-qmp unix:control.socket,server,nowait
-drive id=vda,format=raw,file=var.img,if=none,aio=io_uring,discard=unmap,cache=none,read-only=off
-device virtio-blk-pci,drive=vda
-object memory-backend-memfd,id=mem,size=512M,share=on
-numa node,memdev=mem 
-fsdev local,id=fs0,path=/nix/store,security_model=none
-device virtio-9p-pci,fsdev=fs0,mount_tag=ro-store
```
