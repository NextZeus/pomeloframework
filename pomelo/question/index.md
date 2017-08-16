# 问题答疑

1. 如何获取在线玩家的信息
2. 如何推送消息
3. 同一台服务器部署多款pomelo开发的游戏，服务器推送消息出现问题
4. 客户端连接connector服务器失败
5. rpc connection timeout
6. channelService.broadcast Error:[broadcast] fail to push message to serverId: connector-server-1, err:Error: fail to send message for encode result is empty

#### 解答
1. 问题1 请看[onlineUser](../onlineUser/)
2. 问题2 请看[pushMessage](../pushMessage/)
3. 问题3 请看[multigamesononeserver](../multigamesononeserver)
4. 有可能是connector服务器的port访问权限没有开放 ［实际情况线上是遇到过这种情况的］
5. 一般是有回调函数，没有执行.
6. 打印下route, msg. 看下是否有值是undefined的