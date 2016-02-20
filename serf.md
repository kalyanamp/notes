# Serf

## Introduction

Serf is a tool for:

* cluster membership
* failure detection
* and orchestration

It is:

* decentralized
* fault-tolerant
* and highly available.

It is extremely lightweight: it uses 5 to 10 MB of resident memory and
primarily communicates using infrequent UDP messages.

Serf makes use of gossip protocol to establish communication and transfer
information across the cluster. This leads to two main direct results:

1. **Serf does not have a single point of failure.** Gossip protocols
  inherently work regardless of the number of nodes that go down.
2. **Joinin the cluster - or getting information out of it - requires the
  address of a single cluster member.** All the members in the cluster are
  equal and requests can be forwarded and travel across all nodes without the
  need to keep a list of these nodes.
3. **Serf is eventually consistent.** Since truth is distributed across the
  cluster using gossiping, it takes time to synchronize all the nodes.

The actual protocol is based on SWIM: Scalable Weekly-consistent Infraction-
style process group Membership protocol [2].

### Membership

Serf maintains a list of cluster members. This list is propagated through
gossiping and shared from all nodes. It gets updated when nodes join,
gracefully leave or are detected as failed. The user is able to specify a
custom script to run when the list gets modified.

### Failure detection and recovery

Failure detection is provided through periodic random probing of nodes.
When a node fails to reach a node within a timeout it tries to reach it
indirectly (through another node). If that fails as well the node is marked as
failed.

The user can specify custom scripts that will be triggered when a failure is
detected. Recovery is done by periodically re-checking if the node came back.

### Custom event propagation

The user may extend the Serf messages with custom events and queries. Serf
will propagate events and run associated scripts on the nodes. Queries work
similarly with the result of the triggered scripts being returned to the
caller.

## References

1. https://www.serfdom.io/
2. Das, Abhinandan, Indranil Gupta, and Ashish Motivala. "Swim: Scalable
  weakly-consistent infection-style process group membership protocol."
  Dependable Systems and Networks, 2002. DSN 2002. Proceedings. International
  Conference on. IEEE, 2002.
