# Pomelo框架总结
## A fast,scalable,distributed game server framework for Node.js

## 联系我
Pomelo交流群 @老顽童-NextZeus
微信群 BJ NodeJS Club @老顽童

### 适用场景
* 多人游戏：手游，社交游戏，网页游戏，MMORPG ,ARPG
* 实时应用：聊天，消息推送，等等

### 特点
* 高性能：基于Node.js开发，分布式多进程
* 扩展性强：灵活的服务器扩展，通过配置servers.json或者命令动态管理服务器群
* 轻量级：几乎零配置；基于nodejs开发效率高；javascript语言入门容易；API设计简单，容易上手，开发快速
* 多平台支持：js,flash,android,ios,cocos,c
* 库和工具辅助开发，pomelo-cli,pomelo-admin,pomelo-robot
* 案例丰富，参考资料众多[wiki管理不太好，混乱]
* 支持插件架构，如master高可用，在线状态
* 可自定义协议，组件
* 松耦合：所有的component,库，工具都可以用npm module的形式扩展进来
* 目前版本已更新到2.2.5 支持nodeV6+

### 安装
> npm install pomelo -g
> 
> pomelo init helloWorld
> 
> cd helloWorld & sh npm-install.sh
>
> cd game-server & pomelo start
> 
> cd web-server & node app.js
> 


### Pomelo启动流程

