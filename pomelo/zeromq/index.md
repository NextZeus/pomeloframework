# Zeromq

## Centos6.5 Install ZeroMQ

```

1. cd /etc/yum.repos.d/
2. wget http://download.opensuse.org/repositories/home:/fengshuo:/zeromq/CentOS_CentOS-6/home:fengshuo:zeromq.repo
3. cd ~/
4. sudo yum install  libtool pkgconfig build-essential autoconf automake
5. sudo yum install zeromq-devel


```

### install zeromq error

```

g++版本低[4.7], 指定新版本g++

In my case, g++4.9 is installed in /opt/rh/devtoolset-3/root/usr/bin/g++ and
执行下面命令 指定g++编译路径
CC=/opt/rh/devtoolset-3/root/usr/bin/gcc CXX=/opt/rh/devtoolset-3/root/usr/bin/g++ npm install zmq

如果没有新的版本g++，则需要升级一下。

sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update
sudo apt-get install gcc-5.1 g++-5.1
sudo rm /bin/usr/g++
sudo ln -s /usr/bin/g++-5 /usr/bin/g++


参考资料

http://zeromq.org/distro:centos
https://github.com/nodejs/node-gyp/issues/940


```

## Pomelo中配置RPC client & server 

```

var zmq = require('pomelo-rpc-zeromq-renew');

//具体都有哪些可配置的字段 看过源码也不清楚 参考lordofpomelo

app.set('proxyConfig', {
    rpcClient: zmq.client,
    cacheMsg: true,
    interval: 30,
    lazyConnection: true,
    enableRpcLog: true
});

app.set('remoteConfig', {
    rpcServer: zmq.server,
    cacheMsg: true,
    interval: 30
});

```

## CentOS Upgrade gcc g++

CentOS 系统自带的 gcc 或者 g++ 的版本是：g++ (GCC) 4.4.6. GCC 版本太旧，导致了很多使用上的不便，如：无法使用g++ -std=c++11 命令来编译 C++11、无法使用Vim的很多插件（YouCompleteMe等）。因此，有必要对它进行升级。

对 GCC 升级无法直接使用：

```

> yum update gcc

```

以下是升级的详细过程:

1. 使用 redhat developer toolset 1.1 的repo，安装GCC

```

> cd /etc/yum.repos.d
> wget http://people.centos.org/tru/devtools-1.1/devtools-1.1.repo 
> yum --enablerepo=testing-1.1-devtools-6 install devtoolset-1.1-gcc devtoolset-1.1-gcc-c++

```

2. 替换系统中原来的GCC
        
2.1 通过通过第一步会把 GCC 安装到以下目录：
```

> /opt/centos/devtoolset-1.1/root/usr/bin/

```

2.2 接下来需要修改系统的配置，使默认的 gcc 和 g++ 命令使用的是新安装的版本。

```

> ln -s /opt/centos/devtoolset-1.1/root/usr/bin/* /usr/local/bin/
> hash -r

```    
3. 现在查看 g++ 的版本号：

```
    > g++ --version

```



## zmq module

sudo npmi zmq -g

npm link zmq



#### 参考资料
[centos-upgrade-gcc-g++](http://www.wengweitao.com/centos-sheng-ji-gcc-he-g-de-fang-fa.html)
