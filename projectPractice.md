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










