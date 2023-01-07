---
title: "树莓派(一): 联网、固定ip、改为64位、内网穿透"
date: "2021-02-06"
categories:
  - Linux

tags:
  - 树莓派

toc: true
---

折腾一下，云服务器和nas都偏贵.

买的4B版 4G内存树莓派，包括主板、电源、外壳、散热片。自己外接鼠标、键盘、HDMI线（它这个是小头，最好买机器的时候直接附带买了）

<!--more-->

#####  一、WIFI联网
1. 树莓派连接显示器直接连接
2. 树莓派网线连接电脑登录 ``vim /etc/wpa_supplicant/wpa_supplicant.conf``
	```conf
	ctrl_interface=DIR=/var/run/wap_supplicant GROUP=netdev
	update_config=1
	#country=GB

	network={
	    ssid="WIFI名"
	    psk="密码"
	    key_mgmt=WPA-PSK # 这里不用改
	}
	```
3. 以上配置在树莓派默认桌面版重启后即可连接wifi。 但是我刷了官方提供的0.4G的lite版却不可以。新方式：把 ``wpa_supplicant.conf``恢复原样，使用提示的``sudo raspi-config`` -> ``System Options`` -> ``Wireless LAN``配置。（另外说一下这个版本默认不开启ssh）
##### 二、固定IP
``vim /etc/dhcpcd.conf``
```conf
static ip_address=192.168.0.101/24 #自己设置，注意别冲突，可以事先ping一下
static ip6_address=fd51:42f8:caae:d92e::ff/64
static routers=192.168.0.1
static domain_name_servers=192.168.0.1 8.8.8.8 fd51:42f8:caae:d92e::1
```
#####  三、SSH连接
忘记是否自带ssh了， 账号：``pi`` , 密码：``raspberry`` 。 root可以登录上后直接切。

##### 四、更改为64位
树莓派默认是32位，实际通过设置可以更改为64位
``cd /boot; ls | grep kernel``
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210206142713647.png)
如果存在 kernel8.img 就可以修改
``vim /boot/config.txt`` 末尾添加 ``arm_64bit=1`` 后重启。（如果出现问题导致不能启动内存卡用电脑读取修改回来）

##### 五、内网穿透
树莓派放在家里不费电，做自己的小型服务器很棒，但是问题在于不能直接连入公网。有兴趣的话你再查查有什么好的方式。我是因为本来有一个腾讯云服务器，两端都下载frp，通过配置，使树莓派接入公网。参考：[树莓派 + frp](https://blog.csdn.net/weixin_40973138/article/details/103222901)

1. 包下载：[https://github.com/fatedier/frp/releases](https://github.com/fatedier/frp/releases)，注意两边系统不同，下载的包也不同，树莓派下载arm的，云服务器一般下载amd64的。配置文档 [https://gofrp.org/docs/](https://gofrp.org/docs)
2. 公网服务器解压后启动frps ``./frps -c ./frps.ini`` （要先启动服务端）
3. 树莓派配置 ``frpc.ini`` 更改 ``server_addr``为公网ip后启动frpc ``./frpc -c ./frpc.ini``
4. 使用任何一台电脑，``ssh -oPort=6000 pi@公网ip`` 输入树莓派密码就能ssh登录树莓派啦 (如果端口被拒绝检查公网防火墙端口限制)
5. frp挂起，可以使用nohup，也可以加入systemctl(建议)，拿服务端举例。
	- ``nohup ./frps -c ./frps.ini &`` ，日志会输入到 ``nohup.out``里，这种方式遇到端点没办法自启动了
	- ``vim /lib/systemd/system/frps.service``
		```conf
		[Unit]
		Description=frps service
		After=network.target syslog.target
		Wants=network.target

		[Service]
		Type=simple
		#启动服务的命令（此处写你的frp的实际安装目录）
		ExecStart=/usr/share/frp/frps -c /usr/share/frp/frps.ini

		[Install]
		WantedBy=multi-user.target
		```
		``systemctl start frps``启动
		``systemctl enable frps``开机自启动
6. 我遇到的问题。 经常 frpc 连接2分钟后反复断开重连，偶尔也能长时间保持。报错是 ``work connection closed before response StartWorkConn message: EOF``
	贴个匹配错误点的日志
	![在这里插入图片描述](https://img-blog.csdnimg.cn/2021022313310765.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3o3NzI1MzI1MjY=,size_16,color_FFFFFF,t_70#pic_center)
	此问题，各种尝试无解，包括更换树莓派系统、frp版本、也包括有人说把 frp加入systemctl 管理可以解决。
	用云服务器自己连接自己，无中断现象。
	把树莓派拿公司，连接很稳定。
	所以结论还是家庭网络问题。   不知道是和运营商、路由器之类的什么有关，没有能力去找出来...



##### 六、使用

1. [树莓派仪表盘](https://make.quwj.com/project/10) （Nginx + PHP)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210205150135747.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3o3NzI1MzI1MjY=,size_16,color_FFFFFF,t_70)


2. 高级玩法： [树莓派实验室](https://shumeipai.nxez.com/)

