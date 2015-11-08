# Notes on AUFS

These notes are made based on AUFS v3.x-rcN-201111205

## Concepts

### Branch

A branch is nothing but a directory on the system. Any filesystem can be a
branch, except sysfs, procfs and unionfs.

* If you specify such filesystems as an aufs branch, aufs will return an error
saying it is unsupported. If you enable `CONFIG_AUFS_ROBR`, you can use aufs as 
a non-writable branch of another aufs.

### xino

xino stands for "external inode number".

### whiteout

Whiteout files and directories store information about entries that were
deleted in an AUFS mount but not in the read-only branch.

## State

The AUFS state can be viewed by the sysfs. It is found under `/sys/fs/aufs`:

```
$> ls -la /sys/fs/aufs
-r--r--r-- 1 root root 4096 Oct 22 00:24 config
drwxr-xr-x 2 root root    0 Oct 22 00:23 si_8beacfc1019d8b7
...
```

The `si_<ID>` entries found in `/sys/fs/aufs` represent different AUFS mounts.
To find the SI ID of an AUFS mount, simple list the current mount points:

```
$> cat /proc/mounts | grep aufs
none /aufs/example_1/target aufs rw,relatime,si=8beacfc1019d8b7 0 0
...
```

The branches are not shown in the mount options listed in `/proc/mounts`.
However given the SI ID, one can list the branches of a certain AUFS mount:

```
$> ls -la /sys/fs/aufs/si_8beacfc1019d8b7
-r--r--r-- 1 root root 4096 Oct 22 00:24 br0
-r--r--r-- 1 root root 4096 Oct 22 00:24 br1
-r--r--r-- 1 root root 4096 Oct 22 00:24 xi_path
```

Where the `br*` files contain the `BRANCH` syntax:

```
$> cat /sys/fs/aufs/si_8beacfc1019d8b7/br1
/aufs/example_1/ro=ro
```

### What is sysfs

`sysfs` is a virtual file system provided by the Linux kernel that exports
information about various kernel subsystems, hardware devices, and associated
device drivers from the kernel's device model to user space through virtual
files. In addition to providing information about various devices and kernel
subsystems, exported virtual files are also used for their configuring.

`sysfs` provides functionality similar to the `sysctl` mechanism found in
BSD operating systems, with the difference that `sysfs` is implemented as a
virtual file system instead of being a purpose-built kernel mechanism.

### What is procfs

The proc filesystem (`procfs`) is a special filesystem in Unix-like operating
systems that presents information about processes and other system information
in a hierarchical file-like structure, providing a more convenient and
standardized method for dynamically accessing process data held in the kernel
than traditional tracing methods or direct access to kernel memory.

Typically, it is mapped to a mount point named `/proc` at boot time. The proc
file system acts as an interface to internal data structures in the kernel.
It can be used to obtain information about the system and to change certain
kernel parameters at runtime (`sysctl`).

### What is the difference between sysfs and procfs

`proc` is the old one, it is more or less without rules and structure. And at
some point it was decided that proc was a little to chaotic and a new way was
needed. Then `sysfs` was created, and the new stuff that was added was put
into `sysfs` like device information. So in some sense they do the same, but
`sysfs` is a little bit more structured.

## Mounting

### Setting up the branches

* `br:BRANCH[:BRANCH ...]` is the mount option to setup the initial branches
of the AUFS mount.
* Each `BRANCH` placeholder should be following the branch syntax:
  - `BRANCH := dir_path[=permission[+attribute]]`
  - `permission := rw|ro|rr`
  - `attribute := wh|nolwh`
* A branch should not be shared as the writable branch between multiple aufs.
A readonly branch can be shared.
* The maximum number of branches is configurable at compile time (127 by
default).
* When an unknown permission or attribute is given, aufs sets `ro` to that
branch silently.

#### First branch

The first branch in the list is by default set as as `rw`. All changes made in
the AUFS mount will end up in the first branch as well.

### Create policy

The create policy defines where are the new files going to end up if there
are multiple `rw` branches. The values of the `create` mount option can be
the following:

1. `tdp` or `top-down-parent`
2. `rr` or `round-robin`
3. `mfs` or `most-free-space`

There are other policies as well. Additionally, there are polices that define
the internal copy-up in AUFS.

### User's direct branch access policy

`udba` can be:

1. `none`: which means that if a user makes a change directly to a branch,
there is not guarantee that/when this change will appear in the mount.
2. `reval`: every change is propagated. This has a performance hit on AUFS.
3. `notify`.

## References

1. http://aufs.sourceforge.net/aufs3/man.html
2. http://www.thegeekstuff.com/2013/05/linux-aufs/
3. http://lwn.net/Articles/265240/
4. http://unix.stackexchange.com/questions/4884/what-is-the-difference-between-procfs-and-sysfs
