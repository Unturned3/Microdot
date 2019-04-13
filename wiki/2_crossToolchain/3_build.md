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

First, we need to substitute our identity as the `builder` user. This
allows us to work in a clean environment with access to the variables
that we previously defined in the initialization scripts.

Note that if you exit the shell that you're working in or shut down your
system during the middle of the build process, you need to run the
substitute user command again before resuming from where you left off.

```bash
su - builder
```

Then, we will setup a few directories and links in the cross compiler
building area for convenience purposes. The `$sysroot` folder contains
an imagined root file system for our target, and it will contain target-
specific files, such as its C library. The `lib` folder contains the
various libraries and support files that our tools will be using. We then
point `lib64` to `lib`, because sometimes the build system of a package
will install its contents into the `lib64` folder when we are building
on a 64 bit system. We want to keep all the files in `lib` so we link them.
We also point `$sysroot/usr` to `.` because we don't want anything to be
installed into a separate `usr` directory.

```bash

mkdir -pv $sysroot
mkdir -pv $install/lib
mkdir -pv $sysroot/lib

ln -sfv $install/lib $install/lib64
ln -sfv $sysroot/lib $sysroot/lib64
ln -sfv . $sysroot/usr

cd $install/src		# go to the directory containing all the packages
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

> Unpack and change directory into the binutils-2.30 package

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

make -jN
make install	# install the compiled files
```
* `--prefix`

	This option specifies the location of installation. In this case, we
	want to install all the binutils to `$install`, which is `/opt/cross`

* `--target`

	This option specifies the target architecture that binutils will
	be built for.

* `--with-sysroot`

	This option specifies the location of the system root. This is where
	all the header files, libraries, etc. for the target system are
	contained. When linking binaries for the target, binutils tools will
	look into this sysroot directory for header files, etc.

* `--with-lib-path`

	This option explicitly states the directory to find libraries in when
	binutils tools are used. In this case, libraries will be installed
	in `$sysroot/lib`. This prevents the tools from accidentally linking
	a target binary to a library located on the host system, such as
	`/usr/lib`.

* `make -jN`

	Replace N with the number of CPU cores that your system has. This
	enables `make` to speed up the build job by working in parallel across
	multipul CPU cores. For example, if you have a dual-core system, then
	you would type `make -j2`.


### gcc (bootstrap compiler)

At this stage, we do not have a functional C library yet, so a complete
`gcc` cannot be built (as it relies on certain parts of the libc). Instead,
we will build a minimum gcc compiler called the `bootstrap compiler`, and
use it to build the libc headers, the `libgcc` support library, and the
complete C library. This minimum compiler has no dependency on the libc.

> Unpack and change directory into the gcc-7.3.0 package

`gcc` does depend on a few external libraries to implement certain functions.
These external libraries doesn't depend on anything else, so we can just
build them directly with gcc here. We will unpack the libraries into the
source folder of `gcc`:

```bash

tar -xf ../mpfr\*
tar -xf ../mpc\*
tar -xf ../gmp\*

```

Then we have to "clean up" the names by leaving just the letters and
removing the version number. For example, we will rename `mpfr-4.0.1` as
`mpfr`.

```bash
mv mpfr\* mpfr
mv mpc\* mpc
mv gmp\* gmp
```

`gcc` build system will automatically detect the presence of these
directories and compile them before compiling `gcc` it self. Now, we
will configure and build the bootstrap compiler. Again, we do this in a
separate directory:

```bash

mkdir build
cd build

../configure \
	--target=$target \
	--prefix=$install \
	--with-newlib \
	--without-headers \
	--with-sysroot=$sysroot \
	--with-native-system-header-dir=/include \
	--enable-shared \
	--enable-languages=c,c++ \	# we only want a c/c++ compiler right now
	--disable-nls \
	--disable-decimal-float \
	--disable-libgomp \
	--disable-libmudflap \
	--disable-libssp \
	--disable-libatomic \
	--disable-libquadmath \
	--disable-libvtv \
	--disable-libsanitizer \
	--disable-libstdcxx \
	--disable-threads \
	--disable-multilib
```

