

## 6.1.1 The Services Provided by the Link Layer

- Framing

- Link access

A medium access control (MAC) protocol specifies the rules by
which a frame is transmitted onto the link.

- Reliable delivery

- Error detection and correction.

# 6.4 Switched Local Area Networks

- MAC Addresses

是link layer地址，6 bytes大小，一共有2^48次方可能。
每个设备的MAC都是唯一的，MAC地址一供48bit，前24bit(3个字节)是company从IEEE购买空缺的固定的，后24bit自行设定。

In particular, when a company wants to manufacture adapters, it purchases a chunk
of the address space consisting of 2^24 addresses for a nominal fee. IEEE allocates the
chunk of 2^24 addresses by fixing the first 24 bits of a MAC address and letting the
company create unique combinations of the last 24 bits for each adapter.


- Address Resolution Protocol (ARP)

参考: https://www.saminiir.com/lets-code-tcp-ip-stack-1-ethernet-arp/

Because there are both network-layer addresses (for example, Internet IP addresses)
and link-layer addresses (that is, MAC addresses), there is a need to translate between
them.

DNS针对Internet中所有主机，而ARP仅仅针对相同subnet中的网络，不同子网的ARP请求可能会返回错误。

one important difference between the two resolvers is that DNS
resolves host names for hosts anywhere in the Internet, whereas ARP resolves IP
addresses only for hosts and router interfaces on the same subnet.

如果A/B是不同的子网，那么A向B发送数据包时填写的MAC是和gateway的(由gateway与A在同一网段中的Interface通过ARP协商)，到了gateway由gateway来更换成目标MAC(由gateway与B在同一网段中的interface通过ARP协商)

ARP过程结束后HOST会存储到translation table以便于后续的查询，ARP的算法参考上面链接。

ARP 协议包结构

```
struct arp_hdr
{
    uint16_t hwtype;  -->'Ethernet': 0x0001
    uint16_t protype;  -->协议类型, 如：IPV4 0x0800
    unsigned char hwsize;   -->hardware size, 如: 6 bytes for MAC address
    unsigned char prosize;  -->protocol size, 如: 4 bytes for IP address
    uint16_t opcode;  -->ARP类型:  ARP request (1), ARP reply (2), RARP request (3) or RARP reply (4).
    unsigned char data[];
} __attribute__((packed));
```

- Ethernet Frame


```
struct eth_hdr
{
    unsigned char dmac[6];
    unsigned char smac[6];
    uint16_t ethertype;  -->  如IPV4(0x0800)/ARP
    unsigned char payload[];
} __attribute__((packed));

```

