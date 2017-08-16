# Session

```
frontend-session

session.bind(uid);
session.set('uid', uid);
session.set('roomId',roomId);

//下面两种方式 将会把settings 同步到backend-session

//1. pushAll 
session.pushAll(function(err){
    if(!!err){
        console.error('set rid for session service failed! error is : %j', err.stack);
    }
});

//2 
session.push('uid');
session.push('roomId');


backend-session
var uid = session.get("uid");
var roomId = session.get("roomId");

```