![启动模型](https://camo.githubusercontent.com/5e2819755651743710be78547da919109291801f/687474703a2f2f706f6d656c6f2e6e6574656173652e636f6d2f7265736f757263652f646f63756d656e74496d6167652f73746172745f6d6f64656c2e706e67)

1. createApp 创建app应用实例
2. 加载配置app.configure() & 默认component
3. master服务器启动，通过配置信息和启动参数启动其他服务器群

![组件加载](https://camo.githubusercontent.com/fd833b788c4d0f211aed0abd4dd631feed3dacde/687474703a2f2f706f6d656c6f2e6e6574656173652e636f6d2f7265736f757263652f646f63756d656e74496d6167652f636f6d706f6e656e74735f6c6f61642e706e67)

### App Configuration

```

var pomelo = require('pomelo);
var app = pomelo.createApp();
//<!--env:development|production serverType: master/gate/connector/....-->
app.configure(<env>,<serverType>, function(){
	//<!--configure filter-->
	app.before(pomelo.filters.toobusy());//接口访问限制
	app.filter(pomelo.filters.serial()); // configure builtin filter: serialFilter
    app.filter(pomelo.filters.time()); //开启conn日志，对应pomelo-admin模块下conn request
    app.rpcFilter(pomelo.rpcFilters.rpcLog());//开启rpc日志，对应pomelo-admin模块下rpc request
	
	//<!--启动系统监控-->
	app.enable('systemMonitor');
	
	//<!--注册admin module-->
	//<!--enable systemMonotor后 注册的admin module才可使用-->
	var onlineUser = require('./app/modules/onlineUser');
    if (typeof app.registerAdmin === 'function') {
        app.registerAdmin(onlineUser, {app: app});
    }
	
	//<!--加载配置-->
	app.loadConfig('mysql', app.getBase() + '/config/mysql.json');
	
	//<!--configure router 路由函数-->
	app.route('chat', routeUtil.chat);
	
	// proxy configures
    app.set('proxyConfig', {
        cacheMsg: true,
        interval: 30,
        lazyConnection: true
        enableRpcLog: true
    });

    // remote configures
    app.set('remoteConfig', {
        cacheMsg: true,
        interval: 30
    });
	
	//<!--设置内部connector组建 心跳时长 通信协议-->
	app.set('connectorConfig',{
            connector: pomelo.connectors.hybridconnector,
            heartbeat: 30,
            useDict: true,
            useProtobuf: true,
            handshake: function (msg, cb) {
                cb(null, {});
            }
    });
    
    //<!--设置变量-->
    app.set(key,value);
    
    //<!--加载用户自定义组件 组件导出的都是工厂函数，app可自动识别，讲其自身作为opt参数传递给组件,方便访问app上下文-->
    app.load(helloWorldComponent, opt)
    
    //<!--使用插件-->
    var statusPlugin = require('pomelo-status-plugin');
    app.use(statusPlugin, {
	    status:{
    		host:	'127.0.0.1',
    		port:	6379
	    }
	});	
	
	//<!--启动app -->
	app.start();
});

process.on('uncaughtException', function(err){
	console.error('uncaughtException : ', err, err.stack());
});

```

### 为什么采用node.js开发
1. node.js: fast, scalable,realtime,network符合游戏服务器的要求
2. noee.js在网络io上支持高并发，非阻塞的优势满足游戏服务器网络密集型，实时性要求极高的特点
3. node.js天生单线程，处理复杂逻辑的时候 无需考虑线程同步，锁，死锁等问题
4. 语言优势 js开发可实现快速迭代

### 运行架构图
![MMO运行架构图](https://camo.githubusercontent.com/8f6a9b8343b9f4240548718647340eaaf870a133/687474703a2f2f706f6d656c6f2e6e6574656173652e636f6d2f7265736f757263652f646f63756d656e74496d6167652f6d6d6f4172636869746563747572652e706e67)

运行架构说明：

1. 客户端通过websocket长连接连到connector服务器群。
2. connector负责承载连接，并把请求转发到后端的服务器群。
3. 后端的服务器群主要包括按场景分区的场景服务器(area)、聊天服务器(chat)和状态服务器等(status)， 这些服务器负责各自的业务逻辑。真实的案例中还会有各种其它类型的服务器。
4. 后端服务器处理完逻辑后把结果返回给connector， 再由connector广播回给客户端。
5. master负责统一管理这些服务器，包括各服务器的启动、监控和关闭等功能。

### pomelo解决的问题
游戏服务器端，往往需要处理大量的各种各样的任务，比如：管理客户端的连接，维护游戏世界端状态，执行游戏的逻辑等等
每一项任务所需的系统资源也可能不同，如：IO密集或CPU密集等。
而这些复杂的任务只有一个单独的服务器进程很难支撑和管理起来的。所以游戏服务器往往是由多个类型的服务器进程组成的集群。
每隔服务器进程专注于一块具体的业务，如：连接服务，场景服务，聊天服务等。这些服务器进程相互协作，对外提供完整的游戏服务

由于存在上述的复杂性，，游戏服务器端的开发者往往需要花费大量的时间精力在诸如服务器类型的划分，进程数量的分配，以及这些
进程的维护，进程间的通讯，请求的路由等等这些底层的问题上。 而这些都是一些重复而繁琐的工作。pomelo则是为解决这些问题而生的框架
下面看下它的框架结构

### pomelo框架

![框架](https://camo.githubusercontent.com/1c88f142423bed8f67b19a71689e6360ce6c1ebc/687474703a2f2f706f6d656c6f2e6e6574656173652e636f6d2f7265736f757263652f646f63756d656e74496d6167652f706f6d656c6f2d617263682e706e67)

1. server management, pomelo是个真正多进程、分布式的游戏服务器。因此各游戏server(进程)的管理是pomelo很重要的部分，框架通过抽象使服务器的管理非常容易。
2. network, 请求、响应、广播、RPC、session管理等构成了整个游戏框架的脉络，所有游戏流程都构建在这个脉络上。
3. application, 应用的定义、component管理，上下文配置， 这些使pomelo framework的对外接口很简单， 并且具有松耦合、可插拔架构。

### 服务器类型
pomelo框架提供了一套灵活，快捷的服务器类型系统。 通过pomleo框架，开发者可以自由地定义自己的服务类型，分配管理进程资源。
在pomelo框架中，根据服务器的职责的不同，服务器主要分为fronted 和 backend两类型，二者关系如下图所示：

![](https://camo.githubusercontent.com/5935f0403ef84c20197af32d6ca0d86069c742b3/687474703a2f2f706f6d656c6f2e6e6574656173652e636f6d2f7265736f757263652f646f63756d656e74496d6167652f7365727665722d747970652e706e67)

##### fronted
前端服务器负责承载客户端的连接，，与客户端之间的所有请求和响应都会经过fronted.
同时也负责维护客户端session，并把请求路由给backend server
##### backend
后端服务器负责接收fronted 发过来的请求，实现具体的游戏逻辑，并把消息回推给fronted server,再最终发送给客户端

### Request/Response
pomelo中有四种消息类型：request, response, notify , push
客户端可以向服务器发送两种类型的消息：request, notify
request/response是一组，服务器端接收到请求，处理完逻辑后会做出响应
notify是单向的，客户端通知服务器端的消息， 服务器端无需返回响应
push 是服务器主动推送消息给客户端,客户端注册监听事件，处理具体的事件逻辑 比如：公告，邮件等

整个请求响应过程如下图：

![](http://img.hb.aicdn.com/5c9e12db624e5d47d552d3dedcd12e70e90e20e66d69-WTDIyk_fw658)

#### 请求流程处理

![](https://camo.githubusercontent.com/f7a405773f551d70ffc001f3768166d13b479687/687474703a2f2f706f6d656c6f2e6e6574656173652e636f6d2f7265736f757263652f646f63756d656e74496d6167652f726571756573742d666c6f772e706e67)

1. filter: before, after 过滤器； 请求处理前和请求处理后调用
2. handler 业务逻辑接口

##### filter过滤器

```

module.exports = function(){
    return new Filter();
}

var Filter = function(opt){
    
}

var filter = Filter.prototype;

//before
filter.before = function(msg, session, next){
    next();
    //进入到下一个before filter
}

//after
filter.after = function(err, msg, session, resp, next){
    //todo 
    next();
}

```

##### handler

```

module.exports = function(){
    return bearcat.getBean(Handler);
}

var Handler = function(){
    this.$id = 'Handler';
}

handler.methodName = function(msg, session, next){

}


```

##### errorHandler 全局异常处理

```
/**
* err 前面流程发生的异常
* resp 是前面流程传递过来，要返回给客户端的响应信息
* 其他参数与前面的handler一样
*/
errorHandler = function(err, msg, resp , session, next){
    //todo 
}

app.set('errorHandler',errorHandler);

```

### Session
session可以看成一个简单的key/value的对象，主要是维护当前玩家状态信息，比如：当前玩具的id, fronted服务器等
session对象由客户端所连接的frontend服务器维护。
在分发请求给backend服务器时，frontend服务器会克隆session， 连同请求一起发送给backend服务器。
在backend服务器上，session应该是只读的，或者起码只是本地读写的一个对象。
任何直接在session上的修改，只对本服务器进程生效，并不会影响玩家的全局状态信息。
如需修改全局session里的状态信息，需要调用frontend服务器提供的RPC服务。

```
//connector/entryHandler.entry

session.bind(uid);
session.set(key,value);

session.pushAll();//同步session给后端服务器

前端和后端session 都可以通过session.set session.pushAll() 互相同步session信息

```

### Channel
应用场景：推送广播消息
Channel是服务器向客户端推送消息的通道。 
Channel可以看成是一个玩家ID的容器。 比如一个公会的成员，就是一个容器，公会内部聊天，只推送给公会的成员
Channel只适用于服务器进程本地，及在服务器进程A创建的channel和在服务器进程B创建的channel是不同的，互相独立的

channel分两类： 具有channel_name和匿名channel, 区别在于匿名channel推送消息需制定uids。
推送消息过程：
1. channel所在的服务器进程将消息推送到玩家客户端所连接的frontend进程[依赖底层的RPC框架]
2. 通过frontend进程将消息推送给玩家客户端[推送钱会根据玩家所在的frontend服务器ID分组,log中会现实groups:[{'connector-server-1':[uid1]}]]


### servers.json 

1. max-connections(optional) 前端server可以hold的最大连接数
2. cpu 设置服务器的cpu相关性的过程,只适用于unix platform
3. 如果是同一台服务器部署多个pomelo创建的游戏实例在运行，要注意serverId区别开，不能写成一样的serverId。

### start game-server with optional env

```
//源码
/**
 * Setup enviroment.
 */
var setupEnv = function(app, args) {
    //              'env'         , 创建app opt参数  ||     NODE_ENV         ||     'development'
    app.set(Constants.RESERVED.ENV, args.env        || process.env.NODE_ENV || Constants.RESERVED.ENV_DEV, true);
};

var app = pomelo.createApp({
    env:    process.env.NODE_ENV
});

>$ pomelo start -e production

```

### [对Pomelo服务器框架的重新认识](comprehendPomelo/)


### [常见问题答疑](question/)
### 早期写的简书文章
[Pomelo游戏开发~不再更新](http://www.jianshu.com/c/f42580039b45)

### 其他文章
0. [installPomeloDep](installPomeloDep/)
1. [connector](connector/)
2. [router](router/)
4. [pomelo-cli](pomelo-cli/)
5. [pomelo-rpc-zeromq](pomelo-rpc-zeromq/)
6. [component](component/)
7. [hotupdate](hotupdate/)
8. [bearcat](bearcat/)
9. [addserver](addserver/)
10. [pomelo-schedule](pomelo-schedule/)
11. [pushMessage](pushMessage/)
12. [notify](notify/)
13. [sessionClosed](sessionClosed/)
14. [heartbeat](heartbeat/)
15. [线上服务器部署](deploy/)
16. [session](session/)
17. [globalErrorHandler](globalErrorHandler/)
18. [globalFilter](globalFilter/)
19. [onlineUser在线玩家信息](onlineUser/)
20. [filters](filters/)
21. [distributedDeployment](distributedDeployment/)
22. [highavailability](highavailability/)

