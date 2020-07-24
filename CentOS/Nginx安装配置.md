# CentOS7 下 nginx 服务器的简单安装与搭建


## 安装nginx
1.下载nginx服务器安装包，这里以nginx1.14.2.tar.gz为例，将文件放到 /usr/local 目录下
2.将安装包解压到目录下，并进入解压出来的文件夹	

    [root@centos local]# tar -xzvf nginx-1.14.2.tar.gz
    [root@centos local]# cd nginx-1.14.2

3.进行配置和安装

```shell
//这是最原始的安装，不带任何功能组件
[root@centos nginx-1.14.2]# ./configure && make && make install

//为了支持https协议，建议用下面这种安装，先安装openssl
[root@centos nginx-1.14.2]# yum -y install openssl openssl-devel
[root@centos nginx-1.14.2]# ./configure --with-http_ssl_module
[root@centos nginx-1.14.2]# make
[root@centos nginx-1.14.2]# make install
```

到这里nginx就安装完毕了，默认安装在 /usr/local 目录下
假如先执行了原始安装，后面配置ssl时出现问题，只要先把nginx服务器关掉，回到nginx-1.14.2文件夹，用第二种方式再次安装覆盖掉就行，注意备份配置文件

## 第二步，nginx配置
安装好后local下出现了一个 nginx 文件夹，点进去
再进入里面的conf文件夹
命令行就直接 cd /usr/local/nginx/conf
在这个目录下有一个nginx.conf文件，这个就是nginx的配置文件
然后有ssl证书的，就可以把证书和密钥放到和 nginx.conf 一个目录下，剩下的就只有配置了
![ssl证书和密钥文件](https://img-blog.csdnimg.cn/2019072717143754.png)
nginx配置需要专门的学习才能掌握，这里直接给我的配置文件
我删减掉了一些注释，并没有对这个文件进行测试，有#的行就是被注释掉的行，我的服务器上运行着两个web项目

```nginx
worker_processes  1;

events {
    worker_connections  1024;
}
  
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;
    
    server {
        listen       80;
        listen 443 ssl;
        server_name www.jakuxa.cn ; 		#这里的域名改成自己的
        ssl on;												#off表示同时可以处理http和https请求，on表示只能进行https
        ssl_certificate 1_www.jakuxa.cn_bundle.crt;       #这里就是引用两个证书文件，注意路径（当前是同目录下）
        ssl_certificate_key 2_www.jakuxa.cn.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; 
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;

        location /testProject/ {
             proxy_pass http://118.89.22.111:8080/testProject/;			#这里就是第一个项目地址，注意这里监听的是tomcat服务器默认端口
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection "upgrade";
        }

        location /anotherProject/ {
           # root   html;
           # index  index.html index.htm;
             proxy_pass http://118.89.22.111:8080/anotherProject/;					#第二个项目地址
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection "upgrade";
        }
		
		#假如只有一个项目的话，就这么写
			#location /testProject {         #最后不要加“ / ”就可以了
	           # root   html;
	           # index  index.html index.htm;
	           #proxy_pass http://118.89.22.111:8080/anotherProject;					
	           #proxy_http_version 1.1;
	           #proxy_set_header Upgrade $http_upgrade;s
	           #proxy_set_header Connection "upgrade";
	       #}

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```

}


## 第三步 nginx服务器的启动，停止和重启

```shell
//先切换到nginx文件夹
[root@centos local]# cd /usr/local/nginx

//启动前检查，如果配置错误会提示
[root@centos nginx]# ./sbin/nginx -t

//启动服务器
[root@centos nginx]# ./sbin/nginx -c ./conf/nginx.conf

//重启服务器，每次更改配置，都要运行下面这条语句
[root@centos nginx]# ./sbin/nginx -s reload

//关闭服务器
[root@centos nginx]# ./sbin/nginx -s quit
```



# Nginx静态资源配置

> 静态配置其实非常简单，就加下面这个就解决了
>
> 但如果出问题无法访问资源，那么首先检查nginx监听的80和443端口，链路上有没有问题。
>
> 确保两个端口都正常开放，且nginx启动无误后，再检查配置项中 location和alias地址的结尾都要加上 “/”
>
> 最后检查url地址中是否用了合适的协议(如https)

```nginx
location /md/ {
  	alias /usr/local/markdownImg/;
  	autoindex on;		#开启目录功能
}

# 在上面alias虚拟目录配置下，访问https://www.jakuxa.cn/md/a.jpg
# 实际指定是/usr/local/markdownImg/a.jpg。
```

