# Pomelo High Availability [高可用]

## 高可用性
[High_availability](https://en.wikipedia.org/wiki/High_availability)

High availability is a characteristic of a system, which aims to ensure an agreed level of operational performance, usually uptime, for a higher than normal period.

高可用性HA（High Availability）指的是通过尽量缩短因日常维护操作（计划）和突发的系统崩溃（非计划）所导致的停机时间，以提高系统和应用的可用性

## pomelo master 高可用

##### 参考lordofpomelo master高可用

* [下载zookeeper](http://www-eu.apache.org/dist/zookeeper/)
* 解压到/Users/xxx目录下 [随意] 比如解压到/Users/xxx/zookeeper目录
* 配置conf/zoo.cfg, 可直接改名zoo_sample.cfg为zoo.cfg
* 修改dataDir=/pomelo/master [这个目录是存储数据的目录，手动创建，参考lordofpomelo] 其他默认即可
* bin/zkServer.sh start ［启动zk服务］ [zookeeper/bin目录下 有zkServer.sh zkCli.sh]
* bin/zkCli.sh 登录到zookeeper, ls / 查看有哪些目录 
* 切换到lordofpomelo/game-server 
* 执行 node scripts/createZKMasterhaNode.js 创建/pomelo/master Node[节点]
* 修改game-server app.js配置插件

- app.js配置

```
var masterhaPlugin = require('pomelo-masterha-plugin');

app.use(masterhaPlugin, {
    zookeeper:
    {
        server: 'localhost:2181',
        path: '/pomelo/master', [刚才创建的节点]
        username:  "pomelo",
        password:  "pomelo"
    }
});

```

➜  game-server git:(master) ✗ node scripts/createZKMasterhaNode.js 
这一步是创建/pomelo/master 目录
创建成功会提示

Connected to the server.
Node: /pomelo is created successfully.
Node: /pomelo/master is created successfully.

然后

➜  game-server git:(master) ✗  pomelo start [ 启动game-server ]

➜  game-server git:(master) ✗  pomelo masterha [启动备用master ]


```

