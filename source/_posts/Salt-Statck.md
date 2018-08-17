---
title: Salt Statck
date: 2017-03-23 18:39:47
categories: 技术
tags: salt
---

- 简介
> SaltStack是一个服务器基础架构集中化管理平台，具备配置管理、远程执行、监控等功能，一般可以理解为简化版的puppet和加强版的func。SaltStack基于Python语言实现，结合轻量级消息队列（ZeroMQ）与Python第三方模块（Pyzmq、PyCrypto、Pyjinjia2、python-msgpack和PyYAML等）构建。
> 通过部署SaltStack环境，我们可以在成千上万台服务器上做到批量执行命令，根据不同业务特性进行配置集中化管理、分发文件、采集服务器数据、操作系统基础及软件包管理等，SaltStack是运维人员提高工作效率、规范业务配置与操作的利器。
 
- 使用版本
> 2016.11
> https://github.com/saltstack/salt
 
- 测试环境
```
1台master， 2台minion
```        
 
- 模块及配置
> salt-master
> salt-minion
> [salt-api](http://corey.wang/2017/03/23/salt-api/)
 
- salt命令解析 [blog](http://arlen.blog.51cto.com/7175583/1424684)  [blog2](http://noodle.blog.51cto.com/2925423/1744607)  [官方文档](https://docs.saltstack.com/en/latest/ref/cli/salt.html)

- <b>个人整理</b>
```salt '<target>' <function> [arguments]```

```
salt '*' cp.get_file salt://jdk-8u121-linux-x64.rpm  /tmp/jdk-8u121-linux-x64.rpm  #把salt-master端对应文件拷贝到minion端相应目录下, salt://默认为/srv/salt，master中可以配置
salt '*' cmd.run 'rpm -ivh /tmp/jdk-8u121-linux-x64.rpm' #远程安装jdk， 可以增加template=jinja，使命令支持jinja模板，方便不同机器自动替换相应的参数
salt '*' cp.get_file salt://apache-tomcat-8.0.35.zip /tmp/apache-tomcat-8.0.35.zip #copy tomcat到minion

salt '*' cmd.run 'cd ~ & mkdir tools' #远程创建tools文件夹，如果该文件夹已存在，*会报错*，使用salt '*' file.mkdir /tmp/wangchao36-test命令也可以
salt '*' cmd.run 'unzip /tmp/apache-tomcat-8.0.35.zip -d /root/tools/' #解压tomcat
salt '*' cmd.run 'chmod +x /root/tools/apache-tomcat-8.0.35/bin/*.sh & /root/tools/apache-tomcat-8.0.35/bin/startup.sh' #启动tomcat
salt '*' cmd.run 'ps -ef | grep tomcat'  # 查看tomcat关键词的进程
 
 
salt '*' pkg.install httpd #执行安装命令, 相当于apt-get, yum
salt '*' lowpkg.bin_pkg_info /tmp/jdk-8u121-linux-x64.rpm # 查询rpm包信息，lowpkg相当于rpm命令

salt '*' cmd.script salt://scripts/runme.sh #远程执行一个脚本

salt '*' cmd.run 'pip install psutil' #ps模块需要提前安全psutil
salt '*' ps.pgrep tomcat full=true  #查询tomcat相关进程的pid
salt '*' ps.psaux tomcat #ps aux |grep tomcat
 
salt '*' file.append /root/tools/apache-tomcat-8.0.35/bin/catalina.sh dddd  #在文件尾增加
salt '*' file.grep /root/tools/apache-tomcat-8.0.35/bin/catalina.sh ddd #文件中查找
salt '*' file.comment_line /root/tools/apache-tomcat-8.0.35/bin/catalina.sh ddd #行加注释
salt '*' file.seek_read /root/tools/apache-tomcat-8.0.35/bin/catalina.sh 1000 0 #读
salt '*' file.seek_write /root/tools/apache-tomcat-8.0.35/bin/catalina.sh '#xxx'1000 #写 1000为offset
```

