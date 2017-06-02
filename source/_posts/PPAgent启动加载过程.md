---
title: PPAgent启动加载过程
date: 2017-06-02 17:02:47
tags: pinpoint
---
#### 知识点
> -javaagent xxx.jar,  实现premain()
> Class.getConstructor
> java.lang.reflect.Constructor
> ServiceLoader.load, 简单理解是获取某个接口（服务）的实现类，扫描META-INF/services下的配置
> TraceMetadataProvider接口的实现类
> ClassFileTransformer.transform, 重点看ProfilePluginLoader实现类


#### 图一
<img src="/img/article/PPAgent启动加载过程.jpg"/>
#### 图二
<img src="/img/article/DefaultTraceMetadataLoaderService加载过程.jpg"/>
#### 图三
<img src="/img/article/DefaultAgent加载过程.jpg"/>
