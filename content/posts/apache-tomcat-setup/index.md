---
title: "Linux 环境下部署Apache Tomcat Server"
date: 2021-12-30T10:47:04+08:00
description: "Linux下Tomcat Server的部署"
tags: ["linux", "Tomcat"]
categories: ["linux"]
---

Tomcat 是 Apache 软件基金会（Apache Software Foundation）的 Jakarta 项目中的一个核心项目，由Apache、Sun 和其他一些公司及个人共同开发而成。由于有了Sun 的参与和支持，最新的 Servlet 和JSP 规范总是能在Tomcat 中得到体现，因为Tomcat 技术先进、性能稳定，而且免费，因而深受Java 爱好者的喜爱并得到了部分软件开发商的认可，成为目前比较流行的 Web 应用服务器。

<!--more-->

## Tomcat结构

Tomcat中最顶层的容器是Server，代表着整个服务器，一个Server可以包含至少一个Service，用于具体提供服务。
Service主要包含两个部分：Connector和Container。

Tomcat 的核心就是这两个组件，他们的作用如下：

- Connector用于处理连接相关的事情，并提供Socket与Request和Response相关的转化;
- Container用于封装和管理Servlet，以及具体处理Request请求；

## Tomcat服务器搭建

本次使用`JDK11`和`Tomcat10`版本进行部署.

### JDK环境安装

#### 下载安装

[Java Archive Downloads - Java SE 11 (oracle.com)](https://www.oracle.com/java/technologies/javase/jdk11-archive-downloads.html)

```bash
~$ wget https://download.java.net/java/ga/jdk11/openjdk-11_linux-x64_bin.tar.gz

~$ tar -zxf openjdk-11_linux-x64_bin.tar.gz
~$ sudo mv jdk-11 /usr/local/jdk
```

#### 配置JDK环境变量

```bash
~$ sudo vim /etc/profile.d/jdk.sh
JAVA_HOME=/usr/local/jdk
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME CLASSPATH PATH

# 验证
[xdl@CentOS] ~$ source /etc/profile
[xdl@CentOS] ~$ java -version
openjdk version "11" 2018-09-25
OpenJDK Runtime Environment 18.9 (build 11+28)
OpenJDK 64-Bit Server VM 18.9 (build 11+28, mixed mode)
```

### 安装Tomcat

[Apache Tomcat® - Apache Tomcat 10 Software Downloads](https://tomcat.apache.org/download-10.cgi)

安装

```bash
~$ tar -zxf apache-tomcat-10.0.14.tar.gz
~$ sudo mv apache-tomcat-10.0.14 /usr/local/tomcat
```

## Tomcat管理

### 启动Tomcat

```bash
~$ /usr/local/tomcat/bin/startup.sh
```

### 查看启动日志

```bash
~$ tail -f /usr/local/tomcat/logs/catalina.out
```

### 查看监听端口

```bash
~$ sudo ss -antpl |grep java
```

可以看到Tomcat监听了`8080,8005`

### 开放防火墙端口

```bash
~$ sudo firewall-cmd --add-port=8080/tcp --permanent
~$ sudo firewall-cmd --reload
```

浏览器访问 `http://ip:8080`便可以访问测试页面

### 修改配置文件

```bash
~$ sudo vim /usr/local/tomcat/conf/server.xml
```

### 停止服务

```bash
~$ /usr/local/tomcat/bin/shutdown.sh
```

JAVA应用的`war`包放在`/usr/local/tomcat/webapps/`里，并删除`ROOT`目录。改变server.xml文件中的server区域

```xml
<Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

        <Context path="" docBase="hjc" debug="0" privileged="true"/>  #加入此行
      </Host>
    </Engine>
  </Service>
</Server>
```

### 重启Tomcat

```bash
~$ /usr/local/tomcat/bin/shutdown.sh
~$ /usr/local/tomcat/bin/startup.sh
```

### 设置Tomcat的环境变量

```bash
$ sudo vim /etc/profile.d/tomcat.sh
#TOMCAT
export TOMCAT_HOME=/usr/local/tomcat
export PATH=$PATH:$TOMCAT_HOME/bin
```
