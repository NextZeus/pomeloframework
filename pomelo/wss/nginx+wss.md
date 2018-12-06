## 不需要代码配置证书

### nginx 配置不跨服连接 wss

- nginx1.3+默认支持wss连接，基本上不需要特殊的配置
- 逻辑服务器ip: a.b.c.d[内网IP或者公网IP都可以，如果可以内网访问，最好用内网IP]

```

map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

upstream game-gate {
    server a.b.c.d:3014;
}

upstream game-connector01 {
    server a.b.c.d:3050;
}

upstream game-connector02 {
    server a.b.c.d:3051;
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
	error_log ./logs/https.pomelo.game.com.error.log;

	location /game-gate {
            proxy_pass                      https://game-gate;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
	
	location /game-connector01 {
            proxy_pass                      https://game-connector01;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
	
	location /game-connector02 {
            proxy_pass                      https://game-connector02;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
}

```

### 客户端连接

url:	'wss://pomelo.game.com/game-gate'
