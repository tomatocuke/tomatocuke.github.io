---
title: "Ubuntu搭建LNMP"
date: "2020-08-06"
categories:
  - Linux
toc: true
---



本文介绍Linux环境下Nginx+MySQL+Redis+PHP的环境搭建。

其中Nginx和Redis的搭建比较简单，MySQL在配置连接上有点不同，区分8.0和8.0以下，问题主要原因在于默认密码加密方式的更换。 PHP的槽点较多，当你看到这篇文章时，PHP不一定在哪个版本了，仅供参考。

<!--more-->


> 操作系统 Ubuntu Server 18.04.1 LTS 64位

#### 一、nginx
1. 安装 ``apt install nginx -y``
2. 启动 ``service nginx start``  (如果先安装PHP，附带安装apache2，占用80端口导致nginx无法启动)


#### 二、MySQL
- 安装 ``apt install mysql-server -y``
- 安装 ``apt install mysql-client -y`` (可选，包含一系列工具，备份压测等)
- 启动 ``service mysql start``
- 进入 ``mysql -uroot -p`` (如果找不到mysql按照提示安装)
- 修改密码
	-  ``use mysql;``
	-  8.0.11以下：``update user set authentication_string=password("你的密码") where user="root";`` ，
	- 8.0.11以上： ``update user set plugin="mysql_native_password"; `` 、``alter user 'root'@'localhost' identified with mysql_native_password by '你的密码';``
	- ``update `user` set Host = '%' where User = 'root';``  (远程连接需要允许host)
	- ``flush privileges;``
- 遇到错误``The user specified as a definer ('mysql.infoschema'@'localhost') does not exist``
	- ``DROP USER 'mysql.infoschema'@'localhost';``
	- ``CREATE USER 'mysql.infoschema'@'localhost' IDENTIFIED BY 'password';``
	- ``GRANT SELECT ON *.* TO `mysql.infoschema`@`localhost`;``

#### 三、PHP
###### 1. 安装php
-  ``apt install php -y``
-  ``apt install php-xml -y``
-  ``apt install php-mysql -y`` (这个也要自己装，PDO扩展显示no value困扰我好一会儿)
###### 2. 安装php-fpm
- 安装php-fpm  ``apt install php-fpm -y``   &
 - nginx配置 `` vim /etc/nginx/conf.d/test.conf``
	```conf
	server {
	    listen 80 default;
	    server_name 127.0.0.1;
	    root /var/www;
	    index index.php;

	    location ~ \.php {
	        fastcgi_pass    127.0.0.1:9000;
	        include         fastcgi.conf;
	    }
	}
	```
	``nginx -s reload``
- 编辑php-fpm配置 ``vim /etc/php/7.2/fpm/pool.d/www.conf`` ，找到 ``listen = /run/php/php7.2-fpm.sock``， 改为 ``listen = 9000`` (这个配合nginx的监听本地9000端口，其自带的那种方式我没研究过)
 - 启动php-fpm ``service php7.2-fpm start``
 - 在 ``/var/www``下创建php输出phpinfo。


#### 五、远程连接MySQL
- 进入mysql & ``use mysql;`` & ``update user set host = '%' where user = 'root';`` & ``flush privileges;``
-  ``vim /etc/mysql/mysql.conf.d/mysqld.cnf`` 注释掉 ``bind-address        = 127.0.0.1`` -> ``service mysql restart``
- 安装防火墙管理 (自带ufw的话也可以、或者云厂商控制台开启端口）
	-  ``apt install firewalld -y``
	- 开启3306端口对外访问 ``firewall-cmd --zone=public --add-port=3306/tcp --permanent``
	- 使之生效 ``firewall-cmd --reload`` ，查看暴露端口``firewall-cmd --list-ports``
- 如果还不可以，查看云服务安全组端口限制


#### 六、安装redis并远程连接
- 安装redis客户端 ``apt install redis -y``
- 安装php的redis扩展 ``apt install php-redis -y``
- 开启远程访问
	- ``vim /etc/redis/redis.conf`` & 注释掉``bind 127.0.0.1 ::1`` & 设置 ``protected-mode no``
	- 开启6379端口对外访问 ，参考上边
- 使用密码 ``vim /etc/redis/redis.conf`` &&  ``requirepass 你的密码``
- 再次进入 ``redis-cli``时，命令不能正常使用 ``(error) ERR invalid password`` ， 输入 ``auth 你的密码`` 后正常
- 记录一个报错 ``Failed opening the RDB file root (in server root dir /var/spool/cron) for saving: Read-only file system``
	- 该目录与``/etc/redis/redis.conf``里的 ``dir /var/lib/redis``不符。
	- 进入cli模式 ``redis-cli``
	- 查看``config get dir`` & 更改配置 ``config set dir /var/lib/redis``  (不能持久)
	- ``config rewrite`` （使之持久）

#### 七、其他
- root使用ssh登录
	- 修改配置 ``vim /etc/ssh/sshd_config`` & 取消注释``Port 22`` & ``PermitRootLogin yes``
	- 修改密码 ``sudo passwd root``

