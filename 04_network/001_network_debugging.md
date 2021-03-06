

# Linux network debugging

- [stackexchange 参考链接](https://unix.stackexchange.com/questions/50098/linux-network-troubleshooting-and-debugging?noredirect=1)

- [Linux networking 101](https://www.actualtechmedia.com/wp-content/uploads/2017/12/CUMULUS-NETWORKS-Linux101.pdf)

I think, general principles of network troubleshooting are:

Find out at what level of TCP/IP stack(or some other stack) occurs the problem.
Understand what is the correct system behavior, and what is deviation from normal system state
Try to express the problem in one sentence or in several words
Using obtained information from buggy system, your own experience and experience of other people(google, various forum, etc.), try to solve the problem until success(or failure)
If you fail, ask other people about help or some advice
As for me, I usually obtain all required information using all needed tools, and try to match this information to my experience. Deciding what level of network stack contains the bug helps to cut off unlikely variants. Using experience of other people helps to solve the problems quickly, but often it leads to situation, that I can solve some problem without its understanding and if this problem occurs again, it's impossible for me to tackle it again without the Internet.

And in general, I don't know how I solve network problems. It seems that there is some magic function in my brain named SolveNetworkProblem(information_about_system_state, my_experience, people_experience), which could sometimes return exactly the right answer, and also could sometimes fail(like here TCP dies on a Linux laptop).

I usually use utils from this set for network debugging:

- `ifconfig` (or `ip link`, `ip addr`) - for obtaining information about network interfaces
- `ping` - for validating, if target host is accessible from my machine. ping is also could be used for basic DNS diagnostics - we could ping host by IP-address or by its hostname and then decide if DNS works at all. And then traceroute or tracepath or mtr to look what's going on on the way there.
- `dig` - diagnose everything DNS
- `dmesg | less` or `dmesg | tail` or `dmesg | grep -i error` - for understanding what the Linux kernel thinks about some trouble.
- `netstat -antp + | grep smth` - my most popular usage of netstat command, which shows information about TCP connections. Often I perform some filtering using grep. See also the new ss command (from iproute2 the new standard suite of Linux networking tools) and lsof as in lsof -ai tcp -c some-cmd.
- `telnet <host> <port>` - is very useful for communicating with various TCP-services(e.g. on SMTP, HTTP protocols), also we could check general opportunity to connect to some TCP port.
- `iptables-save` (on Linux) - to dump the full iptables tables
- `ethtool` - get all the network interface card parameters (status of the link, speed, offload parameters...)
- `socat` - the swiss army tool to test all network protocols (UDP, multicast, SCTP...). Especially useful (more so than telnet) with a few -d options.
- `iperf` - to test bandwidth availability
- `openssl` (s_client, ocsp, x509...) to debug all SSL/TLS/PKI issues.
- `wireshark` - the powerful tool for capturing and analyzing network traffic, which allows you to analyze and catch many network bugs.
- `iftop` - show big users on the network/router.
- `iptstate` (on Linux) - current view of the firewall's connection tracking.
- `arp` (or the new (Linux) `ip neigh`) - show the ARP-table status.
- `route` or the newer (on Linux) `ip route` - show the routing table status.
- `strace` (or `truss`, `dtrace` or `tusc` depending on the system) - is useful tool which shows what system calls does the problem process, it also shows error codes(errno) when system calls fails. This information often says enough for understanding the system behavior and solving a problem. Alternatively, using breakpoints on some networking functions in gdb can let you find out when they are made and with which arguments.
- to investigate firewall issues on Linux: `iptables -nvL` shows how many packets are matched by each rule (`iptables -Z` to zero the counters). The `LOG` target inserted in the firewall chains is useful to see which packets reach them and how they have already been transformed when they get there. To get further NFLOG (associated with `ulogd`) will log the full packet.


# tcpdump

## 跟踪某个Interface的详细包

```
tcpdump -vnes0 -i eth0
```

## 跟踪某个Interface上的DHCP过程

如果不加以下选项，抓到的包不会有MAC信息，也不会有一些debug的信息，只是一行

v -- verbose output
n -- no resolution of hostname (不显示主机名，替换为IP地址)
s -- Define the snaplength (size) of the capture in bytes. Use -s0 to get everything, unless you are intentionally capturing less. 

```
tcpdump -vnes0 -i eth0 port 67 or 68

#感觉用下面这个更好，能够很清楚地看到各种Option是否出现
tcpdump -i br0 -vvv -s 1500 '((port 67 or port 68)
```

## tcpdump 的简单学习
https://danielmiessler.com/study/tcpdump/


```
Here are some additional ways to tweak how you call tcpdump.

-X : Show the packet’s contents in both hex and ascii.
-XX : Same as -X, but also shows the ethernet header.
-D : Show the list of available interfaces
-l : Line-readable output (for viewing as you save, or sending to other commands)
-q : Be less verbose (more quiet) with your output.
-t : Give human-readable timestamp output.
-tttt : Give maximally human-readable timestamp output.
-i eth0 : Listen on the eth0 interface.
-vv : Verbose output (more v’s gives more output).
-c : Only get x number of packets and then stop.
-s : Define the snaplength (size) of the capture in bytes. Use -s0 to get everything, unless you are intentionally capturing less.
-S : Print absolute sequence numbers.
-e : Get the ethernet header as well.
-q : Show less protocol information.
-E : Decrypt IPSEC traffic by providing an encryption key.
```


# Iperf

[Iperf](https://iperf.fr/en/iperf-doc.php#tradeoff)是一个测试网络bandwidth的工具，目前由于需要测试switch的性能，因此使用了这个工具。链接中的网址是命令option大全，测试目前使用到Iperf2而非Iperf3。

测试的基本配置为一台电脑作为client另一台作为server。

参考链接：

[基本命令1](https://www.cnblogs.com/shuqingstudy/p/10767414.html)
[基本命令2](https://www.linode.com/docs/networking/diagnostics/install-iperf-to-diagnose-network-speed-in-linux/)


## 测试switch性能简介
以下是SB的一些测试switch端口转发的一些建议，供参考，基于这些，考虑使用的工具为Iperf:

-	既然用了交换芯片，我们就要测试它的确实是硬件转发。可以使用PC机连接EP2的两个网口，用测试软件以不同的包大小进行冲包，看能达到的速率是什么；在冲包的过程中访问EP2的IP地址看有什么影响。愿意的话可以以单播、组播和广播包分类型冲包，看CPU是否会被冲死。
-	测试多网口的linkup/down功能。插一根网线，两根网线，断一根和全断，查看服务通断的情况。特别注意DPWS、DHCP和FORUM服务是否正常。
-	也可以测试一下网口的自动速率功能。就是和10M、100M、1000M相连，全双工、半双工是否能工作。
-	有条件也可多对接几种SE网口设备，多使用几种线缆

## 测试命令

- IPV4发送接收TCP包

```
server: 打印信息1s间隔
    iperf -s -i 1

client:　1秒间隔,90M bandwith,持续2000秒,目标Server IPV4为192.168.1.30
    iperf -c 192.168.1.30 -i 1 -b 90M -t 2000
```

- IPV6发送接收TCP包

```
server: 打印信息1s间隔
    iperf -s -V -i 1

client:　1秒间隔,90M bandwith,持续2000秒, 目标Server IPV6为fe80::5bad:11e3:fc4e:1ede,走enp0s3的interface
    iperf -V -u -c fe80::5bad:11e3:fc4e:1ede%enp0s3 -b 90M -t 2000 -i 1
```


- IPV4发送接收UDP包

```
server: 打印信息1s间隔
    iperf -s -u -i 1

client:　1秒间隔,90M bandwith,持续2000秒
    iperf -u -c 192.168.1.30 -i 1 -b 90M -t 2000
```

- IPV6发送接收UDP包

```
server: 打印信息1s间隔
    iperf -s -V -u -i 1

client:　1秒间隔,90M bandwith,持续2000秒
    iperf -V -u -c fe80::5bad:11e3:fc4e:1ede%enp0s3 -b 90M -t 2000 -i 1
```

## 遇到的问题

### 利用Iperf发送组播包

#### 什么是组播
参考004_multicast.md

#### 用Iperf配置发送组播包(相同网段)PC需要配置什么

不需要！

利用下面的命令开启Iperf接受组播的server，由打印信息可知，Iperf启动后自动生成了一个组播绑定地址224.1.1.1，该地址也会体现在netstat -gn

```
iperf -s -u -B 224.1.1.1 -i 1
------------------------------------------------------------
Server listening on UDP port 5001
Binding to local address 224.1.1.1
Joining multicast group  224.1.1.1
Receiving 1470 byte datagrams
UDP buffer size:  110 KByte (default)
------------------------------------------------------------
```

### 组播包发送不成功

用一台server(双网卡，一个网卡连接公司网络另一个为本地网卡)与一个client PC(自己的台式机，开启Linux虚拟机)相连接，在server上开启`iperf -s -u -B 224.0.67.67 -i 1`，client连接后数据没有发送成功

- netstat -gn发现224.0.67.67这个组播地址被绑定到了默认的连接公司网络的网卡，而目前Iperf2中-B选项在作为组播地址使用后，没有其他的option用来绑定特定的端口，比较傻，因此考虑更换client和server，本地PC作为client，开启Iperf server后默认网卡就是本地连接网卡，Server服务器开启Iperf client进行组播包的发送，以下是client PC的`netstat -gn`结果：

```
lo              1      224.0.0.251
lo              1      224.0.0.1
enp0s3          2      224.0.67.67
enp0s3          1      224.0.0.251
enp0s3          1      224.0.0.1
enp0s8          1      224.0.0.251
enp0s8          1      224.0.0.1
lo              1      ff02::fb
...
```
如果非要在Iperf使用了-B绑定组播地址的情况下再想指定对应的interface进行组播包的发送，可以[参考这里](https://sourceforge.net/p/iperf/mailman/message/19147976/):
```
the way I got around it was to create a static route whenever I needed
to do it:

route add 224.0.67.67 dev eth1  (this is assuming that "eth1" is the
network device corresponding to the network interface whose hostname is
"node2")
```

### 如何使用Ipv6发送组播包

IP -V for IPv6，但是目前仅针对非link local的IPV6地址，如果要使用link local,需要再指定Interface，

```
example:
iperf -V -u -c fe80::5bad:11e3:fc4e:1ede%enp0s3 -b 90M -t 2000 -i 1
```

参考：(看下面的回复)
https://www.kuncar.net/blog/2018/how-to-test-ipv6-with-iperf/2012/



## Notes

以下参考[Linux networking 101]

- ip 新命令和老命令

    - 老

    If you really want or need to use a deprecated
    command such as ifconfig, arp, or route,
    you still can. You just have to install the nettools
    package on your system. On a Debian
    system like the one shown in the examples in this book, as a
    privileged (root) user, run apt-get install net-tools.

    ```
    route  --> 看路由表

    ifconfig --> 看ip地址

    arp
    ```

    - 新

    ```
    ip route show --> 看路由表

    ip addr --> 看ip地址，还能看是否是dynamic 还是forever(static)
    ```

- Layer2存在广播包回环的问题Bridging Loops

如果是广播包，那么对于Layer2的设备而言就会一直在LAN中广播，如果LAN中有LOOP，就会导致所谓的broadcase storm导致网络瘫痪。如果是Layer3，有TTL字段可以保证这样的包会主动丢弃，不会发生storm这样的情况。

Because bridges forward broadcast packets out
every port, the broadcast is amplified by both
devices when there are multiple paths between two
bridges. For example, if two bridges are connected
with two links, the first bridge receives a broadcast frame from an
attached host. The bridge will take this single frame and send one copy
on each link to the other bridge. This second bridge will receive these
two broadcast frames, one on each link, and will make new copies,
sending them back on each link. This back and forth broadcast
replication, known as a "broadcast storm," will continue forever.
Unlike layer 3 packets, layer 2 frames do not possess a TTL field. A
packet contains a special field that is set by the host that first created the
packet. Each router along the path will decrement this field by 1. If a
misconfiguration in the network causes a similar loop, the TTL field will
eventually be decremented to 0 and the packet will be dropped. Because
a layer 2 frame does not have this field, there is no limit to how many
bridges a frame can pass through. Also, because the packet is being
bridged and not routed, the TTL field will never be examined by any of
the devices and never decremented. The lack of TTL is one of the major
problems with layer 2 networks.
The Spanning Tree Protocol (STP) does not add a TTL field to the frame,
but it will prevent layer 2 loops from forming, preventing the broadcast
storm described earlier. Bridges that speak STP will exchange
information about the network using Bridge Protocol Data Units
(BPDUs). Through this BPDU exchange, the bridges will build a loopfree
"tree" of the network. In our two-switch example, STP would
disable one of the two links and never send traffic over it, until the active
link failed.
