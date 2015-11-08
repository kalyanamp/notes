# Linux init systems

In Unix-based computer operating systems, `init` (short for initialization) is
the first process started during booting of the computer system. Init is a
daemon process that continues running until the system is shut down. It is the
direct or indirect ancestor of all other processes and automatically adopts
all orphaned processes.

Init is started by the kernel using a hard-coded filename; a kernel panic will
occur if the kernel is unable to start it. Init is typically assigned process
identifier _1_.

Several replacement init implementations have been created, attempting to
address design limitations in the standard versions. These include:

* launchd: MacOS X init process
* Service Management Facility: developed for Solaris
* systemd: Most recent, adopted by Ubuntu
* Upstart: Previous Ubuntu's init system

As of March 2015, systemd has been adopted by major Linux distributions
although it remains controversial. The critisism against systemd comes from
the fact that it is considered overengineered.

## History

### Research Unix/BSD style

**Research Unix** init ran the initialization shell script located in `/etc/rc`
then launched `getty` on terminals under the control of `/etc/ttys`.
There are no runlevels; the `/etc/rc` file determines what programs are run by
init. The advantage of this system is that it is simple and easy to edit
manually. However, new software added to the system may require changes to
existing files that risk producing an unbootable system.

**BSD** init was, prior to 4.3BSD, the same as Research UNIX's init; in 4.3BSD,
it added support for running a windowing system such as X on graphical
terminals under the control of `/etc/ttys`. To remove the requirement to edit
`/etc/rc`, BSD variants have long supported a site-specific `/etc/rc.local`
file that is run in a sub-shell near the end of the boot sequence.

A **fully modular** system was introduced with NetBSD 1.5 and ported to
FreeBSD 5.0 and successors. This system executes scripts in the `/etc/rc.d`
directory. Unlike System V's script ordering, which is derived from the
filename of each script, this system uses explicit dependency tags placed
within each script. The order in which scripts are executed is determined by
the `rcorder` script based on the requirements stated in these tags.

#### getty

`getty`, short for "get teletype", is a Unix program running on a host
computer that manages physical or virtual terminals (TTYs). When it detects a
connection, it prompts for a username and runs the 'login' program to
authenticate the user.

#### /etc/ttys

The `/etc/ttys` file contains information that is used by various routines to
initialize and control the use of terminal special files. This information is
read with the `getttyent` library routines. There is one line in the ttys
file per special device file.

##### Fields

* Name of the terminal special file as it is found in `/dev`
* Command to execute for the line, usually `getty`.
* Type of terminal usually connected to that TTY line.
* The remaining fields set flags in the `ty_status` entry (see `getttyent`)
or specify a window system process that `init` will maintain for the
terminal line.

### System-V style

When compared to its predecessors, AT&T's UNIX System III introduced a new
style of system startup configuration, which survived (with modifications)
into UNIX System V and is therefore called the "SysV-style init".

At any moment, a running System V is in one of the predetermined number of
states, called **runlevels**. At least one runlevel is the normal operating
state of the system; typically, other runlevels represent:

* single-user mode (used for repairing a faulty system)
* system shutdown
* and various other states.

Switching from one runlevel to another causes a per-runlevel set of scripts to
be run, which typically mount filesystems, start or stop daemons, start or
stop the X Window System, shutdown the machine, etc.

The `/etc/inittab` file, defines what each configured runlevel does in a given
system. It sets the default runlevel with the `:initdefault:` entry.

On most systems, all users can check the current runlevel with either the
`runlevel` or `who -r` command. The root typically changes the current runlevel
by running the `telinit` or `init` commands.

## Replacements

Traditionally, one of the major drawbacks of init is that it starts tasks
**serially**, waiting for each to finish loading before moving on to the next.
When startup processes end up I/O blocked, this can result in long delays
during boot.

### launchd

In computing, launchd, a unified, open-source service-management framework,
starts, stops and manages daemons, applications, processes, and scripts in
Apple OS X environments.

