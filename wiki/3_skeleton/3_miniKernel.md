# Configuring a Minimal Kernel

```bash
# clean up the source directory
make distclean

# configure a kernel with minimum features enabled
make ARCH=$arch CROSS_COMPILE=$target- tinyconfig

# launch the configuration interface
make ARCH=$arch CROSS_COMPILE=$target- nconfig
```

Now you should be seeing a text based configuration interface. You can navigate
around using the arrow keys, ESC key, and return key. Every indentation in the
recipe means that you need to enter a sub-menu to access the options. "Y" means
that you need to select the corresponding option ("N" means deselect).

```

64-bit kernel: Y

General Setup
	Default hostname: Microdot
	Initial RAM filesystem and RAM disk (initramfs/initrd) support: Y
		Support initial ramdisk/ramfs compressed using bzip2: N
		Support initial ramdisk/ramfs compressed using LZMA: N
		Support initial ramdisk/ramfs compressed using XZ: N
		Support initial ramdisk/ramfs compressed using LZO: N
		Support initial ramdisk/ramfs compressed using LZ4: N 

	Configure standard kernel features (expert users)
		Multiple users, groups and capabilities support: Y
		Posix Clocks & timers: Y
		Enable support for printk: Y
		Enable signalfd() system call: Y
		Enable timerfd() system call: Y
		Enable eventfd() system call: Y

Executable file formats / Emulations
	Kernel support for ELF binaries: Y
	Kernel support for scripts starting with #!: Y

Device Drivers
	Generic Driver Options
		Maintain a devtmpfs filesystem to mount at /dev: Y
			Automount devtmpfs at /dev, after the kernel mounted the rootfs: Y
	Character devices
		Enable TTY: Y
			Virtual terminal: N
			Unix98 PTY support: N
			Legacy (BSD) PTY support: N
		Serial drivers
			8250/16550 and compatible serial support: Y
				Console on 8250/16550 and compatible serial port: Y

File systems
	 Enable POSIX file locking API: Y
	 Pseudo filesystems
	 	/proc file system support: Y
	 	sysfs file system support: Y

```

Save, exit, and compile the kernel. The finished image should be around 750kB.
If we used `make defconfig` instead of `make tinyconfig` and hand-tuning the
kernel, we would end up with a 8MB image, which contains a plethora of bloat
that we'll never use. `make defconfig` is what most tutorials out there on the
internet instructs you to do.

```bash
make ARCH=$arch CROSS_COMPILE=$target- -jN

# copy the image to /opt
cp arch/x86/boot/bzImage /opt
```

In the next section, we will pack the content of targetfs into a `initramfs`,
and boot our Microdot system.










