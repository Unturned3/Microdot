
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
