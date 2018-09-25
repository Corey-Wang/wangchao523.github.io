---
title: mac使用pf做端口转发
date: 2018-09-25 09:55:08
tags: mac, proxy
---
## 背景
线上一个问题，但和cookie有关，需要用到域名，所以通过本地host来将域名指向127.0.0.1， 但由于https端口只能是443了，mac无法直接绑定，需要使用ROOT权限。
所以想到可以使用端口转发的技术，网上搜到了pfctl。

## 操作步骤
### 新建转发规则
在这个目录下`/etc/pf.anchors` 新建一个文件，如443（名称自定定义就行），内容如下：
```
rdr pass on lo0 inet proto tcp from any to any port 443 -> 127.0.0.1 port 8443
```
lo0是网卡，可以通过`ifconfig`查看127.0.0.1对应的网卡名

### 将配置添加到pf.conf中
`/etc/pf.conf`
```
scrub-anchor "com.apple/*"
nat-anchor "com.apple/*"
rdr-anchor "com.apple/*"
rdr-anchor "forwarding"
dummynet-anchor "com.apple/*"
anchor "com.apple/*"
load anchor "com.apple" from "/etc/pf.anchors/com.apple"
load anchor "forwarding" from "/etc/pf.anchors/443"
```
其中4行，8行的`forwarding`是新增的，`com.apple`是系统自带的。注意这里一定要保证顺序

### 启动pf
```
sudo pfctl -ef /etc/pf.conf
```

### 关闭pf
```
sudo pfctl -d
```

### 强制重启
```
sudo pfctl -E
```

### 更多介绍
```
# 自己来看吧
man pfctl
```