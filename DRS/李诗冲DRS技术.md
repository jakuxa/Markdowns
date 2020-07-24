## 李诗冲DRS系统

> 不对接jres，单纯spring-cloud

- 注册中心 eureka，而非zookeeper
- 系统用gradle构建，且版本不能高于4.8.1

### 系统总体结构

![image-20200704171108601](/Users/jakuxa/Library/Application Support/typora-user-images/image-20200704171108601.png)



### svn暨项目目录

> trunk是最新的

| 项目、模块名      | 模块说明       | 备注        |
| ----------------- | -------------- | ----------- |
| drs-as-6.0        |                | svn废弃目录 |
| drs-db-6.0        | 数据库脚本打包 |             |
| drs-db-dc         |                | 废弃目录    |
| drs-db-others     | 异构脚本       |             |
| drs-plantform-6.0 | java微服务     | gradle构建  |
| drs-template-6.0  |                |             |

### 模块解析

#### drs-plantform-6.0

- fam-service，内含网关和注册中心，基本不用去动

- fam-module，内含对应不同业务的多个微服务模块

  ![image-20200704171430215](/Users/jakuxa/Library/Application Support/typora-user-images/image-20200704171430215.png)

- fam-common，通用模块，通用jar包

### gradle打包

> 切换工作路径到drs-plantform-6.0目录下，打包至ftp服务器，前端drs-web同理
>
> $:  ./build.sh lisc $ftp账号 $ftp密码

