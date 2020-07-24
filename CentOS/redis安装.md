##### redis安装

```shell
$ tar zxvf redis-5.0.8.tar.gz
$ mv redis-5.0.8 redis
$ cd redis
$ make
$ cd src
$ make install PREFIX=/usr/local/redis
$ cd /usr/local/redis
# 启动服务
$ /usr/local/redis/bin/redis-server /usr/local/redis/redis.conf
# 客户端连接
$ /usr/local/redis/bin/redis-cli
# 客户端关闭
$ /usr/local/redis/bin/redis-cli shutdown
# 设置开机启动
vi /etc/rc.local
> /usr/local/redis/bin/redis-server /usr/local/redis/redis.conf
# 修改配置
vi /usr/local/redis/redis.conf
> daemonize yes
> port 6034
> #bind 127.0.0.1
> requirepass sabrina@Redius
```



##### tcl安装

```shell
$ wget http://downloads.sourceforge.net/tcl/tcl8.6.1-src.tar.gz  
$ sudo tar xzvf tcl8.6.1-src.tar.gz  -C /usr/local/  
$ cd  /usr/local/tcl8.6.1/unix/  
$ sudo ./configure  
$ sudo make  
$ sudo make install
```

