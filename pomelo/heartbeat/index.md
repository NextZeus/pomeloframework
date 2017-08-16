#heartbeat

使用pomelo2.2.5版本时，设置心跳如下：

```
var heartbeat = 10;
app.set('connectorConfig', {
    connector : pomelo.connectors.hybridconnector,
    heartbeat : heartbeat,
    timeout:    heartbeat * 2,
    // disconnectOnTimeout: true,
    handshake: function (msg, cb) {
        // console.log('connector handshake msg: ', msg);
        cb(null, {code:200, sys:{heartbeat:heartbeat}});
    }
});

```

结果总是提示
> client 1 timeout 


### 源码修改 this 都替换成了 self 

```
var Package = require('pomelo-protocol').Package;
var logger = require('pomelo-logger').getLogger('pomelo', __filename);

/**
 * Process heartbeat request.
 *
 * @param {Object} opts option request
 *                      opts.heartbeat heartbeat interval
 */
var Command = function(opts) {
    opts = opts || {};
    this.heartbeat = null;
    this.timeout = null;
    this.disconnectOnTimeout = opts.disconnectOnTimeout;

    if(opts.heartbeat) {
        this.heartbeat = opts.heartbeat * 1000; // heartbeat interval
        this.timeout = opts.timeout * 1000 || this.heartbeat * 2;      // max heartbeat message timeout
        this.disconnectOnTimeout = true;
    }

    this.timeouts = {};
    this.clients = {};
};

module.exports = Command;

Command.prototype.handle = function(socket) {
    var self = this;

    console.warn("heart handle socket.id %s", socket.id);
    if(!self.heartbeat) {
        // no heartbeat setting
        return;
    }

    console.warn('heartbeat clients : ', this.clients);

    if(!self.clients[socket.id]) {
        // clear timers when socket disconnect or error
        console.log("register client socket.id: %s", socket.id);
        self.clients[socket.id] = 1;
        socket.once('disconnect', clearTimers.bind(null, self, socket.id));
        socket.once('error', clearTimers.bind(null, self, socket.id));
    }

    // clear timeout timer
    if(self.disconnectOnTimeout) {
        console.warn('begin clear timeout socket.id: %s', socket.id);
        self.clear(socket.id);
    }

    socket.sendRaw(Package.encode(Package.TYPE_HEARTBEAT));

    if(self.disconnectOnTimeout) {
        console.log('reset timeoutId socket.id: %s', socket.id);
        self.timeouts[socket.id] = setTimeout(function() {
            console.error("socket closed socket.id: %s", socket.id);
            logger.info('client %j heartbeat timeout.', socket.id);
            socket.disconnect();
        }, self.timeout);
    }
};

Command.prototype.clear = function(id) {
    var self = this;
    var tid = self.timeouts[id];
    if(tid) {
        clearTimeout(tid);
        delete self.timeouts[id];
    }
};

var clearTimers = function(self, id) {
    delete self.clients[id];
    var tid = self.timeouts[id];
    console.warn("clearTimer id: %s tid:", id, !!tid);
    if(tid) {
        clearTimeout(tid);
        delete self.timeouts[id];
        console.log('left timeouts: ',Object.keys());
    }
};


```


### 参考
[nodejs-cleartimeout-not-working](https://codedump.io/share/Gw1Krv0K4Ou8/1/nodejs-cleartimeout-not-working)