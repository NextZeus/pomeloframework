# 部署

### 配置文件
```
servers.json

"development":{
    "connector":[
            {"id":"connector-server-1", "host":"0.0.0.0", "port":4050, "clientPort": 3050, "clientHost":"aaaa.bbbb.cccc", "frontend": true}
        ],
    "room":[
        {"id":"room-server-1", "host":"0.0.0.0", "port":5050}
    ],
    "gate":[
        {"id": "gate-server-1", "host": "0.0.0.0", "clientPort": 3014, "frontend": true}
    ],
    "data":[
        {"id":"data-server-1", "host":"0.0.0.0", "port":7050}
    ]
}


master.json

"development":{
    "id":"master-server-1",
    "host":"0.0.0.0",
    "port":3005
}

```

```
特别要注意的地方在于，connector服务器的配置，我的选择方案是，添加一个clientHost, 
在gate服务器返回connector服务器信息时，返回clientHost和clientPort

```
参考：https://github.com/NetEase/pomelo/issues/719

## Zeromq
服务器需要安装zeromq，不管是Debian 还是 Centos.
zeromq 在配置proxyClient, remoteServer时会用到。
zeromq我没有做深入的了解，以后了解了，会分享出来。

## Debian Ubuntu14.04
[参考官网install debian/zeromq](http://zeromq.org/distro:debian)
> apt-get update
>
> sudo apt-get install libtool pkg-config build-essential autoconf automake
>
> sudo apt-get install libzmq3-dev
>
> npm i zmq [如果能安装成功 则说明zeromq已安装成功]

## Centos 6
[参考官网install centos/zeromq](http://zeromq.org/distro:centos)

1. 第一步需要配置 RHEL/CentOS 6: http://download.opensuse.org/repositories/home:/fengshuo:/zeromq/CentOS_CentOS-6/home:fengshuo:zeromq.repo

> cd /etc/yum.repos.d/
>
> sudo wget http://download.opensuse.org/repositories/home:/fengshuo:/zeromq/CentOS_CentOS-6/home:fengshuo:zeromq.repo
>
> cd ~
>
> yum install libtool pkgconfig autoconf automake
> 
> yum groupinstall "Development Tools" [build-essential 只能在Centos系统上安装不了]
>
> yum install zeromq-devel

参考：
1. [build-essentials-in-centos](http://www.asim.pk/2010/05/28/build-essentials-in-centos/)

### Centos安装Zeromq遇到的问题

参考：https://github.com/nodejs/node-gyp/issues/940

#### Resolution

Centos拥有两个版本的g++, 一个4.7，一个4.9
g++版本低[4.7], 指定新版本g++

In my case, g++4.9 is installed in /opt/rh/devtoolset-3/root/usr/bin/g++ and
执行下面命令 指定g++编译路径
CC=/opt/rh/devtoolset-3/root/usr/bin/gcc CXX=/opt/rh/devtoolset-3/root/usr/bin/g++ npm install zmq

## Mac 配置zeromq [这个系统倒是比较简单]
> brew install pkg-config
>
> brew install zmq


### 需要注意的地方
1. game-server下package.json dependecies添加 "zmq": "2.15.3"
2. 代码部署到服务器时，不要提交node_modules/zmq
3. 服务器上game-server目录下必须每次重新 npm i zmq
4. 如果遇到build问题，查看gcc -v ，如果低于4.9, 则执行
    * sudo su
    * CC=/opt/rh/devtoolset-3/root/usr/bin/gcc CXX=/opt/rh/devtoolset-3/root/usr/bin/g++ npm install zmq

