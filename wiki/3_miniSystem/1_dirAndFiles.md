
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
mkdir -p $targetfs/{bin,sbin,dev,proc,sys}
mkdir -p $targetfs/etc/init.d
mkdir -p $targetfs/var/{lock,log}

# Create these directories with special permissions
install -dm 0750 $targetfs/root
install -dm 1777 $targetfs/tmp
install -dm 1777 $targetfs/var/tmp

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
please visit [this](www.c.com) link.

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

```bash
cat > $targetfs/etc/inittab << "EOF"

::sysinit:/etc/init.d/startup
::askfirst:-/bin/sh

::ctrlaltdel:/sbin/reboot

::shutdown:/bin/umount -a -r
::restart:/sbin/init


# ::respawn:/sbin/getty -L ttyS0 9600 vt100

EOF
```

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

