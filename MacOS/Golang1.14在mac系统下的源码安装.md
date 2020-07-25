## Golang1.14的源码安装和升级

- 此方法理论上对CentOS等linux系统都通用。

> go1.4之前是由c++编译，而从1.5开始则是用go语言自身编译。
>
> 因此1.4以前的源码编译安装需要gcc，而1.5及以后的版本基本都需要先安装一个go1.4来编译源码，此处以go 1.14.2为例，其他版本应该是大同小异的。

网上教程其实很多，不过我自己看着有点累，他们有不少多余且复杂的操作，以至于我看不懂而不敢去盲目复现。

最后还是结合了官网上的源码安装教程搞定了[官网教程](https://golang.org/doc/install/source)

其实思路非常简单：1.配置编译器环境变量。2.运行脚本。3.将go加入环境变量

### 一、下载

- [golang官网下载地址](https://golang.org/dl/)

- 下载[go1.14.2.src.tar.gz](https://dl.google.com/go/go1.14.2.src.tar.gz)或者其他稳定版。src代表源码

- 下载[go1.4 bootstrap](https://dl.google.com/go/go1.4-bootstrap-20171003.tar.gz)

### 二、安装go-1.4-bootstrap

1. 首先解压go1.4-bootstrap-20171003.tar.gz

   > 我就解压到自己的下载目录了，并重命名叫做go1.4，最终路径如下

   ```shell
   # 注意自己的路径和重命名
   /Users/jakuxa/Download/go1.4
   ```

2. 配置环境变量，将go1.4注册到环境中，接下来要用它作为编译工具

   - **mac系统**

   ```shell
   # 打开mac系统环境变量配置文件
   $ vim ~/.bash_profile
   
   # 在任意地方添加两行环境变量
   > export GOROOT_BOOTSTRAP=/Users/jakuxa/Download/go1.4
   > CGO_ENABLED=0
   
   # 刷新生效
   $ source ~/.bash_profile
   
   # 进入go1.4/src目录下，运行make.bash
   $ cd /Users/jakuxa/Download/go1.4/src
   $ sh make.bash
   ```

   - **windows系统**

   > 下载前，一定要确保电脑上安装了gcc，如果没有gcc或者有其他困难

   **那么你看到这白看了，建议直接go最新版客户端，放弃源码编译安装吧**

   **因为我尝试在windows10上源码安装时，用gcc编译go1.4源码会报错，我想恐怕是gcc版本不对，但我不想浪费这么多精力去寻找合适的版本**

   >  [gcc安装参考](https://www.cnblogs.com/raina/p/10656106.html)
   
   ```powershell
   # ctrl+R cmd打开命令行
   > gcc -v
   # 有版本号信息说明已经装了gcc，没有则需要安装
   # 剩余步骤windows的步骤和mac基本一致，请参考mac去做吧，记得运行cmd时要以管理员身份打开
   # 另外唯一的区别mac中运行的.bash文件，windows下对应去运行.bat后缀的文件，如make.bat和all.bat
   ```
   
   > 确保有gcc后，windows添加环境变量
   >
   > 右键  此电脑 -> 属性 -> 高级系统设置  -> 环境变量 ->系统变量 新建
   >
   > > 变量名: CGO_ENABLED
   > >
   > > 变量值: 0
   >
   > > 变量名: GOROOT_BOOTSTRAP
   > >
   > > 变量值: /Users/jakuxa/Download/go1.4	
   > >
   > > #注意改成windows下的路径，这里我就用这个指代了
   


### 三、安装go1.14

1. 解压go1.14.2.src.tar.gz

   > 最终路径如下

   ```shell
   # 注意自己的路径
   /Users/jakuxa/go
   ```

2. 安装并配置

   - **mac系统**

   ```shell
   $ cd /Users/jakuxa/go/src
   $ sh all.bash
   
   # 打开mac系统环境变量配置文件
   $ vim ~/.bash_profile
   
   # 这是golang源码路径，以后安装第三放包都在这里
   > export GOPATH=/Users/jakuxa/项目/golang
   # go路径
   > export GOROOT=/Users/jakuxa/go/
   # 修改之前的GOROOT_BOOTSTRAP
   > export GOROOT_BOOTSTRAP=/Users/jakuxa/go/
   # 加入到path中
   > export PATH=$PATH:$GOROOT/bin
   
   # 刷新生效
   $ source ~/.bash_profile

   # 检查安装是否成功
   $ go version
   ```
   
   - **windows系统**
   
   ```powershell
   > cd /Users/jakuxa/go/src
   > all.bat
   ```
   
   > 添加环境变量到path
   >
   > 还记得当初JAVA_HOME怎么配的吗，xD
   
   ```powershell
   > go version
   ```

### 四、go的升级

1. 下载新版源码包，假设为go1.15.1.src.tar.gz，并解压到下载文件夹，总之先不要覆盖目前安装的go目录：

   ```
   /Users/jakuxa/Download/go
   ```

2. 用go1.14编译安装

   > 仍然是编译源码，实际上就是覆盖安装，但不需要再用go1.4了
   >
   > go可以用老版本编译新版本，所以等1.14安装好后，就可以编译1.15或者以上版本

   ```shell
   # 到1.15的src目录下执行相同操作
   $ cd /Users/jakuxa/Download/go/src
   $ sh all.bash
   ```

3. 黏贴覆盖老版本go

   ```bash
   # 可以鼠标操作去复制黏贴，不一定非要用命令
   $ mv /Users/jakuxa/Download/go /Users/jakuxa/go
   # 如果go根目录也改变了，则需要去.bash_profile中修改GOPATH坏境变量
   # 如果打算安装两个版本，则不要覆盖老版本，并通过修改GOPATH在新老版本间切换
   $ go version
   ```

