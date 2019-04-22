---
title: 推荐使用Hexo搭建个人博客
tags:
  - Hexo
categories:
  - 技术
toc: false
date: 2017-03-01 18:20:29
---

Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

## Hexo博客部署到Github
[超完美教程](http://www.cnblogs.com/imapla/p/5533000.html)

# 使用Travis-CI自动发布
## 第一步：生成access token
进入Github个人主页，找到：Settings -> Developer settings -> Personal access tokens，然后取Generate new token，参照下图配置即可。
![image.png](/images/2019/04/22/b15f3cb0-64f3-11e9-a6eb-db97a35784ce.png)
这里生成的Token，接下来会用到，请先妥善保存好。
## 第二步：注册并开启Travis-CI项目构建
使用 GitHub账户登录[Travis-CI](https://travis-ci.org/)官网 ，进去后能看到已经自动关联了 GitHub 上的仓库。这里我们选择需要启用的项目，即 blog-source。然后点击后面的Settings进入设置界面。
![image.png](/images/2019/04/22/d06d3800-64f3-11e9-a6eb-db97a35784ce.png)
## 第三步：配置Travis-CI自动构建
进入设置界面后可以参考我的配置：
![image.png](/images/2019/04/22/d9b50320-64f3-11e9-a6eb-db97a35784ce.png)
配置主要注意一下两点即可：
- Build pushed branches
当分支收到新的push之后构建
- Environment Variables -> GH_TOKEN
GH_TOKEN，是我们第一步在github中生成的access token，因为要从github上将代码拉到travis-ci机器上进行构建，所以需要该token授权。
## 第四步：配置hexo的_config.yml
因为我们的博客托管在github pages，所以我们是以git的方式进行deploy的，hexo如何配置使用git方式进行deploy，请自行Google。下面截取了我的_config.yml文件中关于git deploy配置的片段。
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
- type: git
  # 下方的GH_TOKEN会被.travis.yml中sed命令替换
  repo: https://GH_TOKEN@github.com/gaoyoubo/blog.mspring.org.git
  branch: master
```
## 第五步：配置构建脚本.travis.yml
在hexo项目的根目录创建.travis.yml文件，该文件就是travis的构建脚本，下面是我的脚本配置，我会在脚本中详细注释每一步的作用。
```
# 指定语言为node_js，nodejs版本stable
language: node_js
node_js: stable

# 指定构建的分支
branches:
  only:
    - hexo

# 指定node_modules缓存
cache:
  directories:
    - node_modules

# 构建之前安装hexo-cli，因为接下来会用到
before_install:
  - npm install -g hexo-cli

# 安装依赖
install:
  - npm install

# 执行脚本，先hexo clean 再 hexo generate，会使用hexo的同学应该不陌生。
script:
  - hexo clean
  - hexo generate

# 上面的脚本执行成功之后执行以下脚本进行deploy
after_success:
  - git init
  - git config --global user.name "GaoYoubo"
  - git config --global user.email "gaoyoubo@foxmail.com"
  # 替换同目录下的_config.yml文件中GH_TOKEN字符串为travis后台配置的变量
  - sed -i "s/GH_TOKEN/${GH_TOKEN}/g" ./_config.yml
  - hexo deploy
```
## travis-ci的成功截图
![image.png](/images/2019/04/22/2e0a3710-64f4-11e9-a6eb-db97a35784ce.png)
# 使用HexoClient管理你的文章
- HexoClient在启动之后选择好hexo安装的目录，会自动读取Hexo目录中的文章。
- HexoClient中支持新建、修改文章，新建修改文章之后点击发布按钮能够将文章更改提交到git，并自动通过travis自动发布。（前提是按照上面步骤配置好）
- HexoClient支持七牛图片上传，七牛10G存储空间，每月10G流量免费，可以自行注册配置七牛，配置好后将七牛的ak、sk、bucket、域名配置到HexoClient中

