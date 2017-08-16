# 生命周期回调

生命周期回调能够让开发者在不同类型的服务器生命周期中进行具体操作。提供的生命周期回调函数包括：beforeStartup，afterStartup，beforeShutdown，afterStartAll。其具体的功能说明如下：

### beforeStartup(app, cb) before application start components callback ####Arguments

* app - application object
* cb - callback function

### afterStartup(app, cb) after application start components callback ####Arguments

* app - application object
* cb - callback function
### beforeShutdown(app, cb) before application stop components callback ####Arguments

* app - application object
* cb - callback function
### afterStartAll(app) after all applications started callback ####Arguments

* app - application object

具体使用方法：在game-server/app/servers/某一类型服务器/ 目录下添加lifecycle.js文件，具体文件内容如下：

```

module.exports.beforeStartup = function(app, cb) {
	// do some operations before application start up
	cb();
};

module.exports.afterStartup = function(app, cb) {
	// do some operations after application start up
	cb();
};

module.exports.beforeShutdown = function(app, cb) {
	// do some operations before application shutdown down
	cb();
};

module.exports.afterStartAll = function(app) {
	// do some operations after all applications start up
};

```