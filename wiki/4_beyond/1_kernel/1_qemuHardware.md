
# Adapting the kernel to support various QEMU devices

Previously, in the `3_miniSystem` section, we configured a minimum kernel that
doesn't support any devices. As an exercise to familiarize you with the task
of configuring a kernel, we will now attempt to get Microdot to support all the
virtual devices provided by `QEMU`.

A quick search through [wikipedia](https://en.wikipedia.org/wiki/QEMU#x86) 
and the man pages reveals a list of features that `qemu-system-x86_64` provides:

* PCI Host Bridge: Intel 440FX
* PCI IDE interface with hard disk and CD-ROM support
* PCI Network Adapter E1000
* PCI UHCI USB controller
* Cirrus CLGD 5446 PCI VGA card
* SMP, up to 255 CPUs
* etc...

We will now go throuh each one of these, explain what they do, and configure
our kernel to support them. I am assuming that you followed the instructions
in `wiki/3_miniSystem/3_miniKernel.md` and configured the kernel accordingly 
before reading this document.

Note that very basic command and instructions, such as changing directory into
the Linux kernel source folder or opening up another terminal as a normal
user and launching QEMU with it, are not written. How to do such things are
covered in documents in previous sections. Also, execute all commands in this
section as the builder user, unless the docs specifically say to use another
user, or it's launching QEMU to test the system.



# Adding support for the PCI bus and the ACPI subsystem

You might have noticed that when we issue the `poweroff` command in our mini
Linux system, the system halts, but the QEMU process does not exit, as if it
doesn't know the system has stopped running. This is the case because our Linux
system has no way of telling the hardware that the system is going down. We
will add support for the ACPI subsystem to allow our system to communicate
with the various hardware components available. ACPI is generally present on all
modern PC hardware.

The PCI bus is a generic extension bus used by many PC systems, and many
hardware devices are attached to the system via the PCI bus, such as hard disk
controllers, network cards, USB controllers, etc. The ACPI subsystem will
be automatically enabled if you enable PCI support in the kernel.

```text
Bus options (PCI etc.)
	PCI support: Y
```

Compile the kernel, copy the `bzImage` to `/opt`, and Run QEMU. When you boot
into the system, typing the `lspci` command should show you a list of PCI
devices. `Busybox lspci` only shows the device ID and no human readable names,
unlike the normal `lspci` that you would find in many Linux distros. You can
search up the IDs online and find out the various devices detected on the PCI
bus.

If you type `poweroff`, the system will shutdown, QEMU will exit, and return you
to the terminal prompt, during which you will see a message from `ACPI` saying
that its preparing the system for a sleep state.


# Adding support for the networking subsystem & specific network devices

In order to use the ethernet card provided by QEMU, we will first need to add
support for the networking subsystem, which contains code that support sockets,
TCP/IP, 

```text
Networking support: Y
	Networking options:
		Unix domain sockets: Y
		TCP/IP networking: Y
	Wireless: N		# we have no wireless networking device in QEMU
```

Besides that, we also need to include support for the specific network card
hardware that we are provided with. If you look in the QEMU man pages, you can
see that this would be the "e1000" network card. We will use the `SymSearch`
function provided by `nconfig` to search for `e1000`. Launch nconfig, and press
`F8`, type in `e1000`, and press enter. The displayed message should look
something like this:

```text
Symbol: E1000 [=n]
Type  : tristate
Prompt: Intel(R) PRO/1000 Gigabit Ethernet support
	Location:
		-> Device Drivers
			-> Network device support (NETDEVICES [=n])
				-> Ethernet driver support (ETHERNET [=n])
					-> Intel devices (NET_VENDOR_INTEL [=n])
```

The above search results shows us the location of the config option for
`Intel(R) PRO/1000 Gigabit Ethernet support`. We can now add support for the
`e1000` ethernet card, and remove the support for a few other unnecessary
things.

> Note: Once you are in the `Ethernet driver support` menu, you should notice
> that support for a lot of hardware vendors are included. Uncheck everything,
> except for the Intel devices specified below.

```text
Device Drivers:
	Network device support: Y
		Ethernet driver support: Y
			Intel (82586/82593/82596) devices: N
			Intel devices: Y
				Intel(R) PRO/1000 Gigabit Ethernet support: Y
```

Save, exit, and compile, then launch QEMU. After the system has booted, type
in the `ip link` command, and you should see the `eth0` interface present.
We will cover network configuration in another section.

# Symmetric Multi-Processing (SMP)

This feature allows our kernel to utilize more than 1 CPU cores. Personal
computers commonly ships with 4 cores or even 8 cores these days, and even
embedded systems like the Raspberry Pi has a quad-core cpu, so there's really
no reason not to enable this feature. Go into `nconfig` again and configure
the following:

```text

Processor type and features:
	Symmetric multi-processing support: Y

	# each CPU supported adds 8KB to the kernel image.
	# we will probably never have 64 CPUs on a system
	Maximum number of CPUs: 8
```

