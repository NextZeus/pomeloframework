# 循环任务篇

## 使用情景
```
棋牌：洗牌 发牌 每个人出牌 结算发奖励 再到洗牌。。。
麻将：洗牌 发牌 出牌＋摸牌 结算法奖励 再到洗牌。。。
赛马，轮盘赌：下注 赛马过程 结算发奖

像以上这种有不同阶段的游戏，每个阶段的变化，都会需要服务器主动告知客户端。 这个时候服务器端就需要有个环形的任务一直在默默的工作着，保证整个游戏循环能够正常的切换。

```

## 方案
1. 利用setTimeout 定时去执行每个阶段
2. 利用pomelo-schedule第三方库
3. 使用node-schedule第三方库

## Demo

```javascript
<!--1 setTimeout  -->

function bet(){
    setTimeout(game,6000);
}

function game(){
    setTimeout(settlement,5000);
}

function settlement(){
    setTimeout(bet,5000);
}

setTimeout(bet,0);

<!--2 pomelo-schedule  -->

function bet(){
    schedule.schedulerJob({
        start:  Date.now() + PHASE_TIME.GAME
    },game,{NAME:    '开始游戏'});
}

function game(){
    schedule.schedulerJob({
        start:  Date.now() + PHASE_TIME.REWARD
    },settlement,{NAME:    '开始游戏'});
}

function settlement(){
    //TODO
}

var PHASE_TIME = {
    BET:10000,
    GAME:   8000,
    REWARD:5000
}

schedule.schedulerJob({
    start:  Date.now(),
    period: PHASE_TIME.BET + PHASE_TIME.GAME + PHASE_TIME.REWARD
},bet,{NAME:    '开始下注'});


```