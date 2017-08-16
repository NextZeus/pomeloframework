# 定时任务

用户能够通过配置文件或者pomelo-cli的命令addCron和removeCron对定时任务进行动态调度。 定时任务是针对具体服务器而言，例如需要在chat服务器中配置定时任务：

首先在game-server/app/servers/chat目录下增加cron目录，在game-server/app/servers/chat/cron目录下编写具体的执行的任务的代码chatCron.js，例如：

```
module.exports = function(app) {
  return new Cron(app);
};
var Cron = function(app) {
  this.app = app;
};
var cron = Cron.prototype;

cron.sendMoney = function() {
  console.log('%s server is sending money now!', this.app.serverId);
};

```

然后在game-server/config/目录下增加定时任务配置文件crons.json，具体配置文件如下所示：

```
{
   "development":{
        "chat":[
             {"id":1, "time": "0 30 10 * * *", "action": "chatCron.sendMoney"},
             {"id":2, "serverId":"chat-server-1", "time": "0 30 10 * * *", "action": "chatCron.sendMoney"}
        ]
    },
    "production":{
        "chat":[
             {"id":1, "time": "0 30 10 * * *", "action": "chatCron.sendMoney"},
             {"id":2, "serverId":"chat-server-1", "time": "0 30 10 * * *", "action": "chatCron.sendMoney"}
        ]
  }
}

```

在配置文件crons.json中，id是定时任务在具体服务器的唯一标识，且不能在同一服务器中重复；time是定时任务执行的具体时间，时间的定义跟linux的定时任务类似，一共包括7个字段，每个字段的具体定义如下：

```
*     *     *     *   *    *        command to be executed
-     -     -     -   -    -
|     |     |     |   |    |
|     |     |     |   |    +----- day of week (0 - 6) (Sunday=0)
|     |     |     |   +------- month (0 - 11)
|     |     |     +--------- day of month (1 - 31)
|     |     +----------- hour (0 - 23)
|     +------------- min (0 - 59)
+------------- second (0 - 59)

```

0 30 10 * * * 这就代表每天10:30执行相应任务；
serverId是一个可选字段，如果有写该字段则该任务只在该服务器下执行，如果没有该字段则该定时任务在所有同类服务器中执行；
action是具体执行任务方法，chatCron.sendMoney则代表执行game-server/app/servers/chat/cron/chatCron.js中的sendMoney方法。

通过pomelo-cli的addCron和removeCron命令可以动态地增加和删除定时任务，其中addCron的必要参数包括：id,action,time；removeCron的必要参数包括：id；serverId和serverType是两者选其一即可。例如：

```

addCron id=8 'time=0 30 11 * * *' action=chatCron.sendMoney serverId=chat-server-3

removeCron id=8

```