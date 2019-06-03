# 安装和配置ubuntu

本文主要描述重装/安装ubuntu系统后的主要配置。


## 笔记本安装linux系统

本文以安装ubuntu为例而展开。

参考：
https://cloud.tencent.com/developer/article/1046051

最近脑子发热突然很想学习linux，希望linux作为使用MAC前的一个踏板，另外陪伴自己10余年的笔记本电脑已经明显跟不上时代的步伐，windows从一定意义上非常不好使，种种原因加速了用老笔记本电脑来安装linux的动力，怎么着都得让笔记本在丢掉之前再次发挥一下余热。

本人HP probook 4411亲测下来可以安装ubuntu 14/18.10，而ubuntu 16/18.04在安装过程中都会出现问题。

### 制作Linux系统U盘

- 准备一个超过5GB的U盘，备份其中的数据到其他的移动硬盘因为后续要格式化这个U盘，所有数据都没用了
- 官网下载ubuntu的iso镜像文件
- [软碟通官网](https://cn.ultraiso.net/xiazai.html) 下载utraiso，试用版即可，只要能够在U盘作出一个启动盘
  
  ![image1](./resource/1.png)

- 然后再软件打开我们的linuxiso镜像，文件-打开-选择
  
![image2](./resource/2.png)

- 然后把这个系统写入到u盘上边去,启动-写入硬盘映像
![image3](./resource/3.png)


![image4](./resource/4.png)

- 在写入之前需要先格式化u盘，选择FAT32即可，然后需要注意就是硬盘驱动器选择自己的u盘，写入方式为USB-HDD+，这里要看仔细，注意备份数据
  
![image5](./resource/5.png)


- 耐心等待进度条，这一步完成之后u盘就做好了，可以插入笔记本装系统了


### 配置笔记本U盘启动/安装linux

- 首先把u盘插入笔记本，开机设置u盘为第一启动顺序，我用的HP按ESC进入bios，设置boot为usb  
- 根据傻瓜式的引导来安装ubuntu



## 20180520 ubuntu下运行windows程序

不得不承认某些APP在windows下是很友好的，比如notepad++，在linux下不能直接使用。

在我们从Windows过渡到Ubuntu后，很多本来在Windows上玩的很熟的软件却无法在Ubuntu上运行。如果说应用软件还能在Ubuntu 上找到相应的替代品的话，那么Windows上的游戏就真的没辙了，现在热门的一些游戏几乎都没有Linux版本，当然也就无法在Ubuntu上运行了。 很多人虽然对Linux抱有很大的兴趣，但仍旧无法摆脱Windows，因为你不得不面对Linux上复杂的软件应用和无法运行热门游戏的现实。不过现在 Linux已经不再像以前那样和Windows毫无瓜葛了，我们只要借助一款名为“Wine”的软件就可以架起Linux和Windows的桥梁，在 Ubuntu上运行Win程序和游戏不再是梦想。

### 什么是Wine？

[Wine](https://www.winehq.org/)是 Wine Is Not an Emulator 的所写，是一个在Xwindows和Unix上执行的Windows APIs，也可以认为Wine是一个Windows兼容层。通过Wine，我们可以在Ubuntu中运行windows程序。

在Ubuntu中，可以通过两种方法安装Wine，一种是自己编译Wine代码，另一种是直接下载安装。自己编译Wine代码的好处是编译好的Wine兼 容性好，在运行程序时很少会出现问题，比较稳定，缺点是步骤麻烦。下载安装Wine的好处自然是方便，缺点就是不稳定，经常容易出现问题。这里还是推荐大 家自己编译Wine，如果编译不成功，再通过下载安装Wine。

因为只想快速使用notepad++，我选择了[直接下载的方式](https://wiki.winehq.org/Ubuntu)

参考连接：
https://blog.51cto.com/vincent/100851


## 20180527 ubuntu配置shadowsocks client