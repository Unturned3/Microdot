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
	--disable-mudflap \
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
enter key, spacebar, and ESC key to navigate. Select the following option:
```
Settings --->
	[*] Don't use /usr
```

Save and exit the configuration interface. Now compile & install busybox:

```bash
make -jN ARCH=$arch CROSS_COMPILE=$target- CONFIG_PREFIX=$targetfs install
```








