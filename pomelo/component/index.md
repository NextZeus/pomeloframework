# 组件

### add a component to pomelo

```
// components/HelloWorld.js
module.exports = function(app, opts) {
  return new HelloWorld(app, opts);
};

var DEFAULT_INTERVAL = 3000;

var HelloWorld = function(app, opts) {
  this.app = app;
  this.interval = opts.interval || DEFAULT_INTERVAL;
  this.timerId = null;
};

HelloWorld.name = '__HelloWorld__';

HelloWorld.prototype.start = function(cb) {
  console.log('Hello World Start');
  var self = this;
  this.timerId = setInterval(function() {
    console.log(self.app.getServerId() + ": Hello World!");
    }, this.interval);
  process.nextTick(cb);
}

HelloWorld.prototype.afterStart = function (cb) {
  console.log('Hello World afterStart');
  process.nextTick(cb);
}

HelloWorld.prototype.stop = function(force, cb) {
  console.log('Hello World stop');
  clearInterval(this.timerId);
  process.nextTick(cb);
}
```

每个component都定义start,afterStart, stop这些hook函数，供pomelo管理其生命周期时进行调用

### app.js加载组件
```
// app.js
var helloWorld = require('./app/components/HelloWorld');

app.configure('production|development', 'master', function() {
  app.load(helloWorld, {interval: 5000});
});

```

###  说明
> 定义的组件一般往外导出的是一个工厂函数，而不是一个对象。当app加载component时，如果是一个工厂函数，那么app会将自己作为上下文信息以及后面的
opts作为参数传递给这个函数，使用这个函数的返回值作为component对象。
> pomelo会先按照顺序执行完所有component的start后，才会按照顺序执行所有component的afterStart.