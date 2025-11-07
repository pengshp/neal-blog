---
title: Ceph 分布式系统搭建
date: 2020-08-08T02:47:34+08:00
categories: [Linux]
tags: [Linux]
---

![ceph-blog](https://pengshp.coding.net/p/images/d/images/git/raw/master/ceph-blog.jpg "Ceph")

上一篇介绍了Ceph分布式文件系统，下面介绍Ceph分布式文件系统的安装配置和使用

<!--more-->

# 一、环境准备

## 1. 环境说明

- 操作系统：CentOS Linux 7
- node[1-3]添加一块5G的磁盘
- Ceph 版本：nautilus

| 主机名 | IP地址      | 用途                     |
| ------ | ----------- | ------------------------ |
| admin  | 10.10.10.60 | 管理节点 安装ceph-deploy |
| node1  | 10.10.10.61 | mon mgr osd              |
| node2  | 10.10.10.62 | osd                      |
| node3  | 10.10.10.63 | osd                      |
| client | 10.10.10.64 | client测试               |

## 2.配置相应的环境

> 以下操作在所有节点配置

2.1 新建用户

```shell
## all
[xdl@admin] ~$ sudo useradd -G wheel ceph                        
[xdl@admin] ~$ echo 'ceph' |sudo passwd --stdin ceph
```

2.2 修改hosts

```shell
[ceph@admin ~]$ sudo vim /etc/hosts
# Ceph
10.10.10.60 admin
10.10.10.61 node1
10.10.10.62 node2
10.10.10.63 node3
10.10.10.64 client
```

2.3 配置时间同步

```shell
[root@admin] ~$ vim /etc/chrony.conf
server ntp.aliyun.com iburst

[root@admin] ~$ systemctl restart chronyd.service
[root@admin] ~$ chronyc sources -v
```

2.4 配置admin节点与各node节点的ssh免密登录

```shell
## admin
[xdl@admin] ~$ su - ceph
[ceph@admin ~]$ ssh-keygen -t rsa -b 1024
[ceph@admin ~]$ ssh-copy-id ceph@node1
[ceph@admin ~]$ ssh-copy-id ceph@node2
[ceph@admin ~]$ ssh-copy-id ceph@node3
```

2.5 配置ceph 软件源

```shell
[root@node1] ~$ vim /etc/yum.repos.d/ceph.repo
[ceph]
name=Ceph packages for $basearch
baseurl=https://mirrors.cloud.tencent.com/ceph/rpm-nautilus/el7/$basearch
enabled=1
priority=2
gpgcheck=1
gpgkey=https://mirrors.cloud.tencent.com/ceph/keys/release.asc

[ceph-noarch]
name=Ceph noarch packages
baseurl=https://mirrors.cloud.tencent.com/ceph/rpm-nautilus/el7/noarch
enabled=1
priority=2
gpgcheck=1
gpgkey=https://mirrors.cloud.tencent.com/ceph/keys/release.asc

[ceph-source]
name=Ceph source packages
baseurl=https://mirrors.cloud.tencent.com/ceph/rpm-nautilus/el7/SRPMS
enabled=0
priority=2
gpgcheck=1
gpgkey=https://mirrors.cloud.tencent.com/ceph/keys/release.as

[root@node1] ~$ yum makecache
```

# 二、安装配置

## admin管理节点配置

1. 安装ceph-deploy

```shell
[root@admin] ~$ yum search ceph
[root@admin] ~$ yum install -y ceph-deploy python-setuptools
```

2. 创建节点node1

```shell
[root@admin] ~$ su - ceph
[ceph@admin ~]$ mkdir cluster
[ceph@admin ~]$ cd cluster/
[ceph@admin cluster]$ ceph-deploy new node1
[ceph@admin cluster]$ ls
ceph.conf  ceph-deploy-ceph.log  ceph.mon.keyring
```

## node1节点配置

1. 安装软件包

```shell
[root@node1] ~$ yum makecache
[root@node1] ~$ yum install -y ceph ceph-radosgw
[root@node1] ~$ ceph --version
ceph version 14.2.16 (762032d6f509d5e7ee7dc008d80fe9c87086603c) nautilus (stable)
```

2. 配置mon 和 mgr

```shell
# 初始化mon; admin节点以ceph用户执行
[ceph@admin cluster]$ ceph-deploy mon create-initial

# 赋予各个节点使用命令免密码权限
[ceph@admin cluster]$ ceph-deploy admin node1 node2 node3


# 安装ceph-mgr,只有luminous源才有，为dashboard做准备
[ceph@admin cluster]$ ceph-deploy mgr create node1
```

3. 添加osd

各个节点提供的存储空间不能太小，最好5G以上, /dev/sdb 是为osd 准备的空闲磁盘（无需格式化）

```shell
[ceph@admin cluster]$ ceph-deploy osd create --data /dev/sdb node1
[ceph@admin cluster]$ ceph-deploy osd create --data /dev/sdb node2
[ceph@admin cluster]$ ceph-deploy osd create --data /dev/sdb node3

# 查看
[ceph@admin cluster]$ ssh node1 lsblk -f
```

4. 查看ceph 的状态

```shell
[ceph@admin cluster]$ ssh node1 sudo ceph -s
```

![image-20210118181739086](https://pengshp.coding.net/p/images/d/images/git/raw/master/image-20210118181739086.png)

## Dashboard配置

1. 创建node1管理密钥

```shell
[ceph@node1 ~]$ sudo ceph auth get-or-create mgr.node1 mon 'allow profile mgr' osd 'allow *' mds 'allow *'
[mgr.node1]
	key = AQB2RAVgLC29BhAABkyYcjDyrjShJ1C9N2hlaQ==
```

2. 开启ceph-mgr 管理域

```shell
[ceph@node1 ~]$ sudo ceph-mgr -i node1
```

3. 查看ceph 的状态，确认mgr 的状态为 active

```shell
[ceph@node1 ~]$ sudo ceph status
```

![image-20210118182037802](https://pengshp.coding.net/p/images/d/images/git/raw/master/image-20210118182037802.png)

4. 打开 dashboard 模块

```shell
[ceph@node1 ~]$ sudo ceph mgr module enable dashboard
```

5. 绑定开启dashboard 模块的 ceph-mgr 节点的IP地址

```shell
[ceph@node1 ~]$ sudo ceph config-key set mgr/dashboard/node1/server_addr 10.10.10.61
set mgr/dashboard/node1/server_addr
```

6. 查看监听端口

```shell
[ceph@node1 ~]$ sudo netstat -antp |grep 7000
tcp        0      0 10.10.10.61:7000        0.0.0.0:*      LISTEN      12994/ceph-mgr
```

浏览器访问： http://10.10.10.61:7000 

## 客户端的使用

客户端需要更新内核版本到4.x以上

```shell
[xdl@client] ~$ uname -r
5.10.8-1.el7.elrepo.x86_64
```

1. 安装软件

```shell
[root@client] ~$ yum makecache

[root@client] ~$ yum install -y python-setuptools

[root@client] ~$ su - ceph
[ceph@client ~]$ sudo yum install -y ceph ceph-radosgw

[ceph@client ~]$ ceph --version
ceph version 14.2.16 (762032d6f509d5e7ee7dc008d80fe9c87086603c) nautilus (stable)
```

2. 在admin节点赋予client使用命令免权限

```shell
[ceph@admin cluster]$ ceph-deploy admin client
```

3.  修改client下该文件呢的读权限

```shell
[ceph@client ~]$ sudo chmod +r /etc/ceph/ceph.client.admin.keyring
```

4. 修改client 的ceph 配置文件，这一步是解决映射镜像时出错问题

```shell
[ceph@client ~]$ sudo vim /etc/ceph/ceph.conf
[global]
rbd_default_features = 1
```

5. client 节点创建块设备镜像，单位为MB

```shell
[ceph@client ~]$ rbd create foo --size 4096
```

6. client 节点映射镜像到主机

```shell
[ceph@client ~]$ sudo rbd map foo --name client.admin
/dev/rbd0
```

7. client 节点格式化块设备

```shell
[ceph@client ~]$ sudo mkfs -t ext4 -m 0 /dev/rbd/rbd/foo
```

8. client 节点挂载块设备

```shell
[ceph@client ~]$ sudo mkdir /mnt/ceph
[ceph@client ~]$ sudo mount /dev/rbd/rbd/foo /mnt/ceph/
[ceph@client ~]$ df -lhT
文件系统       类型      容量  已用  可用 已用% 挂载点
/dev/rbd0      ext4      3.9G   16M  3.8G    1% /mnt/ceph
```

客户端重启后，设备需要重新做映射（6），不然会卡死

9. 测试

```shell
[ceph@client ~]$ cd /mnt/ceph/
[ceph@client ceph]$ sudo touch test.txt
[ceph@client ceph]$ ls
lost+found  test.txt
```

