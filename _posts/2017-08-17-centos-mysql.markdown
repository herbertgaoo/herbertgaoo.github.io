---
layout: post
title: centos7安装Mysql数据库和远程访问配置
date: 2017-08-17 10:50:24.000000000 +09:00
tag: Linux Mysql
---

 平时很少操作服务器相关的东西，昨天正好有这样的机会，帮朋友在centos下安装Mysql服务。虽然内容比较low，但我还是觉得应该写个笔记记录一下，毕竟后面会忘的。

### 系统环境
yum update升级以后的系统版本为
```bash
[root@yl-web yl]# cat /etc/redhat-release
CentOS Linux release 7.1.1503 (Core)
```

### Mysql 安装
建议安装顺序
```bash
#yum install mysql
#yum install mysql-devel
#yum install mysql-server
```
一般情况下安装mysql-server会报如下错误：

``` bash
[root@yl-web yl]# yum install mysql-server
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.sina.cn
 * extras: mirrors.sina.cn
 * updates: mirrors.sina.cn
No package mysql-server available.
Error: Nothing to do
```
原因是CentOS 7 版本将MySQL数据库软件从默认的程序列表中移除，用mariadb代替了。所以有两种方法解决：1.`使用mariadb`，2.`安装Mysql` 下面分别介绍两种方法。

#### · 安装mariadb
MariaDB数据库管理系统是MySQL的一个分支，主要由开源社区在维护，采用GPL授权许可。开发这个分支的原因之一是：甲骨文公司收购了MySQL后，有将MySQL闭源的潜在风险，因此社区采用分支的方式来避开这个风险。MariaDB的目的是完全兼容MySQL，包括API和命令行，使之能轻松成为MySQL的代替品。

``` bash
[root@yl-web yl]# yum install mariadb-server mariadb
```

mariadb数据库的相关命令是：
systemctl start mariadb  #启动MariaDB
systemctl stop mariadb  #停止MariaDB
systemctl restart mariadb  #重启MariaDB
systemctl enable mariadb  #设置开机启动
所以先启动数据库, 然后就可以正常使用mariadb了。（PS: 默认root用户没有密码）

``` bash
[root@yl-web yl]# systemctl start mariadb
```
``` bash
[root@yl-web yl]# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 5.5.41-MariaDB MariaDB Server

Copyright (c) 2000, 2014, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)

MariaDB [(none)]>
```
As you see, 这不是处女座想要的Mysql，所以接下来介绍安装Mysql。

### 下载安装mysql-server

``` bash
# wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
# rpm -ivh mysql-community-release-el7-5.noarch.rpm
# yum install mysql-community-server
```
安装完成后启动Mysql服务。
``` bash
# service mysqld restart
```
初次安装的Mysql root用户没有密码。

``` bash
[root@yl-web yl]# mysql -u root
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.6.26 MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.01 sec)

mysql>
```
看到mysql后，修改root用户密码, 修改后立即生效。
``` bash
mysql>use mysql;
mysql> update user set password=passworD("root") where user='root';
mysql> flush privileges;
mysql> exit;
```

### 配置Mysql及远程登陆

mysql的配置文件位于`/etc/my.cnf`, 编辑my.cnf文件，在文件最后加入默认编码配置。
``` bash
[mysql]
default-character-set =utf8
```

#### 远程连接配置
把在所有数据库的所有表的所有权限赋值给位于所有IP地址的root用户。
``` bash
mysql> grant all privileges on *.* to root@'%'identified by 'password';
```
在执行上面命令时可能会报错，提示`Please use mysql_upgrade to fix thiserror.`, 此时需要运行如下命令：

``` bash
[root@yl-web yl]#  mysql_upgrade -u root -p
Enter password:
```

然后就可以远程连接Mysql了。(PS: 需要服务器防火墙开启3306端口)





