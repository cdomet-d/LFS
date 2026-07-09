# LFS

Here's a compendium of the things I found interresting while doing this project.

## Various notes and sources

- [The difference between a 32 and 64 bit system](https://www.reddit.com/r/explainlikeimfive/comments/14px4oe/eli5_what_is_the_difference_between_a_32bit_and/)
- [The Mine of Information](https://moi.vonos.net/)
  - A delopper blog about various things

### LFS Packages triva and explainations

- **Ncurses**: [tic](https://linux.die.net/man/1/tic) - the [terminfo](https://en.wikipedia.org/wiki/Terminfo) entry-description compiler

## Notes on the LFS chapters

### III.ii [Cross Compilation](https://www.linuxfromscratch.org/lfs/view/stable/partintro/toolchaintechnotes.html)

  - [Explained on Unix Stack Exchange](https://unix.stackexchange.com/questions/668844/why-is-the-canadian-cross-used-for-cross-compilation-in-linux-from-scratch/668847#668847)

I really struggled to understand this section of the LFS manual, so here's my understanding.

Overall, cross compilation is used to generate code for a machine `target` on a machine `host`. Those two machines can be completely different, and using `host`'s native compiler toolchain would produce binaries that can run on it's system and architecture but not on the target's.

While native compilation is always simpler, sometimes a target system is too slow, or doesn't have enough memory (for instance, embedded systems), and so we must compile that system's binaries on another, more powerful machine.

### 7.3. Preparing Virtual Kernel File Systems

  - Chroot environnement and the [Virtual Kernel Filesystem](https://www.kernel.org/doc/html/latest/filesystems/vfs.html)
  - [devtmpfs and the device tree](https://www.kernel.org/doc/html/latest/devicetree/usage-model.html)
  - 7.3.2 : [devpts](https://www.baeldung.com/linux/dev-pts) and also [a better explanation of terminal multiplexor](https://en.wikipedia.org/wiki/Terminal_multiplexer)

### 8.2.[Package Management](https://www.linuxfromscratch.org/lfs/view/stable/chapter08/pkgmgt.html)

In this part we must choose a package management technique ; I've opted for the timestamp tracking, which I've implemented like follows :

First, in the host, I created this script - which I named `install-log.sh` - in a directory **that's accessible in the chroot environnement**. For me, that meant `$LFS/var/log/pckmgmt`.

```sh
#!/bin/bash
# Usage: ./log_installed.sh [timestamp_file] [output_log]

TIMESTAMP_FILE="$1"
OUTPUT_LOG="$2"

if [ ! -f "$TIMESTAMP_FILE" ]; then
    echo "Error: Timestamp file '$TIMESTAMP_FILE' not found."
    echo "Create it first with: date +%s > <timestamp-file>"
    exit 1
fi

TIMESTAMP=$(cat "$TIMESTAMP_FILE")
echo "Logging files newer than: $(date -d "@$TIMESTAMP" '+%Y-%m-%d %H:%M:%S')"

find / -type f -newermt "@$TIMESTAMP" \
    -not -path "/proc/*" \
    -not -path "/sys/*" \
    -not -path "/dev/*" \
    -not -path "/run/*" \
    -not -path "/tmp/*" \
    -not -path "/var/tmp/*" >> "$OUTPUT_LOG" 2>/dev/null

echo "Logged $(wc -l < "$OUTPUT_LOG") files to $OUTPUT_LOG"
echo "------" >> "$OUTPUT_LOG"
```

Then, in the same directory, I created a script to run the installation tracking  more smoothly. I've named it `log.sh` and will refer to it as such thereafter.

```sh
#!/bin/bash

set -x

if [ $# -eq 0 ]
  then
    echo "Usage: log <package-name>"
	exit 1
fi

bash /path/to/logging-dir/install-log.sh \
		/path/to/logging-dir/<timestamp-file> \
		/path/to/logging-dir/"$1".log
```

Finally, in the chroot environnement, I :

  - exported a `LOGGER` environnement variable, which holds the path to the logging script - it's also where I've elected to store the logs ;
  - created a date alias to make the timestamp file management less of a pain;

```sh
export LOGGER=/path/to/logging-dir/
alias date='date +%s > $LOGGER/<timestamp-file>'
```

I guess those could  be added to `chroot` command used to enter our building environnement, but I honestly didn't feel comfortable modifying it ; I'll take the pain of having to type two commands.

## Utilities

As I was starting to loose my mind from repetitive tasks, I remembered the Shell is a powerful things and made a few tools to automate annoying stuff: 

### Check if all the packages where successfully installed

```sh
function chkinstall { while IFS= read -r line; do   command -v "$line" || echo "Not Found: $line" ; done < installed ;}
```

This expects an `installed` file to be in the directory where you run this. To create it, I created this function:

```sh
function winstalled { printf '%s\n' "$1" | sed 's/, and /\n/; s/, /\n/g' > installed ; }
```

That you use by copy-pasting the "installed programs" paragraph in your current binary LFS page, IE for kbd : 

```sh
	winstalled  chvt, deallocvt, dumpkeys, fgconsole, getkeycodes, kbdinfo, kbd_mode, kbdrate, loadkeys, loadunimap, mapscrn, openvt, psfaddtable (link to psfxtable), psfgettable (link to psfxtable), psfstriptable (link to psfxtable), psfxtable, setfont, setkeycodes, setleds, setmetamode, setvtrgb, showconsolefont, showkey, unicode_start, and unicode_stop
```

Then your package management process can look like that:

## Useful links

- [Ask questions the smart way](http://catb.org/~esr/faqs/smart-questions.html#before)
- [Dynamic Linking](https://lwn.net/Articles/961117/)

## My own commands

### Aliases

```sh
export LOGGER=/var/log/pckmgmt/
alias date='date +%s > $LOGGER/install_timestamp'
alias pcki='date && make install && log'
```

### Functions

```sh
function cwd { basename $(pwd) ;}
function log { bash /log.sh $(cwd) ;}
function leave { export DONE=$(cwd) && cd .. && rm -rf $DONE ;}
function winstalled { printf '%s\n' "$1" | sed 's/, and /\n/; s/, /\n/g' > installed ; }
function chkinstall { while IFS= read -r line; do   command -v "$line" || echo "Not Found: $line" ; done < installed ;}
```