There are two main programs in the launchd system: launchd and launchctl.

**launchd** manages the daemons at both a system and user level. Similar to
xinetd, launchd can start daemons on demand. Similar to watchdogd, launchd
can monitor daemons to make sure that they keep running. launchd also has
replaced init as PID 1 on Mac OS X and as a result it is responsible for
starting the system at boot time.

Configuration files define the parameters of services run by launchd. Stored
in the LaunchAgents and LaunchDaemons subdirectories of the Library folders,
the property list-based files have approximately thirty different keys that
can be set. Launchd itself has no knowledge of these configuration files or
any ability to read them - that is the responsibility of "launchctl".

**launchctl** is a command line application which talks to launchd using IPC
and knows how to parse the property list files used to describe launchd jobs,
serializing them using a specialized dictionary protocol that launchd
understands. launchctl can be used to load and unload daemons, start and
stop launchd controlled jobs, get system utilization statistics for launchd
and its child processes, and set environment settings.

### Service Management Facility

Service Management Facility (SMF) is a feature of the Solaris operating system
that creates a supported, unified model for services and service management on
each Solaris system and replaces `init.d` scripts. SMF introduces:

* Dependency order. Services sometimes depend on one another for proper
operation, and a robust system should know each service's dependencies.
* Configurable boot verbosity
* Delegation of tasks to non-root users. A service can be configured to run
within a limited set of privileges, rather than as the all-powerful root user.
* Parallel starting of services. This speeds up the boot process by starting
multiple services simultaneously.
* Automatic service restart after failure. Works in conjunction with the
Solaris Fault Manager, allowing software recovery in the event of hardware
faults (cpu, memory), admin error such as accidental kills, and software core
dumps. If a service fails, it needs to be corrected before other services that
depend upon it are affected.

### upstart

Upstart is an event-based replacement for the traditional init daemon. The
traditional init is unable to handle various non-startup-tasks on a modern
desktop computer elegantly, including:

* The addition or removal of USB flash drives and other portable storage /
network devices while the machine is running.
* The discovery and scanning of new storage devices, without locking the
system, especially when a disk may not even power on until it is scanned.
* The loading of firmware for a device, which may need to occur after it is
detected but before it is usable.

Upstart's event-driven model allows it to respond to events asynchronously as
they are generated.

### systemd

systemd is a suite of basic building blocks for a Linux system. It provides a
*system and service manager* that runs as PID 1 and starts the rest of the
system. systemd provides:

* aggressive parallelization capabilities
* uses socket and D-Bus activation for starting services
* offers on-demand starting of daemons
* keeps track of processes using Linux control groups
* supports snapshotting and restoring of the system state
* maintains mount and automount points
* implements an elaborate transactional dependency-based service control logic

systemd supports SysV and LSB init scripts and works as a replacement for
sysvinit. Other parts include:

* a logging daemon
* utilities:
  - control basic system configuration like the hostname, date, locale
  - maintain a list of logged-in users
  - run containers and virtual machines
  - system accounts
  - runtime directories and settings
* daemons:
  - simple network configuration
  - network time synchronization
  - log forwarding
  - name resolution.

See Lennart's blog story for a longer introduction, and the three status
updates since then. Also see the Wikipedia article. If you are wondering
whether systemd is for you, please have a look at this comparison of init
systems by one of the creators of systemd.

#### D-Bus

D-Bus is an inter-process communication (IPC) and remote procedure call (RPC)
mechanism that allows communication between multiple computer programs (that
is, processes) concurrently running on the same machine.

Every connection to a bus is identified in the context of D-Bus by what is
called a bus name.[3] A bus name consists of two or more dot-separated strings
of letters, digits, dashes, and underscores. An example of a valid bus name is
`org.freedesktop.NetworkManager`.

## References

1. https://en.wikipedia.org/wiki/Init
2. http://www.freedesktop.org/wiki/Software/systemd/
3. http://0pointer.de/blog/projects/why.html
4. http://0pointer.de/blog/projects/systemd.html
