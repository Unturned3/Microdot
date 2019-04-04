# Linux Boot Process Overview

Understanding how does a Linux system start and what components are
loaded at which stage is vital for ensuring the proper functionality of
our own mini system.

1. BIOS gets loaded into RAM from the motherboard, looks for the 1st bootable
device it finds (hard drive, CD-ROM, etc), then it loads the bootloader
(grub, syslinux, ...) installed on that device.

2. The bootloader then loads the kernel image (vmlinuz/bzImage) into memory
and then passes the control over to the running kernel.

3. The kernel mounts the root file system, then executes a userspace program
to initialize the system, which is usually `/sbin/init`.

4. `/sbin/init` is the first userspace process that the kernel launches and its
PID is always 1. This process cannot exit during the runtime of the system
or else you will end up with a kernel panic (when the kernel runs into
an fatal error).  All other userspace processes are descendants of init,
and all orphaned processes are automatically adopted by init.  Commonly,
init reads `/etc/inittab` in order to carry out specific initialization
tasks, such as loading modules, mounting disks, running other scripts,
starting daemon processes, etc.

5. `/sbin/init` can now open terminals or start a graphical user interface
for the user to use.
