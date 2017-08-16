# pomelo protobuf

在pomelo 0.9版本中提供了对decodeIO的protobuf的支持

## 客户端

使用最新的pomelo-jsclient-websocket, 同时在客户端添加命名为pomelo-decodeIO-protobuf的component，并将其挂载到window对象下。对应的component.js如下所示:

```

{
  "name": "boot",
  "description": "Main app boot component",
  "dependencies": {
    "component/emitter":"master",
    "NetEase/pomelo-protocol": "master",
    "pomelonode/pomelo-decodeIO-protobuf": "master",
    "pomelonode/pomelo-jsclient-websocket": "master",
    "component/jquery": "*"
  },
  "scripts": ["index.js"]
}

```

对应的index.js如下所示:

```
var Emitter = require('emitter');
window.EventEmitter = Emitter;

var protocol = require('pomelo-protocol');
window.Protocol = protocol;

var protobuf = require('pomelo-decodeIO-protobuf');
window.decodeIO_protobuf = protobuf; 

var pomelo = require('pomelo-jsclient-websocket');
window.pomelo = pomelo;

var jquery = require('jquery');
window.$ = jquery;

```

## 服务端

在服务端需要使用pomelo-protobuf-plugin，并在app.js中使用对应的插件，具体配置如下：

```
app.configure('production|development', function() {
	app.use(protobuf, {
		protobuf: {

		}
	});
});

```

### 注意事项

* pomelo原有的protobuf和decodeIO-protobufjs不能同时使用，即不能同时使用pomelo-protobuf-plugin插件并在前端服务器开启useProtobuf
* 考虑到与原有的protobuf保持一致，pomelo 0.9版本中支持的decodeIO-protobuf同样采用serverProtos.json和clientProtos.json，不支持decodeIO-protobufjs中的*.proto格式，对于*.proto格式可以采用decodeIO-protobufjs提供的命令行工具转换成json格式
* 具体的使用示例可以参考[lordofpomelo](https://github.com/NetEase/lordofpomelo/tree/decodeIO_protobuf) decodeIO-protobuf分支。