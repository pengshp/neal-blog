---
title: Raspbian 安装最新版 docker
date: 2016-09-10T02:47:34+08:00
categories: [Docker]
tags: [Docker,RaspberryPi]
---

![docker](/images/logo-docker.jpg "Docker")
<!--more-->

### 一.问题描述
执行`docker pull`命令时经常不能`pull images`,显示如下错误
    
    Could not reach any registry endpoint.

Google后才知道`Raspbian`中默认安装的docker版本是`1.3.3`，而docker更新后一些性能不在支持`1.5`及以下的版本，所以才会出现这样的问题。为了能够正常使用，要更新到较新的版本。

### 二.安装配置
#### 1.先卸载旧版本的docker

```sh
$ sudo apt-get remove --purge docker docker-io
```

#### 2.使用官方脚本安装docker

```sh
$ curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

#### 3.为了使用户能够使用docker,把它加入用户组

```sh
$ sudo usermod -aG docker pi
```

这样在执行`docker`命令的时候就不需要加上`sudo`.
根据系统提示，重新启动系统或者重新登录，便可以正常的使用`docker`.

### 4.配置镜像加速器

```sh
$ vim /etc/docker/daemon.json
{
    "registry-mirrors": [
        "https://docker.mirrors.ustc.edu.cn/",
        "https://dockerhub.azk8s.cn"
     ],
     "insecure-registries": []
}
```

### 5.限制容器日志的大小

```shell
~$ vim /etc/docker/daemon.json
{
    "log-driver": "json-file",
    "log-opts": {
    	"max-size":"1m", 
    	"max-file":"2"
     }
}

# 重启Docker
~$ sudo systemctl restart docker
```

若不做配置，容器产生的日志会不断增大，可能到几GB，上面配置每个容器最多产生5MB的日志，最多保留2个日志文件。

#### 容器日志清理脚本

```sh
#!/bin/sh
# Clean Docker log.

echo "======== start clean docker containers logs ========"

logs=$(find /var/lib/docker/containers/ -name *-json.log)

for log in $logs
do
        echo "clean logs : $log"
        cat /dev/null > $log
done

echo "======== end clean docker containers logs ========"
```

执行此脚本，可清理容器产生的日志。

### 6.有时出了问题，可以卸载重新安装

```sh
$ sudo apt-get remove --auto-remove docker-engine
```

### 三.docker教程参考

[ Docker —— 从入门到实践](https://www.gitbook.com/book/yeasy/docker_practice/details)

**参考：**
1.<https://www.raspberrypi.org/blog/docker-comes-to-raspberry-pi/>
2.[Could not reach any registry endpoint](https://stackoverflow.com/questions/38286313/docker-on-raspberry-pi-2-could-not-reach-any-registry-endpoint)





