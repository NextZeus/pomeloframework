# pomelo-schedule

```
/**
 * Created by lixiaodong on 17/3/16.
 */
var schedule = require("pomelo-schedule");

function getTime() {
    return new Date().toISOString().
    replace(/T/, ' ').
    replace(/\..+/, '')
}

var xiazhu = function(data){
    console.log("run Job :" + data.name,' time: ', getTime());
    schedule.scheduleJob({start:Date.now()+15000}, gamestart, {name: '开始游戏'});
}

function gamestart(data) {
    console.log("run Job :" + data.name,' time: ', getTime());
    schedule.scheduleJob({start:Date.now()+15000}, gameend, {name: '开始发奖'});
}

function gameend(data) {
    console.log("run Job :" + data.name,' time: ', getTime());
}

console.log('game start time: ', getTime())
schedule.scheduleJob({start:Date.now(), period:45000}, xiazhu, {name: '开始下注'});

//result
rgame start time:  2017-03-16 10:05:46
run Job :开始下注  time:  2017-03-16 10:05:46
run Job :开始游戏  time:  2017-03-16 10:06:01
run Job :开始发奖  time:  2017-03-16 10:06:16
run Job :开始下注  time:  2017-03-16 10:06:31
run Job :开始游戏  time:  2017-03-16 10:06:46

```

