---
title: K8S 集群搭建指南
date: 2021-07-11T02:47:34+08:00
categories: [K8S]
tags: [K8S]
lightgallery: true
---

![Kubernetes_New](https://hugoblog-img-1251694304.cos.ap-guangzhou.myqcloud.com/blog/Kubernetes_New.png "k8s")

Kubernetes 是一个自动化的容器编排平台，它负责应用的部署、应用的弹性以及应用的管理，本文介绍使用`kubeadm`搭建 K8S 集群。

<!--more-->

## 环境准备

### 主机配置

| 主机名 | IP地址      | 操作系统  | 配置                  |
| ------ | ----------- | --------- | --------------------- |
| Master | 10.10.10.60 | CentOS7.9 | 2CPU  2G内存  20G硬盘 |
| Node1  | 10.10.10.61 | CentOS7.9 | 2CPU  2G内存  20G硬盘 |
| Node2  | 10.10.10.62 | CentOS7.9 | 2CPU  2G内存  20G硬盘 |

自行安装`Docker`

以下环境配置需在三台主机都要执行

1. 关闭selinux

```sh
setenforce 0
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
```

2. 关闭swap分区或禁用swap文件

```sh
$ swapoff -a
$ sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久
```

3. 关闭防火墙

```sh
[root@master ~]# systemctl stop firewalld
[root@master ~]# systemctl disable firewalld
```

4. 修改内核参数

```sh
$ vim /etc/sysctl.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
$ sysctl -p
```

5. 启用内核模块

```sh
$ vim /etc/sysconfig/modules/ipvs.modules
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
```

6. 配置hosts

```sh
$ vim /etc/hosts
10.10.10.60 master
10.10.10.61 node1
10.10.10.62 node2
```

7. 配置时间同步

```sh
# 时间同步
[xdl@k8s-master] ~$ sudo vim /etc/chrony.conf
server ntp.aliyun.com iburst

[xdl@k8s-master] ~$ sudo systemctl restart chronyd.service
```

重启系统

### 安装K8S

1. 配置Kubernets 的yum源

```sh
$ vim /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
	http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```

2. 安装kubelet、kubeadm、kubectl[v1.21.2]

```sh
$ yum install -y kubelet kubeadm kubectl

# 指定版本
$ yum install -y kubelet-1.20.0 kubeadm-1.20.0 kubectl-1.20.0
```

3. 启动kubelet服务

```sh
$ sudo systemctl enable kubelet
$ sudo systemctl start kubelet
```

### 设置Docker

1. 修改Docker配置文件

```sh
[xdl@k8s-master] ~$ vim /etc/docker/daemon.json
{
	"registry-mirrors": [
        "https://xxxxxxx.mirror.aliyuncs.com"
    ],
    "insecure-registries": [],
  	"exec-opts": ["native.cgroupdriver=systemd"]
}

[xdl@k8s-master] ~$ sudo systemctl restart docker.service
```

## kubernetes集群初始化

Master节点创建集群（该操作只在master主机执行）

```sh
$ sudo kubeadm init \
--kubernetes-version=v1.21.2 \
--image-repository registry.aliyuncs.com/google_containers \
--pod-network-cidr=10.244.0.0/16 \
--service-cidr=10.96.0.0/12 \
--apiserver-advertise-address=10.10.10.60
```

由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址。

> 如果失败，使用`kubeadm reset` 重置

```sh
[xdl@k8s-master] ~$ mkdir -p $HOME/.kube
[xdl@k8s-master] ~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[xdl@k8s-master] ~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

node节点加入集群

```sh
# node1
[xdl@k8s-node1] ~$ sudo kubeadm join 10.10.10.60:6443 --token tzbczr.r1lmokzaxfnirp73 \
        --discovery-token-ca-cert-hash sha256:b9e7a95e7ac84f97c2486ff850b9891c61c32d35d57a8248401f62b97660ea2e

# node2
[xdl@k8s-node2] ~$ sudo kubeadm join 10.10.10.60:6443 --token tzbczr.r1lmokzaxfnirp73 \
        --discovery-token-ca-cert-hash sha256:b9e7a95e7ac84f97c2486ff850b9891c61c32d35d57a8248401f62b97660ea2e
```

master节点查看node节点信息

```sh
[xdl@k8s-master] ~$ kubectl get nodes
NAME         STATUS   ROLES                  AGE   VERSION
k8s-master   Ready    control-plane,master   61m   v1.21.2
k8s-node1    Ready    <none>                 47m   v1.21.2
k8s-node2    Ready    <none>                 46m   v1.21.2
```

### kubernetes安装CNI网络插件

kubernetes支持多种网络插件，如：flannel、calico、canal等，任选一种使用即可，本实验选择flannel

只在master节点安装flannel插件即可，该插件使用的是DaemonSet控制器，该控制器会在每个节点上都运行

```sh
#获取flannel配置文件
[root@master ~]# wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

#执行文件启动flannel
[root@master ~]# kubectl apply -f kube-flannel.yml
```

等待几分钟，查看状态

```sh
[xdl@k8s-master] ~$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
kube-system   coredns-558bd4d5db-kzvhc             1/1     Running   0          63m
kube-system   coredns-558bd4d5db-p7x9t             1/1     Running   0          63m
kube-system   etcd-k8s-master                      1/1     Running   0          63m
kube-system   kube-apiserver-k8s-master            1/1     Running   0          64m
kube-system   kube-controller-manager-k8s-master   1/1     Running   0          64m
kube-system   kube-flannel-ds-pfll6                1/1     Running   0          49m
kube-system   kube-flannel-ds-xfvsn                1/1     Running   0          50m
kube-system   kube-flannel-ds-xvsst                1/1     Running   0          52m
kube-system   kube-proxy-9zjlz                     1/1     Running   0          50m
kube-system   kube-proxy-nj8lm                     1/1     Running   0          49m
kube-system   kube-proxy-sqfkx                     1/1     Running   0          63m
kube-system   kube-scheduler-k8s-master            1/1     Running   0          63m
```

全部显示`running`则说明搭建成功了。

### 测试Kubernets 集群

在Kubernetes集群中创建一个pod，验证是否正常运行：

```sh
[xdl@k8s-master] ~$ kubectl create deployment nginx --image=nginx
deployment.apps/nginx created

[xdl@k8s-master] ~$ kubectl expose deployment nginx --port=80 --type=NodePort
service/nginx exposed

[xdl@k8s-master] ~$ kubectl get pods,svc
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-6799fc88d8-5982c   1/1     Running   0          73s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        14h
service/nginx        NodePort    10.104.158.32   <none>        80:32389/TCP   20s

# 测试
[xdl@k8s-master] ~$ curl -I node1:32389
HTTP/1.1 200 OK
Server: nginx/1.21.1
```



#### 参考链接：

1. [K8S安装过程笔记 | 鸿则的博客 (hungtcs.top)](http://blog.hungtcs.top/2019/11/27/23-K8S安装过程笔记/)
2. [Kubernetes/K8S 集群环境搭建_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1oJ411d7Tv)
3. [Kubernetes](https://kubernetes.io/zh/)
