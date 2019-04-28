# Bare Minimum

In this section, we will create a Linux system so small to the extent that it's
somewhat useless. It has no support for any peripheral devices such as keyboard,
screen, storage, or network, and its only means of interacting with the outside
world is via a serial connection. However, it is still a "complete" system, in
the sense that it has a kernel, an init system, and a decent userspace, so it
suffices for the job of demonstrating various part's internal workings and functionality.

This bare-minimum Linux system will be ran inside an emulated environment using
the Quick Emulator (QEMU) because:

1. There really is no point to run it on physical hardware
2. All kinds of hardware setups exist, and this mini system probably won't
	run smoothly on all of them. Using an emulator makes the environment much
	more controlled and eliminates the issue of incompatible hardware.

In the next section, we will expand upon this and transform the bare minimum
system into a more useful one that can boot on real hardware.
