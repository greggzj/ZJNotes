



In Section
4.2, we’ll dive down into the internal hardware operations of a router, including input
and output packet processing, the router’s internal switching mechanism, and packet
queueing and scheduling. In Section 4.3, we’ll take a look at traditional IP forwarding,
in which packets are forwarded to output ports based on their destination IP
addresses. We’ll encounter IP addressing, the celebrated IPv4 and IPv6 protocols and
more. In Section 4.4, we’ll cover more generalized forwarding, where packets may
be forwarded to output ports based on a large number of header values (i.e., not only
based on destination IP address). Packets may be blocked or duplicated at the router,
or may have certain header field values rewritten—all under software control. This
more generalized form of packet forwarding is a key component of a modern network
data plane, including the data plane in software-defined networks (SDN).


# 4.1 Overview of Network Layer

## 4.1.1 Forwarding and Routing: The Data and Control Planes

- Forwarding. When a packet arrives at a router’s input link, the router must move
the packet to the appropriate output link.

- Routing. The network layer must determine the route or path taken by packets as
they flow from a sender to a receiver.



the routing algorithm (Control plane) determines the contents of the routers’ forwarding
tables (Data plane).



# 4.2 What’s Inside a Router?

- Input ports.

a. It performs the physical
layer function of terminating an incoming physical link at a router

b. also performs link-layer functions needed to
interoperate with the link layer at the other side of the incoming link; this is
represented by the middle boxes in the input and output ports.

c. most crucially,
a lookup function is also performed at the input port; this will occur in the
rightmost box of the input port. It is here that the forwarding table is consulted
to determine the router output port to which an arriving packet will be forwarded
via the switching fabric.


- Switching fabric.

- Output ports.

An output port stores packets received from the switching fabric
and transmits these packets on the outgoing link by performing the necessary
link-layer and physical-layer functions

- Routing processor.

The routing processor performs control-plane functions. 

a. executes the routing protocols (which we’ll study in Sections
5.3 and 5.4)

b. maintains routing tables and attached link state information

c. computes the forwarding table for the router.


## 4.2.1 Input Port Processing and Destination-Based Forwarding


## 4.2.2 Switching

- Switching via memory.

The simplest, earliest routers were traditional computers,
with switching between input and output ports being done under direct control of
the CPU (routing processor).

- Switching via a bus.

- Switching via an interconnection network.

## 4.2.3 Output Port Processing

## 4.2.4 Where Does Queuing Occur?


## 4.2.5 Packet Scheduling


# 4.3 The Internet Protocol (IP): IPv4, Addressing,IPv6, and More

## 4.3.2 IPv4 Datagram Fragmentation

The maximum amount of data that a link-layer frame can carry is called
the maximum transmission unit (MTU).

What is a problem
is that each of the links along the route between sender and destination can use
different link-layer protocols, and each of these protocols can have different MTUs.

The solution is to fragment
the payload in the IP datagram into two or more smaller IP datagrams, encapsulate each
of these smaller IP datagrams in a separate link-layer frame; and send these frames over
the outgoing link. Each of these smaller datagrams is referred to as a fragment.
Fragments need to be reassembled before they reach the transport layer at the
destination.

the designers of IPv4 decided to
put the job of datagram reassembly in the end systems rather than in network routers.

## 4.3.3 IPv4 Addressing


The boundary between the host and the physical link is called
an interface.

Thus, an IP address is technically associated with an interface,
rather than with the host or router containing that interface.


To determine the subnets, detach each interface from its host or router, creating
islands of isolated networks, with interfaces terminating the end points of the
isolated networks. Each of these isolated networks is called a subnet.


### Obtaining a Host Address: The Dynamic Host Configuration Protocol


## 4.3.4 Network Address Translation (NAT)

## 4.3.5 IPv6

# 4.4 Generalized Forwarding and SDN



## 4.4.3 OpenFlow Examples of Match-plus-action in Action

