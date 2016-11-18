# 实战 [以下是我之前的项目部分代码]

## app.js


```
var pomelo = require('pomelo');
var zmq = require('pomelo-rpc-zeromq');
var bearcat = require('bearcat');
var mysql = require('mysql');
var path = require('path');

//mysql config
var mysqlConfig = require('./config/mysql.js');



var app = pomelo.createApp();
app.set('name','appname');

//全局变量设置 有个地方会用到
global.app = pomelo.app;

//下面设置的变量 在自定义pomelo-cli工具会用到 如果不挂载在app上 其他地方引用不到
global.app.bearcat = bearcat;
global.app.async = async;
global.app.mysqlModule = mysql;
global.app.mysqlConfig = mysqlConfig;


//最重要的配置服务器

var Configure = function(){
	app.configure('production|development', function () {
		//todo 坑 ： 加上之后服务器启动之后 一会儿就会宕机 尚未找到原因
        //app.enable('systemMonitor'); 

        app['myLoader'] = myLoader;
        app['redis'] = app.myLoader.load(__dirname + '/lib/redis.js');
        //设置路由函数router
        var routeUtil = app.myLoader.load(__dirname + '/app/util/route.js');
        app.route('chat', routeUtil.chat);
        app.route('gate', routeUtil.gate);
        app.route('connector', routeUtil.connector);
        app.route('data', routeUtil.data);
       

        //全局路由过滤函数
        var globalFilter = require('./app/servers/filter/globalFilter.js');
        app.globalFilter(globalFilter()); //处理未登录玩家 黑名单 白名单等

        //全局错误处理
        var GlobalHandler = require('./app/globalHandler/globalErrorHandler.js');
        var globalErrorHandler = new GlobalHandler();
        app.set('globalErrorHandler',globalErrorHandler.globalHandler);
        app.set('errorHandler',globalErrorHandler.globalHandler);

        app.connDispatch = {};//链接调度
        app.roomDispatch = {};//房间调度

		//rpc client 
        app.set('proxyConfig', {
            rpcClient: zmq.client
        });
		
		//rpc server
        app.set('remoteConfig', {
            rpcServer: zmq.server
        });

    });
    
    
    //gate服务器配置 
    app.configure('production|development', 'gate', function () {
        app.set('connectorConfig',
            {
                connector: pomelo.connectors.hybridconnector,
                heartbeat: 8, //心跳
                useDict: true,
                useProtobuf: true,
                disconnectOnTimeout: true 超时断开连接
            });
    });
    
    //chat 服务器配置
    app.configure('production|development', 'chat', function () {

    });
    
    //connector服务器配置
    app.configure('production|development', 'connector', function () {
        app.set('connectorConfig',
            {
                connector: pomelo.connectors.hybridconnector,
                heartbeat: 8,
                useDict: true,
                useProtobuf: true,
                disconnectOnTimeout : true
            });
    });
    
    //data 应用服务器配置
    app.configure('production|development', 'data', function () {
	    //配置mysql
        var mysqlConf = app.myLoader.load(__dirname + '/config/mysql.js');
        app['mysql'] = app.myLoader.load(__dirname + '/lib/mysql.js');
        app['mysql'].config(mysqlConf);
        global.app.mysql = app.mysql;
    });
    
    //master
    app.configure('production|development', 'master', function() {
        /＊
        这里可以写一些初始化服务器的功能 比如说删除一些以前的无用信息
        服务器机器人的初始化
        清理redis缓存信息
        等等
        ＊／
        //清理停服状态  每次停服都是先设置状态 再主动踢出玩家 
        global.app.redis.get('gameServerStatus',function(){});           
    });
    
}

//use bearcat start app
var contextPath = require.resolve('./context.json');
bearcat.createApp([contextPath]);

bearcat.start(function() {
    Configure();
    // start app
    app.start();
});

//配置日志信息格式
var logger = require('pomelo-logger').configure(path.join(app.getBase(),'/lib/log4js.json'),{base : app.getBase()});


//异常错误捕捉处理
process.on('uncaughtException',function(err){
	//错误日报 发送邮件 nodemailer库
    var mailModel = bearcat.getBean('sendMailModel');
    mailModel.sendMail(err.stack);
});


//正式环境 去掉connsole log输出
var env = app.get('env');
if(env != 'development'){
   console.log = function(){};
   console.info = function(){};
   console.warn = function(){};
}

```

## 配置文件
### package.json 主要是版本
```
{
  "name": "appname",
  "version": "0.0.1",
  "private": false,
  "dependencies": {
    "pomelo": "1.1.3",
    "request": "2.34.0",
    "zmq": "2.6.0",
    "crc": "0.2.1",
    "bearcat": "^0.2.37",
    "pomelo-logger": "0.1.2",
    "nodemailer": "1.3.0",
    "nodemailer-smtp-transport": "0.1.13",
    "async": "0.2.10",
    "mysql": "^2.9.0", [版本问题坑过一次,老是断开连接]
    "pomelo-rpc-zeromq": "0.0.8",
    "redis": "0.10.0",
    "redis-jsonify": "0.0.4",
    "underscore": "1.7.0",
    "ws": "*",
    "socket.io": "0.9.17",
    "pomelo-protocol": "*",
    "pomelo-protobuf": "*",
    "fs": "0.0.2",
    "express": "^4.13.3",
    "body-parser": "^1.14.1"
  }
}
```
### context.json

```
# bearcat配置
{
	"name": "appname",
	"beans": [],
	"scan": "app" //指的是game-server/app文件夹下的*.js
}

```


