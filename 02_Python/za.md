# 20180106 Datetime和String转换

今日谈谈python对于字符串和日期对象的互相转换问题。

- strptime = "string parse time"
- strftime = "string format time"

## String转datetime

参考：https://stackoverflow.com/questions/466345/converting-string-into-datetime

```
from datetime import datetime
datetime_object = datetime.strptime('Jun 1 2005  1:33PM', '%b %d %Y %I:%M%p')
```

## Datetime转string

参考：https://stackoverflow.com/questions/10624937/convert-datetime-object-to-a-string-of-date-only-in-python

```
import datetime
t = datetime.datetime(2012, 2, 23, 0, 0)
t.strftime('%m/%d/%Y')

```

## 代号含义

Reference:
- http://strftime.org/
- Python 官网： https://docs.python.org/3.5/library/datetime.html#strftime-and-strptime-behavior

```
Code    Meaning Example
%a	Weekday as locale’s abbreviated name.	Mon
%A	Weekday as locale’s full name.	Monday
%w	Weekday as a decimal number, where 0 is Sunday and 6 is Saturday.	1
%d	Day of the month as a zero-padded decimal number.	30
%-d	Day of the month as a decimal number. (Platform specific)	30
%b	Month as locale’s abbreviated name.	Sep
%B	Month as locale’s full name.	September
%m	Month as a zero-padded decimal number.	09
%-m	Month as a decimal number. (Platform specific)	9
%y	Year without century as a zero-padded decimal number.	13
%Y	Year with century as a decimal number.	2013
%H	Hour (24-hour clock) as a zero-padded decimal number.	07
%-H	Hour (24-hour clock) as a decimal number. (Platform specific)	7
%I	Hour (12-hour clock) as a zero-padded decimal number.	07
%-I	Hour (12-hour clock) as a decimal number. (Platform specific)	7
%p	Locale’s equivalent of either AM or PM.	AM
%M	Minute as a zero-padded decimal number.	06
%-M	Minute as a decimal number. (Platform specific)	6
%S	Second as a zero-padded decimal number.	05
%-S	Second as a decimal number. (Platform specific)	5
%f	Microsecond as a decimal number, zero-padded on the left.	000000
%z	UTC offset in the form +HHMM or -HHMM (empty string if the the object is naive).	
%Z	Time zone name (empty string if the object is naive).	
%j	Day of the year as a zero-padded decimal number.	273
%-j	Day of the year as a decimal number. (Platform specific)	273
%U	Week number of the year (Sunday as the first day of the week) as a zero padded decimal number. All days in a new year preceding the first Sunday are considered to be in week 0.	39
%W	Week number of the year (Monday as the first day of the week) as a decimal number. All days in a new year preceding the first Monday are considered to be in week 0.	39
%c	Locale’s appropriate date and time representation.	Mon Sep 30 07:06:05 2013
%x	Locale’s appropriate date representation.	09/30/13
%X	Locale’s appropriate time representation.	07:06:05
%%	A literal '%' character.	%
```


#20190808 Linux 下使用Pyserial访问串口返回`OSError: [Errno 13] Permission denied:`

- reference： https://stackoverflow.com/questions/27858041/oserror-errno-13-permission-denied-dev-ttyacm0-using-pyserial-from-pyth

单次解决方法：`sudo chmod 666 /dev/ttyUSB0`


#20190808 Python 获取本机所有网卡的IP地址(IP Address)方法

简单点，如下，但是在Linux下会有些问题

```
gethostbyname_ex(gethostname())[2]
```

## refencece: 

- [三种获取本机IP方法中文版]https://www.chenyudong.com/archives/python-get-local-ip-graceful.html
- [Linux的gethostbyname](https://linux.die.net/man/3/gethostbyname)
- [三种获取本机IP方法英文版]https://stackoverflow.com/questions/24196932/how-can-i-get-the-ip-address-of-eth0-in-python#


## gethostbyname在Linux下遇到的问题

在Linux下使用dhcp server python library时发现一个问题，主机分配出来的地址是127.0.0.1而不是想要的192.168.1.2，深究发现问题出在作者使用的`gethostbyname_ex(gethostname())[2]`上。

Python的socket module调用的gethostbyname(_ex)函数会调用C语言同名函数，所以该函数的返回值是完全依赖于不同操作系统中的gethostbyname运行结果的。

对于Linux而言(目前没看过代码)，该函数返回结果是有以下几个获取的:
- any or all of the name server
- a broken out line from /etc/hosts
- Network Information Service (NIS or YP), depending upon the contents of the order line in /etc/host.conf

上述优先级从高到低排列，因此如果Linux机器中/etc/hosts目录下有类似以下的语句：
```
127.0.0.1      localhost
127.0.1.1      admin123-Precision-3630-Tower
```
gethostbyname就会返回admin123-Precision-3630-Tower对应的IP地址127.0.1.1，而非期望的本机网口对应的196的IP地址。

### 解决方法

如果想用纯python来解决，一个比较好的解决方法是修改/etc/hosts，对于ubuntu18.04，只注释掉hostname这一行(127.0.1.1      admin123-Precision-3630-Tower)就可以保证gethostbyname_ex返回主机所有网卡的ip地址。


### 进一步思考，如何获取本机IP地址

如果主机只有一个网卡(可能比较少见了，笔记本都有无线和有线，除非是台式机),或者希望获取默认网卡(eth0?)的IP地址，可以有以下三种方法，最优雅的是第三种，第三种在某种程度上也能解决上述问题，但是只能够获取到默认网卡的IP，如果要获取全部网卡IP，就需要获取所有网卡名(如何获取？),后续可以关注下。

- 不推荐：靠猜测去获取本地IP方法

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import socket
import fcntl
import struct

def get_ip_address(ifname):
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    return socket.inet_ntoa(fcntl.ioctl(
        s.fileno(),
        0x8915,  # SIOCGIFADDR
        struct.pack('256s', ifname[:15])
    )[20:24])

print "br1 = "+ get_ip_address('br1')
print "lo = " + get_ip_address('lo')
print "virbr0 = " + get_ip_address('virbr0')
```

- 通过hostname来获取本机IP

```
import socket
print(socket.gethostbyname(socket.gethostname()))

# 有可能出现这个情况
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
socket.gaierror: [Errno -2] Name or service not known
```

有些机器可能没有设置Hostname，就会出现上面的错误，另外的问题就是上面的那个Linux下获取到127字段IP地址的问题，需要手动修改/etc/hosts


- 通过 UDP 获取本机 IP

方法比较优雅，但是这个目前只能获取到默认网卡对应的IP，获取其他网卡的IP的方法未知。

这个方法没有任何的依赖，也没有去猜测机器上的网络设备信息。

而且是利用 UDP 协议来实现的，生成一个UDP包，把自己的 IP 放如到 UDP 协议头中，然后从UDP包中获取本机的IP。

这个方法并不会真实的向外部发包，所以用抓包工具是看不到的。但是会申请一个 UDP 的端口，所以如果经常调用也会比较耗时的，这里如果需要可以将查询到的IP给缓存起来，性能可以获得很大提升。

```
# 在 shell 中可以一行调用，获取到本机IP
python -c "import socket;print([(s.connect(('8.8.8.8', 53)), s.getsockname()[0], s.close()) for s in [socket.socket(socket.AF_INET, socket.SOCK_DGRAM)]][0][1])"
10.12.189.16


# 可以封装成函数，方便 Python 的程序调用
import socket

def get_host_ip():
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        s.connect(('8.8.8.8', 80))
        ip = s.getsockname()[0]
    finally:
        s.close()

    return ip
```