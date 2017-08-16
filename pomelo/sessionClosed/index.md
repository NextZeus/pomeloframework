#session closed

```
session on [connector-server-1] is closed with session id: 1

```

## protobuf设置不对导致的

```
app.set('connectorConfig', {
    connector : pomelo.connectors.hybridconnector,
    heartbeat : 30,
    useDict: true,
    useProtobuf : true,
    disconnectOnTimeout: true,
    handshake: function (msg, cb) {
        // console.log('connector handshake msg: ', msg);
        cb(null, {status:200, sys:{heartbeat:30}});
    }
});

源码 : /pomelo/lib/connectors/common/coder.js
var encodeBody = function(server, route, msgBody) {
    // encode use protobuf 设置protobuf之后 就会走这里
    //console.log('encode use protobuf==>>',!!server.protobuf,!!server.protobuf.getProtos().server[route],msgBody);
    if(!!server.protobuf && !!server.protobuf.getProtos().server[route]) {
        msgBody = server.protobuf.encode(route, msgBody);
    } else if(!!server.decodeIO_protobuf && !!server.decodeIO_protobuf.check(Constants.RESERVED.SERVER, route)) {
        msgBody = server.decodeIO_protobuf.encode(route, msgBody);
    } else { //没有设置protobuf
        msgBody = new Buffer(JSON.stringify(msgBody), 'utf8');
    }
    return msgBody;
};
```