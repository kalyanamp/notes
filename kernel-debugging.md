# Kernel debugging

## Ftrace

Ftrace is built into the Linux kernel. To enable it run:

```bash
echo 1 > /sys/kernel/debug/tracing/tracing_on
```

### Set the tracer

Next, set a ftrace tracer. To list the available tracers run:

```bash
cat /sys/kernel/debug/tracing/available_tracers
```

Select one and configure it by doing:

```bash
echo "function_graph" > /sys/kernel/debug/tracing/current_tracer
```

### Filter traced functions

Filter the functions you care about using a pattern as follows:

```bash
echo "*mount*" > /sys/kernel/debug/tracing/set_ftrace_filter
```

### Set the PID to trace syscalls from

Possibly, set the PID of the process that makes the syscalls:

```bash
echo 12345 > /sys/kernel/debug/tracing/set_ftrace_pid
```

### Read the log

The log is found in `/sys/kernel/debug/tracing/trace`.

## KGDB

KGDB is a kernel debugger that requires two machines that are connected via a
serial connection.

### Host (OSX) preparation

Make sure the host (MacOS) has GDB:

```bash
brew install gdb
```

Enable the serial port through the hypervisor and make sure that hardware
virtualization is disabled:

1. Run `vagrant halt` to shutdown the VM
1. Using the VirtualBox UI, map the VM serial port (COM1) to host
   `/dev/ttys003`
1. Start the VM (through the VirtualBox UI again) in headless state
1. Run `vagrant reload`

### Set the KGDB serial port

Select a serial port [1] to enable KGDB:

```bash
echo ttyS0 > /sys/module/kgdboc/parameters/kgdboc
```

### Download vmlinux image

Download the `vmlinux` image from `/boot`. This is useful for GDB to read the
symbols from.

### Start KGDB

Configure the "Linux Magic System Request Key" system. The magic SysRq key is a
'magical' key combo you can hit which the kernel will respond to regardless of
whatever else it is doing, unless it is completely locked up [5].

SysRq invocations via keyboard are restricted by the bitmap set in
`/proc/sys/kernel/sysrq`. However, setting sysrq using `/proc/sysrq-trigger` is
always possible for root. To enable KGDB run:

```bash
echo g > /proc/sysrq-trigger
```

### Connect

Install the `linux-image-$(uname -r)-dbgsym` package in Ubuntu to get the Linux
kernel symbols:

```bash
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C8CAB6595FDFF622 
codename=$(lsb_release -c | awk  '{print $2}')
tee /etc/apt/sources.list.d/ddebs.list << EOF
deb http://ddebs.ubuntu.com/ ${codename}      main restricted universe multiverse
deb http://ddebs.ubuntu.com/ ${codename}-security main restricted universe multiverse
deb http://ddebs.ubuntu.com/ ${codename}-updates  main restricted universe multiverse
deb http://ddebs.ubuntu.com/ ${codename}-proposed main restricted universe multiverse
EOF

apt-get update
apt-get install linux-image-$(uname -r)-dbgsym
```

Then copy the debug image to a different VM. The image can be found in:
`/usr/lib/debug/boot/vmlinux-$(uname -r)`.

In the different VM, map the same serial device. Then run:

```bash
$> gdb ./vmlinux
(gdb) set remotebaud 115200
(gdb) target remote /dev/ttyS0
```

## References

1. http://www.tldp.org/HOWTO/Remote-Serial-Console-HOWTO/preparation-setport.html
2. https://www.kernel.org/doc/htmldocs/kgdb/EnableKGDB.html
3. https://www.kernel.org/doc/Documentation/trace/ftrace.txt
4. https://lwn.net/Articles/365835/
5. https://www.kernel.org/doc/Documentation/sysrq.txt
7. https://wiki.ubuntu.com/Kernel/Systemtap
8. https://wiki.ubuntu.com/Kernel/Systemtap
