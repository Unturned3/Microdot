
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
* editing files using a CLI text editor

A lot of jargon is used throughout the documentation. If there are terms or
concepts that you don't know about, please consult Google and make sure
that understand them before moving on, or else you might get stuck later
because you missed something out previously.

Many people on the internet has already explained certain important
concepts that we will be working with; So instead of reinventing the wheel
and rewriting what others have wrote, I will put links to them in the
documentation.

If you feel like that something isn't explained clearly enough in the docs,
feel free to 
[open up a new Issue on GitHub](github.com/Unturned3/Microdot/issues) 
and tell me about it there.

Useful websites to learn about the Linux command line:
* [linuxcommand.org](http://linuxcommand.org)
* [ryanstutorial.net/linuxtutorial](https://ryanstutorial.net/linuxtutorial)
* [linux-tutorial.info](https://linux-tutorial.info)

# Host System Requirements

You need to have a working Linux environment (Ideally a mainstream
distribution such as Debian or Arch) on your machine. It can either be
installed directly onto the computer's hard disk or inside a virtual
machine. VMs are comparatively easier to set up, but the downside is
that the guest operating system won't have access to all the CPU power
on the computer, so it might take longer when compiling software. If you
do not have a working Linux system yet, just visit Google and you can
find plenty of good tutorials out there about how to set up a VM.

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



