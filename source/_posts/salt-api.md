---
title: salt-api
date: 2017-03-23 18:42:26
categories: 技术
tags: salt
---

###基础说明
> salt-api对外提供https api接口，方便使用salt相关的功能
> salt-api需要部署在master的机器上
> [官方文档] (http://salt-api.readthedocs.io/en/latest/ref/netapis/all/saltapi.netapi.rest_cherrypy.html#a-rest-api-for-salt)，附件中是官方文档pdf版
> [参考blog](http://ju.outofmemory.cn/entry/97116)
 
###接口安全方面
> salt-api使用前需要调用login api（帐号密码）来获取token，后续的api请求使用token即可

###依赖 
> 使用pip安装salt时，默认已经安装了salt-ai
> salt-api对外提供http服务，依赖Python的CherryPy库，这里需要检查本机是否已安装
 
###配置过程
> 生成https证书
```
cd  /etc/pki/tls/certs
# 生成自签名证书, 过程中需要输入key密码及RDNs
make testcert
cd /etc/pki/tls/private/
# 解密key文件，生成无密码的key文件, 过程中需要输入key密码，该密码为之前生成证书时设置的密码
openssl rsa -in localhost.key -out localhost_nopass.key
```
 
> 创建salt-api的用户
```
useradd -M -s /sbin/nologin salt-api
echo "salt-api-passwd" | passwd salt-api —stdin
```
 
> 配置权限, [eauth官网文档](https://docs.saltstack.com/en/latest/topics/eauth/index.html)
```
#/etc/salt/master.d/eauth.conf
external_auth:
  pam:
    salt-api:
      - .*
      - '@wheel'
      - '@runner'
```
> 配置api端口、证书等, 这里使用了cherrypy库（python的web库）
```
#/etc/salt/master.d/api.conf
rest_cherrypy:
  port: 8000
  ssl_crt: /etc/pki/tls/certs/localhost.crt
  ssl_key: /etc/pki/tls/private/localhost_nopass.key
```
 
 
> 启动 
```
salt-api -d --log-file-level=all
#这里的日志级别为all是为了测试用，日志级别包含：'all', 'garbage', 'trace', 'debug', 'profile', 'info', 'warning','error', 'critical', 'quiet'. Default: 'warning'
```
 
###api接口列表
```
返回JSON格式，head中增加Accept:application/json
```
> [/](http://salt-api.readthedocs.io/en/latest/ref/netapis/all/saltapi.netapi.rest_cherrypy.html#id1?_blank) 

```
#GET
获取当前的client
#POST
执行salt命令
```
> [/login](http://salt-api.readthedocs.io/en/latest/ref/netapis/all/saltapi.netapi.rest_cherrypy.html#login)
```
Log in to recieve a session token
```
> [/logout](http://salt-api.readthedocs.io/en/latest/ref/netapis/all/saltapi.netapi.rest_cherrypy.html#logout)
```
#POST
Destroy the currently active session and expire the session cookie
```
> [/minions](http://salt-api.readthedocs.io/en/latest/ref/netapis/all/saltapi.netapi.rest_cherrypy.html#minions)
```
#GET /minions/(mid)
有mid时，查询mid机器的详细信息，没有mid时查询所有的minion的详细信息
#POST /minions
远程执行命令，并返回jobid，相当于/执行命令中的client=local_async
这里并没直接返回结果，需要通过jobs结果查询返回结果
适合异步的场景
```
> [/jobs](http://salt-api.readthedocs.io/en/latest/ref/netapis/all/saltapi.netapi.rest_cherrypy.html#jobs)
```
#GET /jobs/(jid)
A convenience URL for getting lists of previously run jobs or getting the return from a single job
有jid时，查询该job的详细结果
无jid，返回job列表（执行job时的参数和jobid）
```
> [/run](http://salt-api.readthedocs.io/en/latest/ref/netapis/all/saltapi.netapi.rest_cherrypy.html#run)
```
#POST
不使用token的方式执行，这里需要每次请求带着用户名密码等认证信息
```
> [/events](http://salt-api.readthedocs.io/en/latest/ref/netapis/all/saltapi.netapi.rest_cherrypy.html#events)
```
#GET 需要深入了解
```
> [/ws](http://salt-api.readthedocs.io/en/latest/ref/netapis/all/saltapi.netapi.rest_cherrypy.html#ws)
```
#GET /ws/(token)
Return a websocket connection of Salt's event stream
对url api的补充，大数据量时可以使用这个。
```
> [/hook](http://salt-api.readthedocs.io/en/latest/ref/netapis/all/saltapi.netapi.rest_cherrypy.html#hook)
```
#POST /hook   具体用途还需了解
Fire an event in Salt with a custom event tag and data
```
> [/stats](http://salt-api.readthedocs.io/en/latest/ref/netapis/all/saltapi.netapi.rest_cherrypy.html#stats)
```
#GET /stats
Expose statistics on the running CherryPy server
```

