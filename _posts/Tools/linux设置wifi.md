---
title: linux设置wifi
date: 2020-11-12 19:48:20
categories: Tools
tags: linux
---

# 查看wifi网卡

`sudo ifconfig -a`
一般w开头的是wifi网卡

# 启动wifi网卡

`sudo ifconfig <wifi网卡名> up`

# 搜索wifi

`sudo iwlist <wifi网卡名> scan`

这个命令能看到每个wifi的所有信息,但我只需要wifi名字就可以了。所以可以使用下面这条命令进行筛选

`sudo iwlist <wifi网卡名> scan | grep -i ssid`

# 生成连接wifi的配置文件

`sudo wpa_supplicant {wifi名} {wifi密码} > /etc/wpa_supplicant/{wifi名}.conf`

**使用这条命令的时候，虽然我加了sudo，但是还是提示权限不足。可以直接`su` 切换到`root`执行**

# 连接wifi

`sudo wpa_supplicant -i {wifi网卡名} -c /etc/wpa_supplicant/{wifi名}.conf -B`

# 自动分配IP

`sudo dhclient {wifi网卡名}`

查看是否连接成功: `sudo ifconfig`



