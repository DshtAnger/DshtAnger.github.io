---
title: Linux下修改网卡默认Mac地址
date: 2016-09-15 13:38:58
tags: [Linux,WiFi]
---

在进行一些WiFi攻防测试时，有时候需要手动修改Mac地址。这里记录下一些经测试的可行方法和问题，以下在Uububtu、Kali等系统使用过

<!-- more -->

## 临时修改
```
sudo ifconfig wlan0 down
sudo ifconfig wlan0 hw ether aa:bb:cc:dd:ee:ff
sudo ifconfig up
```
或者用`macchanger`工具
```
sudo ifconfig wlan0 down
sudo macchanger -r wlan0 //随机mac
sudo macchanger -m aa:bb:cc:dd:ee:ff wlan0  //指定mac
sudo ifconfig wlan0 up
```
这样修改后，连接到目标WiFi后，有时会获取到ip地址，但出现默认路由获取不到的问题。直接的现象就是明明连上了wifi，浏览器里出现下面这样的提示：
```
未连接到互联网
请试试以下办法：
	检查网线、调制解调器和路由器
	重新连接到 Wi-Fi 网络
DNS_PROBE_FINISHED_NO_INTERNET
```

解决办法如下：
```
sudo dhclient wlan0
```
最后`重启网络服务`：
```
sudo /etc/init.d/networking restart 
```

完整的设置临时随机mac脚本如下：
```
#!/bin/bash
#48:d2:24:39:da:6a
ifconfig wlan0 down
macchanger -r wlan0
ifconfig wlan0 up
dhclient wlan0
ifconfig wlan0
```
也可以使用永久修改的方式指定dhcp获取各种地址信息，推荐这种方法。

## 永久修改
`sudo vim /etc/network/interfaces`
```
#加入如下内容
auto wlan0
iface wlan0 inet dhcp
pre-up ifconfig wlan0 hw ether aa:bb:cc:dd:ee:ff
```
重启网络服务后一般问题就解决了：
```
sudo /etc/init.d/networking restart 
```
