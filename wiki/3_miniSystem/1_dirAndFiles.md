
# Constructing the Skeleton

Before proceeding onto installing software using our cross toolchain, we will
first create the skeleton of our Microdot system. That is, we need to create
a few important directories and files, and we'll name these
folders according to the established convention. The skeleton will be built in
the `$targetfs` directory. The contents of this folder will later be copied
onto a bootable disk image.

The directory structure that we will construct here is a greatly simplified
version of the one specified by the Filesystem Hierarchy Standard, with a bunch
of folders omitted. For more details regarding the Linux directory structures,
check out the following links:

* [PUT LINK HERE](https://www.a.com)
* [PUT LINK HERE](https://www.b.com)

# Important Directories

```bash
# essential directories
# note that we are not using the /usr directory
mkdir -p $targetfs/{bin,sbin,dev,proc,sys,lib}
mkdir -p $targetfs/etc/init.d
mkdir -p $targetfs/var/{lock,log}

# Create these directories with special permissions
install -dm 0750 $targetfs/root   # 0750: rwxr-x---
install -dm 1777 $targetfs/tmp    # 1777: rwxrwxrwx, with sticky bit set
install -dm 1777 $targetfs/var/tmp    # same as above

# link "lib64" to "lib" so all libraries can be installed neatly in one place
ln -sfv $targetfs/lib $targetfs/lib64
```

# Configuration & Log Files

`/etc/mtab` is a file that shows a list of mounted file systems, and
`/proc/mounts` contains exactly that information, so we symlink these two
together.
```bash
ln -sfv ../proc/mounts $targetfs/etc/mtab
```

The `/etc/passwd` file is a text-based database of information about users, such
as their user name, user/group ID number, login shell, and so on. For now, we
only need the root user in here. For more information about `/etc/passwd`,
please visit [this](www.c.com)(DEADLINK: FIXME) link.

```bash
cat > $targetfs/etc/passwd << "EOF"
root::0:0:root:/root:/bin/ash
EOF
```

`/etc/group` is another text-based database of information about the groups
present on the system. For now, we only have the root group.

```bash
cat > $targetfs/etc/group << "EOF"
root:x:0:
EOF
```

The `/var/log/lastlog` file is used by programs like `init` or `login` to record
information about users that logged into the system. These programs won't write
to the `lastlog` file if it doesn't exist, so we create them here.

```bash
touch $targetfs/var/log/lastlog
chmod -v 664 $targetfs/var/log/lastlog
```

# System Initialization Scripts

We will use `busybox init` as our init system. When the kernel passes control
to `/sbin/init`, it will read `/etc/inittab` and carry out actions specified in
there. The `inittab` file format for busybox init is a bit different from the
traditional Sys V Init system. For more information, check out 
[this](https://git.busybox.net/busybox/tree/examples/inittab) page from
`busybox.net`.

```bash
cat > $targetfs/etc/inittab << "EOF"

::sysinit:/etc/init.d/startup   # execute a script that we will write ourselves
::respawn:/sbin/getty -L ttyS0 9600 vt100   # start a terminal on ttyS0 (serial line)
# ::askfirst:-/bin/sh

::ctrlaltdel:/sbin/reboot   # what do do when the ctrl-alt-del keys are pressed

::shutdown:/bin/umount -a -r  # what to do during shutdown
::restart:/sbin/init  # what to do during restart

EOF
```

This is the custom "startup" script that `busybox init` will run when the system starts.
The script performs a few tasks that's vital for the operation of the system, such as mounting
the virtual file systems, and printing some messages.

```bash
cat > $targetfs/etc/init.d/startup << "EOF"
#!/bin/sh

# mount pseudo file systems
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs devtmpfs /dev

# print some messages
echo Boot took $(cut -d' ' -f1 /proc/uptime) seconds.
echo Welcome to Microdot Linux!

EOF

chmod +x $targetfs/etc/init.d/startup
```

With the skeleton finished, we can now populate it with some software.

