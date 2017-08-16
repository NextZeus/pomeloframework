# Router 路由函数

## 作用
Router的作用 是为所有的客户端请求指定目标服务器。 开发者可以自己针对不同的服务器，定义不同的路由规则

### example

```

app.configure('development|production','', function(){
    var routeUtil = require(process.cwd() + '/app/util/routeUtil.js');
    app.route('chat', routeUtil.chat);
});


// routeUtil.js
//return target server id 
exports.chat = function(session, msg, app, callback){
    var chatServers = app.getServersByType('chat');

    if(!chatServers || !chatServers.length){
        callback(new Error('can`t find chat servers!'));
        return;
    }
    var server = dispatcher.dispatch(session.rid, chatServers);
    callback(null, server.id);
}

```

