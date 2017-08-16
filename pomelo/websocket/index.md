# pomelo websocket 支持自动重练

在pomelo 0.9版本中，pomelo-jsclient-websocket 支持自动重连。重连发生在连接断开后的5s后，在重连失败后下次重连的时间将是上次重连时间的2倍；所以重连时间依次为5s, 10s，20s,依次类推。默认最大的重连次数是10次，该参数可以在连接初始化过程中进行配置。

```
//设置客户端重连

pomelo.init({
		host: 127.0.0.1,
		port: 3050,
		reconnect: true
	}, function() {

});


//设置客户端重连最大次数

pomelo.init({
		host: 127.0.0.1,
		port: 3050,
		reconnect: true，
  		maxReconnectAttempts： 20
	}, function() {

});


```