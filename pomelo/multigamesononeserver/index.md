## 同一个服务器部署多款pomelo开发的游戏 有的服务器推送消息失败

## 配置问题

1. master.json和servers.json配置中的id, port要保持唯一。

## 解决方法
1. 修改master.json和servers.json配置中的id,port 保证配置的server唯一。

## 了解

在看源码的过程中，发现最终推送消息的地方: zmq.socket.send
zmq创建的socket.send方法已经执行了， 但是执行失败了 也没有抛出任何的错误。


