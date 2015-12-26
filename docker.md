# Notes on Docker

## Storage drivers

### devmapper

## Networking

### docker0 bridge

By default docker will use the `docker0` bridge for all its containers.
Assumming that you are running on an Ubuntu release, you need to install the
`bridge-utils` package to properly inspect the bridge attributes while playing
with containers:

```bash
$> apt-get install bridge-utils -y
...

# Display list of bridges
$> brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242743c0b8f       no

# Display bridge IP and network addresses
$> ip addr show docker0
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 9001 qdisc noqueue state DOWN group default
    link/ether 02:42:74:3c:0b:8f brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:74ff:fe3c:b8f/64 scope link
       valid_lft forever preferred_lft forever

# Display the list of interfaces that are linked to the bridge, by MAC
$> brctl showmacs docker0
port no mac addr                is local?       ageing timer
```

Let's add a couple of containers:

```bash
$> docker run -d redis
5e773946394b57240cc80a046cffb61d1a12d50f36c8182eb33624e97c13e668

$> docker run -d -e MYSQL_ROOT_PASSWORD=test mysql
ba73f2f98230e04010cf688059b1f201547a39db068e335d14b1759bf41cea7c

$> docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
ba73f2f98230        mysql               "/entrypoint.sh mysql"   6 seconds ago       Up 5 seconds        3306/tcp            mad_feynman
5e773946394b        redis               "/entrypoint.sh redis"   6 minutes ago       Up 6 minutes        6379/tcp            pensive_yonath
```

After creating the containers we have to additional [veth][veth] pairs in the host:

```bash
$> ip addr
...
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default
    link/ether 02:42:74:3c:0b:8f brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:74ff:fe3c:b8f/64 scope link
       valid_lft forever preferred_lft forever
11: veth7f6daae: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue master docker0 state UP group default
    link/ether 36:48:bf:1a:81:e3 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::3448:bfff:fe1a:81e3/64 scope link
       valid_lft forever preferred_lft forever
23: vethc5d293a: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue master docker0 state UP group default
    link/ether 6e:58:97:8c:39:5b brd ff:ff:ff:ff:ff:ff
    inet6 fe80::6c58:97ff:fe8c:395b/64 scope link
       valid_lft forever preferred_lft forever
```

we can see two `veth` interfaces in this list of link interfaces. They have
MAC addresses `36:48:bf:1a:81:e3` and `6e:58:97:8c:39:5b` respectively. If we
look into the bridge's interfaces we will see that both of them are
connected in the `docker0` bridge:

```bash
$> brctl showmacs docker0
port no mac addr                is local?       ageing timer
  1     36:48:bf:1a:81:e3       yes                0.00
  2     6e:58:97:8c:39:5b       yes                0.00
```

It is also interesting to look into the network namespaces themselves. To do
so we need to assign names to the network namespaces of each container.
This is an example of how this is done for the redis container:

```bash
$> ps -ax | grep redis
 2483 ?        Ssl    0:09 redis-server *:6379
 7755 pts/0    S+     0:00 grep --color=auto redis

$> ln -s /proc/2483/ns/net /run/netns/redis

# do the same for the mysql container

$> ip netns list
mysql
redis
```

Now we can run networking commands inside the network namespace of the
containers:

```bash
$> ip netns exec redis ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:02
          inet addr:172.17.0.2  Bcast:0.0.0.0  Mask:255.255.0.0
          ...

lo        Link encap:Local Loopback
...

$> ip netns exec mysql ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
    ...
22: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 scope global eth0
       valid_lft forever preferred_lft forever
    ...
```

From the above results it seems that can draw a picture of how `docker0` works:

* Docker defines the `172.12.0.0/16` subnet
* `docker0` has the "gateway" IP: `172.17.0.1`
* Each container defines a veth pair:
  - One side is called `veth<Container id>` and is assigned to the `docker0`
      bridge.
  - The other side is called `eth0` and lives in the container network
      namespace. It gets an IP from the `127.12.0.0/16` subnet.

### Alternative networks

Docker uses the `--network` argunent of the `docker run` command to specify
the container network. To list the Docker networks run:

```bash
$> docker network ls
NETWORK ID          NAME                DRIVER
1ef0200050db        none                null
bf802e48489f        host                host
4e120ad68df2        bridge              bridge
```

## References

1. https://docs.docker.com/engine/userguide/networking/dockernetworks
1. [Information about virtual ethernet device][veth]

[veth]: https://lwn.net/Articles/232688/

