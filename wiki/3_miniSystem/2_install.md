# Installing Basic Software

In this section we will install the essential system software programs into
`$targetfs`. The `$targetfs` directory forms the skeleton of our system, and
we will later "package" the contents into an image readable by the kernel.

## Clean Up the Source Directory

We've finished constructing the cross toolchain, and now we will remove the
directories extracted previously so we can work in a clean environment:

```bash
cd $install/src
rm -rf binutils-2.30 gcc-7.3.0 linux-4.18.5 musl-1.1.20
```

## libgcc

Install the `libgcc` support library, so other dynamic binaries compiled by
`gcc` can actually run on our final system.

> Note: libgcc is a part of the gcc-7.3.0 package. Unpack and change directory into gcc-7.3.0

```bash
tar -xf ../mpfr*
tar -xf ../mpc*
tar -xf ../gmp*

mv mpfr* mpfr
mv gmp* gmp
mv mpc* mpc

mkdir build
cd build

../configure \
	--prefix=$targetfs \
	--host=$target \
	--target=$target \
	--disable-multilib \
	--disable-bootstrap \
	--disable-nls \
	--disable-libmpx \
	--disable-libsanitizer \
	--disable-libmudflap \
	--enable-c99 \
	--enable-long-long \
	--enable-tls \
	--enable-languages=c,c++

make -jN all-target-libgcc	
make install-target-libgcc

$target-strip $targetfs/lib/libgcc_s.so.1
rm -r $targetfs/lib/gcc
```

# static vs dynamic system

You can either build a statically linked userspace or a dynamically linked one.
"Dynamically linked" refers to the situation where the programs in the userspace
all depend on some shared library (such as musl-libc). In this case, musl-libc needs
to be present in the final Microdot system, or else programs such as Busybox won't run.
"Statically linked" refers to the situation where the portion of the shared library is
directly copied into the program that you are compiling. For example, if we compile
Busybox as statically linked, it will copy a portion of musl-libc into the finished
Busybox binary, making it sort of "self-contained" in the sense that we don't need
musl-libc to be present on the final Microdot system.

An obvious downside to statically linking every userspace program is that there will 
be many redundant copies of the same code in every program. Dynamic linking reduces
the redundant space waste by packing the code that every program uses in a shared library.

For more information on this topic, visit [link1](https://stackoverflow.com/questions/1993390/static-linking-vs-dynamic-linking) and 
[link2](https://cs-fundamentals.com/tech-interview/c/difference-between-static-and-dynamic-linking).

However, for our system, we only have 1 binary, which is Busybox. Statically linking
Busybox in this case will result in an overall smaller root file system, but dynamic
linking (and installing musl-libc) will make expanding the system & adding more software
easier. You can choose either option here.

# musl-libc

Install musl-libc onto the target system, so dynamically linked binaries can
run. If you are building a statically-linked system with no shared libs, skip
this step and continue from the busybox section below.

> Note: unpack and change directory into the musl-1.1.20 directory

```bash
./configure \
	CROSS_COMPILE=$target- \
	--prefix=/ \
	--disable-static \
	--target=$target

make -j4
make DESTDIR=$targetfs install-libs
```

# busybox

Busybox is the main component of Microdot's userspace. It contains hundreds of
Linux utilities combined into one tiny executable. Linking it with `musl-libc`
results in an even smaller binary compared to other libraries.

> Note: unpack and change directory into the busybox-1.30.0 directory

```bash
make distclean
make ARCH=$arch defconfig
```

Now, run the following command to launch the "menuconfig" interface. We need to
configure busybox to install everything to `/bin` and `/sbin`, and ignore `/usr`.

```bash
make ARCH=$arch menuconfig
```

You should now see a text-based configuration interface. You can use the arrow keys,
enter key, spacebar, and ESC key to navigate. "Y" means to select the option, and "N"
means to deselect it. We aren't using the `/usr` directory for the sake of simplicity,
and we are disabling `linuxrc` because it is a feature designed to provide
compatibility for the `initrd` (initial ram disk) mechanism. Initramfs is preferred
over initrd in our case.

```
Settings --->
	Don't use /usr: Y
Init Utilities --->
	linuxrc: support running init from initrd (not initramfs): N

# If you are building a statically-linked busybox, select the following as well:

Settings --->
	Build static binary (no shared libs): Y
```

Save and exit the configuration interface. Now compile & install busybox:

```bash
make -jN ARCH=$arch CROSS_COMPILE=$target- CONFIG_PREFIX=$targetfs install
```

Lastly, we have to "setuid" the busybox exectuable. Normally, when a binary
is run, its privileges are the same as the user who started it. However, if
a binary has the setuid bit set, it will execute with the privilege
of the user who owns it. In our case, `root` owns the busybox binary. We need
to set the "setuid" bit of busybox or else commands like `passwd` won't work.
Busybox drops the elevated privileges for most other applets, so this isn't a
security risk.

```bash
chmod 4755 $targetfs/bin/busybox	# the leading '4' is the setuid bit
```

You have to change the ownership of the `busybox` binary to root. However,
you can only do that if you have root privileges.

```bash
su	# login as root uesr
chown root:root -R /opt/targetfs
exit
```





