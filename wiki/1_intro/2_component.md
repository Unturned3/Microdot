
# Components of a Minimal Linux System

Any sensible computer system (that a user can use) probably includes the
following components:

* Bootloader

	When the computer powers on, the operating system can't load
	itself into RAM and start to execute. A small program (the
	bootloader) will copy the operating system from a storage media
	(hard drive, CD-ROM, etc.) into RAM and pass the execution control
	to it.

* Kernel

	The kernel is the core of an operating system, as it interacts
	the platform's hardware, manages the computer's resources
	(such as how much RAM should be given to an application), and
	provides an almost platform-independent environment for userspace
	applications to run in.

* Userspace Applications

	Although vital, the kernel can't actually do anything that's useful
	for the user. The functionality of the operating system depends
	on the userspace applications that runs on top of the kernel. For
	example, one might have programs that initiates a network connection,
	or make backups to a hard drive, etc.

* User Interface

	An operating system can't be used if the user can't interact with
	it (obviously). The UI of the system can be graphical, or command
	line, or just a few buttons (for embedded systems, like a coffee
	machine).


We will hand-pick the vital components that we will include in Microdot
Linux and tailor it to our needs.
For the bootloader, we will use `syslinux`, as it is easy to configure
and has a tiny footprint. For the userspace, we will use the "Swiss Army
Knife of Embedded Linux", aka. `busybox`. It combines roughly 300 common
Linux utilities into 1 program, and it only uses around 900KB of disk
space. For the user interface, we will use the `ash` shell provided in
`busybox`. Of course, for the kernel, we will use the Linux kernel (or
else this project would be named something else, like Microdot Windows)

In the next section, you will learn how these components work together
to bring up a usable system when the computer powers on.