### mysql.js
```
var mysql = require('mysql');
var pool;//mysql 连接池

var db = function(){
    return {config : function(conf){
       pool =  mysql.createPool({
            host     : conf.host,
            user     : conf.user,
            password : conf.password,
            database : conf.database,
            acquireTimeout : conf.acquireTimeout
        });

        pool.getConnection(function(err, connection) {
            if(err){
                console.log("mysql初始化失败!");
            }else{
                console.log("mysql初始化成功!");
                connection.release();
            }
        });
    }};
}

//
exports.mysql = function(){
    return pool;
};


```

### router函数设置
```
var serversConfig = require('../../config/servers.json');

exp.chat = function (route, msg, app, cb) {
    var chat = app.getServersByType('chat')[0];
    if (!chat) {
        console.log('failed to route to chat.');
        return;
    }
    cb(null, chat.id);
};

exp.connector = function (frontendId, msg, app, cb) {
    if (!frontendId) {
        console.log('failed to route to connector.');
        return;
    }
    cb(null, frontendId);
};

//gate服务器
exp.gate = function (serverId, msg, app, cb) {
    var gate = app.getServersByType('gate')[0];
    if (!gate) {
        console.log('failed to route to gate.');
        return;
    }
    cb(null, gate.id);
}

/*
data 应用服务器 应用服务器的设置 可能会比较复杂一些，因为应用服务器会比gate,connector服务器要多
第一个参数session , 在我们的应用里有可能是session, characterId[string] 
*/
exp.data = function (session, msg, app, cb) {
    var env = app.env;
    var envConfig = serversConfig[env];
    var servers = envConfig['data'];
    var data,index ;
    var charId;

    if(!!session && typeof(session) == 'object'){
        if(typeof(session.get) == 'function'){
            charId  = session.get('character_id');
        }

        if(!charId){//没有角色Id的情况 : 没有登录和同步
            //index = parseInt(Math.random() * servers.length);
            index = servers.length - 1;
        } else {
            index = parseInt(charId) % servers.length ;
        }
    }else if(session && (typeof(session) == 'string' || typeof(session) == 'number')){
        index = session % servers.length ;
    }else{
        index = parseInt(Math.random() * servers.length);
    }

    data = servers[index];

    cb(null, data.id);
}

```

### glboalErrorHandler

```
//错误码对照表
var errCode = require('../../lib/errCode');
如下
"chat.chatHandler.send":{
	"error_code":1001 //策划填写错误提示
}

var GlobalHandler = function(app){
    this.app = app;
}
var br = require('bearcat');

GlobalHandler.prototype.globalHandler = function(err, msg, resp, session, next){
    var route = msg.route || msg.__route__ ;
    var charId = session.get('character_id');//登陆的时候 会把角色ID放到session中
    var mailModel = br.getBean('sendMailModel');
    var backendSessionService = this.app['backendSessionService'];

    if(err){
        if('no_redis_char' == err || 'redis_err' == err){
            backendSessionService.kickBySid(session.frontendId,session.id,function(err){
                if(err){
                    mailModel.sendMail(new Error('======>>>>kick char err').stack);
                }
            });
        }else if('update_redis_char' == err){
            //更新缓存错误
            backendSessionService.kickBySid(session.frontendId,session.id,function(err){
                if(!err){
                    global.app.rpc.redis.delete('char_' + charId ,function(err){
                        if(err){
                            mailModel.sendMail(new Error('redis del charInfo err').stack);
                        }
                    });
                }
            });
        }else if(!!errCode[route] && !!errCode[route][err]){//错误码中有对应的错误提示
            next(null,{status : errCode[route][err]});
        }else if(!!err['condition_lack']){
            next(null,{ status : 10000 ,conditionId : err['condition_lack'].conditionId });
        }else if('redis_err' == err) {
            mailModel.sendMail(err.stack);
        }else if(!!err.status){
            next(null,err);
        }else {
            console.log('此错误没有对应状态码!!!!!!!!',err);
            next(null,{status : 500});
        }
    }else{
        next(null,err);
    }
}

module.exports = GlobalHandler;

```



### gate 

```

var dispatcher = require('../../../util/dispatcher');
var pomelo = require('pomelo');
var bearcat = require('bearcat');
var async = require('async');
var util = bearcat.getBean('util');


var Handler = function() {
    this.$id = 'gateHandler';
    this.app = pomelo.app;
};

var handler = Handler.prototype;


handler.queryEntry = function(msg,session,next){
	var self = this;
	var connectors = self.app.getServersByType('connector');
    if(!connectors || connectors.length === 0) {
        next(null, {
            status: 500
        });
        return;
    }
    
    async.waterfall([
        function(cb){
        	//查看服务器是否处于停服状态 状态存储在redis缓存中 设置master时，delete掉
            global.app.redis.get('gameServerStatus',function(err,data) {
                if (!err && !data) {
                    next(null, {status: 430});
                    return;
                }
                cb(err);
            });
        },
        function(cb){
            global.app.rpc.chat.chatRemote.getOnlinePlayersNumber(null,function(err,data){
                console.log('服务器同时在线人数==',data,serverLimit);
                if(!err){
                	 //查看服务器在线人数是否超过限制
                    if(data >= serverLimit){
                        next(null,{ status : 888});
                        return;
                    }
                } else {
                    next(null,{ status : 500});
                    return;
                }
                cb();
            });
        }
    ], function (err) {
    	//分配一个connector服务器 ；算法是Math.random()*connectorServers.length
        var res = dispatcher.dispatch(session.id,connectors);
        next(null, {
            status: 0,
            host: res.host,
            port: res.clientPort
        });
    });
}

```









