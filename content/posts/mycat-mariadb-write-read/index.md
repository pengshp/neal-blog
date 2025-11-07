---
title: MyCat实现MySQL数据的读写分离
date: 2021-08-04T02:47:34+08:00
categories: [MySQL]
tags: [Linux,MySQL]
---

![mycat](https://hugoblog-img-1251694304.cos.ap-guangzhou.myqcloud.com/blog/image-20210804225925453.png "MyCat")

MyCat是开源社区在阿里cobar基础上使用Java进行二次开发的开源数据库中间件,本文介绍使用 `MyCat` 实现读写分离.

<!--more-->



## MyCat简介

MyCat是开源社区在阿里cobar基础上使用Java进行二次开发的开源数据库中间件；是一个开源的分布式数据库系统，是一个实现了MySQL协议的的Server，前端用户可以把它看作是一个数据库代理，用MySQL客户端工具和命令行访问，而其后端可以用MySQL原生（Native）协议与多个MySQL服务器通信，也可以用JDBC协议与大多数主流数据库服务器通信，其核心功能是分表分库，即将一个大表水平分割为N个小表，存储在后端MySQL服务器里或者其他数据库里。

Mycat发展到目前的版本，已经不是一个单纯的MySQL代理了，它的后端可以支持MySQL、SQL Server、Oracle、DB2、PostgreSQL等主流数据库，也支持MongoDB这种新型NoSQL方式的存储，未来还会支持更多类型的存储。

对于DBA来说，可以这么理解Mycat： Mycat就是MySQL Server，而Mycat后面连接的MySQL Server，就好象是MySQL的存储引擎,如InnoDB，MyISAM等，因此，Mycat本身并不存储数据，数据是在后端的MySQL上存储的，因此数据可靠性以及事务等都是MySQL保证的；

对于架构师来说，可以这么理解Mycat：Mycat是一个`强大的数据库中间件`，不仅仅可以用作`读写分离、以及分表分库、容灾备份`，而且可以用于多租户应用开发、云平台基础设施、让你的架构具备很强的适应性和灵活性，借助于即将发布的Mycat智能优化模块，系统的数据访问瓶颈和热点一目了然，根据这些统计分析数据，你可以自动或手工调整后端存储，将不同的表映射到不同存储引擎上，而整个应用的代码一行也不用改变。

[Mycat1.6官网](http://www.mycat.org.cn/)

[Mycat1权威指南 · 语雀 (yuque.com)](https://www.yuque.com/ccazhw/tuacvk)

## MyCat作用

1. 读写分离

   ![mk](https://hugoblog-img-1251694304.cos.ap-guangzhou.myqcloud.com/blog/image-20210804225925453.png "MyCat-2")

2. 数据分片

    垂直拆分（分库）、水平拆分（分表）、垂直+水平拆分（分库分表）


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f61991a54ff0478bbe6825f414605291~tplv-k3u1fbpfcp-watermark.image "数据分片")

3. 多数据源整合

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d62be6fd910645d6b5fd0de43129f6e4~tplv-k3u1fbpfcp-watermark.image "数据整合")

## MyCat原理
Mycat 的原理中最重要的一个动词是“拦截”，它拦截了用户发送过来的 SQL 语句，首先对 SQL 语句做了一些特定的分析：如分片分析、路由分析、读写分离分析、缓存分析等，然后将此 SQL 发往后端的真实数据库，并将返回的结果做适当的处理，最终再返回给用户。

## MyCat安装
1. 安装java运行环境JDK

```sh
$ sudo wget http://dl.mycat.org.cn/jdk-8u20-linux-x64.tar.gz
$ sudo tar -zxf jdk-8u20-linux-x64.tar.gz
$ sudo mv jdk1.8.0_20 /usr/local/jdk

# 配置环境变量
~$ sudo vim /etc/profile.d/jdk.sh
JAVA_HOME=/usr/local/jdk
JRE_HOME=$JAVA_HOME/jre
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME JRE_HOME CLASSPATH PATH

# 验证
[xdl@CentOS] ~$ source /etc/profile
[xdl@CentOS] ~$ java -version
java version "1.8.0_20"
Java(TM) SE Runtime Environment (build 1.8.0_20-b26)
Java HotSpot(TM) 64-Bit Server VM (build 25.20-b23, mixed mode)
```

## MyCat安装

```sh
~$ sudo wget http://dl.mycat.org.cn/1.6.7.1/Mycat-server-1.6.7.1-release-20200209222254-linux.tar.gz
~$ tar -zxf Mycat-server-1.6.7.1-release-20200209222254-linux.tar.gz -C /usr/local
~$ ls /usr/local/mycat
bin  catlet  conf  lib  logs  version.txt

# 设置java路径
~$ sudo vim /usr/local/mycat/conf/wrapper.conf
wrapper.java.command=/usr/local/jdk/bin/java

wrapper.startup.timeout=300

# 若启动出错，调整虚拟内存的大小
wrapper.java.additional.9=-Xmx2G
wrapper.java.additional.10=-Xms512M

# 设置环境变量
~$ sudo vim /etc/profile.d/mycat.sh
MYCAT_HOME=/usr/local/mycat

~$ source /etc/profile
```

## MyCat读写分离架构

![mycat](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b83bc2375814d509b800ab7a74a6e14~tplv-k3u1fbpfcp-watermark.image "MyCat-3")

应用直接连接MyCat的逻辑表testdb,MyCat后端连接真实的MySQL数据库实例，并把写数据和读数据分开，减轻单个MySQL实例的压力。

## 配置读写分离
配置文件

①schema.xml：定义逻辑库，表、分片节点等内容

②rule.xml：定义分片规则

③server.xml：定义用户以及系统相关变量，如端口等

1、修改配置文件server.xml 修改mycat用户信息，与MySQL区分，如下：

```sh
~$ sudo vim /usr/local/mycat/conf/server.xml
...p110
<user name="root" defaultAccount="true">
        <property name="password">mycatpw</property>
        <property name="schemas">testdb</property>
        <property name="defaultSchema">testdb</property>
```
这里主要设置了MyCat的用户名，密码，逻辑库的name等属性

2、修改配置文件 schema.xml

删除`<schema>`标签间的表信息，`<dataNode>`标签只留一个，`<dataHost>`标签只留一个，`<writeHost> <readHost>`只留一对

```sh
~$ sudo vim /usr/local/mycat/conf/schema.xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

    <schema name="testdb" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1">

    </schema>
    <dataNode name="dn1" dataHost="host1" database="test" />
    <dataHost name="host1" maxCon="1000" minCon="10" balance="3"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        <!-- can have multi write hosts -->
        <writeHost host="hostM1" url="10.10.10.11:3306" user="root"
                   password="zxcv">
            <!-- can have multi read hosts -->
            <readHost host="hostS2" url="10.10.10.12:3306" user="root" password="zxcv" />
        </writeHost>
    </dataHost>
</mycat:schema>
```

这里设置数据节点，以及对应的后端真实的MySQL实例属性，写库和读库的属性

dataHost标签中有一个balance属性，用于设置读写分离

- balance="0", 不开启读写分离机制，所有读操作都发送到当前可用的 writeHost 上；
- balance="1"，全部的 readHost 与 stand by writeHost 参与 select 语句的负载均衡；
- balance="2"，所有读操作都随机的在 writeHost、readhost 上分发；
- balance="3"，所有读请求随机的分发到 readhost 执行，writerHost 不负担读压力

MariaDB数据库账号设置

```sh
MariaDB [(none)]> grant all privileges on *.* to root@'%' identified by 'zxcv';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
```

### 启动MyCat

```sh
# 前台启动
~$ sudo /usr/local/mycat/bin/mycat console

# 后台启动
~$ sudo /usr/local/mycat/bin/mycat start

# 查看日志
~$ sudo tail -f /usr/local/mycat/logs/wrapper.log
STATUS | wrapper  | 2021/08/04 16:49:51 | --> Wrapper Started as Daemon
STATUS | wrapper  | 2021/08/04 16:49:51 | Launching a JVM...
INFO   | jvm 1    | 2021/08/04 16:50:17 | Wrapper (Version 3.2.3) http://wrapper.tanukisoftware.org
INFO   | jvm 1    | 2021/08/04 16:50:17 |   Copyright 1999-2006 Tanuki Software, Inc.  All Rights Reserve
INFO   | jvm 1    | 2021/08/04 16:50:17 |
INFO   | jvm 1    | 2021/08/04 16:50:44 | MyCAT Server startup successfully. see logs in logs/mycat.log
```

### 测试MyCat

```sh
~$ mysql -h 10.10.10.13 -P 8066 -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.6.29-mycat-1.6.7.4-release-20200105164103 MyCat Server (OpenCloudDB)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]>
MySQL [(none)]> show databases;
+----------+
| DATABASE |
+----------+
| testdb   |
+----------+
1 row in set (0.00 sec)
```
视频教程参考：[MyCat性能最好的开源数据库中间件_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1mK4y1S7hS)
