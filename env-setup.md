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

Once we have those files, we can then start our machine using the Qemu CLI :

```bash
	qemu-system-x86_64 \
	-enable-kvm \ # uses the Linux Kernel hypervisor module, allowing the guest to run on real hardware
	-m 8G \ # 8GB of RAM
	-cpu host \ # allows us to use the computer's CPU directly instead of emulating one
	-smp 20 \ # allocates 20 cores to the guest
	-drive file=$LFS_PATH/debian-host.qcow2,if=virtio \ # mounts the host disk to the guest
	-drive file=$LFS_PATH/lfs.qcow2,if=virtio \ # mounts the lfs disk to the guest
	-boot d \ # boots the guest from the cd-rom - this allows us to install an .iso on the host disk
	-vga virtio \ 
	-netdev user,id=net0,hostfwd=tcp::2222-:22 \
	-device virtio-net-pci,netdev=net0 \
	-display none
```

## Cross compilation theory

I really striggled to understand [this section](https://www.linuxfromscratch.org/lfs/view/stable/partintro/toolchaintechnotes.html) of the LFS manual, so here's my understanding.

Overall, cross compilation is used to generate code for a machine `target` on a machine `host`. Those two machines can be completely different, and using `host`'s native compiler toolchain would produce binaries that can run on it's system and architecture but not on the target's.

While native compilation is always simpler, sometimes a target system is too slow, or doesn't have enough memory (for instance, embedded systems), and so we must compile that system's binaries on another, more powerful machine.

## Useful links

- [Ask questions the smart way](http://catb.org/~esr/faqs/smart-questions.html#before)
- [Dynamic Linking](https://lwn.net/Articles/961117/)

- **Cross Compilation**
  - [Cross Compilation according to LFS](https://www.linuxfromscratch.org/lfs/view/11.0/partintro/toolchaintechnotes.html) and also [explained on Unix Stack Exchange](https://unix.stackexchange.com/questions/668844/why-is-the-canadian-cross-used-for-cross-compilation-in-linux-from-scratch/668847#668847)

- **Qemu**
  - [QEMU Invocation](https://www.qemu.org/docs/master/system/invocation.html)
  - [QEMU Overlays](https://zakariakebairia.com/posts/qemu-overlay-images) -> allows to have a read-only base disk, writing changes to another file and preserving VM state in case something goes wrong.

- **7.3. Preparing Virtual Kernel File Systems**
  - Chroot environnement and the [Virtual Kernel Filesystem](https://www.kernel.org/doc/html/latest/filesystems/vfs.html)
  - [devtmpfs and the device tree](https://www.kernel.org/doc/html/latest/devicetree/usage-model.html)
  - 7.3.2 : [devpts](https://www.baeldung.com/linux/dev-pts) and also [a better explanation of terminal multiplexor](https://en.wikipedia.org/wiki/Terminal_multiplexer)