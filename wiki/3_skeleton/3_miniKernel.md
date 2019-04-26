# Configuring a Minimal Kernel

```bash
# clean up the source directory
make distclean

# configure a kernel with minimum features enabled
make ARCH=$arch CROSS_COMPILE=$target- tinyconfig

# launch the configuration interface to tune the kernel
make ARCH=$arch CROSS_COMPILE=$target- nconfig
```

This is the configuration recipe:

```
64-bit kernel: Y
General Setup
	Default hostname: Microdot
	Configure standard kernel features (expert users)
		Multiple users, groups and capabilities support: Y
		Posix Clocks & timers: Y
		Enable support for printk: Y
		Enable signalfd() system call: Y
		Enable timerfd() system call: Y
		Enable eventfd() system call: Y
Enable the block layer: Y
	Partition Types
		Advanced partition selection: Y
			PC BIOS (MSDOS partition tables) support: Y
			EFI GUID Partition support: Y
Executable file formats / Emulations
	Kernel support for ELF binaries: Y
	Kernel support for scripts starting with #!: Y
Device Drivers
	Generic Driver Options
		Maintain a devtmpfs filesystem to mount at /dev: Y
		Automount devtmpfs at /dev, after the kernel mounted the rootfs: Y
```
