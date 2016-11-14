# Pomelo

![https://github.com/NetEase](http://img.hb.aicdn.com/36e5b65a8350952199e46b602bd2ff878498982b1136d-BvcHCk_fw658)

## Overview

### Intro

* pomelo 是一个与以往单进程的游戏框架不同，拥有**高性能**，**高可伸缩性**，**分布式多进程**的游戏服务器框架。Easy configure , Easy use!
* 它包括基础开发框架和一系列相关工具和库
* **pomelo-rpc**,**pomelo-rpc-zeromq**,**pomelo-scheduler**,**pomelo-status-plugin**,**bearcat**,**pomelo-logger**,**seq-queue**,**pomelo-cli**,**pomelo-robot**,**pomelo-admin-web**
* 组成：框架，库，工具，客户端库(unity,cocos)

### 选择Pomelo理由

* 架构的可伸缩性好，采用多进程单进程的运行架构扩展服务器非常方便，node.js的网络io优势提供了高可伸缩性，写好的应用只需要简单地修改一下配置就能轻松地伸缩扩充
* 易用. pomelo基于轻量级的nodejs，其开发模型与web应用的开发类似，基于convention over configuration的理念， 几乎零配置， api的设计也很精简，很容易上手，开发快速
* 框架的松耦合和可扩展性好. 遵循node.js微模块的原则， 框架本身只有很少的代码，所有component、库、工具都可以用npm module的形式扩展进来，任何第三方都可以根据自己的需要开发自定义module，并把它整合到pomelo的框架中
* 完整的demo和文档. 不过要吐槽一句 坑很多 对于初次接触的人来说 ，现在基本上也没有人来维护这个框架了。而且和nodejs的版本兼容的不是太好。 Pomelo开发者群里面 经常有人问到: 你们的nodejs用的哪个版本，我的版本安装pomelo失败。

#### 安装[node版本兼容问题 经常安装失败]
* 第一种方式
> npm install pomelo -g

* 第二种方式
> git clone https://github.com/NetEase/pomelo.git
> 
> cd pomelo
> 
> npm install -g

### 项目结构

项目创建成功后的项目结构如下图

![](https://github.com/NetEase/pomelo/wiki/images/HelloWorldFolder.png)

### 启动 

#### pomelo start 启动服务器
#### pomelo list  查看服务器状态
#### pomelo stop 停止应用 

![](http://img.hb.aicdn.com/700d3d6d392d5824ad70f1ebc775081a117cbdd3ac9de-NrUZpo_fw658)


>到此pomelo的介绍算是结束了


















