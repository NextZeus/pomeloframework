# pomelo+bearcat

### context.json
```
{
	"name": "app-name",
	"beans": [],
	"scan": "app" 
}

```

### 启动

```
var contextPath = require.resolve('./context.json');
bearcat.createApp([contextPath]);

bearcat.start(function() {
	// app configure
	// start app
	app.start();
});

```

### 单个模块

```
//单个模块使用bearcat

var Model = function(){
    this.$id = "model";
}

bearcat.module(Model);

```