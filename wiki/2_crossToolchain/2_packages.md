# Required Packages

The following packages will be used when we build the
cross compilation toolchain. Each package's functionality will
be explained individually later. You need to download
these files and store them inside the /opt/cross/src directory.

* [binutils-2.30.tar.gz](https://ftp.gnu.org/gnu/binutils/binutils-2.30.tar.gz)
* [linux-4.18.5.tar.gz](https://mirrors.edge.kernel.org/pub/linux/kernel/v4.x/linux-4.18.5.tar.gz)
* [mpfr-4.0.1.tar.xz](https://ftp.gnu.org/gnu/mpfr/mpfr-4.0.1.tar.xz)
* [mpc-1.1.0.tar.gz](https://ftp.gnu.org/gnu/mpc/mpc-1.1.0.tar.gz)
* [gmp-6.1.2.tar.xz](https://ftp.gnu.org/pub/gnu/gmp/gmp-6.1.2.tar.xz)
* [gcc-7.3.0.tar.gz](https://ftp.gnu.org/gnu/gcc/gcc-7.3.0//gcc-7.3.0.tar.gz)
* [musl-1.1.20.tar.gz](http://git.musl-libc.org/cgit/musl/snapshot/musl-1.1.20.tar.gz)
* [busybox-1.30.0.tar.bz2](https://busybox.net/downloads/busybox-1.30.0.tar.bz2)

Alternatively, you can find a mirror site that is closer to your geographical location
and download the packages there, instead of using the links given above.

> Note: you must use the _exact_ version specified for
> each package, as it has been observed that other versions
> might break the toolchain.)
