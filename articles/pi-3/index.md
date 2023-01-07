---
title: "树莓派(三): 更换ubuntu系统、ubuntu server系统wifi连接"
date: "2021-02-09"
categories:
  - Linux

tags:
  - 树莓派

toc: true
---

<!--more-->

#### 一、准备
1. 树莓派（支持WIFI）
2. 读卡器
3. 一台电脑（就是看我的这台）

##### 一、刻录系统
1. ubuntu官网已经提供树莓派的系统 [https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview)
2. 下载软件 ，选择系统时注意32位还是64位，另外是桌面版还是server版。 我下载了桌面版感觉没啥用，又换了server版。 选择SD卡后等待，中途不能断。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210209092455832.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3o3NzI1MzI1MjY=,size_16,color_FFFFFF,t_70)
3.  刻录后根据文档去设置wifi 。

#### 二、ubuntu连接wifi设置
1. 如果你连接wifi是5G频段，需要设置 ``/etc/default/crda``中``REGDOMAIN=CN`` (太坑了)
2. ``vim /etc/netplan/xxxxxxx.yaml  `` # 不同的机器文件名不一样
设置的格式和上边ubuntu提供的倒是差不多。 另外需要注意缩进4个空格。
	```conf
	network:
	    ethernets:
	        eth0:
	            dhcp4: true
	            optional: true
	    version: 2
	    wifis:
	        wlan0:
	            dhcp4: true
	            access-points:
	                "你的WFI名":
	                    password: "你的WIFI密码"
	```
3. 根据帖子是
	```shell
	sudo netplan try
	sudo netplan apply
	```
	但是我try后没有生效，``reboot``重启后生效的。看过好多种设置wifi的方法了，太懵了。 祝你成功吧。

