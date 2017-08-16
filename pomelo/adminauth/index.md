# 服务器连接认证机制

服务器与master之间的连接需要进行认证以提高服务的安全性，目前在pomelo-admin中提供了一个简单的服务器认证，
可以看[admin auth](https://github.com/NetEase/pomelo-admin/blob/master/lib/util/utils.js#L117)
使用连接认证，需要在 config 目录下添加 adminServer.json 的文件

```
[{
    "type": "connector",
    "token": "agarxhqb98rpajloaxn34ga8xrunpagkjwlaw3ruxnpaagl29w4rxn"
}, {
    "type": "chat",
    "token": "agarxhqb98rpajloaxn34ga8xrunpagkjwlaw3ruxnpaagl29w4rxn"
},{
    "type": "gate",
    "token": "agarxhqb98rpajloaxn34ga8xrunpagkjwlaw3ruxnpaagl29w4rxn"
}
]

```

**type** 是serverType, **token** 是一个字符串，你可以自己生成
你可以通过自己定义的认证还是来完成认证的工作
具体情况可以参考 [admin auth server](https://github.com/NetEase/pomelo-admin#server-master-auth)