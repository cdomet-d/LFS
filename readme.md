# Compiling the kernel

## Oh the places you'll go

Reading the LSF Handbook, I've also made detours to :

- [How to extract and compiles packages](https://tldp.org/HOWTO/Software-Building-HOWTO.html#toc2)
  - Usefulness : undertermined
- [Building packages](https://moi.vonos.net/linux/beginners-installing-from-source/)

## Various notes and sources

- [The difference between a 32 and 64 bit system](https://www.reddit.com/r/explainlikeimfive/comments/14px4oe/eli5_what_is_the_difference_between_a_32bit_and/)
- [The Mine of Information](https://moi.vonos.net/)
  - A delopper blog about various things

### LFS Packages triva and explainations

- **Ncurses**: [tic](https://linux.die.net/man/1/tic) - the [terminfo](https://en.wikipedia.org/wiki/Terminfo) entry-description compiler

### Notes on the LFS chapters

## Cross compilation theory

### III.ii [Cross Compilation](https://www.linuxfromscratch.org/lfs/view/stable/partintro/toolchaintechnotes.html)

  - [Explained on Unix Stack Exchange](https://unix.stackexchange.com/questions/668844/why-is-the-canadian-cross-used-for-cross-compilation-in-linux-from-scratch/668847#668847)

I really struggled to understand this section of the LFS manual, so here's my understanding.

Overall, cross compilation is used to generate code for a machine `target` on a machine `host`. Those two machines can be completely different, and using `host`'s native compiler toolchain would produce binaries that can run on it's system and architecture but not on the target's.

While native compilation is always simpler, sometimes a target system is too slow, or doesn't have enough memory (for instance, embedded systems), and so we must compile that system's binaries on another, more powerful machine.

### 7.3. Preparing Virtual Kernel File Systems

  - Chroot environnement and the [Virtual Kernel Filesystem](https://www.kernel.org/doc/html/latest/filesystems/vfs.html)
  - [devtmpfs and the device tree](https://www.kernel.org/doc/html/latest/devicetree/usage-model.html)
  - 7.3.2 : [devpts](https://www.baeldung.com/linux/dev-pts) and also [a better explanation of terminal multiplexor](https://en.wikipedia.org/wiki/Terminal_multiplexer)

### 8.2.[ Package Management](https://www.linuxfromscratch.org/lfs/view/stable/chapter08/pkgmgt.html)

In this part we must choose a package management technique ; I've opted for Install-log, of which [I found a modern fork](https://github.com/Bjonk23/install-log) (thanks to @Bjonk23).

I've taken the liberty of packing the repository in a tarball since I currently don't have git on my LFS host and don't plan to install it. You can easily download it thusly:

```sh
wget https://github.com/cdomet-d/LFS/raw/refs/heads/master/install-log.tar
```

You can then follow [the original ReadMe](https://github.com/Bjonk23/install-log).

## Useful links

- [Ask questions the smart way](http://catb.org/~esr/faqs/smart-questions.html#before)
- [Dynamic Linking](https://lwn.net/Articles/961117/)