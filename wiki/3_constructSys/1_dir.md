
# Creating Files & Directories

Before proceeding onto installing software using our cross toolchain, we will
first create the skeleton of our Microdot system. That is, we need to create
a few important directories that will be populated later. We will name these
folders according to the established convention. For more details regarding the
Linux directory structures, check out the following links:

* [https://www.a.com](a)
* [https://www.b.com](b)
* [https://www.c.com](c)

The skeleton will be built in the `$targetfs` directory. The contents of this
folder will later be copied onto a bootable disk image.

```bash
# essential directories
mkdir -p $targetfs/{bin,sbin,dev,etc,proc,sys}
mkdir -p $targetfs/{mnt,opt,root,home,usr/{lib,bin,sbin}}
mkdir -p $targetfs/var/{lib,spool,lock,cache,log,run}

# link all "lib" and "lib64" folders to "usr/lib"
ln -sfv $targetfs/usr/lib $targetfs/lib
ln -sfv $targetfs/usr/lib $targetfs/lib64
ln -sfv $targetfs/usr/lib $targetfs/usr/lib64

# Create these directories with special permissions
install -dm 0750 $targetfs/root
install -dm 1777 $targetfs/tmp
install -dm 1777 $targetfs/var/tmp
```

Now we will create a few important files that controls how the system runs:

```bash
ln -sfv ../proc/mounts $targetfs/etc/mtab

cat > $targetfs/etc/passwd << "EOF"
root::0:0:root:/root:/bin/ash
EOF

cat > $targetfs/etc/group << "EOF"
root:x:0:
bin:x:1:
EOF

touch $targetfs/var/log/lastlog
chmod -v 664 $targetfs/var/log/lastlog
```
