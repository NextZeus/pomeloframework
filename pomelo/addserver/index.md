# add a new server

比如添加data-server
1. adminServer.json 中添加 {type: "data", token:""}
2. routeUtil.js 配置data服务器的路由函数
3. app.js 注册路由函数 app.route('data', routeUtil.data);
