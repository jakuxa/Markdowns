## Oracle在CentOS 7下的静默安装（无图形界面）

> 参考https://blog.csdn.net/chenghuikai/article/details/85776622
>
> 并在此基础上添加了试错和补充

- 系统：CentOS 7.5
- Oracle版本：linux.x64_11gR2

- 软件包：
  - linux.x64_11gR2_database_1of2.zip
  - linux.x64_11gR2_database_2of2.zip	



### 前置准备

#### 更改主机名称

> 这一步建议不做，那么在下面的步骤中，注意这个“oracledb”，将它们改成服务器实际的主机名

```shell
hostnamectl set-hostname oracledb
echo "127.0.0.1     oracledb" >>/etc/hosts
```

#### 关闭SeLinux

```shell
sed -i "s/SELINUX=enforcing/SELINUX=disabled/" /etc/selinux/config  
setenforce 0
```

#### 安装依赖

```shell
yum -y install binutils compat-libcap1 compat-libstdc++-33 gcc gcc-c++ glibc glibc-devel ksh libaio libaio-devel libgcc libstdc++ libstdc++-devel libXi libXtst make sysstat unixODBC unixODBC-devel
```

#### 创建用户与用户组

```shell
groupadd oinstall
groupadd dba
groupadd oper
useradd -g oinstall -G dba oracle
# 设置用户密码
passwd oracle
```

#### **配置内核参数和资源限制**

- 在`/etc/sysctl.conf`添加如下参数，如果系统中某个参数高于下面的参数的值 ，保留较大的值,下面的数值只是官方要求的最小值，可以根据系统调整数值，以优化系统性能

```shell
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 536870912
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
```

- 使内核参数生效

```shell
sysctl -p
```

- 在`/etc/security/limits.conf`中添加如下参数

```shell
oracle              soft    nproc   2047
oracle              hard    nproc   16384
oracle              soft    nofile  1024
oracle              hard    nofile  65536
```

- 在/etc/pam.d/login文件中，添加下面内容

```shell
session required /lib64/security/pam_limits.so
session required pam_limits.so
```

- /etc/profile 文件中添加如下内容

```shell
if [ $USER = "oracle" ]; then
   if [ $SHELL = "/bin/ksh" ]; then
       ulimit -p 16384
       ulimit -n 65536
    else
       ulimit -u 16384 -n 65536
   fi
fi
```

- 使用`/etc/profile`文件生效

```shell
source /etc/profile
```

- 禁用使用Transparent HugePages(启用Transparent HugePages,可能会导致造成内存在运行时的延迟分配，Oracle官方建议使用标准的HugePages)

  > 查看是否启用 如果显示 `[always]`说明启用了

```shell
cat /sys/kernel/mm/transparent_hugepage/enabled
```

- 禁用Transparent HugePages,在/etc/grub.conf添加如下内容

