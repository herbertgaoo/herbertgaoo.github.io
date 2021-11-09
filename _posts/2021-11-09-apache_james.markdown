---
layout: post
title: Apache James 邮件服务部署
date: 2021-11-09 18:54:24.000000000 +08:00
tag: James
---

## 邮件服务器可选软件
---
>软件1、`Apache James`  
>软件2、`linagora`  
>软件3、`sendmail`  

## 官网  

---
http://james.apache.org/


>James Components(组件):
> - Emailing protocols: SMTP, LMTP, POP3, IMAP, ManageSieve, JMAP
> - Mailet container: independent, extensible and pluggable email processing agents
> - Storage API: Mailbox API / Search API / User API
> - Storage Implementations: Cassandra / PostgreSQL / HSQLDB / MySQL / ElasticSearch...
> - Administration: JMX / REST / Command Line
> - James Core

### 端口调用  
>Remote Manager Service started plain:4555  
POP3 Service started plain:110  
SMTP Service started plain:25  
NNTP Service started plain:119  
IMAP Service started plain:143  
POP3 收件箱  
IMAP 收件箱  
SMTP 发件箱  


#### 服务器IP 
> 192.168.2.111

#### 官网下载  
- 1、Binary  
https://james.apache.org/download.cgi

- 2、james-server-app-3.5.0-app.zip
https://archive.apache.org/dist/james/server/3.5.0/


## 环境部署
---

邮件服务器需要调用的端口，查看端口是否被调用, 如果端口被占用，kill了并关掉开机启动项
``` sh
[root@localhost ~]# netstat -tunlp | grep -E '4555|9999|110|25|119|143'
```

### 安装jdk-1.8
- james-3.5.0 可以使用 jdk-1.8
- james-3.6.0 必须使用 jdk-1.8以上
``` sh
[root@localhost ~]# yum install jdk-8u121-linux-x64.rpm -y
```

### 安装james & 启动

``` sh
[root@localhost ~]# ls /backup/james-server-app-3.5.0-app.zip

[root@localhost ~]# yum install unzip -y

[root@localhost ~]# cd /backup/ && unzip james-server-app-3.5.0-app.zip

[root@localhost ~]# mv /backup/james-server-app-3.5.0 /app/

[root@localhost ~]# /app/james-server-app-3.5.0/bin/run.sh
```

- 出现以下信息启动成功  
``` sh
09-Nov-2021 19:58:03.442 INFO [main] org.apache.james.app.spring.JamesAppSpringMain.main:46 - Apache James Server is successfully started in 38766 milliseconds.
```
#### james目录
- `bin`: 程序各种工具和运行程序目录
- `lib`: 依赖包
- `conf`: 配置目录，我们配置就修改这里面的配置文件
- `var`: jamse server存储数据目录


### james配置-实现pop3，smtp收发邮件

- 配置hosts文件
``` sh
[root@localhost ~]# vim /etc/hosts
192.168.2.111 ibmtest.com
```

- 简单for循环查找配置关键字
``` sh
for i in $(find . -type f)
do
   echo $i
   grep authorized $i
done
```
#### 1、添加邮箱后缀  
`注意：如果是在本地模拟则需在hosts中做域名映射，否则会出现意想不到的问题`
``` sh
[root@localhost ~]# vim /app/james-server-app-3.5.0/conf/domainlist.xml
```
``` xml
<!-- Before -->
<domainlist class="org.apache.james.domainlist.jpa.JPADomainList">
   <autodetect>true</autodetect>
   <autodetectIP>true</autodetectIP>
   <defaultDomain>localhost</defaultDomain>
</domainlist>


<!-- After -->
<domainlist class="org.apache.james.domainlist.jpa.JPADomainList">
   <autodetect>false</autodetect>
   <autodetectIP>false</autodetectIP>
   <defaultDomain>ibmtest.com</defaultDomain>
</domainlist>

```


- ibmtest.com       邮箱域名不可以有下划线
- autodetect:true   表示自动侦测主机名,设成false会使用指定的servername
- autodetectIP:true 表示会为你的servername加上ip，所以直接false就行
- defaultDomain     就改成你所需要的域名就行

>注: 
[可能遇到的报错]:
nested exception is java.net.UnknownHostException: xxx:xxx
将xxx添加映射到/etc/hosts后重启即可.


