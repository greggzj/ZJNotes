
application layer-->Transport Layer (TCP/UDP)--> Network Layer data panel (IP) --> Network layer control panel --> Link Layer

## reference

- Computer networking

## Ipv4

![ipv4](./resource/ipv4.png)

- Time-to-live. The time-to-live (TTL) field is included to ensure that datagrams
do not circulate forever (due to, for example, a long-lived routing loop) in the
network. This field is decremented by one each time the datagram is processed by
a router. If the TTL field reaches 0, a router must drop that datagram.

### Ipv4 分段fragment

以太网帧Ethernet Frames理论上最大能够承载1,500 bytes的数据，世纪中wide-area links只能够承载不超过576 bytes的数据。

link-layer frame能够承载的最大传输单元成为maxium transmission unit(MTU)。IP datagram封装在link layer frame中由一个router传递给另一个router，而不同的router可以使用不同的link layer protocol，导致了使用不同的MTU。

如果IP dategram从一个支持大MTU的router转发到小MTU的router，大MTU的router就会将IP datagram进行分段，也就是fragment。

Fragments会在目标机器传递到transport layer之前进行重组。即，TCP/UDP希望从Network layer得到的是完整的没有分段的数据。重组只会发生在端到端的两端，不会发生在router上。（所以router不会有network layer?)

![ipv4fra](./resource/ip_fragment.png)

### Ipv4 addressing

Ipv4 中IP地址与网卡interface绑定(对于Ipv4,一个Interface一个IP地址？？而Ipv6一个interface可以有多个IP)

格式：32 bits long (4 bytes), 最大2的32次方个IP 地址。

![ipv4add](./resource/ipv4_addr.png)

子网掩码subnet mask: 223.1.1.0/24中的/24，表示32位中的leftmost 24 bits属于subnet address。

上图中host interface(233.1.1.1/233.1.1.2/233.1.1.3)以及router interface
(233.1.1.4)组成了一个子网223.1.1.0/24。


![rs](./resource/router_subnet.png)

对于子网subnet的定义，看上面这张图会清晰一些，上图中，223.1.1.0/24, 223.1.2.0/24, 和 223.1.3.0/24三个子网与之前图4.18比较类似，除此以外还有三个子网：
- 223.1.9.0/24, R1与R2连接的子网
- 223.1.8.0/24, R2与R3连接的子网
- 223.1.7.0/24, R3与R1连接的子网


#### 无类别域间路由选择 Classless Interdomain Routing (CIDR—pronounced cider) [RFC 4632]

- CIDR的定义

将32-bit的IP地址划分为两个部分，a.b.c.d/x，x表示 x most significant bits（第一部分），通常
称为network prefix，一个organization通常有相同的第一部分，在Organization外部的router向
Organization转发数据时只需识别network prefix部分，这样就会减少router里面转发表的size。

剩下的bit为第二部分，最为Organization内部使用，在内部也可以有各种subnet，比如，a.b.c.d/21是
orgnization的network prefix，内部就可以定义一个a.b.c.d/24作为一个organization内部的subnet。

在CIDR被正是采纳之前，network 的prefix是固定的8, 16, 24bit这样的长度，既所谓的A/B/C类地址。
但是这样划分会有一些问题，不灵活，无法很好的支持日益增长的IP需求，比如一个C类地址(/24)subnet，
一共可以有2的8次方减2=254个hosts(减去的两个是保留地址)，254个对于一个Organization可能太少了,
假设这个organization有2000个Hosts,而如果给其B类地址(65534个)subnet，又白白浪费了63000个地址(
其他orgnization无法使用)。

- 利用CIDR寻址

![ca](./resource/cidr_address.png)

如上图，ISP: Fly-By-Night-ISP 对于外界使用200.23.16.0/20(前20bit)来通信，外接只知道
这个ISP的地址是200.23.16.0/20，并不知道在这个ISP内部还有8个其他的organization，每一个
都有自己的subnets存在，这种使用单个前缀来表示多个network的能力称为**地址聚合address 
aggregation(或者是路由聚合route aggregation/summarization)**。

