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
 * Increase connection count
 */
pro.increaseConnectionCount = function() {
  this.connCount++;
};

/**
 * Remote logined user
 *
 * @param uid {String} user id
 */
pro.removeLoginedUser = function(uid) {
  if(!!this.logined[uid]) {
    this.loginedCount--;
  }
  delete this.logined[uid];
};

/**
 * Decrease connection count 玩家离线 
 *
 * @param uid {String} uid
 */
pro.decreaseConnectionCount = function(uid) {
  if(this.connCount) {
    this.connCount--;
  }
  if(!!uid) {
    this.removeLoginedUser(uid);
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


```
