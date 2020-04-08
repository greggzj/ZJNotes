

# Multicast

Multicast 又称为多播、组播。

参考TCP.IP.Illustration Chapter9

目前总结：
    如果是一个局域网内组播，发送端只看下路由表(netstat -rn)
    接收端看下组播的路由(netstat -gn)就行了，主要是接收端在对应网卡上是否绑定组播的IP


## iperf 发送组播包

refer to 001_network_debugging.md中Iperf