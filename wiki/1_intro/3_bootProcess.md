# Linux Boot Process Overview

Understanding how does a Linux system start and what components are
loaded at which stage is vital for ensuring the proper functionality of
our own mini system. Generally, for a standard `x86_64` machine, this is
what happens during power on:

1. The BIOS (basic input-output system) gets loaded into RAM from the
motherboard, looks for the 1st bootable device it finds (hard drive,
CD-ROM), then it loads the bootloader installed on that device. In our
case, it would be `syslinux`.

2. The bootloader then loads the kernel into memory and then passes the
control over to the running kernel.

3. The kernel mounts the root file system, then executes a userspace program
to initialize the system, which is usually `/sbin/init`. In our case, the
init program is provided by `busybox`.

4. `/sbin/init` is the first userspace process that the kernel launches and
its process ID is always 1. This process cannot exit during the runtime
of the system or else you will end up with a kernel panic (when the kernel
runs into an fatal error).  All other userspace processes are descendants
of init, and all orphaned processes are automatically adopted by init.
Commonly, init reads its configuration file, `/etc/inittab`, and carries
out tasks specified by the file, such as opening a text terminal, mounting
other disks, starting a network connection, etc.

5. The user can now interact with the system.

On a conventional PC, this bootup process typically takes around 20 seconds
to a minute. On the other hand, Microdot Linux can boot up within a
second, because its components are highly optimized for both speed and
size. Since the system only has a handful of programs, it uses suprisingly
little memory, and can boot with as little as 32MB of RAM.
