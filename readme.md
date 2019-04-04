

# Microdot Linux

The Microdot Linux project will guide you through the process of
creating a miniature but fully functional Linux system from the
ground up, with special emphasis on toolchain creation and kernel
configuration. By default, Microdot Linux only contains a minimum
environment (kernel, busybox, and musl-libc), however other programs
can easily be added by compiling them from source, or by downloading
them through an experimental package manager. 

# Getting Started

The Microdot Wiki is where all the tutorials and documentation are
stored. Check out the Wiki to start building your own Microdot Linux!

You can also visit the FAQ section to learn more
about the Microdot project.


# FAQ

### What does "Microdot" mean?

Microdots are a steganographic technique that involves
photographically shrinking a large chunk of text to the
size of a typographical dot, such as a period. This term
is used here to figuratively show the fact that Microdot
Linux compresses a lot of functionality into a tiny
amount of space.

### Why did you build the Microdot project?

Initially, MicroLinux started out as a personal project to develop a
tiny Linux system that uses as little disk space and RAM as possible
(intended as a hypervisor guest). However, I faced some major challenges
during the process:

* How do I make a cross compilation toolchain?
* How do I properly configure the Linux kernel?
* How can I compile a kernel with minimal bloat?
* What are the bare minimum components needed for a functional system?


### Who is the intended audience of the Microdot Project?

The Microdot Project is primarily intended for people who
is interested in constructing a custom Linux system (it seems
like that this is quite a popular activity for Linux users).
However, besides that, the Microdot Project also provides
useful information on cross compilation, which could be useful
outside the scope of Linux system building.

### What's the difference between the Microdot Project and others?

Tiny Core Linux: does an exellent job making a tiny Linux distro,
but isn't primarily focused on teaching people _how_ to make one

Linux From Scratch: teaches people how to manually assemble
a working Linux system, but the resulting system is still quite
big (200+ MB). Does not teach you anything about how to
properly configure a suitable kernel. Complicated process
of constructing a cross toolchain.

Microdot Project: focuses on how to make a very small but
useful Linux system. Includes sufficient information on
important aspects such as toolchain construction and kernel
configuration. Also has a package manager, so its more
like a complete distro. 

### What can Microdot Linux be used for?

Microdot Linux is made to be modular and extendable,
meaning that you can tailor it to your needs. Because 
it comes with no bloat and has a small size, it can
also be used as an embedded linux system, where resources
are constrained. For example, you can replace Rasbian with
Microdot on a Raspberry Pi in order to save more memory
and processing power for your applications. 



