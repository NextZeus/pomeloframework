# globalErrorHandler 全局错误处理

```
//errorCode.js 错误码模块
var br = require("bearcat");

var ErrorCode = function () {
    this.$id = "errorCode";
}

var code = {
    "data.dataHandler.attack":{
        "error1":    1001
    }
}

ErrorCode.prototype.getErrorCode = function (route, name) {
    return code[route][name] || 500;
}

br.module(ErrorCode);


//errorHandler.js

var GlobalHandler = function () {

}

GlobalHandler.prototype.globalHandler = function (err,msg,resp,session,next) {
    var route = msg.route || msg.__route__;
    var errorCode = bearcat.getBean("errorCode");

    console.warn('globalHandler-----error', err);
    if(!!err){
        return next(null,{code: errorCode.getErrorCode(route,err)});
    }

    next();
}

module.exports = GlobalHandler;


//app.js 配置
app.configure('production|development', function(){
    var globalErrorHandler = require("./app/globalHandler/errorHandler");
    var errorHandler = new globalErrorHandler();
    app.set("globalErrorHandler", errorHandler.globalHandler);
    app.set("errorHandler", errorHandler.globalHandler);
});

```