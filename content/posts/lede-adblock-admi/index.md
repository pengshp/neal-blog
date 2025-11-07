---
title: LEDE/OpenWrt路由器使用Adblock屏蔽小米广告
date: 2018-07-13T02:47:34+08:00
categories: [OpenWrt]
tags: [OpenWrt]
---
小米的生态链产品做的不错，但无奈小米手机上的广告太多，实在无法忍受，有网友推出了`小米净化APP`用于屏蔽小米的广告，本文介绍如何在`LEDE/OpenWrt`路由器使用`Adblock`插件屏蔽小米烦人的广告。
<!--more-->
## 准备
* 刷了`LEDE/OpenWrt`的路由器
* 小米手机
* 网络

### 1、远程登录到路由器后台
```sh
ssh root@192.168.31.1  #改为自己的IP
BusyBox v1.25.1 () built-in shell (ash)

     _________
    /        /\      _    ___ ___  ___
   /  LE    /  \    | |  | __|   \| __|
  /    DE  /    \   | |__| _|| |) | _|
 /________/  LE  \  |____|___|___/|___|         lede-project.org
 \        \   DE /
  \    LE  \    /  ----------------------------------------------
   \  DE    \  /    Reboot (17.01.4, r3560-79f57e422d)
    \________\/    ----------------------------------------------

root@HiWiFi:~#
```
### 2、安装中文支持包
```sh
root@HiWiFi:~# opkg update
root@HiWiFi:~# opkg install luci-i18n-base-zh-cn
```
### 3、安装Adblock插件
```sh
root@HiWiFi:~# opkg install adblock luci-app-adblock 
root@HiWiFi:~# opkg install luci-i18n-adblock-zh-cn
```
### 4、设置
在`WebUI`管理界面设置`Adblock`的黑名单，加入如下的域名
依次点击：服务--->Adblock--->高级--->编辑黑名单
```sh
a.stat.xiaomi.com
a.union.mi.com
abtest.mistat.xiaomi.com
adinfo.ra1.xlmc.sec.miui.com
adv.sec.miui.com
api.ad.xiaomi.com
api.ra2.xlmc.sec.miui.com
api.tuisong.baidu.com
api.tw06.xlmc.sec.miui.com
app01.nodes.gslb.mi-idc.com
app02.nodes.gslb.mi-idc.com
app03.nodes.gslb.mi-idc.com
applog.uc.cn
beha.ksmobile.com
bss.pandora.xiaomi.com
calopenupdate.comm.miui.com
cdn.ad.xiaomi.com
cm.p4p.cn.yahoo.com
cm066.getui.igexin.com
connect.rom.miui.com
data.mistat.xiaomi.com
e.ad.xiaomi.com
etl.xlmc.sandai.net
fcanr.tracking.miui.com
fclick.baidu.com
get.sogou.com
hm.xiaomi.com
hub5pn.wap.sandai.net
idx.m.hub.sandai.net
image.box.xiaomi.com
info.analysis.kp.sec.miui.com
info.sec.miui.com
logupdate.avlyun.sec.miui.com
m.bss.pandora.xiaomi.com
m.irs01.com
m.sjzhushou.com
master.wap.dphub.sandai.net
mdap.alipaylog.com
migc.g.mi.com
migcreport.g.mi.com
migrate.driveapi.micloud.xiaomi.net
mis.g.mi.com
mlog.search.xiaomi.net
new.api.ad.xiaomi.com
notice.game.xiaomi.com
nsclick.baidu.com
o2o.api.xiaomi.com
p.alimama.com
pdc.micloud.xiaomi.net
ppurifier.game.xiaomi.com
pre.api.tw06.xlmc.sandai.net
r.browser.miui.com
reader.browser.miui.com
report.adview.cn
resolver.gslb.mi-idc.com
resolver.msg.xiaomi.net
sa.tuisong.baidu.com
sa3.tuisong.baidu.com
sdk.open.phone.igexin.com
sdk.open.talk.gepush.com
sdk.open.talk.igexin.com
sdkconfig.ad.xiaomi.com
sec-cdn.static.xiaomi.net
sec.resource.xiaomi.net
security.browser.miui.com
sg.a.stat.mi.com
staging.admin.e.mi.com
test.ad.xiaomi.com
test.api.xlmc.sandai.net
test.e.ad.xiaomi.com
test.new.api.ad.xiaomi.com
tracking.miui.com
tw13b093.sandai.net
union.dbba.cn
update.avlyun.sec.miui.com
www.adview.cn
yun.rili.cn
zhwnlapi.etouch.cn
api.comm.miui.com
```
备注：以上域名引自小米净化APP

### 5、注意事项
Adblock的配置的`Download Utility`最好选择`uclient-fetch`,并且安装SSL库
```sh
root@HiWiFi:~# opkg install libustream-openssl
```
![ad](/images/adblocka.png "AdBlock")
拦截来源列表里推荐勾选上`reg_cn`，对国内的广告拦截效果较好。
### 6、重启小米路由器
```sh
root@HiWiFi:~# reboot
```
此时再打开小米的APP，如：`小米路由器`就发现启动时的广告没了，清爽的感觉又回来了！
