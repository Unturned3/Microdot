
# Skill Requirements

Building a Linux system from the ground up is not an easy task, and
it requires familiarity with the Linux command line. As a minimum,
you should be able to do basic operations like these:

* copying/moving/renaming files
* changing/listing directories
* printing file contents
* deleting files
* creating files/directories
* using variables

You also need to know how to edit files from the command line, using a
text editor such as GNU Nano or Vi/Vim.

Useful websites to learn about the Linux command line:
* linuxcommand.org
* ryanstutorial.net/linuxtutorial
* linux-tutorial.info

# Host System Requirements

You need to have a working Linux environment (Ideally a mainstream
distribution such as Debian or Arch) on your machine. It can either be
installed directly onto the computer's hard disk or inside a virtual
machine. VMs are comparatively easier to set up, but the downside is
that the guest operating system won't have access to all the CPU power
on the computer, so it might take longer when compiling software.

I used a Debian 9 system installed in a VM to build Microdot Linux, because
I didn't bother to do a hard drive install and mess with the Apple boot
loader. The VM has 16GB of free disk space, 2GB of RAM, and 2 CPU cores. 

Here's a list of required packages for building Microdot Linux. You can
use Google to find distribution-specific instructions for installing
these packages onto your host Linux system.

* gcc (The GNU compiler collection)
* wget (a simple tool to download files)
* make (a build tool that simplifies the compilation process)
* bc (needed for building the Linux kernel)
* qemu (Quick Emulator, used to run and test Microdot Linux)
* ncurses (a text-based menu interface for certain configuration scripts)
* gawk (a text-processing language)



