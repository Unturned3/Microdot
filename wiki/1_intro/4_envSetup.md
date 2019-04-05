# Setting up the build environment

First, we need to create a few directories in `/opt` to store the all
the files needed during the build. The `targetfs` directory stands for
"target file system", and it will contain files that forms the bootable
disk image. The `cross` directory contains our cross compilation toolchain
that we will build in the next section. The `cross/src` directory contains
all the source packages that we will be compiling. 

> Note: execute the following commands as the root user, since you (as a
> normal user) wouldn't have write access to the `/opt` directory.

```bash
mkdir /opt/targetfs
mkdir /opt/cross
mkdir /opt/cross/src
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
the new user. The `-k` option specifies the "skeletal files" that will
be copied to the new user's home directory. Here, we tell it to copy
nothing by specifying `/dev/null`, otherwise it might pollute the new home
directory. Lastly, we specify `builder` as the name of the new user.

The `passwd` command sets a password for the new user.
> Note: you cannot see the password as you type it in. Your keyboard is
> working fine!

Finally, the `su - builder` command allows us to change our user to
`builder` without logging out and logging in again, which is quite
convenient.
> Note: the dash in `su - builder` is important! It will spawn a login
> shell for the new user instead of just simply substituting our identity.
> This will reset all the old environmental variables and provide us with
> a clean environment. If you forget the dash then the point of creating
> a new user is defeated.

```bash
groupadd builder
useradd -s /bin/bash -g builder -m -k /dev/null builder
passwd builder
su - builder
```


# Setup the Initialization Scripts for `Builder`

```bash
cat > ~/.bash_profile << "EOF"
	exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
```

```bash
cat > ~/.bashrc << "EOF"

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
```

```bash
source ~/.bash_profile
exit
```

# Change permission and ownership

> Note: run the following commands as the root user
```bash
chown builder:builder -R /opt/cross 
chown builder:builder -R /opt/targetfs
chmod 755 -R /opt/cross
```
