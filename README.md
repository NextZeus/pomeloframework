# 目的
* 学而时习之 温故而知新
* 实战经验分享
## 文章系列
[pomelo游戏服务器开发系列](http://nextzeus.github.io/pomelo/)


### pomelo-admin connect game-admin 是指定的吗?

可以看下pomelo-admin源码， connect方法 ，在源码里可以看到 第一个参数id 赋值给了clientId。

```

var Client = function(opt) {
	this.id = "";
	this.reqId = 1;
	this.callbacks = {};
	this.listeners = {};
	this.state = Client.ST_INITED;
	this.socket = null;
	opt = opt || {};
	this.username = opt['username'] || "";
	this.password = opt['password'] || "";
	this.md5 = opt['md5'] || false;
};

Client.prototype = {
	connect: function(id, host, port, cb) {
		this.id = id;
		var self = this;

		console.log('try to connect ' + host + ':' + port);
		this.socket = new MqttClient({
			id: id
		});

		this.socket.connect(host, port);
    },,,
}

连接到服务器之后，服务器会显示一条log，可以看到clientId 就是'game-admin'.

```

这个值是任意的字符串. connect 主要关键参数是username password host port, 只要保证这四个参数正确，就可以连接到 pomelo服务器。

### pomelo-admin-web 的onlineUser模块 可以显示的玩家信息是怎么得到的

首先说一下这些信息存储在哪里 以及 什么时候存储的

1. pomelo/lib/components/connection.js  __connection__ 组件负责维护登录的玩家连接信息
2. pomelo/lib/common/service/connectionService.js 这里会存储一些登录玩家计数信息

```

var Service = function(app) {
  this.serverId = app.getServerId();
  this.connCount = 0;
  this.loginedCount = 0;
  this.logined = {};
};

/**
 * connection statistics service
 * record connection, login count and list
 */
var Service = function(app) {
  this.serverId = app.getServerId();
  this.connCount = 0;
  this.loginedCount = 0;
  this.logined = {};
};

module.exports = Service;

var pro = Service.prototype;


/**
 * Add logined user.
 *
 * @param uid {String} user id
 * @param info {Object} record for logined user
 */
pro.addLoginedUser = function(uid, info) {
  if(!this.logined[uid]) {
    this.loginedCount++;
  }
  info.uid = uid;
  this.logined[uid] = info;
};

/**
 * Update user info.
 * @param uid {String} user id  保存一些玩家信息 admin获取的onlineUser信息
 * @param info {Object} info for update.
 */
pro.updateUserInfo = function(uid, info) {
    var user = this.logined[uid];
    if (!user) {
        return;
    }

    for (var p in info) {
        if (info.hasOwnProperty(p) && typeof info[p] !== 'function') {
            user[p] = info[p];
        }
    }
};

/**
 * Get statistics info  获取玩家在线信息
 *
 * @return {Object} statistics info
 */
pro.getStatisticsInfo = function() {
  var list = [];
  for(var uid in this.logined) {
    list.push(this.logined[uid]);
  }

  return {serverId: this.serverId, totalConnCount: this.connCount, loginedCount: this.loginedCount, loginedList: list};
};



//登录的时候 将玩家信息存储到__connection__组件中
var connectionService = pomelo.app.components.__connection__;
connectionService.updateUserInfo(uid,playerInfo);


```
## poemlo start 后面有-D 是后台运行么？比方关闭shell 端口还在？

```
推荐你使用 pomelo help , 

$: pomelo start --help

$ pomelo start --help

  Usage: start [options]

  Options:

    -h, --help                    output usage information
    -e, --env <env>               the used environment
    -D, --daemon                  enable the daemon start
    -d, --directory, <directory>  the code directory
    -t, --type <server-type>,     start server type
    -i, --id <server-id>          start server id

```

具体的help 在 pomelo/bin/pomelo.js中 ;
我只能帮你到这了。。。

## pomelo统计在线人数

详情请看源码: pomelo/lib/service/connectionService.js 

## pomelo  + wss + nginx

由于自己的服务器有gate , connector 两种front服务器，所有就选择这样配置。

```
app.configure('production|development', 'connector|gate', function() {

  app.set('connectorConfig',

    {
      connector : pomelo.connectors.hybridconnector,
      heartbeat : 3,
      useDict : true,
      useProtobuf : true,
      ssl: {
        type: 'wss',
          key: fs.readFileSync('./keys/server.key'), //根据域名产生的证书文件
          cert: fs.readFileSync('./keys/server.crt'),
      }
    });
});

```

### nginx 配置wss

- nginx1.3+默认支持wss连接，基本上不需要特殊的配置

```
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

server {
	listen       443 ssl;
	server_name  pomelo.game.com;

	ssl on;
	ssl_certificate ./game.com.crt;
	ssl_certificate_key ./game.com.key;
	ssl_session_timeout 5m;
	ssl_protocols SSLv2 SSLv3 TLSv1;
	ssl_ciphers HIGH:!aNULL:!MD5;
	ssl_prefer_server_ciphers on;

    access_log  ./logs/https.pomelo.game.com.access.log main;

	location / {
	    index  index.html index.htm;
	}

	error_page   500 502 503 504  /50x.html;
	location = /50x.html {
		root   html;
	}
}

```

### 客户端访问
```
url:	'wss://pomelo.game.com:3014'
```

## dump memory 
1. install pomelo-cli2 -g
2. install heapdump at /usr/lib/node_modules/pomelo-cli2/ 
3. use gate-server-1
4. dump memory /home/node/dump --force

#### dump memory 遇到的Error 
1. pomelo-admin need heapdump 
solution: npm install heapdump -s at /usr/lib/node_modules/pomelo-cli2/ 
2. 需要gcc version 4.8+
