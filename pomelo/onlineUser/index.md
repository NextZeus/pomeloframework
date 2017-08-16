# OnlineUser
## 如何在线玩家的信息

```

var async = require('async');

module.exports = function (opts) {
    return new Module(opts);
};

module.exports.moduleId = 'onlineUser';

var Module = function (opts) {
    opts = opts || {};
    this.app = opts.app;
};

Module.prototype.monitorHandler = function (agent, msg, cb) {
    var app = this.app;

    var connection = app.components.__connection__;
    if (!connection) {
        cb({
            serverId: agent.id,
            body    : 'error'
        });
        return;
    }

    var connectionService = this.app.components.__connection__;
	if(!connectionService) {
		// logger.error('not support connection: %j', agent.id);
		return;
	}
    var info = connectionService.getStatisticsInfo();
    console.log('serverId: ' ,agent.id, ' info: ', info);
    cb(null, {
        serverId: agent.id,
        body    : info
    });
};

Module.prototype.clientHandler = function (agent, msg, cb) {
    var app = this.app;
    var servers = app.getServersByType('connector');
    var onLineUser = {};
    if(servers){
        async.mapSeries(servers,function(server,callback){
            //请求每个connector server 统计每个服务器的在线玩家信息
            agent.request(server.id, module.exports.moduleId, msg, function(err,info){
                if(err){
                    cb(null,{body : 'err'});
                    return;
                }
                delete info.body.loginedList;
                onLineUser[server.id] = info.body;
                callback();
            });
        },function(err,res){
            console.log('onLineUser: ', onLineUser);
            cb(null,{
                body : onLineUser
            });
        });
    }else{
        cb(null,{boyd : onLineUser});
    }
};


```

### 检测某个用户是否在线

```
//在frontend-server 获取在线状态
var sessionService = global.app.get('sessionService');
	
if (!!sessionService.getByUid(uid)) {
    return next("DUPLICATE_LOGIN");
}

//在backend-server 获取在线状态
var player = {id:1,name:'li',uid:1001, online:false};
var uid = player.uid;//玩家账户ID
var backendSessionService = global.app.get('backendSessionService');
var connectorServerId = session.get('fid');//session中保存的connector 服务器ID

backendSessionService.getByUid(connector.id,uid,function(err,sessions){
    if(!!sessions){
        //一个角色只有一个session
        var session = sessions[0];
        if(session.uid == uid){
            player.online = true;//在线
        }
    }
});                   

```