* `--with-newlib`

	Tells `gcc` to not rely on any C libraries

* `--without-headers`

	Tells `gcc` to not use any of the hosts existing header files


* `--with-native-system-header-dir`

	Tells `gcc` to look for headers in this directory. Note that
	this directory is relative to the previously set `--with-sysroot`
	option, so `/include` actually refers to `$sysroot/include`

* `--enable-shared`

	Tells `gcc` to build libraries as shared (aka. dynamic). This
	affects the `libgcc` and `libstdc++` libraries later, and makes
	it possible to dynamically link to them.

* `--disable-xxxxxx`

	Tells `gcc` to disable all these features. They are not needed
	when building a bootstrap compiler, and it would waste a lot of
	build time if we don't disable them.

Compile & Install the bootstrap compiler:

```bash
make -jN all-gcc	# only build the gcc compiler, nothing else
make install-gcc	# only install what we built
```

If you look under `$install/bin`, you would see that `gcc` and various
other tools has been installed in there, all prefixed with our target
architecture triplet, which is `x86_64-linux-musl`. If you were 
to invoke `$target-gcc` and try to compile some C code, you would be
greeted with a load of error messages saying that certain files are
missing. This is because our bootstrap compiler has no access to any
part of the C library, which makes it impossible to compile a runnable
binary. However, this bootstrap compiler can compile _freestanding_ code,
that is, code that does not use the C library at all. The Linux kernel is
an example of _freestanding_ code. Yes, we can use this `gcc` to compile
the kernel right now if we want to, but there's no point in doing it.


### musl-libc headers

In this step we will install the header files provided by `musl-libc`. The
headers are needed for us to build our compiler support library (`libgcc`)
in the next step. We will tell the build system of `musl-libc` to use our
previously made bootstrap compiler instead of the one on our host system.

> Unpack and change directory into the musl-1.1.10 package

```bash
./configure \
	CROSS_COMPILE=$target- \
	--prefix=/ \
	--target=$target
```

* `CROSS_COMPILE=$target-`

	We use this variable to specify the `cross compiler prefix` that the
	build system will use. So instead of invoking just `gcc`, it will
	prepend the prefix that we provided, so `gcc` turns into `$target-gcc`,
	which is `x86_64-linux-musl-gcc` in this case. This allows the build
	system to use our bootstrap compiler.

* `--prefix=/`

	The `prefix` option in `musl-libc` doesn't seem to be able to control
	the installation location. So we set it to `/` here, and rectify it
	later via the `DESTDIR` variable during the installation step.

Compile and install the `musl-libc` headers:

```bash
make -jN DESTDIR=$sysroot install-headers
```

Now, if you look into `$sysroot/include`, you should see all the familiar
C header files like `stdio.h` and `stdlib.h`, and so on. These headers
will be used when building `libgcc` and other binaries later on.


### static libgcc

`libgcc` is a low-level compiler support library and it is used by all
dynamic binaries produced by `gcc`. It contains "support code" that
implements certain features, such as 64-bit division, etc. `musl-libc`
requires `libgcc`, so we will build a static one here in order to construct
`musl-libc` in the next step.

> Change directory into the previously unpacked `gcc` folder. `libgcc` is a
> part of the `gcc` package.

```bash
cd build
make -jN MAKE="make enable_shared=no" all-target-libgcc
make -jN MAKE="make enable_shared=no" install-target-libgcc
```

* `MAKE="make enable_shared=no"`

	The `make` command will recursively pass the `MAKE` variable to itself
	when it builds certain subtasks required by a target. We pass the
	`enable_shared=no` flag here to make sure only a static `libgcc` is
	built.

* `all-target-libgcc`

	This tells `make` to only build `libgcc` and nothing else.


### complete musl-libc

Now we are ready to build the final C library.

> Change directory into the previously unpacked `musl` folder.

