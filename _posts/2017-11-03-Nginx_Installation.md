---
layout: post
title: Nginx部署文档（二进制包安装）
---


# Nginx部署文档（二进制包安装）
> 创建时间：2016-06-27
> 修改时间：2017-03-04 
> 修改时间：2017-03-06

## 文档目的
使用二进制包安装Nginx服务器

## 基础知识
Nginx（发音同 engine x）是一款轻量级的Web 服务器／反向代理服务器及电子邮件（IMAP/POP3）代理服务器，并在一个BSD-like 协议下发行。由俄罗斯的程序设计师Igor Sysoev所开发，最初供俄国大型的入口网站及搜寻引擎Rambler（俄文：Рамблер）使用。 其特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页伺服器中表现较好.目前中国大陆使用nginx网站用户有：新浪、网易、 腾讯,另外知名的微网志Plurk也使用nginx。

反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。这里讲得很直白。反向代理方式实际上就是一台负责转发的代理服务器，貌似充当了真正服务器的功能，但实际上并不是，代理服务器只是充当了转发的作用，并且从真正的服务器那里取得返回的数据。我们让nginx监听一个端口，譬如80端口，但实际上我们转发给在8080端口的tomcat，由它来处理真正的请求，当请求完成后，tomcat返回，但数据此时没直接返回，而是直接给nginx，由nginx进行返回，这里，我们会以为是nginx进行了处理，但实际上进行处理的是tomcat。很多用到nginx的地方都是作为静态伺服器，这样可以方便缓存那些静态文件，比如CSS，JS，html，htm等文件。

## 常用命令
开启nginx	/usr/local/nginx/sbin/nginx -r start
关闭nginx	/usr/local/nginx/sbin/nginx -r stop
重启nginx 	/usr/local/nginx/sbin/nginx -r reload
检查nginx	/usr/local/nginx/sbin/nginx -t
nginx配置文件	/usr/local/nginx/conf/nginx.conf
	

## 系统环境
操作系统：Linux version 3.10.0-229.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.2 20140120 (Red Hat 4.8.2-16) (GCC) )



## 操作步骤
### 1. 安装依赖
首先由于nginx的一些模块依赖一些lib库，所以在安装nginx之前，必须先安装这些lib库，这些依赖库主要有g++、gcc、openssl-devel、pcre-devel和zlib-devel 所以执行如下命令安装
```[root@iZ23f4c42gwZ bin]# yum -y install gcc-c++```

```[root@iZ23f4c42gwZ bin]# yum -y install pcre pcre-devel```

```[root@iZ23f4c42gwZ bin]# yum -y install zlib zlib-devel```

``` [root@iZ23f4c42gwZ bin]# yum -y install openssl openssl-devel```

```[root@iZ23f4c42gwZ bin]# yum -y install perl-devel perl-ExtUtils-Embed```

### 2. 安装nginx
```[root@iZ23f4c42gwZ bin]#find -name nginx (安装之前，最好检查一下是否已经安装有nginx)```

```[root@iZ23f4c42gwZ bin]#yum remove nginx(如果系统已经安装了nginx，那么就先卸载)```

```[root@iZ23f4c42gwZ bin]#cd /usr/local```

```[root@iZ23f4c42gwZ local]#cp  /soft/nginx-1.7.4.tar.gz  ./```

```[root@iZ23f4c42gwZ local]#tar  zxvf nginx-1.7.4.tar.gz```

```[root@iZ23f4c42gwZ local]#cd nginx-1.7.4```

```[root@iZ23f4c42gwZ nginx-1.7.4]# ./configure --with-http_perl_module --with-ld-opt="-Wl,-E" --with-http_ssl_module (默认安装在/usr/local/nginx，请求转发模块和https模块)```

>（注：这里解压完tar包后不要修改nginx-1.7.4名字为 nginx，因为之后编译的时候会再生成一个nginx，如果已修改名字，会出现文件重名冲突！（报错：cp: `conf/koi-win' and `/usr/local/nginx/conf/koi-win' are the same file））

>（ 注：在上面的configure选项中“--with-http_stub_status_module”可以用来启用 Nginx 的 NginxStatus 功能，以监控 Nginx 的当前状态。）

更多的安装配置 
>./configure --prefix=/usr/local/nginx 
--with-openssl=/usr/include (启用ssl) 
--with-pcre=/usr/include/pcre/ (启用正规表达式) 
--with-http_stub_status_module (安装可以查看nginx状态的程序) 
--with-http_memcached_module (启用memcache缓存) 
--with-http_rewrite_module (启用支持url重写)
  
```[root@iZ23f4c42gwZ nginx-1.7.4]# make && make install```

### 3. 启动nginx
```[root@iZ23f4c42gwZ nginx-1.7.4]# /usr/local/nginx/sbin/nginx ```
>(注：启动：确保系统的中的80 端口没被其他程序占用)

```[root@iZ23f4c42gwZ nginx-1.7.4]#netstat -ntlp|grep 80   (检查是否启动成功)```

```[root@iZ23f4c42gwZ nginx-1.7.4]# ps -ef | grep nginx    （检查nginx是否开启)```

```[root@iZ23f4c42gwZ nginx-1.7.4]# /usr/local/nginx/sbin/nginx –t （检查nginx是否配置正确）```

> （注：以下是停止 & 重载配置文件命令）

```[root@iZ23f4c42gwZ nginx-1.7.4]# /usr/local/nginx/sbin/nginx -s stop （停止Nginx服务）```

``` [root@iZ23f4c42gwZ nginx-1.7.4]# /usr/local/nginx/sbin/nginx -s reload （重新加载配置文件）```
 
>（注：可能出现问题：[error] open() "/usr/local/nginx/logs/nginx.pid" failed (2: No such file or directory)
解决方法：[root@localhost nginx]# /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
　　使用nginx -c的参数指定nginx.conf文件的位置）
 
``` [root@iZ23f4c42gwZ nginx-1.7.4]# vi /etc/rc.d/rc.local```
>（配置Nginx开机启动）
在文件末尾添加“/usr/local/nginx/sbin/nginx”
 
``` [root@iZ23f4c42gwZ nginx-1.7.4]# cd /usr/local/nginx/conf```

``` [root@iZ23f4c42gwZ nginx-1.7.4]# vi nginx.conf```
  

### 3. 关闭防火墙

``` sudo firewall-cmd --zone=public --add-port=888/tcp --permanent  ```
>（注：端口号根据项目情况而定）
sudo firewall-cmd --reload


### 4. 修改生产环境配置

Server下的listen表示监听端口，server name 表示监听域名(可以写IP），一般客户访问的方式是“域名：端口号”
Upstream 表示均衡负载，里面的 server 后面跟的是需要均衡负载的Tomcat服务器的IP地址




### 常见问题
#### 问题一：报错“cp: `conf/koi-win' and `/usr/local/nginx/conf/koi-win' are the same file”


#### 解决方法：

这里解压完tar包后不要修改nginx-1.7.4名字为 nginx，因为之后编译的时候会再生成一个nginx，如果已修改名字，会出现文件重名冲突！（报错：cp: `conf/koi-win' and `/usr/local/nginx/conf/koi-win' are the same file）



#### 问题二：[error] open() "/usr/local/nginx/logs/nginx.pid" failed (2: No such file or directory)
#### 解决方法：
[root@localhost nginx]# /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf    使用nginx -c的参数指定nginx.conf文件的位置


