# Pomelo

![https://github.com/NetEase](http://img.hb.aicdn.com/36e5b65a8350952199e46b602bd2ff878498982b1136d-BvcHCk_fw658)

## Overview

### 介绍

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
#### pomelo stop  停止应用 

![](http://img.hb.aicdn.com/700d3d6d392d5824ad70f1ebc775081a117cbdd3ac9de-NrUZpo_fw658)

### 术语解释

#### gate服务器
* 负责前端的负载均衡
* 配置中 只有**clientPort**字段 没有**port**字段
* 客户端首先向gate服务器发送请求，gate会给客户端分配具体的connector服务器。具体的分配逻辑自定
* "frontend": true


#### connector服务器
* 负责接收客户端的连接请求，创建与客户端的连接，维护客户端的session信息
* 接收客户端对后端服务器的请求，按照用户配置的路由策略，将请求路由给具体的后端服务器
* 扮演一个中间角色，负责接收请求，并将处理结果发送给客户端
* 配置中同时具有clientPort和port,clientPort用来监听客户端的连接，port端口用来给后端提供服务
* "frontend": true

#### 应用逻辑服务器
* gate服务器和connector服务器又都被称作前端服务器，应用逻辑服务器是后端服务器
* 后端服务器之间通过rpc进行交互
* 后端服务器不会和客户端有直接连接，只需监听其提供服务的端口

#### master服务器
* 加载配置文件，通过读取配置文件，启动所配置的服务器集群，并对所有服务器进行管理

#### RPC调用
* 分类: sys 系统rpc调用 和 用户自定义的rpc 调用
* sys rpc调用 
	* 后端向前端请求session信息
	* 后端通过channel推送消息时对前端服务器发起的rpc调用
	* 前端服务器将用户请求给后端服务器

#### route router
* route用来标识一个具体服务或者客户端接受服务端推送消息的位置; 例如"chat.chatHandler.send", chat就是服务器类型，chatHandler是chat服务器中定义的一个Handler，send则为这个Handler中的一个handle方法
* 可以粗略地认为router就是根据用户的session以及其请求内容，做一些运算后，将其映射到一个具体的应用服务器id。可以通过application的route调用给某一类型的服务器配置其router
* 如果不配置的话，pomelo框架会使用一个默认的router。pomelo默认的路由函数是使用session里面的uid字段，计算uid字段的crc32校验码，然后用这个校验码作为key，跟同类应用服务器数目取余，得到要路由到的服务器编号
* 注意这里有一个陷阱，就是如果session没有绑定uid的话，此时uid字段为undefined，可能会造成所有的请求都路由到同一台服务器。所以在实际开发中还是需要自己来配置router

#### session sessionService
* session 是由sessionService创建
* BackendSessionService创建 BackendSession
* SessionService,FrontendSession,Session

##### gate session
![gateSession](http://img.hb.aicdn.com/6116eb2f3cbbd164efc33424b96f4a45636ae6b15fc52-Qe0ABz_fw658)

##### connector session
![connectorSession](http://img.hb.aicdn.com/156999223104cd6cd65746fbbf71c46330ad30fb689d2-KXxy7V_fw658)

##### backend session[chat]
![1](http://img.hb.aicdn.com/9e8e5463f176e4dd7dcaf908d17baf85df69cfd566f87-DkjEXv_fw658)
![](http://img.hb.aicdn.com/5539b15e78fc1f4b55e465ce49dccd5621782bc851619-7rBA0q_fw658)

##### chat 
![](http://img.hb.aicdn.com/6c560ae36918eee1431fcfa5285dc5a4e662665e34a7a-nbc6qu_fw658)


#### channel 
* 可以看作是一个玩家ID的容器，主要用于需要广播推送消息的场景，例如聊天。 个人私聊，世界频道，工会频道，都是根据channel去筛选， 要给具体的channel里的玩家推送消息
* channel 在应用服务器之前不是共享的。服务器A上创建的channel，只有服务器 A才能给它推送消息

#### filter
* 分类：before after
* before 对请求进行过滤 做一些前置处理 例如检测玩家session信息，以及是否登陆，以及黑名单白名单等
* after 进行请求后置处理 特殊处理 发送邮件等

#### handler
* 实现具体业务逻辑 在before filter和after filter之间，next回调函数串通
* msg, session, next 参数，通过next 进入到after filter处理流程

#### error handler
* 处理全局异常情况的地方 可发邮件通知维护



>到此pomelo的介绍算是结束了





## 实战














