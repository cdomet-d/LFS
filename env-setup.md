# Setting up the Virtual Machine

For various access and security reasons, we will be working in a virtual machine with two separate disks :

- The host system disk, on which we will create the cross-compilation toolchain,
- The LFS disk, where our distribution will eventually live.

We elected to use a headless QEMU virtual machine to benefit from improved performances thanks to [KVM hardware acceleration](https://en.wikipedia.org/wiki/Kernel-based_Virtual_Machine).

Since our host machine shares the guest's architecture and we're intending to mostly work remotely through a SSH tunnel, using VirtualBox is redundant. Using KVM + QEMU this allows us to exploit the host kernel instead of relying on hardware emulation and a type 2 hypervisor. It saves a lot of time during the compilation portion of the project.

## Initializing the VM

Creating a QEMU Virtual Machine is essentially creating a [`.qcow2` file](https://en.wikipedia.org/wiki/Qcow) (see also [`.qcow2` QEMU documentation](https://www.qemu.org/docs/master/system/images.html#cmdoption-image-formats-arg-qcow2)), then mounting it to a `-drive` trough a QEMU command.

In concrete terms, for our case, we will create two .qcows2 files :

```bash
	qemu-img create -f qcow2 host.qcow2 10G
	qemu-img create -f qcow2 lfs.qcow2 30G
```

## Things to look up 

**COW overlays** -> allows to have a read-only base disk, writing changes to another file and preserving VM state in case something goes wrong.
