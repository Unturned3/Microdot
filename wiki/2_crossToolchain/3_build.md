# Constructing the Cross Compilation Toolchain

In this section, we will use the source packages that we previously
downloaded to make a cross toolchain that runs on `x86_64` and produces
code for `x86_64`. Here, we are using the cross compiler not to generate
code for different platforms, but to isolate the final binaries from
the host system's environment, because by definition a cross compiler
cannot rely on anything in the host environment. Doing so eliminates the
risk of our host system contaminating the binaries that will be used
in Microdot Linux.


### Environment Setup

```bash
$ su - builder

$ mkdir -pv $sysroot
$ mkdir -pv $install/lib
$ mkdir -pv $sysroot/lib

$ ln -sfv $install/lib $install/lib64
$ ln -sfv $sysroot/lib $sysroot/lib64
$ ln -sfv . $sysroot/usr

$ cd $install/src 
```

### How to Work With Source Code Packages

For every section below, you need to use the `tar` command to uncompress
and untar the archives (such as `.tar.gz` or `.tar.xz`, etc). Then you
need to change directory into the folder that's created by the untarred
package. Instructions in these sections assume that you are starting in the
untarred folder of a package. An example of how this is done is given in
the _linux kernel headers_ section.

### linux kernel headers

First, we will install the Linux kernel headers into the `$sysroot`
directory. The kernel headers can allow an application to know what
features are enabled and usable in the kernel's interface. The headers
are needed by `busybox` because it implements a lot of its functions
by utilizing the kernel's API. We have to install the headers first,
because installing it later would cause it to remove vital files installed
in the `$sysroot` by `musl-libc` and `gcc`.

```bash
tar -xf linux-4.18.5.tar.gz		# uncompress & untar the package
cd linux-4.18.5					# cd into the folder created

# First, we use the "distclean" target to clean up the source folder
make distclean					

# specify the architecture via the variable ARCH
make ARCH=$arch headers_check

# specify that we want the headers to be installed in $sysroot
make ARCH=$arch INSTALL_HDR_PATH=$sysroot headers_install

cd ..							# leave the source folder
```

For every section onwards, it is assumed that you will first untar the
specified package, change directory into it, and go on from there. Do
not delete the source folder after finishing a section, because it might
be needed later.

### binutils

The binutils package contains the linker, assembler, and a few other tools
to manipulate binary files. When you invoke `gcc` to compile C code,
`gcc` actually uses tools from `binutils` to assemble the final binary.
This is why binutils is built first, as `gcc` needs it in order to compile
code, _and_ both `gcc` and `musl-libc` performs several tests on the
binutils tools and determine what features to enable/disable.

The binutils built in this step will be installed in $install/bin and the
tool's name will all be prefixed with the target arch triplet
(`x86_64-linux-musl` in this case). This set of binutils tools will be used
by `gcc` in the finished cross toolchain later, as it is configured to run
on our host machine, but generates code for the target architecture. Even
though our host and targets are the same architecture, this is done to
isolate the target against possible contaminations from the host system.

```bash

mkdir build		# create a folder named "build" inside the source folder
cd build		# build binutils in this folder instead

../configure \			# the backslashe breaks 1 line into several
	--prefix=$install \
	--target=$target \
	--with-sysroot=$sysroot \
	--with-lib-path=$sysroot/lib \
	--disable-werror \		# prevents warnings from stopping the build
	--disable-nls		# disable internationalization

make -j4
make install	# install the compiled files
```
* --prefix

	This option specifies the location of installation. In this case, we
	want to install all the binutils to `$install`, which is `/opt/cross`

* --target

	This option specifies the target architecture that binutils will
	be built for.

* --with-sysroot

	This option specifies the location of the system root. This is where
	all the header files, libraries, etc. for the target system are
	contained. When linking binaries for the target, binutils tools will
	look into this sysroot directory for header files, etc.

* --with-lib-path

	This option explicitly states the directory to find libraries in when
	binutils tools are used. In this case, libraries will be installed
	in `$sysroot/lib`. This prevents the tools from accidentally linking
	a target binary to a library located on the host system, such as
	`/usr/lib`.

* make -j4

	Replace "4" with the number of CPU cores that your system has. This
	enables `make` to speed up the build job by working in parallel across
	multipul CPU cores.


### gcc (bootstrap compiler)

### musl-libc headers

### static libgcc

### complete musl-libc

### complete gcc

### testing the cross compiler
