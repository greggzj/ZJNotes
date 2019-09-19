

# 树莓派刷openwrt

## Reference

[1] [官网树莓派版本对应Firmware和论坛](https://openwrt.org/toh/raspberry_pi_foundation/raspberry_pi)

[2] [独立package下载地址(如python, luci在外部同级目录下)](http://downloads.openwrt.org/releases/18.06.4/packages/aarch64_cortex-a53/packages/)

[3] [UI界面配置LUCI的独立安装](https://openwrt.org/docs/guide-user/luci/luci.essentials)

[4] [包管理工具opkg简介，包含proxy使用方法](https://openwrt.org/docs/guide-user/additional-software/opkg)

[5] [Luci JsonRPC使用](https://github.com/openwrt/luci/wiki/JsonRpcHowTo)

[61] [/etc/config/wireless](https://openwrt.org/docs/guide-user/network/wifi/basic)
[62] [wifi 配置](https://oldwiki.archive.openwrt.org/doc/uci/wireless#regenerate_configuration)

[7] [/etc/config/network](https://openwrt.org/docs/guide-user/base-system/basic-networking)

[8] [UCI (Unified Configuration Interface)简介1--techref](https://openwrt.org/docs/techref/uci)

[9] [UCI (Unified Configuration Interface)简介2--Usage](https://openwrt.org/docs/guide-user/base-system/uci)

[10] [对于uci api的使用的理解](https://wiki.teltonika.lt/view/UCI_command_usage)

[111] [树莓派snapshot版本](https://downloads.openwrt.org/snapshots/targets/brcm2708/bcm2710/)
[112] [大神制作snapshot稳定版本](https://forum.openwrt.org/t/18-06-on-raspberry-pi-3-b/18670/40)

[12] https://openwrt.org/docs/guide-developer/overview

## Usage

这次测试Wifi，用树莓派作为client和AP，与被测对象进行通信，树莓派中刷上了之前就期待的Openwrt系统。

这次用的是Raspberry 3B +，倒腾了半天，着实被坑了一把，根据[1]的描述，对于树莓派3B+，官网的release
版本无法支持wifi功能(表现为刷了之后不会再/etc/config/目录下生成wireless文件，使用wifi up/down也
不会有任何效果，根本检测不到wifi硬件)，貌似还有个country code无法设置的issue。

官网给出的解决办法是使用snapshot版本，然而使用snapshot版本又是一堆坑，以下是对于snapshot版本的看法，
来自于https://openwrt.org/releases/snapshot：

- snapshot有对于更多hardware和device的支持
- snapshot 默认支持SSH，但是版本不包含Luci, 需要自己手动安装
- snapshot 不稳定，每天会编译，与之对应的kmod/dependency packages一定要用对应的，不然过了
几个小时更新后可能就会因为不匹配导致无法工作

由于release铁定不支持wifi，没办法，要用3b+就得上snapshot，下载了个最新的snapshot后发现，uci
commit后会报I/O error，瞬间觉得此版本可能有诈，果不其然，该版本ssh也无法连接。于是只能够再寻找
高人，最后在[112]中找到了有个大神，自己结合某个wireless可用的snapshot，结合其他SDK编译了一个目前
来看非常好使的img，终于可以在Luci中正常使用wireless功能了！



## 配置连接网络

连接Internet主要目的是使用opkg来安装一些Package：

- 将树莓派网线直连外网，或者上级路由器，而后配置vi /etc/config/network

```
config interface 'lan'
        #option type 'bridge'
        option ifname 'eth0'
        option proto 'dhcp'
        #option ipaddr '192.168.1.1'
        #option netmask '255.255.255.0'
        #option ip6assign '60'
```

- (optional，针对公司网络)vi /etc/opkg.conf，增加一行http代理

```
option http_proxy http://101.231.121.17:80/
```



