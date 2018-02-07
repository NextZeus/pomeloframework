由于自己的服务器有gate , connector 两种front服务器，所有就选择这样配置。

```
app.configure('production|development', 'connector|gate', function() {

  app.set('connectorConfig',

    {
      connector : pomelo.connectors.hybridconnector,
      heartbeat : 3,
      useDict : true,
      useProtobuf : true,
      ssl: {
        type: 'wss',
          key: fs.readFileSync('./keys/server.key'),
          cert: fs.readFileSync('./keys/server.crt'),
      }
    });
});

```

### nginx 配置不跨服连接 wss

- nginx1.3+默认支持wss连接，基本上不需要特殊的配置

```
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

server {
	listen       443 ssl;
	server_name  pomelo.game.com;

	ssl on;
	ssl_certificate ./game.com.crt;
	ssl_certificate_key ./game.com.key;
	ssl_session_timeout 5m;
	ssl_protocols SSLv2 SSLv3 TLSv1;
	ssl_ciphers HIGH:!aNULL:!MD5;
	ssl_prefer_server_ciphers on;

    access_log  ./logs/https.pomelo.game.com.access.log main;

	location / {
	    index  index.html index.htm;
	}

	error_page   500 502 503 504  /50x.html;
	location = /50x.html {
		root   html;
	}
}

```

### 客户端访问

url:	'wss://pomelo.game.com:3014'

## nginx域名转发到其他服务器

#### 思路

不同于域名:端口号直连方式，需要配置nginx,并监听gate, connector多个clientPort 
nginx负责监听这些clientPort, proxy_pass到对应的upstream。

#### 注意
pomelo wss连接，nginx配置不支持降级，即proxy_pass必须使用https

```

逻辑服务器ip: a.b.c.d[内网IP或者公网IP都可以，如果可以内网访问，最好用内网IP]

upstream gate {
    server a.b.c.d:3014;
}

upstream connector01 {
    server a.b.c.d:3050;
}

upstream connector02 {
    server a.b.c.d:3051;
}

server {
        listen   3014;
        server_name  pomelo.game.com;

        ssl on;
        ssl_certificate /etc/nginx/ssl/pomelo.game.com.crt;
        ssl_certificate_key /etc/nginx/ssl/pomelo.game.com.key;
        ssl_session_timeout 5m;
        ssl_protocols SSLv2 SSLv3 TLSv1;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        access_log  /etc/nginx/logs/pomelo.game.com.access.log main;
        error_log /etc/nginx/logs.pomelo.game.com.error.log;

        location / {
            # WebSocket support (nginx 1.4)
            proxy_pass                      https://gate;

            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
}

server {
        listen   3050;
        server_name  pomelo.game.com;

        ssl on;
        ssl_certificate /etc/nginx/ssl/pomelo.game.com.crt;
        ssl_certificate_key /etc/nginx/ssl/pomelo.game.com.key;
        ssl_session_timeout 5m;
        ssl_protocols SSLv2 SSLv3 TLSv1;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        access_log  /etc/nginx/logs/pomelo.game.com.access.log main;
        error_log /etc/nginx/logs.pomelo.game.com.error.log;

        location / {
            # WebSocket support (nginx 1.4)
            proxy_pass                      https://connector01;

            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
}

server {
        listen   3051;
        server_name  pomelo.game.com;

        ssl on;
        ssl_certificate /etc/nginx/ssl/pomelo.game.com.crt;
        ssl_certificate_key /etc/nginx/ssl/pomelo.game.com.key;
        ssl_session_timeout 5m;
        ssl_protocols SSLv2 SSLv3 TLSv1;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        access_log  /etc/nginx/logs/pomelo.game.com.access.log main;
        error_log /etc/nginx/logs.pomelo.game.com.error.log;

        location / {
            # WebSocket support (nginx 1.4)
            proxy_pass                      https://connector02;

            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
}

```