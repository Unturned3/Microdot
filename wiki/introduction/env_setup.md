# Setting up the build environment

First, we need to create a few directories in `/opt` to store the all
the files needed during the build. The `targetfs` directory stands for
"target file system", and it will contain files that forms the bootable
disk image. The `cross` directory contains our cross compilation toolchain
that we will build in the next section. The `cross/src` directory contains
all the source packages that we will be compiling. 

> Note: the `#` symbol indicates that you should be typing the command
> as the root user. On the other hand, the `$` symbol indicates that the
> commands should be executed as a normal user (without any elevated 
> privileges).

```bash
# mkdir /opt/targetfs
# mkdir /opt/cross
# mkdir /opt/cross/src
```


# Create the Builder User

Instead of using your own user account to build Microdot, we will create
a dedicated user named `builder` to do this. First, your default user
account will include a lot of environmental variables, and some of which
can interfere with our build process. Adding a new user account enables
us to eliminate dangerous/unused environmental variables. Also, you
probably don't want your default user's home directory to be cluttered
with a pile of temporary build files. 

First, we add a group named `builder`, then we use `useradd` to actually
create the new user. The `-s` option specifies the login shell that
`builder` will be using. The `-g` option specifies the group that `builder`
belongs to. The `-m` option tells `useradd` to create a home directory for
the new user. The `-k` option specifies the `skeletal files` that will
be copied to the new user's home directory. Here, we tell it to copy
nothing by specifying `/dev/null`, otherwise it might pollute the new home
directory. Lastly, we specify `builder` as the name of the new user.

```bash
# groupadd builder
# useradd -s /bin/bash -g builder -m -k /dev/null builder
# passwd builder
# su - builder
```


# Set up ~/.bash\_profile and ~/.bashrc

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

# Change permission and ownership

# chown builder:builder -R /opt/cross 
# chown builder:builder -R /opt/targetfs
# chmod 755 -R /opt/cross
