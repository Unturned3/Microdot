# Configuring a Minimal Kernel

The Linux kernel is made out of many subsystems and it supports a lot of
different hardware platforms. If we never manually configure it and
leave it to use the defaults, then a lot of unneeded subsystems and code
will be added to the final product. How much space does this waste?
An unmodified default configuration will generate a 8 to 10 megabyte kernel
image. On the other hand, a nicely tuned manual config will keep the image
under 1MB with plenty of functionality included.

Configuring the kernel basically boils down to selecting code that supports the
various hardware devices that you want to use, and including features that will
benefit userspace programs. For example, you might want to select the relevant
drivers for your onboard ethernet card, or for a hard drive, etc. Please read
[this](https://wiki.gentoo.org/wiki/Kernel/Gentoo_Kernel_Configuration_Guide)
Gentoo Wiki page in order to consolidate your understanding of the various
concepts involved. It might have some Gentoo-specific terminologies that
we won't use here.

> Unpack & change directory into the linux-4.18.5 directory

```bash
# clean up the source directory
make distclean

# configure a kernel with minimum features enabled
make ARCH=$arch CROSS_COMPILE=$target- tinyconfig

# launch the configuration interface
make ARCH=$arch CROSS_COMPILE=$target- nconfig
```

Now you should be seeing a text based configuration interface, with entries
similar to the following (hashtags are comments)

```text
64-bit kernel

# configure various kernel features
General setup

# should the kernel use any loadable modules?
Enable loadable module support

# include code to handle block devices? (hard disk, etc.)
Enable the block layer

# configure features related to CPUs
Processor type and features

# power management features
Power management and ACPI options

# system bus features
Bus options (PCI etc.)

Executable file formats / Emulations

# networking subsystem
Networking support

# support code for different hardware devices
Device Drivers

Firmware Drivers

# file system support (ext4, FAT, etc.)
File systems

Kernel hacking
Security options
Cryptographic
Virtualization
Library routines
```



You can navigate
around using the arrow keys, ESC key, and return key. Every indentation in the
recipe means that you need to enter a sub-menu to access the options. "Y" means
that you need to select the corresponding option ("N" means deselect). Again,
a hashtag ("#") indicates a comment. For each option, you can hit the "?" key to
show a description/explanation (this is great for learning how things work).
The descriptions sometimes will
point you to more information in the Linux documentation.

Configure the kernel according to the following recipe. As we said, this section
focuses on constructing a bare minimum system, and you should notice that we
didn't really include support for any real hardware.

```text
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

		# printk() is what the kernel uses to print messages onto the screen
		Enable support for printk: Y

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

Save, exit, and compile the kernel. The finished image should be around 700kB.
If we used `make defconfig` instead of `make tinyconfig` and hand-tuning the
kernel, we would end up with a 8MB image, which contains a plethora of bloat
that we'll never use. `make defconfig` is what most tutorials out there on the
internet instructs you to do.

```bash
make ARCH=$arch CROSS_COMPILE=$target- -jN

# copy the image to /opt (you probably need root permission for this)
cp arch/x86/boot/bzImage /opt
```

In the next section, we will pack the content of targetfs into a `initramfs`,
and boot our Microdot system.










