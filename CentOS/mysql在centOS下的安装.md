## mysql5.7.25在CentOS 7下的源码安装

> **源码安装耗时极长，对于本身资源有限的小服务器并不友好，除非有较长空余时间且机器性能优秀，否则建议使用rpm安装包方式安装**



#### 5.7.25带boost源码包资源：

- https://cdn.mysql.com/archives/mysql-5.7/mysql-boost-5.7.25.tar.gz

#### 5.7.25 rpm合集包资源：

- https://cdn.mysql.com/archives/mysql-5.7/mysql-5.7.25-1.el7.x86_64.rpm-bundle.tar



### 安装

#### 安装依赖

```shell
yum install -y  gcc gcc-c++ cmake ncurses ncurses-devel bison
```

#### 添加用户与新建文件夹

```shell
useradd -s /sbin/nologin mysql
mkdir -p /data/mysql/data
chown -R mysql /data/mysql
```

#### 解压

```shell
tar -zxvf mysql-boost-5.7.25.tar.gz -C /usr/local/mysql/
```

#### 编译安装

```shell
cd /usr/local/mysql/mysql-5.7.25
cmake -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_BOOST=boost
# 强烈建议这一步make install在后台运行，且这一步骤极有可能持续数小数，建议晚上操作
# nohup make && make install &
make && make install

#编译这步等待较久时间
```



### 配置

```shell
vim /etc/my.cnf
```

```shell
## 此处可以修改port，以及max_allowed_packet等数据
## 这里以3306为例

[client]
port        = 3306
socket      = /tmp/mysql.sock

[mysqld]
port        = 3306
socket      = /tmp/mysql.sock
user = mysql


basedir = /usr/local/mysql
datadir = /data/mysql/data
pid-file = /data/mysql/mysql.pid

log_error = /data/mysql/mysql-error.log
slow_query_log = 1
long_query_time = 1
slow_query_log_file = /data/mysql/mysql-slow.log


skip-external-locking
key_buffer_size = 32M
max_allowed_packet = 1024M
table_open_cache = 128
sort_buffer_size = 768K
net_buffer_length = 8K
read_buffer_size = 768K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
thread_cache_size = 16
query_cache_size = 16M
tmp_table_size = 32M
performance_schema_max_table_instances = 1000

explicit_defaults_for_timestamp = true
#skip-networking
max_connections = 500
max_connect_errors = 100
open_files_limit = 65535

log_bin=mysql-bin
binlog_format=mixed
server_id   = 232
expire_logs_days = 10
early-plugin-load = ""

default_storage_engine = InnoDB
innodb_file_per_table = 1
innodb_buffer_pool_size = 128M
innodb_log_file_size = 32M
innodb_log_buffer_size = 8M
innodb_flush_log_at_trx_commit = 1
innodb_lock_wait_timeout = 50

[mysqldump]
quick
max_allowed_packet = 16M

[mysql]
no-auto-rehash

[myisamchk]
key_buffer_size = 32M
sort_buffer_size = 768K
read_buffer = 2M
write_buffer = 2M
```

#### 赋权给用户mysql

```shell
chown -R mysql /usr/local/mysql
chown -R mysql /data/mysql/data
# 把数据库数据目录用户和组更改为mysql （ 数据库数据目录：/data/mysql/data），方法同上!
# 注：/data/mysql/data目录下一定要为空才行
```

#### 初始化mysql

```shell
./mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql/data
```

> **注:到这一步很容易出问题，在初始化的时候一定要加上面的参数，而且在执行这一步操作前/data/mysql/data 这个目录必须是空的；在这里指定的basedir 和 datadir 目录必须要和/etc/my.cnf 配置的目录一直才行。**

- 拷贝可执行文件

```shell
cp mysql.server /etc/init.d/mysqld
```



### 服务器启动

```shell
service mysqld start
```

- 测试

```shell
./mysql -hlocalhost -uroot -p
# 无须密码即可登录
# 登录后可以设置密码
```

#### 添加环境变量

```shell
vim /etc/profile
```

```shell
# 新增一行
PATH=/usr/local/mysql/bin:$PATH
```

```shell
source /etc/profile
```

#### 开机启动

```shell
systemctl enable mysqld
```



### 添加远程用户

```sql
--实例，创建一个账号jakuxa,密码aaa@123的用户
CREATE USER 'jakuxa'@'%' IDENTIFIED BY 'aaa@123';
--赋予所有权限给jakuxa，包括远程登录权限
GRANT ALL PRIVILEGES ON *.* TO 'jakuxa'@'%' WITH GRANT OPTION;
--如果登录地址是固定的，那么可以将'root'@'%'中的百分号改为ip地址，如'root'@'192.168.0.49'
--赋予root远程登录权限
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION; 
--刷新生效
flush privileges;
```

#### 更改用户密码

```sql
set password for 用户名@localhost = password('新密码');
--例 set password for root@localhost = password('123456');
--如果密码123456不通过，则可以通过调整密码政策
set global validate_password_policy=0; --仅限制密码长度，默认为至少8位
set global validate_password_length=6; --改为至少6位
```



## mysql5.7.25在CentOS 7下的rpm包安装

### 忽略依赖强制安装

```shell
# 解压软件包合集
tar -xvf mysql-5.7.25-1.el7.x86_64.rpm-bundle.tar
# 安装其中的server和client，如下
rpm -ivh mysql-community-client-5.7.25-1.el7.x86_64.rpm --nodeps --force
rpm -ivh mysql-community-common-5.7.25-1.el7.x86_64.rpm --nodeps --force
rpm -ivh mysql-community-libs-5.7.25-1.el7.x86_64.rpm --nodeps --force
rpm -ivh mysql-community-libs-compat-5.7.25-1.el7.x86_64.rpm --nodeps --force
rpm -ivh mysql-community-server-5.7.25-1.el7.x86_64.rpm --nodeps --force
```

### 开机启动mysql

```shell
systemctl enable mysqld
systemctl start mysqld
```

### 配置mysql

```shell
# 查询默认root密码
more /var/log/mysqld.log |grep password 
```

```shell
# 开始配置，按步骤执行
mysql_secure_installation

# 删除临时用户
Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
# 是否禁止远程登录
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : n
# 删除测试数据库
Remove test database and access to it? (Press y|Y for Yes, any other key for No) :y
# 刷新权限表?
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
```

### 修改端口

```shell
vim /etc/my.cnf

# 在[mysqld]下新增一行
port=6033

#重启生效
systemctl restart mysqld
```

