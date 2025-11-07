---
title: CentOS7 配置MariaDB数据库
date: 2018-11-02T02:47:34+08:00
categories: [CentOS]
tags: [CentOS, MariaDB]
---

自从`MySQL`数据库被oracle公司收购不在开源后，MySQL的作者就重新开发了一个完全兼容MySQL的数据库软件取名叫MariaDB.在MySQL的基础上加入了一些新的特性，很快得到了开源社区的支持。新版的CentOS7也使用MariaDB替代了MySQL，本文介绍在CentOS7上安装完MySQL后的一些基本配置。

<!--more-->

#### 安装MariaDB数据库

```sh
$ yum update
$ yum install -y mariadb-server
```

#### 基本概念

- mysqld ——服务器程序，守护进程
- mysql ——客户端
- 3306-----监听端口
  **服务器管理工具：**
  1、mysqlaccess 用于创建账户和设置权限
  2、mysqladmin 命令行的数据库管理工具，交互式地查询服务器的状态和使用量，以及关闭服务；
  3、mysqlshow 即可显示各数据库和各表的信息，又可查看服务器状态；
  4、mysqldump 导出dump文件

#### 修改MariaDB的配置文件

```sh
$ sudo vim /etc/my.cnf.d/server.cnf
[mysqld]
# 让MariaDB支持中文编码
character-set-server=utf8
collation-server=utf8_general_ci
```

```sh
$ sudo vim /etc/my.cnf.d/mysql-clients.cnf
[mysql]
default-character-set=utf8
```

更改编码为utf-8，加入上面的配置后便可以在数据库表中插入中文的数据。

#### 启动MariaDB数据库服务

```sh
sudo systemctl start mariadb.service
sudo systemctl enable mariadb.service
```

#### 初始化数据库

    mysql_secure_installation

#### 修改root用户密码

```sh
$ mysqladmin -u root -p flush-privileges password “xxxxx”
```

#### 创建新的用户

使用`MariaDB`时不建议使用root账户进行操作，建议新建一个用户进行操作，可以保证数据库的安全性。

```sh
mysql -u root -p -e "GRANT ALL ON *.* TO 'vicent'@'%';"
```

上面新建了一个叫vicent的新账户，设置对所有的数据库都有权限，最后的`%`表示通过`vicent`这个账户可以从任何地方连接到这个`MariaDB`服务器。

#### 查看用户信息

```sh
mysql -u root -p -e "SELECT User,Host FROM mysql.user;”
```

#### 删除用户

```sql
> drop user 用户名@localhost;
```

#### 修改用户密码

```sql
> set password for vicent@‘%’ = password(‘123’);
```

#### 远程连接到MariaDB数据库

使用mycli连接数据库，`mycli`是`Python`实现的`MySQL`客户端，支持代码高亮和智能提示，用于如下命令安装`mycli`

```sh
pip3 install mycli
```

在另一台电脑上连接到`MariaDB`数据库：

```sh
mycli -h 192.168.22.156 -u vicent
```

学习`MariaDB`数据库，建议先看看《SQL基础教程》这本书学习`SQL`语句。本书是一本非常适合初学者入门`SQL`语句的书。
