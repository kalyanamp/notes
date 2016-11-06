# Linux traffic control

Traffic control is the name given to the sets of queuing systems and mechanisms
by which packets are received and transmitted on a router. This includes
deciding which (and whether) packets to accept at what rate on the input of an
interface and determining which packets to transmit in what order at what rate
on the output of an interface.

* Separating internal queues for different classes of applications is useful
	since it can protect the local network from contention.
* Traffic control is the set of tools which allows the user to have granular
	control over these queues and the queuing mechanisms of a networked device.
* The power to rearrange traffic flows and packets with these tools is
	tremendous and can be complicated, but is no substitute for adequate
	bandwidth.
* Packet switching networks differ from circuit based networks in that they are
	stateless.
* This however means that it's hard to differentiate traffic coming from
	different applications of different significance and priority.
* Traffic control adds statefullness to a stateless network.

## Concepts

### Queues

* Contain a finite number of packets.
* Normally FIFO.
* Single queue in Linux called `pfifo_fast` which it is slightly more complex
	than a FIFO queue.
* With TC, apart from queue and dequeue we can use operations such as:
	- Delay packet
	- Rearrange packet
	- Drop packet
	- Prioritize packet in multiple queues.

### Flows

* Distinct connection and stream of packets between two hosts.
* Flows are important when trying to divide the bandwidth between different
	flows in the system.

### Tokens and buckets

* Only dequeue packets if there are available tokens.
* Generate tokens at the controllable rate to control use of bandwidth.
* TC keeps the tokens in a bucket which can fill up when there is no demand,
	i.e.: no packets in the queue waiting to be dequeued.

### Packets and frames

* Frame describes a layer 2 (data link) unit of data.
* Packet describes a layer 3 (network) unit.
* Damn OSI model!

## Elements of traffic control

* Shaping: delaying packets output to meet a desired rate.
* Scheduling: arranging and rearranging packets for output.
	- E.g.: SFQ attempts to prevent a single client or flow from dominating the
	  network usage.
* Classifying: sorting traffic into different internal queues.
	- Packets can cascade through multiple classifiers.
* Policing: policers measure and limit traffic in a particular queue.
  - Not the same as shaping which can delay a packet.
* Dropping: discarding an entire packet, flow or classification.
* Marking: altering a packet.

## Components

* `qdisc` for scheduling.
	- `disc` is a scheduler.
	- There are classful `disc`s which can contain `class`es and provide a
		handle to attach `filter`s.
	- There are classless `disc`s which contain no classes and it is not
		possible to attach filers.
	- `root` `disc` for outgoing traffic (egress) and `ingress` for incoming
		traffic (ingress).
	- `root` is a full `disc` and it can support multiple classes, filters and
		policers.
	- `ingress` is quite limited and it can only have a filter and a policer to
		control the incoming packages.
* `class` for shaping
	- Classes can only exit inside a classful `disc`.
	- A class can either contain subclasses or it is a leaf class which contains
		a `qdisc`.
	- Each class can have an arbitrary number of filters attached to it.
* `filter` and `classifier` for classifying.
	- Must contain a classifier phase.
	- Must contain a policer phase.
	- Can be attached either to classful `qdisc`s or classes.
	- Packet enters the `root` `qdisc` and then it can either be processed by one
		of the attached filters or it will be passed to the filters of its
		subclasses.
* `policer` as part of a `filter` for policing.
* A `policer` with a `drop` action for dropping.
* The `dsmark` `qdisc` for marking.
* `handle`: every class or classful `qdisc` requires an identifier.
	- It consists of a major number and a minor number.
	- When minor is 0 it identifies a `qdisc`.

## Tool

The `tc` tool is used to configure `qdisc`s, classes, filters, policers etc.

## Examples

### Limiting bandwidth

```bash
tc qdisc add dev ens3 handle 10: root htb default 1
tc class add dev ens3 parent 10: classid 10:1 htb rate 1000000kbit
tc class add dev ens3 parent 10: classid 10:10 htb rate 100kbit
tc qdisc add dev ens3 parent 10:10 handle 100: netem rate 100kbit
iptables -A POSTROUTING -t mangle -j CLASSIFY --set-class 10:10 -p udp --dport 53 -d 8.8.8.8
```

* Adds `qdisc` in `root` (egress) of type `htb` with handle `10:0`.
	- HTB (hierarchical token bucket) uses the concepts of tokens and buckets
		along with the class-based system and filters to allow for complex and
		granular control over traffic.
	- HTB sets the transmission rate but the class can stil borrow tokens from
		other buckets.
* Adds class `10:1` in `qdisc` `10:0` with rate of 1000000kbit.
* Adds class `10:10` in `qdisc` `10:0` with rate of 100kbit.
* Adds `qdisc` in class `10:10` of type `netem` with rate 100kbit
	- `netem` stands for network emulation.
	- It can emulate delay, rate (bandwidth) or latency.
* Iptables set the classid to all the packets that use protocol UDP and go to
	8.8.8.8 port 53.

### Limiting bandwidth with fall-back

```bash
tc qdisc add dev ens3 handle 10: root htb default 1
tc class add dev ens3 parent 10: classid 10:1 htb rate 500kbit
tc class add dev ens3 parent 10: classid 10:10 htb rate 100kbit
tc qdisc add dev ens3 parent 10:10 handle 100: netem rate 100kbit
iptables -A POSTROUTING -t mangle -j CLASSIFY --set-class 10:10 -p udp --dport 53 -d 8.8.8.8
```

* Same as the above except that the fall-back rate limiting, in class `10:1`,
	is 500kbit.

## References

[TLDP]: http://tldp.org/HOWTO/Traffic-Control-HOWTO/

1. [TLDP Traffic control HOWTO][TLDP]
