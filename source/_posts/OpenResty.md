---
title: OpenResty
date: 2017-04-25 18:20:29
categories: 技术
tags: OpenResty
---
### 一句话介绍
> OpenResty = Nginx + LuaJit + more lib of Lua
> [openresty官网](http://openresty.org/cn/)
> 推荐学习教程：[OpenResty 最佳实践 By 极客学院](http://wiki.jikexueyuan.com/project/openresty-best-practice/)

### 安装OpenResty
> *没有比官网的文档更好的了*[OpenResty 安装](http://openresty.org/cn/installation.html)

### Nginx Lua模块化执行顺序
```
set_by_lua #流程分之处理判断变量初始化
rewrite_by_lua #转发、重定向、缓存等功能(例如特定请求代理到外网)
access_by_lua #IP 准入、接口权限等情况集中处理(例如配合 iptable 完成简单防火墙)
content_by_lua #内容生成
header_filter_by_lua    #应答 HTTP 过滤处理(例如添加头部信息)
body_filter_by_lua #应答 BODY 过滤处理(例如完成应答内容统一成大写)
log_by_lua #回话完成后本地异步完成日志记录(日志可以记录在本地，还可以同步到其他机器)
```

### Lua Code Cache 
> [lua-nginx-module directives](https://github.com/openresty/lua-nginx-module#directives)
```
# http, server, location, location if
lua_code_cache off; #方便开发，线上一定要关闭，否则影响性能
```

### Lua Kafka
> 使用的开源库：https://github.com/doujiang24/lua-resty-kafka
> 直接下载源码到任意目录（最好是放到OpenResty/lualib下，方便统一管理）
```
# for http {}
lua_package_path "/path/to/lua-resty-kafka/lib/?.lua;;";
```

> 如果kafka使用了域名，需要在nginx.conf中开启dns解析，否则会导致域名解析失败，配置如下：
```
# for http {}
resolver 10.10.10.10;
```
