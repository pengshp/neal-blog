---
title: MooseFS网络分布式文件系统指南
date: 2021-01-20T02:47:34+08:00
categories: [Linux]
tags: [Linux]
---



![moosefs-pro-architecture](https://pengshp.coding.net/p/images/d/images/git/raw/master/moosefs-pro-architecture.png "MooseFS")

MooseFS是一个具备冗余容错功能、高可用、高性能、可扩展的网络分布式文件系统，它将数据分别存放在多个物理服务器或一个服务器的多块磁盘，确保一份数据有多个备份副本，然而对于访问MFS的客户端或者用户来说，整个网络分布式文件系统集群看起来就像一个虚拟磁盘一样。MooseFS分为普通版和Pro版本，Pro版本需要授权。

<!--more-->

# MooseFS 特性

a.高可靠性：每一份数据可以设置多个备份（多分数据），并可以存储在不同的主机上

b.高可扩展性：可以很轻松的通过增加主机的磁盘容量或增加主机数量来动态扩展整个文件系统的存储量

c.高可容错性：我们可以通过对mfs进行系统设置，实现当数据文件被删除后的一段时间内，依旧存放于主机的回收站中，以备误删除恢复数据

d.高数据一致性：即使文件被写入、访问时，我们依然可以轻松完成对文件的一致性快照；



# MooseFS 应用场景

- 大规模高并发的线上数据存储及访问（小文件，大文件都适合）

- 大规模的数据处理，如日志分析，小文件强调性能不用HDFS

# MooseFS 组成

- 管理服务器 (master server) 简称 `master` ：

  这个组件的角色是管理整个mfs文件系统的主服务器，除了分发用户请求外，还用来存储整个文件系统中每个数据文件的metadata信息，metadate（元数据）信息包括文件（也可以是目录，socket，管道，块设备等）的大小，属性，文件的位置路径等。

- 云数据备份服务器 (Metadata backup servers) 简称 `metalogger`：

  这个组件的作用是备份管理服务器master的变化的metadata信息日志文件，文件类型为changelog_ml.*.mfs。以便于在管理服务器出问题时，可以经过简单的操作即可让新的主服务器进行工作。

- 数据存储服务器组（chunk servers）简称data：

  这个组件就是真正存放数据文件实体的服务器了，这个角色可以有多台不同的物理服务器或不同的磁盘及分区来充当，当配置数据的副本多于一份时，据写入到一个数据服务器后，会根据算法在其他数据服务器上进行同步备份。

- 客户机服务器组（client servers）简称 `client`：

  这个组件就是挂载并使用mfs文件系统的客户端，当读写文件时，客户端首先会连接主管理服务器获取数据的metadata信息，然后根据得到的metadata信息，访问数据服务器读取或写入文件实体，mfs客户端通过fuse mechanism实现挂载mfs文件系统的，因此，只有系统支持fuse，就可以作为客户端访问mfs整个文件系统。

# MooseFS 原理

1. Master记录着管理信息，比如：文件路径|大小|存储的位置(ip,port,chunkid)|份数|时间等，元数据信息存在于内存中，会定期写入metadata.mfs.back文件中，定期同步到metalogger，操作实时写入changelog.*.mfs，实时同步到metalogger中。master启动将metadata.mfs载入内存，重命名为metadata.mfs.back文件。

2. 文件以chunk大小存储，每chunk最大为64M，小于64M的，该chunk的大小即为该文件大小（验证实际chunk文件略大于实际文件），超过64M的文件将被切分，以每一份（chunk）的大小不超过64M为原则；块的生成遵循规则：目录循环写入(00-FF 256个目录循环，step为2)、chunk文件递增生成、大文件切分目录连续。

3. Chunkserver上的剩余存储空间要大于1GB（Reference Guide有提到），新的数据才会被允许写入，否则，你会看到No space left on device的提示

4. 文件可以有多份copy，当goal为1时，文件会被随机存到一台chunkserver上，当goal的数大于1时，copy会由master调度保存到不同的chunkserver上，goal的大小不要超过chunkserver的数量，否则多出的copy，不会有chunkserver去存。



# MooseFS 集群搭建

## 实验架构图

![blog-mfs2](https://pengshp.coding.net/p/images/d/images/git/raw/master/blog-mfs2.png)

## 环境准备

- 操作系统：CentOS7
- MooseFS版本：3.0.115
- chunk servers 加入 5G 的磁盘

> 以下步骤在所有节点操作

### 1. 配置MooseFS软件源

```shell
# 导入验证密钥
curl "https://ppa.moosefs.com/RPM-GPG-KEY-MooseFS" > /etc/pki/rpm-gpg/RPM-GPG-KEY-MooseFS

# 添加yum 源
# For CentOS8
$ curl "http://ppa.moosefs.com/MooseFS-3-el8.repo" > /etc/yum.repos.d/MooseFS.repo

# For CentOS7
$ curl "http://ppa.moosefs.com/MooseFS-3-el7.repo" > /etc/yum.repos.d/MooseFS.repo

# For CentOS6
$ curl "http://ppa.moosefs.com/MooseFS-3-el6.repo" > /etc/yum.repos.d/MooseFS.repo
```

### 2. 配置解析

```shell
# 所有节点
[root@CentOS] ~$ echo '10.10.10.11 mfsmaster' >> /etc/hosts
```

## 搭建步骤

### Master Server

```shell
$ yum install moosefs-master moosefs-cgi moosefs-cgiserv moosefs-cli

[root@CentOS] ~$ vim /etc/mfs/mfsexports.cfg
10.10.10.0/24   /   rw,alldirs,maproot=0
# ·  表示MFS文件系统
# rw,alldirs,maproot=0    表示客户端拥有的权限
# rw  读写方式分享
# alldirs  允许挂载任何指定的子目录
# maproot   映射为root用户还是指定的用户，为0则不映射
# password  指定客户端密码

# 修改master server 配置文件
[root@CentOS] ~$ vim /etc/mfs/mfsmaster.cfg
WORKING_USER = mfs
WORKING_GROUP = mfs
SYSLOG_IDENT = mfsmaster
DATA_PATH = /var/lib/mfs
EXPORTS_FILENAME = /etc/mfs/mfsexports.cfg

# 启动服务
[root@CentOS] ~$ systemctl enable --now moosefs-master.service

# 配置MooseFS CGI Server
# MooseFS CGI 监控接口用于观察和分析 MooseFS 当前的状态
[root@CentOS] ~$ systemctl enable --now moosefs-cgiserv.service
[root@CentOS] ~$ netstat -antpl |grep 9425
tcp        0      0 0.0.0.0:9425            0.0.0.0:*               LISTEN      2703/python2

# 浏览器访问 
firefox http://10.10.10.11:9425
```

![img](https://pengshp.coding.net/p/images/d/images/git/raw/master/clipboard.png)

### Metadata backup servers (Metaloggers)

元数据备份节点配置

```shell
$ yum install moosefs-metalogger

[root@CentOS] ~$ vim /etc/mfs/mfsmetalogger.cfg
WORKING_USER = mfs
WORKING_GROUP = mfs
SYSLOG_IDENT = mfsmetalogger
DATA_PATH = /var/lib/mfs
BIND_HOST = 10.10.10.12
MASTER_HOST = mfsmaster
MASTER_PORT = 9419

# 启动服务
[root@CentOS] ~$ systemctl enable --now moosefs-metalogger.service
[root@CentOS] ~$ netstat -antpl |grep mfs
tcp        0      0 10.10.10.12:42183       10.10.10.11:9419        ESTABLISHED 2384/mfsmetalogger
```

### Chunkservers配置

① 磁盘设置

每台 chunkserver 上添加一块5G的磁盘

```shell
# 分区
[root@CentOS] ~$ parted --align optimal /dev/sdb
GNU Parted 3.1
使用 /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel gpt
(parted) mkpart mfschunks1 0% 100%
(parted) q
信息: You may need to update /etc/fstab.

# 格式化
[root@CentOS] ~$ mkfs -t xfs /dev/sdb1

# 挂载
[root@CentOS] ~$ mkdir /mnt/mfschunks1
[root@CentOS] ~$
[root@CentOS] ~$ mount /mnt/mfschunks1
[root@CentOS] ~$ chown mfs.mfs /mnt/mfschunks1
[root@CentOS] ~$ chmod 770 /mnt/mfschunks1

# 设置开机自动挂载
[root@CentOS] ~$ vim /etc/fstab
/dev/sdb1                                 /mnt/mfschunks1         xfs     defaults        0 0
```

② chunk 服务设置

```shell
# 安装软件包
$ yum install moosefs-chunkserver

# 修改配置文件
[root@CentOS] ~$ vim /etc/mfs/mfschunkserver.cfg
WORKING_USER = mfs
WORKING_GROUP = mfs
SYSLOG_IDENT = mfschunkserver
DATA_PATH = /var/lib/mfs
HDD_CONF_FILENAME = /etc/mfs/mfshdd.cfg
LABELS = chunkserver1
BIND_HOST = 10.10.10.13
MASTER_HOST = mfsmaster
MASTER_PORT = 9420

# 修改挂载的磁盘配置
[root@CentOS] ~$ vim /etc/mfs/mfshdd.cfg
/mnt/mfschunks1

# 启动服务，不能手动挂载 mount -a
[root@CentOS] ~$ systemctl enable --now moosefs-chunkserver.service

[root@CentOS] ~$ netstat -antpl |grep mfs
tcp     0      0 0.0.0.0:9422         0.0.0.0:*   LISTEN      2070/mfschunkserver
tcp     0      0 10.10.10.13:38923    10.10.10.11:9420    ESTABLISHED 2070/mfschunkserver
```

![mfs-1](https://pengshp.coding.net/p/images/d/images/git/raw/master/mfs-1.png)

### Client客户端设置

```shell
$ yum install fuse libfuse2
$ yum install moosefs-client

# 挂载MFS文件系统
[root@dev ~]# mkdir -p /mnt/mfs
[root@dev ~]# mfsmount /mnt/mfs/ -H mfsmaster
mfsmaster accepted connection with parameters: read-write,restricted_ip,admin ; root mapped to root:root

# 查看挂载的情况
[root@dev ~]# df -h |grep mfs
mfsmaster:9421   10G  577M  9.5G    6% /mnt/mfs
```

![mfs-2](https://pengshp.coding.net/p/images/d/images/git/raw/master/mfs-2.png)

### 测试

在客户端新建目录folder1 ，设置目录有一份拷贝（goal=1）,新建目录folder2 ，设置目录有一份拷贝（goal=2）

```shell
[root@dev ~]# mkdir -p /mnt/mfs/folder1
[root@dev ~]# mkdir -p /mnt/mfs/folder2
[root@dev ~]# mfssetgoal -r 1 /mnt/mfs/folder1
/mnt/mfs/folder1:
 inodes with goal changed:                       1
 inodes with goal not changed:                   0
 inodes with permission denied:                  0
[root@dev ~]# mfssetgoal -r 2 /mnt/mfs/folder2
/mnt/mfs/folder2:
 inodes with goal changed:                       0
 inodes with goal not changed:                   1
 inodes with permission denied:                  0
```

注意：副本的数量不能超过chunk server的数量。



新建文件测试，查看文件的副本信息

```shell
[root@dev ~]# echo 'Hello MFS!' > test.txt
[root@dev ~]# cp test.txt /mnt/mfs/folder1
[root@dev ~]# cp test.txt /mnt/mfs/folder2

[root@dev ~]# mfscheckfile /mnt/mfs/folder1/test.txt
/mnt/mfs/folder1/test.txt:
 chunks with 1 copy:              1  # <---- one copy
[root@dev ~]# mfscheckfile /mnt/mfs/folder2/test.txt
/mnt/mfs/folder2/test.txt:
 chunks with 2 copies:            1  # <---- two copies
```

![mfs-3](https://pengshp.coding.net/p/images/d/images/git/raw/master/mfs-3.png)

### 安全的停止MooseFS集群

1. 客户端卸载所有的挂载点 

```shell
[root@dev ~] $ lsof -n |grep mfsmount
[root@dev ~] $ umount /mnt/mfs
```

2. 停止 Chunkserver 服务

```shell
[root@CentOS] ~$ systemctl stop moosefs-chunkserver.service
```

3. 停止 Master Server 服务

```shell
[root@CentOS] ~$ systemctl stop moosefs-master.service
```

4. 停止 Metalogger 服务

```shell
[root@CentOS] ~$ systemctl stop moosefs-metalogger.service
```



参考：

[MooseFS官网](https://moosefs.com/)