```shell
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

- 重新启动系统以使更改成为永久更改



### 安装Oracle

#### 创建oracle安装目录

```shell
mkdir -p /data/app/
chown -R oracle:oinstall /data/app/
chmod -R 775 /data/app/
```

#### 配置oracle用户环境变量

- 在文件`/home/oracle/.bash_profile`里添加下面内容(具体值根据实际情况修改)

```shell
umask 022
export ORACLE_HOSTNAME=oracledb
export ORACLE_BASE=/data/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/
export ORACLE_SID=ORCL
export PATH=.:$ORACLE_HOME/bin:$ORACLE_HOME/OPatch:$ORACLE_HOME/jdk/bin:$PATH
export LC_ALL="en_US"
export LANG="en_US"
export NLS_LANG="AMERICAN_AMERICA.ZHS16GBK"
export NLS_DATE_FORMAT="YYYY-MM-DD HH24:MI:SS"
```

#### 重启下系统

```shell
reboot
```

#### 解压文件

```sql
unzip -q linux.x64_11gR2_database_1of2.zip -d /data
unzip -q linux.x64_11gR2_database_2of2.zip -d /data
mkdir -p /data/etc
cp /data/database/response/* /data/etc/
```

- 在`/data/etc/db_install.rsp`修改以下变量的值

```shell
oracle.install.option=INSTALL_DB_SWONLY
DECLINE_SECURITY_UPDATES=true
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/data/app/oracle/inventory
SELECTED_LANGUAGES=en,zh_CN
ORACLE_HOSTNAME=oracledb
ORACLE_HOME=/data/app/oracle/product/11.2.0
ORACLE_BASE=/data/app/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.isCustomInstall=true
oracle.install.db.DBA_GROUP=dba
oracle.install.db.OPER_GROUP=dba
```

#### 开始安装

```shell
su - oracle
cd /data/database
./runInstaller -silent -responseFile /data/etc/db_install.rsp -ignorePrereq
```

- 如果出现如下报错

```shell
Checking swap space: 0 MB available, 150 MB required. Failed <<<<
# 解决方法https://www.cnblogs.com/a9999/p/6957280.html
```

- 安装完成后有如下提示，如果有类似如下提示，说明安装完成

```shell
The following configuration scripts need to be executed as the "root" user. 
#!/bin/sh 
#Root scripts to run

/u01/app/oraInventory/orainstRoot.sh
/u01/app/oracle/product/11.2.0/db_1/root.sh
To execute the configuration scripts:

1. Open a terminal window 
2. Log in as "root" 
3. Run the scripts 
4. Return to this window and hit "Enter" key to continue

Successfully Setup Software.
```

- 使用root用户执行脚本

```shell
su - root
sh /data/app/oracle/inventory/orainstRoot.sh
sh /data/app/oracle/product/11.2.0/root.sh
```

- 配置监听程序

```shell
su - oracle
netca /silent /responsefile /data/etc/netca.rsp

#如果提示netca命令不存在
则检查ORACLE_HOME路径下有没有netca软件，并source重新加载/home/oracle/.bash_profile

#输出结果
[oracle@oracledb ~]$ netca /silent /responsefile /data/etc/netca.rsp

Parsing command line arguments:
    Parameter "silent" = true
    Parameter "responsefile" = /data/etc/netca.rsp
Done parsing command line arguments.
Oracle Net Services Configuration:
Profile configuration complete.
Oracle Net Listener Startup:
    Running Listener Control:
      /data/app/oracle/product/11.2.0/bin/lsnrctl start LISTENER
    Listener Control complete.
    Listener started successfully.
Listener configuration complete.
Oracle Net Services configuration successful. The exit code is 0
```

- 上一步配置监听程序可能会出错误，提示无法监听，处理如下

```shell
# 错误信息，如下
Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.4.181)(PORT=1521)))
TNS-12541: TNS:no listener
 TNS-12560: TNS:protocol adapter error
  TNS-00511: No listener
   Linux Error: 111: Connection refused
Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=IPC)(KEY=EXTPROC1521)))
TNS-12541: TNS:no listener
 TNS-12560: TNS:protocol adapter error
  TNS-00511: No listener
   Linux Error: 111: Connection refused

# 注意HOST，将HOST中的地址改为本地地址,修改/etc/hosts，可以是内网地址，可以是localhost
# 打开防火墙的1521端口
```

- 查看监听端口

```shell
netstat -tnpl | grep 1521
# 如何什么也没返回那上面一定做错了，解决上面的再继续
```



### 创建数据库

- 编辑应答文件`/data/etc/dbca.rsp`

```shell
[GENERAL]
RESPONSEFILE_VERSION = "11.2.0"
OPERATION_TYPE = "createDatabase"
[CREATEDATABASE]
GDBNAME = "orcl"
SID = "orcl"
SYSPASSWORD = "oracle"
SYSTEMPASSWORD = "oracle"
SYSMANPASSWORD = "oracle"
DBSNMPPASSWORD = "oracle"
DATAFILEDESTINATION =/data/app/oracle/oradata
RECOVERYAREADESTINATION=/data/app/oracle/fast_recovery_area
CHARACTERSET = "AL32UTF8"
TOTALMEMORY = "1638"
```

- 执行静默建库

```shell
su - oracle
dbca -silent -responseFile /data/etc/dbca.rsp
```

- 执行过程如下

```shell
[oracle@oracledb ~]$ dbca -silent -responseFile /data/etc/dbca.rsp
Copying database files
1% complete
3% complete
11% complete
18% complete
26% complete
37% complete
Creating and starting Oracle instance
40% complete
45% complete
50% complete
55% complete
56% complete
60% complete
62% complete
Completing Database Creation
66% complete
70% complete
73% complete
85% complete
96% complete
100% complete
Look at the log file "/data/app/oracle/cfgtoollogs/dbca/orcl/orcl.log" for further details.
```

- 查看进程

```shell
ps -ef | grep ora_ | grep -v grep

# 执行结果
[oracle@oracledb ~]$ ps -ef | grep ora_ | grep -v grep
oracle   19304     1  0 18:33 ?        00:00:00 ora_pmon_orcl
oracle   19306     1  0 18:33 ?        00:00:00 ora_vktm_orcl
oracle   19310     1  0 18:33 ?        00:00:00 ora_gen0_orcl
oracle   19312     1  0 18:33 ?        00:00:00 ora_diag_orcl
oracle   19314     1  0 18:33 ?        00:00:00 ora_dbrm_orcl
oracle   19316     1  0 18:33 ?        00:00:00 ora_psp0_orcl
oracle   19318     1  0 18:33 ?        00:00:00 ora_dia0_orcl
oracle   19320     1  0 18:33 ?        00:00:00 ora_mman_orcl
oracle   19322     1  0 18:33 ?        00:00:00 ora_dbw0_orcl
oracle   19324     1  0 18:33 ?        00:00:00 ora_lgwr_orcl
oracle   19326     1  0 18:33 ?        00:00:00 ora_ckpt_orcl
oracle   19328     1  0 18:33 ?        00:00:00 ora_smon_orcl
oracle   19330     1  0 18:33 ?        00:00:00 ora_reco_orcl
oracle   19332     1  0 18:33 ?        00:00:00 ora_mmon_orcl
oracle   19334     1  0 18:33 ?        00:00:00 ora_mmnl_orcl
oracle   19336     1  0 18:33 ?        00:00:00 ora_d000_orcl
oracle   19338     1  0 18:33 ?        00:00:00 ora_s000_orcl
oracle   19361     1  0 18:34 ?        00:00:00 ora_qmnc_orcl
oracle   19376     1  0 18:34 ?        00:00:00 ora_cjq0_orcl
oracle   19396     1  0 18:34 ?        00:00:00 ora_q000_orcl
oracle   19398     1  0 18:34 ?        00:00:00 ora_q001_orcl
```

- 查看监听状态

```shell
$ lsnrctl status

#如果提示没有lsnrctl这个命令，说明oracle监听没起来，则执行以下语句
$ lsnrctl start

#结果
[oracle@oracledb ~]$ lsnrctl status

LSNRCTL for Linux: Version 11.2.0.1.0 - Production on 02-JAN-2019 18:36:15

Copyright (c) 1991, 2009, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=IPC)(KEY=EXTPROC1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 11.2.0.1.0 - Production
Start Date                02-JAN-2019 18:20:21
Uptime                    0 days 0 hr. 15 min. 54 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /data/app/oracle/product/11.2.0/network/admin/listener.ora
Listener Log File         /data/app/oracle/diag/tnslsnr/oracledb/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=oracledb)(PORT=1521)))
Services Summary...
Service "orcl" has 1 instance(s).
  Instance "orcl", status READY, has 1 handler(s) for this service...
Service "orclXDB" has 1 instance(s).
  Instance "orcl", status READY, has 1 handler(s) for this service...
The command completed successfully
```

#### 数据库创建完成，登陆看看

```shell
su - oracle
sqlplus / as sysdba
SQL> select status from v$instance;
```

- 执行select时，全出现以下情况，是因为oracle没有启动，上面启动的只是监听

```shell
SQL> select status from v$instance;
*
ERROR at line 1:
ORA-01034: ORACLE not available
Process ID: 0
Session ID: 0 Serial number: 0
```

- 启动oracle，startup

```shell
#startup的输出提示：
SQL> statup
SP2-0042: unknown command "statup" - rest of line ignored.
SQL> startup
ORA-01078: failure in processing system parameters
LRM-00109: could not open parameter file '/data/app/oracle/product/11.2.0/dbs/initORCL.ora'
```

- 根据提示，将$ORACLEBASE/admin/数据库名称/pfile目录下的init.ora.xxx形式的文件copy到$ORACLE_HOME/dbs目录下initOracle.ora（根据startup提示）即可
- 完成上述后，startup提示01102装载错误，则重启系统即可

```shell
cp /data/app/oracle/admin/orcl/pfile/init.ora.0xxxxxxxx329 /data/app/oracle/product/11.2.0/dbs/initORCL.ora
# 如果复制文件后，还是无法启动，那就是oracle用户对这个文件还没有权限
# 切换root用户进行赋权,777表示该文件所有人都能操作
su root
chmod 777 /data/app/oracle/product/11.2.0/dbs/initORCL.ora
```

```sql
# 查看数据库编码
SQL> select userenv('language') from dual;
# 查看数据库版本
SQL> select * from v$version;
```

#### 激活scott用户，这也是远程登陆的用户

```sql
SQL> alter user scott account unlock;		--最后的分号一定要有
SQL> alter user scott identified by tiger;
SQL> select username,account_status from dba_users;
```



### Oracle开机自启动

- 修改`/data/app/oracle/product/11.2.0/bin/dbstart`

```shell
ORACLE_HOME_LISTNER=$ORACLE_HOME
```

- 修改`/data/app/oracle/product/11.2.0/bin/dbshut`

```shell
ORACLE_HOME_LISTNER=$ORACLE_HOME
```

- 修改`vi /etc/oratab`

```shell
orcl:/data/app/oracle/product/11.2.0:Y
```

```shell
# 切换到root用户
su root
chmod +x /etc/rc.d/rc.local
vim /etc/rc.d/rc.local
# 增加两行
su oracle -lc "/data/app/oracle/product/11.2.0/bin/emctl start dbconsole"
su oracle -lc "/data/app/oracle/product/11.2.0/bin/lsnrctl start"
su oracle -lc "/data/app/oracle/product/11.2.0/bin/dbstart"
```

- 设置oracle用户环境自动加载

```shell
vim /home/oracle/.bash_profile
#注释下面这些内容
# if [ -f ~/.bashrc ]; then
#        . ~/.bashrc
# fi

vim .bashrc
#在末尾添加
source /home/oracle/.bash_profile
```



- 给文件加启动权限

```shell
cd /data/app/oracle/product/11.2.0/bin/
chmod 6751 oracle
cd /var/tmp
chown -R oracle:oinstall .oracle
```



### 创建表空间和用户

```sql
create tablespace 表间名 datafile '数据文件名' size 表空间大小
--create tablespace TB_DRS datafile '/data/app/oracle/tablespace/TB_DRS.DBFBF' size 256M

--自动增长，表空间不足时增加200MB，最大扩展5000MB，若要无限扩大，则 maxsize unlimited;
alter database datafile '/data/app/oracle/tablespace/TB_DRS.DBFBF' autoextend on next 200m maxsize 5000m; 

create user 用户名 identified by 密码 default tablespace 表空间表;
--create user jakuxa identified by linn default tablespace TB_DRS;

--改密码
alter user jakuxa identified by 123456

--用户授权，包括远程登录与资源管理
GRANT connect, resource to jakuxa
```





### 开启Oracle远程连接

> 上面我们已经创建了scott用户，下面就可以用这个用户进行远程连接

- 修改配置文件/data/app/oracle/product/11.2.0/network/admin/listener.ora

```shell
vim /data/app/oracle/product/11.2.0/network/admin/listener.ora
```

```shell
# 修改里面的HOST为主机名，其重点在于与整个安装流程第一步的修改主机名对应，主机解析不能是127.0.0.1，须是内网地址
# 如果我们没有执行第一步的修改主机名，则这里要与系统默认主机名保持一致
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
      (ADDRESS = (PROTOCOL = TCP)(HOST = oracledb)(PORT = 1521))
    )
  )
```

- 如果准确无误，则可以通过这样的连接登录oracle

![image-20200713235743529](/Users/jakuxa/Library/Application Support/typora-user-images/image-20200713235743529.png)