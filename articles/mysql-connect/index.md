---
title: "mysql8远程连接报错"
date: "2019-09-21"
categories:
  - MySQL
---


报错：``Authentication plugin 'caching_sha2_password' cannot be loaded``
原因: MySQL从8.0开始更改了密码的加密方式为`caching_sha2_password`，而一般远程连接软件使用`mysql_native_password`
解决方法：
``vim /etc/my.ini``或``vim /usr/local/etc/my.cnf``

	在``[mysqld]``下添加``default_authentication_plugin=mysql_native_password``

	``ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'xxxxx';``


<!--more-->
