
# Adapting the kernel to support various QEMU devices

Previously, in the `3_miniSystem` section, we configured a minimum kernel that
doesn't support any devices. As an exercise to familiarize you with the task
of configuring a kernel, we will now attempt to get Microdot to support all the
virtual devices provided by `QEMU`.

A quick search through [wikipedia](https://en.wikipedia.org/wiki/QEMU#x86) 
and the man pages reveals a list of devices that `qemu-system-x86_64` provides:

* 