当然还有一种其他的情况，想下，如果Fly-By-Night-ISP中的Organization 1连接到了另一个ISPs-R-Us中，而ISPs-R-Us本身对外地址为199.31.0.0/16, 显然Organization 1的IP地址超出了这个范围，接下来怎么办呢？

比较花开销的做法是Organization 1对内部所有的routers和Hosts重新划分地址，但代价太大。有一个方法可以让该组织无需重新分配地址，Fly-By-Night-ISP继续advertise 200.23.16.0/20 , ISPs-
R-Us继续advertise 199.31.0.0/16，同时advertise 200.23.18.0/23, 当网络外部的router同时遇到
200.23.16.0/20 (from Fly-By-Night-ISP) and 200.23.18.0/23 (from ISPs-
R-Us)时，并想要把数据发给200.23.18.0/23时，会使用长匹配，即挑选最长的network prefix来匹配。


#### Organization内部分配一系列的地址规则

如果一个ISP的地址快是200.23.16.0/20 11001000 00010111 00010000 00000000,里面的organization的地址可以这么分：
Organization 0 200.23.16.0/23 11001000 00010111 00010000 00000000
Organization 1 200.23.18.0/23 11001000 00010111 00010010 00000000
Organization 2 200.23.20.0/23 11001000 00010111 00010100 00000000
… … …
Organization 7 200.23.30.0/23 11001000 00010111 00011110 00000000


### DHCP Dynamic Host Configuration Protocol (DHCP) [RFC 2131]

这个是IPV4下Host获取IP地址的一种方式(IPV6的成为DHCPV6), 除了分配IP地址外，host还能够通过DHCP来获取一些其他的信息比如：subnet mask, default gateway, NTP server， DNS server地址等。

我们可以配置DHCP server每次都给某个设备分配相同的IP地址，也可以每次分配不同的地址。

![dhcp](./resource/dhcp.png)

通常在每一个subnet中有一个DHCP server，如果没有，那么一个DHCP relay agent(通常来说就是一个router)会知道DHCP server的地址。

如上图，DHCP server在subnet 223.1.2/24, 旁边有个router连接了223.1.1/24和223.1.3/24就是一个relay agent

对于新加进来的HOST的DHCP过程，4步：

![dhcp](./resource/dhcp_req.png)

- **DHCP server discovery**.

Host先准备UDP包(67端口)DHCP server discovery，该UDP包被封装到一个IP datagram中发送出去。
目标IP地址填写为广播地址255.255.255.255，源地址为0.0.0.0(表示this host). DHCP client将准备好的IP datagram交给link layer，由后者广播到subnet中所有与Host相连的nodes。

- **DHCP server offer(s)**. 

DHCP server收到了DHCP discover消息，回复时仍旧使用255.255.255.255的广播地址，广播到subnet的所有nodes，采用广播而非单播是因为在subnet中可能有多个DHCP server的存在，client需要选择其中的一个。

回复client的消息可以包含：the transaction ID of the received discover message,
the proposed IP address for the client, the network mask, and an IP address
**lease time**—分配的IP地址有效时间，通常设置为几个小时或者几天[Droms 2002].

- **DHCP request**
client会从一个或者多个server的Offer中选择一个并对其进行回复DHCP request消息，将收到的Offer消息中的配置信息在DHCP request中原封不动的传递回去。

- **DHCP ACK**
server回复DHCP request消息，确认请求生效。



Once the client receives the DHCP ACK, the interaction is complete and the
client can use the DHCP-allocated IP address for the lease duration. Since a client
may want to use its address beyond the lease’s expiration, DHCP also provides a
mechanism that allows a client to renew its lease on an IP address.
From a mobility aspect, DHCP does have one very significant shortcoming.
Since a new IP address is obtained from DHCP each time a node connects to a new
subnet, a TCP connection to a remote application cannot be maintained as a mobile
node moves between subnets. In Chapter 6, we will examine mobile IP—an extension
to the IP infrastructure that allows a mobile node to use a single permanent
address as it moves between subnets. Additional details about DHCP can be found in
[Droms 2002] and [dhc 2016]. An open source reference implementation of DHCP
is available from the Internet Systems Consortium [ISC 2016].

### NAT


## Ipv6



https://thenetworkway.wordpress.com/2014/07/02/ipv6-address-assignment-stateless-stateful-dhcp-oh-my/

