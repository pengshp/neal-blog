+++
title = 'Setup Fail2ban'
date = 2023-06-05T08:06:23+08:00
tags = ['linux']
categories = ['Linux']

+++



今天查看我的滴滴云服务器的时候，云监控里发现最近两天有两百多次的SSH暴力破解登录，查了下来源IP，都是韩国釜山，安徽什么的。真是防不胜防啊！于是决定研究下SSH的防暴力破解，我选择的程序是`Fail2ban`。下面介绍如何配置使用。
<!--more-->



### 1、下载安装
`Fail2ban`安装包在`epel`源里，如果没安装，需要装上。
```bash
$ sudo yum install -y epel-release
$ sudo yum install -y fail2ban fail2ban-systemd fail2ban-firewalld
```
### 2、修改配置文件
主配置文件`/etc/fail2ban/jail.conf`不建议修改，自行建一个配置文件,它会覆盖主配置文件的配置项，这样便于管理，升级安装包时也不会被覆盖。
```sh
$ sudo vim /etc/fail2ban/jail.d/ssh.local
[DEFAULT]
# 禁止一个IP24小时
bantime = 24h

# 10分钟内尝试登陆3次失败便加入屏蔽列表
findtime = 10m
maxretry = 3
# 把自己的IP加入这里，免得自己被ban
ignoreip = 127.0.0.1/8 ::1 x.x.x.x

[sshd]
enabled = true
mode    = normal
port    = ssh
logpath = %(sshd_log)s
backend = %(sshd_backend)s
```

### 3、启动Fail2ban服务
```sh
$ sudo systemctl start fail2ban.service
$ sudo systemctl enable fail2ban.service
```

### 4、查看`ssh`状态
```sh
$ fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     0
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned: 20
   |- Total banned:     20
   `- Banned IP list:   1.252.24.138 142.93.153.234 157.230.9.239 \
   159.65.145.175 159.65.148.178 159.65.151.151 165.227.39.62 \
   183.157.142.164 183.230.146.26 192.99.255.47 193.201.224.214 \
   195.231.4.214 198.98.56.196 198.98.62.146 206.189.132.42 \
   209.141.35.22 223.113.91.54 24.90.25.51 59.20.205.178 68.183.99.64
```

### 5、查看实时日志
```sh
$ sudo tail -f /var/log/fail2ban.log
```

### 6、最近一个启动`fail2ban`日志
```sh
journalctl -b -u fail2ban
```

### 7、查看登录失败的日志

```sh
$ sudo cat /var/log/secure |grep 'Failed password'
May 16 20:42:55 10-255-0-83 sshd[21804]: Failed password for root from 185.244.25.105 port 35412 ssh2
May 16 21:08:06 10-255-0-83 sshd[25255]: Failed password for root from 40.73.39.211 port 41718 ssh2
May 16 21:13:51 10-255-0-83 sshd[25935]: Failed password for root from 105.103.132.251 port 19628 ssh2
```

### 8、解锁IP

```sh
$ fail2ban-client set sshd unbanip IP
```
