


## Connectionless Multiplexing and Demultiplexing

It is important to note that a UDP socket is fully identified by a two-tuple consisting
of a destination IP address and a destination port number. As a consequence, if
two UDP segments have different source IP addresses and/or source port numbers, but
have the same destination IP address and destination port number, then the two segments
will be directed to the same destination process via the same destination socket.

## Connection-Oriented Multiplexing and Demultiplexing

One subtle difference between a
TCP socket and a UDP socket is that a TCP socket is identified by a four-tuple:
(source IP address, source port number, destination IP address, destination port
number).
In particular, and in contrast with UDP, two arriving TCP segments with different
source IP addresses or source port numbers will (with the exception of a TCP
segment carrying the original connection-establishment request) be directed to two
different sockets.(即使他们有相同的dest IP / port)



# 3.3 Connectionless Transport: UDP

UDP, defined in [RFC 768], does just about as little as a transport protocol can do.
Aside from the multiplexing/demultiplexing function and some light error checking, it
adds nothing to IP. In fact, if the application developer chooses UDP instead of TCP,
then the application is almost directly talking with IP. UDP takes messages from the
application process, attaches source and destination port number fields for the multiplexing/
demultiplexing service, adds two other small fields, and passes the resulting
segment to the network layer. The network layer encapsulates the transport-layer segment
into an IP datagram and then makes a best-effort attempt to deliver the segment
to the receiving host. If the segment arrives at the receiving host, UDP uses the destination
port number to deliver the segment’s data to the correct application process. Note
that with UDP there is no handshaking between sending and receiving transport-layer
entities before sending a segment. For this reason, UDP is said to be connectionless.

DNS is an example of an application-layer protocol that typically uses UDP.

# 3.4 Principles of Reliable Data Transfer



# 3.5 Connection-Oriented Transport: TCP
