---
title: Linux-Debian配置shadowsocks客户端与终端代理配置
date: 2016-10-24 18:57:31
tags: [Linux,ShadowSocks]
---

无论是自己购买vps搭建ss服务，还是购买别人搭好的ss服务，在Windows、Mac、Andorid、ios上只需要下载对应的软件按步骤操作即可。那么在Linux如上如何配置shadowsocks来科学上网呢，本文就Ubuntu、Kali等Debian系Linux的配置方法做一个记录性的介绍。

<!-- more -->

# 安装shadowsocks
## pip安装
安装依赖
```
sudo apt-get update
sudo apt-get install python-pip
sudo apt-get install python-setuptools m2crypto
```
安装shadowsocks
```
sudo pip install shadowsocks
```

## apt安装
如果是ubuntu16.04 直接用`apt`而不用`apt-get`这是一项改进
```
sudo apt-get install shadowsocks
```
在安装时有时候提示需要安装一些依赖比如python-setuptools、m2crypto ，依照提示进行安装即可

# 启动shadowsocks
通过`sslocal --help`查看帮助
```
$ sslocal --help
option --help not recognized
usage: sslocal [-h] -s SERVER_ADDR [-p SERVER_PORT]
               [-b LOCAL_ADDR] [-l LOCAL_PORT] -k PASSWORD [-m METHOD]
               [-t TIMEOUT] [-c CONFIG] [--fast-open] [-v] [-q]

optional arguments:
  -h, --help            show this help message and exit
  -s SERVER_ADDR        server address
  -p SERVER_PORT        server port, default: 8388
  -b LOCAL_ADDR         local binding address, default: 127.0.0.1
  -l LOCAL_PORT         local port, default: 1080
  -k PASSWORD           password
  -m METHOD             encryption method, default: aes-256-cfb
  -t TIMEOUT            timeout in seconds, default: 300
  -c CONFIG             path to config file
  --fast-open           use TCP_FASTOPEN, requires Linux 3.7+
  -v, -vv               verbose mode
  -q, -qq               quiet mode, only show warnings/errors

Online help: <https://github.com/clowwindy/shadowsocks>
```
`sslocal -s 11.22.33.44 -p 50003 -k "123456" -l 1080 -t 600 -m aes-256-cfb`  

`-s`表示服务器ip或域名  
`-p`表示服务器的端口  
`-l`表示本地端口，默认是1080  
`-k`表示密码，要加""  
`-t`表示超时时间，默认300  
`-m`加密方法，默认aes-256-cfb  

但是为了方便，推荐直接用`sslcoal -c 配置文件路径`这样的方式  
在一个合适的路经位置新建名为`shadowsocks.json`的文件，内容如下：
```
{
"server":"11.11.11.11",		#服务端的IP
"server_port":50003,		#务端的端口
"local_port":1080,		#本地端口，一般默认1080
"password":"123456",		#服务端为你设置的密码
"timeout":600,			#超时设置
"method":"aes-256-cfb"		#加密方法
}
```

确定上面设置没有问题后，终端输入`sslocal -c .../PATH/shadowsocks.json`启动即可，会有如下的提示
```
INFO: loading config from shadowsocks.json
shadowsocks 2.1.0
2016-10-24 19:35:23 INFO     starting local at 127.0.0.1:1080
```
现在就可以去设置-Network处设置代理，代理-手动-Socks主机处填入`127.0.0.1 1080`等对应信息即可

但这样设置代理是全局的，然而访问`baidu.com`、`csdn.net`等网站也走代理就显得比较绕且没有必要，而且终端并不能使用到此代理，接下来逐个解决.  

# 配置浏览器自动切换

以Chrome为例，别的浏览器类似

## 安装插件

安装`SwitchyOmega`插件，但是没有代理之前是不能从谷歌商店安装这个插件的.  

