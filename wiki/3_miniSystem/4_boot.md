
# Assembling & Running the Final System

In this section, we will assemble the parts of our mini system and boot it
via QEMU. First, we need to pack the contents of `$targetfs` into an `initramfs`
archive. An `initramfs` is a compressed 
[cpio](https://en.wikipedia.org/wiki/Cpio) archive that is loaded into RAM at
boot by the bootloader, and the kernel mounts it as a temporary root file
system. You should read [this](https://landley.net/writing/rootfs-intro.html) 
article by Rob Landley to understand what it
is and why is it useful before proceeding to build Microdot.

The procedure to create a `cpio` archive is not very straight forward, but its
easy to understand. If we want to put the content of the `$targetfs` directory
into an initramfs, we have to run the following command (as root):

```bash
cd $targetfs
find . | cpio -H newc -o | gzip > /opt/initramfs.gz
```

This is how the command works:

The `find .` command prints a list of files present in the current directory.
Then the output of that gets piped into the input of `cpio`. It will then read
the list of files given, pack them using the `newc` format, and output the
finished archive. The output from `cpio` is then piped to the input of `gzip`,
which compresses the archive. Finally, the output from `gzip` is redirected to
the file `/opt/initramfs.gz`, and that gives us a gzip compressed cpio archive
of the `$targetfs` directory.

# Running the minimum system

Finally! We made it to the last step. Let's fire up QEMU and boot our system.
You can run the following command as the builder user or a regular user on your system:

```bash
qemu-system-x86_64 \
	-kernel bzImage \
	-append "rdinit=/sbin/init console=ttyS0 quiet" \
	-initrd initramfs.gz \
	-m 32M \
	-nographic
```

The mini system should boot, and you should be soon presented with a login
prompt. Login with root (no password), and do whatever you like. Note that
this tiny system has no presistent storage. That is, if you make any changes
to the files and reboot the machine, all changes would be lost. This is
because our root file system is an `initramfs`, which means that the content
of it resides in `RAM`. Whatever modifications we make to the data in RAM
will be lost if we can't write it to a non-volatile storage device.

If you are done, you can `poweroff` the system. `QEMU`, however, will
not terminate and return you to a terminal prompt, because our systems kernel
has no support for the ACPI subsystem yet, so the kernel has no way of telling
the hardware that the system is shutting down. We can open up a new terminal
and kill `QEMU` with the following command:

```bash
killall qemu-system-x86_64
```
