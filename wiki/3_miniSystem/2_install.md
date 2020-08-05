# Installing Basic Software

In this section we will install the essential system software programs into
`$targetfs`, so the system can actually be used later.

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

# musl-libc

Install musl-libc onto the target system, so dynamically linked binaries can
run.

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





