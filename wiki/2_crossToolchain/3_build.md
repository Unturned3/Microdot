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

### linux kernel headers

First, we will install the Linux kernel headers into the `$sysroot`
directory. The kernel headers can allow an application to know what
features are enabled and usable in the kernel's interface. The headers
are needed by `busybox` because it implements a lot of its functions
by utilizing the kernel's API. We have to install the headers first,
because installing it later would cause it to remove vital files installed
in the $sysroot by `musl-libc` and `gcc`.

### binutils

### gcc (bootstrap compiler)

### musl-libc headers

### static libgcc

### complete musl-libc

### complete gcc

### testing the cross compiler
