# Setting up the build environment

In this section, we will prepare our host system for building Microdot
Linux.
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


# Setup the Initialization Scripts for Builder

Every time a user logs in, a set of scripts are automatically executed
to initialize the users command line environment (set up variables, etc).
By default, a _login shell_ is spawned when a user logs in, and this shell
is initialized by the `.bash_profile` script. Notice the leading dot in the
file name: it makes the file invisible if you run `ls` in a directory. This
is commonly done for configuration files that the user doesn't want to see
all the time. You can force `ls` to show invisible files by invoking it as
`ls -a`.

The commands placed in `.bash_profile` replaces the current _login shell_
with a new shell and with an empty environment except for the `HOME`,
`TERM`, and `PS1` variables. This ensures that no dangerous environmental
variables can leak in to this new shell.

> Note: we uses the `cat` command to write text to a file instead of using
> a text editor here. The syntax below tells `cat` to read input from STDIN
> until reading the characters `EOF`, and redirect all the entered text
> into `~/.bash_profile`.

```bash
cat > ~/.bash_profile << "EOF"
	exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
```

The newly spawned shell by the `.bash_profile` script is a non-login shell,
and it uses `.bashrc` to initialize it self. The `set +h` option turns off
the hashing functionality of the bash shell. This prevents bash from
remembering the paths to previously executed commands. We need this because
we will first be using the host system's programs, then we will use the
programs we built ourselves during the construction stage. If bash
remembers the old paths, then the new programs will be ignored. The `umask
022` command sets the user file creation mask, and this ensures that all
files we make can only be write to by ourselves, but can be read by others.

> Note: a hashtag character ("#") means that the rest of the line is a
> comment. It is used to insert useful information into the code snippits.
> Do not type the comment as a part of the command, as it will be ignored
> by the shell anyways.

```bash
cat > ~/.bashrc << "EOF"

	set +h
	umask 022

	host=x86_64-cross-linux-gnu
	target=x86_64-linux-musl
	arch=x86

	export host target arch
	unset CFLAGS

	install=/opt/cross
	sysroot=/opt/cross/$target
	targetfs=/opt/targetfs
	LC_ALL=POSIX	# turns off localization
	PATH=$install/bin:/bin:/usr/bin

	export install sysroot targetfs LC_ALL PATH

EOF		# tell cat that this is the end of input
```

The `host` and `target` variables contains two system triplets. Basically,
a system triplet specifies the information about a platform. Its format is
generally like this: `architecture-kernel-userspace`. Here, the architecture
for both the host and the target is `x86_64`. For the host triplet, we
added an extra word `cross` in there, to indicate that it is the platform
on which we are building the cross compiler toolchain. The kernel is the
same for both the host and the target, which is `linux`. Finally, the hosts
userspace applications are all using the GNU C library, which is why we put
`gnu` there. However, for the target, we are using a much smaller C library
called musl-libc, so we put `musl` there instead.

The `arch` variable contains the target hardware architecture. Here, `x86`
refers to a generic 64-bit PC architecture.

The `install` variable holds the location in which we would like all of our
compiled programs to be installed in. This prevents the program we build
from getting mixed into the hosts programs.

The `sysroot` variable contains the location of the "root file system" of
the target system. Of course, it is an incomplete and imagined root file
system. It contains the target architecture specific stuff, such as the
C library, etc. Our cross compilers will be installed in `$install` instead
of `$sysroot` because it is meant to run on our host, and not intended
for the target.

The `targetfs` variable points to the directory containing the files that
will form the bootable disk image that we will make later on. This
directory is not used until we get to the stage where we assemble the final
system.

The `PATH` variable holds the location that `bash` will check to find
programs. For example, if the program `ls` or `cat` is not located in
your `PATH`, then you wouldn't be able to execute them. Notice how we
put `$install/bin` before `/bin` and `/usr/bin`. We want the bash shell
to use our new tools immediately when we install them and ignore the
tools that came with the host system.


Finally, we let the current shell re-read the `~/.bash_profile` script,
so all the stuff that we just did can be applied.

```bash
source ~/.bash_profile
exit
```

# Change permission and ownership

Now, we will make `builder` the owner of the directories that we just made.

> Note: run the following commands as the root user
```bash
chown builder:builder -R /opt/cross 
chown builder:builder -R /opt/targetfs
chmod 755 -R /opt/cross
```
