
# Connector

在客户端通信的时候， pomelo目前提供了hybirdconnector和sioconnector，其中hybirdconnector支持tcp,websocket,; sioconnector支持socket.io;
实际编程中，我们可能需要定制自己的connector,pomelo提供了接口

## 内建的connector分析
以sioconnector为例分析

sioconnectot构造函数里需要三个参数 host, port, opts
host,port是要监听socket绑定的
会监听以下几个事件：start, disconnect, error, message

### encode/decode
服务器端收到客户端的请求message或者需要给客户端发送回应或者推送消息时，pomelo会使用connector的decode函数对数据进行解码。

客户端请求信息：
```
{
    id: <requestId>, //客户端产生的请求ID 不同的session 请求ID是互不干涉的[notify类型request reqId null]
    route:  <handlerRoute>, //请求位置 chat.chatHandler.send 
    body:   <requestBody> //请求参数
}    
```

服务器端给客户端的响应活着服务器端的推送消息，会使用connector的encode进行编码，encode(reqId, route, msg)
如果是推送消息，reqId为null； 

decode/encode可通过opts配置 优先使用opts指定的decode/encode


```
//connector 配置
app.set('connectorConfig', {
    connector: pomelo.connectors.hybirdconnector,
    encode : <encode func>, //optional
    decode : <decode func>, //optional
    heartbet:30
});
```
