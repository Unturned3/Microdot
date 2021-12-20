# Required Packages

The following packages will be used when we build the
cross compilation toolchain. Each package's functionality will
be explained later. You need to download these files and store
them inside the `/opt/cross/src` directory. The download links given
here are for your convenience; Alternatively, you can find mirror
sites that are closer to your geographical location to speed up the
downloads.

> __Important__: you must use the _exact_ version specified for
> each package, as it has been observed that certain incompatible
> versions may break the build process in obscure ways.

* [binutils-2.30.tar.gz](https://ftp.gnu.org/gnu/binutils/binutils-2.30.tar.gz)
* [linux-4.18.5.tar.gz](https://mirrors.edge.kernel.org/pub/linux/kernel/v4.x/linux-4.18.5.tar.gz)
* [mpfr-4.0.1.tar.xz](https://ftp.gnu.org/gnu/mpfr/mpfr-4.0.1.tar.xz)
* [mpc-1.1.0.tar.gz](https://ftp.gnu.org/gnu/mpc/mpc-1.1.0.tar.gz)
* [gmp-6.1.2.tar.xz](https://ftp.gnu.org/pub/gnu/gmp/gmp-6.1.2.tar.xz)
* [gcc-7.3.0.tar.gz](https://ftp.gnu.org/gnu/gcc/gcc-7.3.0//gcc-7.3.0.tar.gz)
* [musl-1.1.20.tar.gz](http://git.musl-libc.org/cgit/musl/snapshot/musl-1.1.20.tar.gz)
* [busybox-1.30.0.tar.bz2](https://busybox.net/downloads/busybox-1.30.0.tar.bz2)

> Note: previously it was thought that the ISL package is unnecessary. However, some users
> have reported `gcc` build errors due to missing the ISL package. The exact reason
> is unclear as of now. For more information, see [#27](https://github.com/Unturned3/Microdot/issues/27)
> and [#29](https://github.com/Unturned3/Microdot/pull/29).

* [isl-0.24.tar.xz](https://gcc.gnu.org/pub/gcc/infrastructure/isl-0.24.tar.bz2)

