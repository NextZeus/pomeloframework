# Nginx 配置 支持 Websocket

```

map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

upstream gate_1 {
    server 10.0.1.192:3015;
}
upstream connector_1 {
    server 10.0.1.192:3053;
}
upstream connector_2 {
    server 10.0.1.192:3054;
}
upstream connector_3 {
    server 10.0.1.192:3055;
}
upstream connector_4 {
    server 10.0.1.192:3056;
}
# 多人房多人赛马配置end

server {
        listen       80;
        listen       443 ssl;
        server_name  upgame.pengpengla.com;

        ssl_certificate /etc/nginx/ssl/pomelo.game.com.crt;
        ssl_certificate_key /etc/nginx/ssl/pomelo.game.com.key;
        ssl_session_timeout 5m;
        ssl_protocols SSLv2 SSLv3 TLSv1;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        access_log  /data/logs/nginx/pomelo.game.com.access.log main;

        root /data/app/nginx/html/static;

        location /gate_1 {
             proxy_pass                      http://gate_1;
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection $connection_upgrade;
        }
        location /connector_1 {
             proxy_pass                      http://connector_1;
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection $connection_upgrade;
        }
        location /connector_2 {
             proxy_pass                      http://connector_2;
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection $connection_upgrade;
        }
        location /connector_3 {
             proxy_pass                      http://connector_3;
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection $connection_upgrade;
        }
        location /connector_4 {
             proxy_pass                      http://connector_4;
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection $connection_upgrade;
        }
}

```

# Nginx 支持上万 Websocket连接

### 修改nginx允许的连接数

```
# nginx.conf

events {
    use epoll;
    worker_connections 2000000; //  允许websocket连接数，如果配置的很小，连接数达到这个配置数，就不会再允许连接通过nginx了。nginx会返回500错误
}

```

### 修改系统 open file descriptors


要想支持更高数量的TCP并发连接的通讯处理程序，就必须修改系统对当前用户的进程同时打开的文件数量的软限制(soft limit)和硬限制(hardlimit)。
其中软限制是指Linux在当前系统能够承受的告警范围内进一步限制用户同时打开的文件数，超过则会告警；
硬限制则是根据系统硬件资源状况(主要是系统内存)计算出来的系统最多可同时打开的文件数量，超过则无法打开了

```
$ ulimit -a
-t: cpu time (seconds)              unlimited
-f: file size (blocks)              unlimited
-d: data seg size (kbytes)          unlimited
-s: stack size (kbytes)             8192
-c: core file size (blocks)         0
-m: resident set size (kbytes)      unlimited
-u: processes                       10240
-n: file descriptors                2000000 (设置该配置)
-l: locked-in-memory size (kbytes)  64
-v: address space (kbytes)          unlimited
-x: file locks                      unlimited
-i: pending signals                 15005
-q: bytes in POSIX msg queues       819200
-e: max nice                        0
-r: max rt priority                 0
-N 15:                              unlimited


```

#### 永久修改系统 file descriptors 以及端口号范围

```

sudo vim /etc/sysctl.conf

添加下面四行配置

net.ipv4.ip_local_port_range = 1024 65535 (一个socket连接占一个端口, 端口范围小了，连接数也上不去)
kernel.pid_max=3999999
fs.file-max = 2000000
fs.nr_open = 2000000

sudo vim /etc/security/limits.conf

修改上限为200W

* soft nofile 2000000
* hard nofile 2000000
* soft nproc 2000000
* hard nproc 2000000

```


##### 参考文章连接

[](https://colobu.com/2015/05/22/implement-C1000K-servers-by-spray-netty-undertow-and-node-js/)

[](https://www.jianshu.com/p/c77e7026531a)

[](https://jingyan.baidu.com/article/2f9b480d8bcc9441ca6cc247.html)

[](https://blog.csdn.net/whereismatrix/article/details/50582919)