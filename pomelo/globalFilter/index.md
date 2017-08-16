# globalFilter

```
//globalFilter.js
var Filter = function(){
}

Filter.prototype.before = function (msg,session,next) {
    //todo 服务器是否启动
    console.warn('before-filter');
    next();
}

Filter.prototype.after = function (err,msg,session,resp,next) {
    console.warn('after-filter');
    next();
}

module.exports = function(){
    return new Filter();
}

//app.js配置
app.configure('production|development', function(){
    var globalFilter = require("./app/filter/globalFilter");
    app.globalFilter(globalFilter());
});

```