#### 2、配置DNS
``` sh
[root@localhost ~]# cat /etc/resolv.conf
nameserver 10.4.41.2
nameserver 10.4.43.218
```

##### **DNS查看查找方法**
``` sh
[root@localhost ~]# netstat -rn
#Kernel IP routing table
#Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
#192.168.2.0     0.0.0.0         255.255.255.0   U         0 0          0 eth0
#169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 eth0
#0.0.0.0         192.168.2.1     0.0.0.0         UG        0 0          0 eth0

```


##### **再查看主机dns**
``` dos
#[d:\~]$ ipconfig /all
#DNS 服务器: 10.4.41.16
            10.4.41.18
            10.4.41.1
            10.4.41.2

```

``` xml
[root@localhost ~]# vim /app/james-server-app-3.5.0/conf/dnsservice.xml

<servers>
    <server>10.4.41.2</server>
    <server>10.4.43.218</server>
    <server>10.4.41.16</server>
    <server>10.4.41.18</server>
</servers>
<autodiscover>false</autodiscover>
<authoritative>false</authoritative>

```

- autodiscover  自动查找
- authoritative 自动查找权威的DNS服务器(根域名)


#### 3、配置SMTP
``` sh
[root@localhost ~]# vim /app/james-server-app-3.5.0/conf/smtpserver.xml
```
``` xml
不做修改
<bind>0.0.0.0:25</bind>
<connectionBacklog>200</connectionBacklog>

不释放此配置
<!--
<helloName autodetect="false">ibmtest.com</helloName>
-->

更改<!--重要-->
<authorizedAddresses>0.0.0.0</authorizedAddresses>
<!-- authorizedAddresses文档中描述注释掉可以开启身份验证 -->
```


#### 4、配置POP3
``` sh
[root@localhost ~]# vim /app/james-server-app-3.5.0/conf/pop3server.xml
```
``` xml
不做修改
<bind>0.0.0.0:110</bind>
释放1
<helloName autodetect="false">ibmtest.com</helloName>
```

--------------------------
#### 5、配置IMAP
``` sh
[root@localhost ~]# vim /app/james-server-app-3.5.0/conf/imapserver.xml
```
``` xml
不做修改
<bind>0.0.0.0:143</bind>
释放1
<helloName autodetect="false">ibmtest.com</helloName>
```

--------------------------
#### 6、配置邮件存储
```
[root@localhost ~]# vim /app/james-server-app-3.5.0/conf/mailetcontainer.xml
```
``` xml
<!-- Before1 -->
<context>
<postmaster>postmaster@localhost</postmaster>
</context>

<!-- After1 -->
<!-- postmaster修改为管理员邮箱postmaster@ibmtest.com -->
<context>
<postmaster>postmaster@ibmtest.com</postmaster>
</context>


<!-- RemoteAddrNotInNetwork所在节点注释掉 -->
<!--
<mailet match="RemoteAddrNotInNetwork=127.0.0.1" class="ToProcessor">
    <processor>relay-denied</processor>
    <notice>550 - Requested action not taken: relaying denied</notice>
</mailet>
-->
```



#### 7、配置客户端(james-cli.sh)连接(创建账号使用)
``` sh
[root@localhost ~]# vim /app/james-server-app-3.5.0/conf/jmx.properties
可以不做修改(只能本地创建用户)
jmx.address=127.0.0.1
jmx.port=9999
```


#### 8、创建测试用户

``` sh
[root@localhost ~]# netstat -tunlp | grep java
tcp6    0    0 127.0.0.1:9999    :::*    LISTEN    4940/java 

# 使用james-cli命令配置邮件后缀域名和用户
[root@localhost ~]# cd /app/james-server-app-3.5.0/bin/
./james-cli.sh -h 127.0.0.1 -p 9999 AddUser test1@ibmtest.com test
./james-cli.sh -h 127.0.0.1 -p 9999 AddUser test2@ibmtest.com test
#返回值
AddUser command executed sucessfully in 303 ms.

#查看用户
./james-cli.sh -h 127.0.0.1 -p 9999 ListUsers

```

``` dos
# 配置测试foxmail的windows的hosts文件
C:\Windows\System32\drivers\etc\hosts
192.168.2.111 ibmtest.com

测试配置好的hosts
[D:\~]$ ping ibmtest.com
正在 Ping ibmtest.com [192.168.2.111] 具有 32 字节的数据:
来自 192.168.2.111 的回复: 字节=32 时间<1ms TTL=64
```

