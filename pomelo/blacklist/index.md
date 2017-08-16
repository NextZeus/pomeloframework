# pomelo支持连接黑名单机制

在新版本的pomelo中连接服务器支持静态或者动态添加ip黑名单功能，服务端可以在连接服务器中增加黑名单对攻击的ip进行屏蔽

## 原理

connector每接受一个连接都会抛出一个连接事件, 这个事件中含有该连接的客户端IP. connector会捕获该连接事件, 
并调用用户传入的获取IP黑名单的函数, 如果该客户端IP在黑名单中, 则立刻将对应的socket断开. 以此来实现连接服务器的黑名单过滤功能.

## 使用方法

### 静态添加黑名单

使用时只需要向在connector的connectionConfig配置中传入一个获取IP黑名单的函数即可, 这个函数需要接受一个回调函数作为其参数, 该回调函数形如function(err, list) {...}. 在获取IP黑名单的函数内, 拿到IP黑名单时(该黑名单应为一维JS Array), 以类似于cb(null, self.list)的形式调用IP过滤回调函数，具体使用方法如下：

```
./game-server/app/util/blackList.js
... ...

var self = this;
self.blackList = ['192.168.100.1', '192.168.100.2'];
module.exports.blackListFun = function(cb) {
    cb(null, self.blackList);
};
... ...

```

```
./game-server/app.js
var blackList = require('./app/util/blackList');
    ... ...
    app.configure('production|development', function() {
    ... ...
    app.set('connectorConfig', {
        blacklistFun: blackList.blackListFun
    });
    ... ...
}

```

### 动态添加黑名单

动态添加黑名单可以通过pomelo-cli完成，其中运行输入具体ip或者正则表达式，具体命令如下：

```

blacklist 192.168.100.1
blacklist (([01]?d?d|2[0-4]d|25[0-5]).){3}([01]?d?d|2[0-4]d|25[0-5])

```