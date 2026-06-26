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

Once we have those files, we can then start our machine using the Qemu CLI.

We first need to install the host ; I chose Debian Trixie, but you can do whatever.

```bash
	qemu-system-x86_64 \
	-enable-kvm \ # uses the Linux Kernel hypervisor module, allowing the guest to run on real hardware
	-m 8G \ # 8GB of RAM
	-smp 20 \ # allocates 20 cores to the guest (right now it's overkill but it'll get nice when we're compiling packages)
	-cpu host \ # allows us to use the computer's CPU directly instead of emulating one
	-drive file=$LFS_PATH/host.qcow2,if=virtio \ # mounts the host disk to the guest
	-boot d \ # boots the guest from the cd-rom \
	-cdrom /path/to/installer.iso \
	-vga virtio \ # uses a paravirtualize GPU, which accelerates any graphical renders. 
		# Useful for the install through the GUI, it can be dropped later.
```

Install your host distribution within your guest, then kill your VM and prepare to launch your working version. I personnaly prefer working headless through SSH, so my configuration reflects that. You do you though.

```bash
	qemu-system-x86_64 \
	-enable-kvm \
	-m 8G \
	-smp 20 \
	-cpu host \
	-drive file=$LFS_PATH/host.qcow2,if=virtio \
	-drive file=$LFS_PATH/lfs.qcow2,if=virtio \ # mounts the lfs disk to the guest
	-device virtio-net-pci,netdev=net0 \ # see notes
	-netdev user,id=net0,hostfwd=tcp::<host port>-:<guest port> \ # see notes
	-display none # for headless 
```

Notes:

- anytime your see `virtio` in Qemu options, it means that we're [paravirtualzing](https://www.techtarget.com/searchitoperations/definition/paravirtualization) a hardware element (a network cart, a GPU, etc...). It helps with performance.
  - [VirtIO paravirtualized device](https://docs.kernel.org/driver-api/virtio/virtio.html)
- `-device virtio-net-pci,netdev=net0` and `netdev user,id=net0,hostfwd=tcp::2222-:22` work together to enable SSH. `-device` adds a paravirtualized network card to the guest, allowing it to perform network operations, while `-netdev` creates the user NATS on the host, determining how and on which ports the guest can communicate with the host.
  - This enables us to `ssh -p <port> <username>@localhost` to the guest.

Tips:

- In order to avoid typing your password everytime you ssh into your machine, you can run `sh-copy-id -p <port> <username>@localhost` on the host. This copies your ssh key.
- You should put your `qemu-system` command in a script so you can easily run it and avoid typing the command everytime. I've done so and wrapped it in an alias to get working in a second :

```bash
	startvm='nohup bash /path/to/qemusystem/script.sh > /path/to/qemusystem/vm.log 2>&1 &'
```

That allows you to start your guest headlessly AND recover your terminal after it's running, saving you the trouble to have several terminals open to get in the guest. Any messages or errors are logged to vm.log, allowing for debug if need be.

Since we nohupped the guest, it won't respond to signals anymore; to turn off your VM, either run:

- `poweroff` as sudo from within, (safest)
- `pkill qemu` from the host (less safe, might corrupt disks)

## Creating overlays

One might create overlays of their qemu disks. This allows to have a read-only base disk, writing changes to another file and preserving VM state in case something goes wrong. I recommend doing it before doing any potential breaking changes.

In order to do so, turn off your VM (with `sudo poweroff` in the guest, for example). I usually make sure the guest is down with:

```bash
	pgrep qemu
```

Then, run :

```bash
	qemu-img create \
	-o backing_file=./<DISK-TO-BACKUP>.qcow2,backing_fmt=qcow2 \
	-f qcow2 ./<NAME_OF_OVERLAY>.qcow2
  ```

Obviously this works if your disks are `qcow2` ; idk how to do it otherwise.

Once you have your overlays, you should mount those instead of your base disks; either modify your script or remember to do so if you enjoy typing your command manually everytime. 

It's possible to chain overlays on top of each other, but I'm not doing that. Rather, I recommend you regularly commit your overlays to their base images and create a fresh one. To do so, ensure your VM is off and do : 

```bash
	qemu-img commit <overlay-name.qcow2>
```

That will merge the changes made to your overlays back on the base image. After checking that the base image boots correctly and you haven't lost stuff, you can delete the previous overlay and create a fresh one.

- **Qemu**
  - [QEMU Invocation](https://www.qemu.org/docs/master/system/invocation.html)
  - [QEMU Overlays](https://zakariakebairia.com/posts/qemu-overlay-images)