## 配置客户端foxmail
`设置`
| key    | value  |  
|-----------|-------------|
| Email地址|test1@ibmtest.com |
| 密码     | test              |
| 显示名称 | 随便填写          |
| 发信名称 | 随便填写          |

`服务器 - POP3`
| key    | value  |  
|------------|-----------------|
| 邮箱类型  | POP3                  |
| 账号      | test1@ibmtest.com     |
| 收件服务器| ibmtest.com  端口:110 |
| 发件服务器| ibmtest.com  端口:25  |

`服务器 - IMAP`
| key    | value  |  
|------------|-----------------|
| 邮箱类型  | IMAP                  |
| 账号      | test1@ibmtest.com     |
| 收件服务器| ibmtest.com  端口:143 |
| 发件服务器| ibmtest.com  端口:25  |

### 测试可以收发邮件
james使用数据库存储

>1、将jdbc包拷入此目录  
jdbc_jar下载  
https://mvnrepository.com/artifact/mysql/mysql-connector-java/8.0.27
--> Files --> jar
mysql-connector-java-8.0.27.jar

``` sh
[root@localhost ~]# ls /app/james-server-app-3.5.0/conf/lib/mysql-connector-java-8.0.27.jar
# 注 如果无法初始化建表，使用低版本的jdbc
mysql-connector-java-3.1.14.jar
```


>2、创建mysql数据库及用户
``` sql
mysql> grant all privileges ON james_db.* TO 'james_user'@'%' identified by 'James12!@' with grant option;
mysql> flush privileges;
mysql> create database james_db;
```


>3、修改conf/james-database.properties  
``` sh
注释掉 默认的DERBY存储，使用mysql：   
[root@localhost ~]# vim /app/james-server-app-3.5.0/conf/james-database.properties
database.driverClassName=com.mysql.jdbc.Driver
database.url=jdbc:mysql://192.168.2.111:3306/james_db?rewriteBatchedStatements=true
database.username=james_user
database.password=James12!@

#解决ssl方式方式
#database.url=jdbc:mysql://192.168.2.111:3306/james_db?rewriteBatchedStatements=true&useSSL=false

vendorAdapter.database=MYSQL

openjpa.streaming=false

mailrepositorystore.xml需要定义数据库
sqlResources.xml为创建数据的的sql语句
```


>4、更改邮件信息存储方式  
#数据库配置  
repositoryPath同时存在文件存储(file://)、数据库存储(db://)的，将默认的文件存储注释掉，并释放数据库存储；
对于只存在文件存储的，不进行释放。分别如下图所示：
``` sh
[root@localhost ~]# vim /app/james-server-app-3.5.0/conf/mailetcontainer.xml
<mailet match="All" class="ToRepository">
    <!--注释此行
    <repositoryPath>file://var/mail/address-error/</repositoryPath>
    -->
    <!-- An alternative database repository example follows. -->
    <!--释放此行-->
    <repositoryPath>db://maildb/deadletter/address-error</repositoryPath>
</mailet>
```


>5、使用james-cli命令配置邮件后缀域名和用户  
此时用户信息写入数据库内  
[root@localhost ~]# cd /app/james-server-app-3.5.0/bin/
./james-cli.sh -h 127.0.0.1 -p 9999 AddUser test1@ibmtest.com test
./james-cli.sh -h 127.0.0.1 -p 9999 AddUser test2@ibmtest.com test


## 优化--James内存使用
``` sh
[root@localhost ~]# vim /app/james-server-app-3.5.0/conf/wrapper.conf
# 改成此配置即可
# Initial Java Heap Size (in MB)
wrapper.java.initmemory=256
# Maximum Java Heap Size (in MB)
wrapper.java.maxmemory=1024



[root@localhost ~]# /app/james-server-app-3.5.0/bin/james start

[root@localhost ~]# netstat -tunlp | grep -E '4555|9999|110|25|119|143'
```


## 参考
>https://blog.csdn.net/hjnth/article/details/82931569


>报错:Unable to access mailbox解决方法
https://blog.csdn.net/Fanpei_moukoy/article/details/80202090  
或者更换jdbc的包试试

