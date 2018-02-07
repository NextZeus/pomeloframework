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


## 源码解读

```

app.set('connectorConfig', {
    connector: pomelo.connectors.hybridconnector,
    heartbeat: 30,
    useProtobuf: true, //使用protobuf
    handshake: function (msg, cb) {
        cb(null, {});
    }
});

//pomelo/lib/components/connector.js

var Component = function(app,opts){
	if (opts.useProtobuf) {
   		app.load(pomelo.protobuf, app.get('protobufConfig'));
  	}
}









//pomelo/lib/components/protobuf.js

var protobuf = require('pomelo-protobuf');
var crypto = require('crypto');

this.setProtos(Constants.RESERVED.SERVER, path.join(app.getBase(), this.serverProtosPath));
this.setProtos(Constants.RESERVED.CLIENT, path.join(app.getBase(), this.clientProtosPath));

setProtos method :

serverProtos = protobuf.parse(require(path));
clientProtos = protobuf.parse(require(path));
var protoStr = JSON.stringify(this.clientProtos) + JSON.stringify(this.serverProtos);
this.version = crypto.createHash('md5').update(protoStr).digest('base64');

protobuf.init({encoderProtos:this.serverProtos, decoderProtos:this.clientProtos});


//接下来就进入protobuf--->>pomelo-protobuf

//parse method

var Parser = module.exports;

/**
 * [parse the original protos, give the paresed result can be used by protobuf encode/decode.]
 * @param  {[Object]} protos Original protos, in a js map.
 * @return {[Object]} The presed result, a js object represent all the meta data of the given protos.
 */
Parser.parse = function(protos){
	var maps = {};
	for(var key in protos){
		maps[key] = parseObject(protos[key]);
	}

	return maps;
};

/**
 * [parse a single protos, return a object represent the result. The method can be invocked recursively.]
 * @param  {[Object]} obj The origin proto need to parse.
 * @return {[Object]} The parsed result, a js object.
 */
function parseObject(obj){
	var proto = {};
	var nestProtos = {};
	var tags = {};

	for(var name in obj){
		var tag = obj[name];
		var params = name.split(' ');

		switch(params[0]){
			case 'message':
				if(params.length !== 2){
					continue;
				}
				nestProtos[params[1]] = parseObject(tag);
				continue;
			case 'required':
			case 'optional':
			case 'repeated':{
				//params length should be 3 and tag can't be duplicated
				if(params.length !== 3 || !!tags[tag]){
					continue;
				}
				proto[params[2]] = {
					option : params[0],
					type : params[1],
					tag : tag
				};
				tags[tag] = params[2];
			}
		}
	}

	proto.__messages = nestProtos;
	proto.__tags = tags;
	return proto;
}

//初始化protobuf
Protobuf.init = function(opts){
	//On the serverside, use serverProtos to encode messages send to client
	encoder.init(opts.encoderProtos);

	//On the serverside, user clientProtos to decode messages receive from clients
	decoder.init(opts.decoderProtos);
};

//又延伸到encoder decoder

客户端需要单独写protobuf encode decode


```




















