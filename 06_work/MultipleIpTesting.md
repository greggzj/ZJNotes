

## 环境配置

## 遇到问题和解决办法

### 在EP板子上使用curl命令访问其他设备的web server
解决办法：`curl --noproxy "*" URL:PORT`

默认使用`curl (--interface XX) URL:PORT`会出现zscaler proxy的错误，原因应该是默认curl使用了在env中存在http_proxy/https_proxy走配置好的zscaler gateway，每次发送curl时不走Proxy即可解决。


## 遗留问题