替代方案是从Github上直接下载[最新版](https://github.com/FelisCatus/SwitchyOmega/releases/),然后浏览器地址打开`chrome://extensions/`，将下载的插件托进去安装  

## 设置代理地址
安装好插件会自动跳到设置选项，会弹出提示、你可以跳过  

左边点击`新建情景模式` ---> 选择`代理服务器`随意命名比如SS ---> 点击`创建`，之后在`代理协议`选择`SOCKS5`，代理服务器为`127.0.0.1`，`代理端口`为默认的`1080` ，然后点击左下角`应用选项`保存该情景模式.如下两图
{% asset_img shadowsocks-0.png image %}
{% asset_img shadowsocks-1.png image %}

## 设置自动切换

接着点击左侧自动切换 ，给`规则列表规则`前打钩，该行的情景模式选择前面创建好的`SS`，下一行的`默认情景模式`选择`直接连接`  

再往下`规则列表设置`处选择`AutoProxy`，然后将这个地址`https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt`填进去，点击下面的`立即更新情景模式`，会有提示更新成功  

最后点击左侧的`应用选项`将刚才的所有设置保存，如下图

{% asset_img shadowsocks-2.png image %}

然后点击浏览器右上角的`SwitchyOmega`小圆圈图标，下面选择`自动切换`，然后打开`google.com`试试，打开`ip.gs`看看  

{% asset_img shadowsocks-3.png image %}

# 开机后台自动运行ss

按照之前的操作都需要保持运行sslocal的终端不关闭，当然你可以使用`&、nohup`等工具让命令在后台运行。  

为了避免每次开机或者关闭终端后要重新输入那一串的命令，以及服务的稳定可靠可监控，现在使用`supervisor`将sslocal转化为守护进程在后台运行，再将`supervisor`服务添加进开机自动启动。

[supervisor文档](http://supervisord.org/)

> Linux的后台进程运行有好几种方法，例如nohup，screen等，但是，如果是一个服务程序，要可靠地在后台运行，我们就需要把它做成daemon，最好还能监控进程状态，在意外结束时能自动重启。
> 
> supervisor就是用Python开发的一套通用的进程管理程序，能将一个普通的命令行进程变为后台daemon，并监控进程状态，异常退出时能自动重启。

## 安装
```
sudo apt-get install supervisor
```
## 修改配置文件
```
sudo vim /etc/supervisor/supervisor.conf
```
## 加入如下内容
```
[program:shadowsocks]
command=sslocal -c .../PATH/shadowsocks.json
autostart=true
autorestart=true
user=root
log_stderr=true
logfile=/var/log/shadowsocks.log
```
## 重启服务
```
sudo service supervisor restart
```
去浏览器开个google看看代理是否正常，也可以用`ps -ef|grep sslocal`命令查看sslocal是否在运行

## 让supervisor开机启动
```
sudo vim /etc/rc.local
```
在配置文件的`exit 0`前面加上如下一行
```
service supervisor start
```
现在可以重启机器后直接去浏览器访问google试一下了  

## 参考
[linux-ubuntu使用shadowsocks客户端配置](https://aitanlu.com/ubuntu-shadowsocks-ke-hu-duan-pei-zhi.html)
在此感谢这位博主的精彩文章，以及精彩截图，以上几张图均出自此处

# 为终端设置Shadowsocks代理

常用命令行工具的人经常会在终端下进行一些网络操作，比如下载gradle包，比如使用docker拉取镜像，比如使用npm安装软件  

由于一些你懂我也懂的原因，某些网络操作不是那么理想，直接操作会非常缓慢、甚至很多时候会直接失败，虽然可以采用换源的方式，如docker使用daocloud、npm使用淘宝源、pip使用豆瓣源，但配置终端代理来访问网络终究要显得更`自由`  

Shadowsocks是我们常用的代理工具，它使用socks5协议，而终端很多工具目前只支持http和https等协议，对socks5协议支持不够好，所以我们为终端设置shadowsocks的思路就是将socks协议转换成http协议，然后为终端设置即可  

这里借助工具进行转换。`polipo`是一个轻量级的缓存web代理程序  

## 安装
```
sudo apt-get install polipo
```
## 修改配置文件
```
sudo vim /etc/polipo/config
```
## 设置为以下内容
`socksParentProxy`为ss运行的地址
```
logSyslog = true
logFile = /var/log/polipo/polipo.log
logLevel=4

socksParentProxy = "127.0.0.1:1080"
socksProxyType = socks5
```
##启动
**必须**先关闭正在运行的polipo，然后再次启动，如果只是`restart`话不生效
```
sudo service polipo stop
sudo service polipo start
```
## 验证及使用
```
$ curl ip.gs
当前 IP：115.190.117.22  来自：中国山东青岛 电信
$ export http_proxy=http://localhost:8123
$ curl ip.gs
当前 IP：210.140.193.128 来自：日本日本
$ unset http_proxy
$ curl ip.gs
当前 IP：115.190.117.22  来自：中国山东青岛 电信
```
`8123`	是polipo的默认端口，如有需要，可以修改成其他有效端口  

使用`export`将代理设置为本次会话全局使用(不关闭当前终端)，`unset`取消设置.  

如果想要更长久的设置代理，可以将`export http_proxy=http://localhost:8123`加入`.bashrc`或者`.bash_profile`文件，然后`source ~/.bashrc`生效

## 参考
[Linux、Mac终端设置Shadowsocks代理](http://droidyue.com/blog/2016/04/04/set-shadowsocks-proxy-for-terminal/)  

[Linux终端挂代理方法整理](http://www.jianshu.com/p/8e7d7f57bf59)
