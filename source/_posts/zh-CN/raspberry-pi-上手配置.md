---
title: raspberry-pi 上手配置
lang: zh-CN
date: 2018-04-13 17:41:45
tags: raspberry pi
---

### 安装系统
- 我安装的是little不带桌面的树莓派系统。
<!--more-->
### 登录进树梅派
- 连接显示器和键盘，默认登陆名和密码`pi:raspberry`
- 如果没有显示器和键盘的，需要ssh的话比较麻烦和困难：
	1.需要知道树莓派的ip，这就需要树梅派连接到wifi或者内网中
	2.不知从哪一个版本开始树莓派默认不支持ssh。如果是windows系统，需要在sd卡的根目录新建一个名称为`ssh`的空文件；如果是已经在树梅派系统里面`sudo touch /boot/ssh`。
### 配置树梅派连接WIFI。
	1.配置无线局域网的ssid和密码，先确定树莓派可以搜到我们的ssid，`sudo iwlist wlan0 scan | grep chinac`，我这里的chinac是ssid的名称，然后修改配置`sudo vi /etc/wpa_supplicant/wpa_supplicant.conf `
```shell
country=GB
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
        ssid="无线网的ssid，也就是名称"
        psk="无线密码"
        priority=9 #优先级，数字越大优先连到这个wifi
}

```
	2.配置interfaces,`sudo vi /etc/network/interfaces`，指定wlan0的配置如下，其中wpaconf的配置的地址就是第一步中我们配置的无线局域网的文件的绝对路径。
```
# interfaces(5) file used by ifup(8) and ifdown(8)
# Please note that this file is written to be used with dhcpcd
# For static IP, consult /etc/dhcpcd.conf and 'man dhcpcd.conf'
# Include files from /etc/network/interfaces.d:
source-directory /etc/network/interfaces.d

auto wlan0
allow-hotplug wlan0
iface wlan0 inet dhcp
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

```
	3.重启网卡`sudo /etc/init.d/networking restart`，然后`ifconfig wlan0`查看是否获取到了ip，然后通过这个ip地址进行ssh访问树梅派，而不需要连接显示器和键盘。我的返回如下：
```
pi@raspberrypi:~ $ ifconfig wlan0
wlan0     Link encap:Ethernet  HWaddr b8:27:eb:42:7c:4c  
          inet addr:10.21.2.30  Bcast:10.21.2.255  Mask:255.255.255.0
          inet6 addr: fe80::9cea:caae:37b9:7e30/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:175309 errors:0 dropped:0 overruns:0 frame:0
          TX packets:35595 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:104320764 (99.4 MiB)  TX bytes:3293509 (3.1 MiB)

```
### 参考资料
[明明白白玩 Pi 系列之三](https://sspai.com/post/38900)
