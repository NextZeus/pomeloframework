# pomelo schedule job  | pomelo定时任务

[FROM](https://www.cnblogs.com/brucemengbm/p/6816788.html)

## Steps 步骤

1. add `cron` directory at your special server directory , for example `game`
2. create a new `youCronFileName.js` file with code like below:

```

module.exports = function(app) {
  return new Cron(app);
};
var Cron = function(app) {
  this.app = app;
};

Cron.prototype.yourJobMethod = function() {
    // .... your job logic
};

```

3. create `crons.json` file at `game-server/config` directory with configuration like below:

```

{
    "development":{
        "game": [
            {"id": 1, "time": "0 0/1 * * * *", "action": "youCronFileName.yourJobMethod"}
        ]
    },
    "production":{
        "game": [
            {"id": 1, "time": "0 0/1 * * * *", "action": "youCronFileName.yourJobMethod"}
        ]
    }
}

```

##  Note

1. job time 

```

*     *     *     *   *    *        command to be executed
-     -     -     -   -    -
|     |     |     |   |    |
|     |     |     |   |    +----- day of week (0 - 6) (Sunday=0)
|     |     |     |   +------- month (0 - 11)
|     |     |     +--------- day of month (1 - 31)
|     |     +----------- hour (0 - 23)
|     +------------- min (0 - 59) */n: every n minutes execute the job
+------------- second (0 - 59)


```
2. job id

- id is an optional field, if no id , this job will run at all the same type of the current server. 