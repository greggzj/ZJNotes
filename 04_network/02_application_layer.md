

# Socket

a
socket is the interface between the application layer and the transport layer within
a host.

The only control that the application developer
has on the transport-layer side is (1) the choice of transport protocol and (2) perhaps
the ability to fix a few transport-layer parameters such as maximum buffer and maximum
segment sizes (to be covered in Chapter 3). Once the application developer
chooses a transport protocol (if a choice is available), the application is built using
the transport-layer services provided by that protocol. We’ll explore sockets in some
detail in Section 2.7.


