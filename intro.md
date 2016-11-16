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
* 参数
	* session , args , callback
	* 第一个参数session是用来做路由计算的

#### route router
* route用来标识一个具体服务或者客户端接受服务端推送消息的位置; 例如"chat.chatHandler.send", chat就是服务器类型，chatHandler是chat服务器中定义的一个Handler，send则为这个Handler中的一个handle方法
* 可以粗略地认为router就是根据用户的session以及其请求内容，做一些运算后，将其映射到一个具体的应用服务器id。可以通过application的route调用给某一类型的服务器配置其router
* 如果不配置的话，pomelo框架会使用一个默认的router。pomelo默认的路由函数是使用session里面的uid字段，计算uid字段的crc32校验码，然后用这个校验码作为key，跟同类应用服务器数目取余，得到要路由到的服务器编号
* 注意这里有一个陷阱，就是如果session没有绑定uid的话，此时uid字段为undefined，可能会造成所有的请求都路由到同一台服务器。所以在实际开发中还是需要自己来配置router

#### session sessionService
* session 是由sessionService创建
* BackendSessionService创建 BackendSession
* SessionService,FrontendSession,Session

##### gate session [FrontendSession]
![gateSession](http://img.hb.aicdn.com/6116eb2f3cbbd164efc33424b96f4a45636ae6b15fc52-Qe0ABz_fw658)

##### connector session [FrontendSession]
![connectorSession](http://img.hb.aicdn.com/156999223104cd6cd65746fbbf71c46330ad30fb689d2-KXxy7V_fw658)

##### chat session [BackendSession]
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

#### component
* pomelo 核心是由一系列松耦合的component组成
* 可自己定制component , 在服务器启动时加载，完成一定的功能
* start, afterStart, stop 
* 导出工厂函数，而不是一个对象，app会将自己作为上下文信息以及后面的opts作为参数传递给这个函数，并返回一个component 对象
* app.load(componentName);

#### 增加admin module 
* 线上admin 工具
* 三主体：master,monitor,client
* 注册admin module; app.registerAdmin(moduleName,{app:app});
* moduleId , monitorHandler,masterHandler,clientHandler

##### register admin
![resigter admin](http://img.hb.aicdn.com/b43559c1b702de7d9a5e2060830798322cab8a8813d1c-3Szh64_fw658)


##### process admin module
![adminModule](http://img.hb.aicdn.com/235999b270ec8c35b2137ade7760263981b0314918611-YkzDZR_fw658)

* 线上admin , client 向master服务器发送request，请求中带有moduleId和对应的回调参数

###### module request 
![admin request](https://github.com/NetEase/pomelo/wiki/images/sd_admin.png)

#### 热更新
* hot文件夹下



```
#例子

var Handler = function(app) {
    this.app = app;
    this.channelService = app.get('channelService');
};

var handler = Handler.prototype;

handler.send = function(msg,session,next){
    next();
};

module.exports = {
    id: "chatHandler",
    func: Handler
};

```

#### bearcat
* context.json 配置

```
{
	"name": "arpg_name",
	"beans": [],
	"scan": "app"
}

```

* createApp 

```
var contextPath = require.resolve('./context.json');
bearcat.createApp([contextPath]);

bearcat.start(function() {
    Configure(); //app configure
    // start app
    app.start();
});

```



## pomelo服务器启动过程

### 启动过程

> pomelo start

> pomelo start 启动master服务器， master服务器调用进程创建子进程,即应用服务器

![process](http://img.hb.aicdn.com/37178ff5e355f4dcb4f7ea8fc6ffb782224527ab160f4-Ljz5BA_fw658)
> node <BasePath>/app.js env=development id=connector-server-1

#### Application 初始化

![init](https://github.com/NetEase/pomelo/wiki/images/app_init.png)
create app 创建应用
init app 初始化应用的配置信息
setenv 设置环境变了

processArgs 创建应用服务器的处理过程 ， 就是上面的node .../app.js ....

应用创建成功之后， 还会对app进行一些设置，比如服务器的路由设置，以及上下文变量的设置


### Master服务器启动
![master](https://github.com/NetEase/pomelo/wiki/images/sd_master.png)
* app.start() 后，加载默认的组件，对于master来说加载的组件为master和monitor组件
* Master组件的创建过程会创建MasterConsole, MasterConsole会创建MasterAgent, MasterAgent会创建监听Socket, 用来监听应用服务器的监控和管理请求
* 加载完组件后，会启动所有的组件。Master有自己的组件，还会启动用户自定义的Module, 在app.start()之前调用app.registerAdmin 挂在app上
* 所有的Module挂到app后，Master组件会启动MasterConsoleService.启动MasterConsoleService时，MasterConsoleService会从app处拿到所有挂在其上面的Module，然后将Module注册到自己的Module仓库中，这一步实际上就是Module放到一个以ModuleId做键的Map中，以使得后来有请求时，可以直接进行查询回调
* 然后开启MasterAgent监听，此时，Master组件就可以接受监控管理请求了
* 下一步，启动所有的Module. 
* 启动所有的应用服务器。Master组件完成了所有的其自身的Module的初始化和开启任务后，Master会委托Starter完成整个服务器群的启动。  Master服务器负责管理所有应用服务器

### 应用服务器启动
* 参数启动 
* 配置 app.configure('production|development', 'chat',function(){.....}) 配置数据库以及其他配置信息
* 细节的东西比较多 不一一讲了

### 服务器关闭
* pomelo stop
* 线上服务器可通过自定义Module工具停服







### 简单介绍完毕

>到此pomelo的简单介绍算是结束了 要说的东西实在不是一句两句能说完的 大家可以去Github上去看文档 






## 实战















