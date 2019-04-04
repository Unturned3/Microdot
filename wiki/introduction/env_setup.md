# Setting up the build environment
--------------------------------------------------------------------------

```bash
# mkdir /opt/targetfs
# mkdir /opt/cross
# mkdir /opt/cross/src
```

set the "source_dir" variable to point to where
all the tar archives are located, and copy all
the archives into /opt/cross/src

```bash
# source_dir=/path/to/sources
# cp $source_dir/* /opt/cross/src
```


Create the builder user
--------------------------------------------------------------------------

```bash
# groupadd builder
# useradd -s /bin/bash -g builder -m -k /dev/null builder
# passwd builder
# su - builder
```


# Set up ~/.bash\_profile and ~/.bashrc
--------------------------------------------------------------------------

$ cat > ~/.bash_profile << "EOF"
	exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF

$ cat > ~/.bashrc << "EOF"
	set +h
	umask 022

	alias ls="ls --color=auto"
	alias grep="grep --color=auto"

	host=x86_64-cross-linux-gnu
	target=x86_64-linux-musl
	arch=x86

	export host target arch
	unset CFLAGS

	install=/opt/cross
	sysroot=/opt/cross/$target
	targetfs=/opt/targetfs
	LC_ALL=POSIX
	PATH=$install/bin:/bin:/usr/bin

	export install sysroot targetfs LC_ALL PATH
EOF

$ source ~/.bash_profile
$ exit

Change permission and ownership
--------------------------------------------------------------------------
===============================

# chown builder:builder -R /opt/cross 
# chown builder:builder -R /opt/targetfs
# chmod 755 -R /opt/cross