```bash
make -jN \
	CC=$target-gcc \
	LIBCC=$install/lib/gcc/$target/*/libgcc.a \
	DESTDIR=$sysroot \
	all

make DESTDIR=$sysroot install
```

* `CC=$target-gcc`

	This tells the build system of `musl-libc` to use `$target-gcc` as the
	compiler when compiling the source code.

* `LIBCC=...`

	This tells the build system of `musl-libc` where is the static libgcc
	located.

After running `make install`, you should see files such as `libc.so` inside
the `$sysroot/lib` directory. `libc.so` is our `musl` C library (.so stands
for "shared object", aka. a dynamic library). Binaries built using our cross
compiler toolchain will be linked to this library instead of our host's
`glibc` (GNU C library).

### complete gcc

Now we can build the final `gcc` compiler with less `--disable-xxx` flags,
along with the final dynamic `libgcc` and the C++ standard library,
`libstdcxx`. First, delete the previously extracted `gcc-7.3.0` directory.

```bash
rm -rf $install/src/gcc-7.3.0	# be careful not to mistype anything
```

Then, extract the `gcc` source with `tar -xf` as before, change directory
into it, and extract the three dependency libraries:

```bash
tar -xf ../mpfr\*
tar -xf ../mpc\*
tar -xf ../gmp\*

mv mpfr\* mpfr
mv mpc\* mpc
mv gmp\* gmp
```

Configure & build gcc:

```bash
mkdir build
cd build

../configure \
	--prefix=$install \
	--build=$host --host=$host --target=$target \
	--enable-languages=c,c++ --enable-c99 --enable-long-long \
	--enable-shared --without-newlib \
	--disable-libmudflap --disable-multilib --disable-libsanitizer \
	--disable-bootstrap --disable-nls --with-sysroot=$sysroot

make -jN all-gcc all-target-libgcc all-target-libstdc++-v3
make install-gcc install-target-libgcc install-target-libstdc++-v3
```

* `--build=xxx, --host=xxx, --target=xxx`

	This three flags specifies the architecture of the machine that `gcc`
	is being _built_ on, the machine that `gcc` will _run_ on, and the
	machine that `gcc` will _generate code for_, respectively. We are both
	building and running `gcc` on `x86_64-cross-linux-gnu`, and we want
	`gcc` to generate code for `x86_64-linux-musl`, as that is the
	architecture of Microdot Linux.

* `all-target-libstdc++-v3`

	This tells `make` to build the C++ standard library as well.

After installing everything, you should find `$target-gcc` and `$target-g++`
and a few other tools installed inside `$install/bin`, and the C++ standard
library & headers installed in `$sysroot`. Now we can use this complete
toolchain to compile binaries for Microdot.

### testing the cross compiler

Instead of using `gcc` (which calls the gcc installed on the
host system), we will invoke `$target-gcc`, which translates
to `x86_64-linux-musl-gcc`. This is our cross compiler installed
in the $install/bin directory.

First, write a simple C program in builder's home directory:
(we want to keep the cross toolchain directory clean)

```bash
cd ~
cat > test.c << "EOF"

#include <stdio.h>
int main()
{
	printf("Hello, world!\n");
	return 0;
}

EOF
```

Then, invoke "$target-gcc" to compile this C program. The "-o test" option
tells gcc to name the output executable as "test". The "--static" option
tells gcc to statically link the executable (aka. copy the musl-libc
contents into the final executable file). If we compile the binary
dynamically, it will be linked to `musl-libc`. However, `musl-libc` is only
present inside the `$sysroot/lib` and not installed to the host systems
`/lib` directory. So if we were to run this program, then it wouldn't be
able to find `musl-libc`.

```bash
$target-gcc test.c -o test --static
```

Now run the program, and you should see "Hello, world!" on the screen.

./test

### You Made It!

Congratulations! This is what I consider to be the hardest and most
agitating part of the build process. In the next section, we will utilize
this toolchain to add software to our Microdot system. Then after that,
we will go on to configure our first Linux kernel.
