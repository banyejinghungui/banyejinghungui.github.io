---
layout: post
title: NFS部署文档
---


# NFS服务器部署文档
> 创建时间：2016-10-10

## 文档目的
部署NFS服务，实现不同操作系统之间文件共享功能。

## 基础知识
**NFS(Network Files System)** 即网络文件系统，它最大的功能就是可以透过网络，让不同的机器、不同的操作系统、可以彼此分享个别的档案 (share files)。NFS文件系统协议允许网络中的主机通过TCP/IP协议进行资源共享，NFS客户端可以像使用本地资源一样读写远端NFS服务端的资料，需要注意NFS服务依赖于RPC服务与外部通信，所以必需保证RPC服务能够正常注册服务的端口信息才能正常使用NFS服务。

## 常用命令
rpcinfo -p localhost   查看rpc 使用端口

service nfs start      开启nfs服务

showmount -e [IP]    查看指定IP的共享文件

## 系统环境
服务器操作系统：Linux version 3.10.0-229.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.2 20140120 (Red Hat 4.8.2-16) (GCC) )

服务器IP地址：172.16.1.61

客户机操作系统：Linux version 3.10.0-229.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.2 20140120 (Red Hat 4.8.2-16) (GCC) )

客户机IP地址：172.16.1.62

## 涉及软件
rpcbind
nfs-utils

## 配置文件
主要配置文件：/etc/exports
这个档案就是 NFS 的主要配置文件了！不过，系统并没有默认值，所以这个档
案『 不一定会存在』，你可能必须要使用 vim 主动的建立起这个档案喔！我们
等一下要谈的设定也仅只是这个档案而已吶！

NFS 文件系统维护指令：/usr/sbin/exportfs
这个是维护 NFS 分享资源的指令，我们可以利用这个指令重新分享
/etc/exports 变更的目录资源、将 NFS Server 分享的目录卸除或重新分享等
等，这个指令是 NFS 系统里面相当重要的一个喔！至于指令的用法我们在底下
会介绍。

分享资源的登录档：/var/lib/nfs/*tab
在 NFS 服务器的登录文件都放置到 /var/lib/nfs/ 目录里面，在该目录下有两
个比较重要的登录档， 一个是 etab ，主要记录了 NFS 所分享出来的目录的完
整权限设定值；另一个 xtab 则记录曾经链接到此 NFS 服务器的相关客户端数
据。

客户端查询服务器分享资源的指令：/usr/sbin/showmount
这是另一个重要的 NFS 指令。exportfs 是用在 NFS Server 端，而 showmount
则主要用在 Client 端。这个 showmount 可以用来察看 NFS 分享出来的目录资
源喔！

## 操作步骤
### 1. 安装所需程序
由于nfs依赖于RPC服务，所以在安装nfs前必须先安装RPC服务，安装命令如下：

`yum install rpcbind`

 

然后再安装nfs服务

`yum install nfs-utils`

 

### 2. 创建共享目录
假设我们要共享的目录是/nfsfiles

`mkdir /nfsfiles`


并设置权限

`chmod 777 /nfsfiles`

 

在该文件夹中放入一个文本，作为之后验证使用

`echo “this is my first nfs server” > /nfsfiles/README`

现在进入关键一步，需要把 /nfsfiles 文件共享，就需要修改/etc/exports 中的配置内容

`vim /etc/exports`

 

> 注：exports中的格式是 [共享文件路径] + [可以允许访问的IP地址或IP网段]+[访问权限] , 中间是用空格隔开。

NFS配置共享的参数有：
参数	作用
ro	只读默认
rw	读写模式
root_squash	当NFS客户端使用root用户访问时，映射为NFS服务端的匿名用户。
no_root_squash	当NFS客户端使用root用户访问时，映射为NFS服务端的root用户。
all_squash	不论NFS客户端使用任何帐户，均映射为NFS服务端的匿名用户。
sync	同时将数据写入到内存与硬盘中，保证不丢失数据。
async	优先将数据保存到内存，然后再写入硬盘，效率更高，但可能造成数据丢失。

共享之后的文件夹名字背景会有变化，如下图：
 
### 3. 关闭防火墙
> 以下端口必须开放：
111端口的TCP和UDP
2049端口的TCP

另外，rpc.mount进程所打开的端口，由于每次重启NFS服务这个进程端口都不一样，所以要根据每次开放的端口来设置，TCP和UDP都需要打开。可以通过`rpcinfo -p localhost`命令查看rpc.mount端口号

> 注：网上还有一种方法是修改/etc/sysconfig/nfs 文件中的RQUOTAD_PORT, LOCKD_TCPPORT,LOCKD_UDPPORT, MOUNTD_PORT 四个值来固定端口号，但是在我实际实验用没有效果，原因未知。。

 


> 注：添加完这些端口后，在客户端还是无法使用showmount -e [ip] 查看到NFS服务器上共享的文件，但是可以用mount -t nfs [ip]:/nfsfiles 挂载共享文件，原因未知。。。如果实在不行，可以关闭防火墙，service firewalld stop , 虽然不推荐。。

>firewall-cmd --zone=public --add-port=111/tcp --permanent
firewall-cmd --zone=public --add-port=111/udp --permanent
firewall-cmd --zone=public --add-port=2049/tcp --permanent
注：以下端口根据本机情况设定
firewall-cmd --zone=public --add-port=20048/tcp --permanent
firewall-cmd --zone=public --add-port=20048/udp --permanent
firewall-cmd --zone=public --add-port=54161/udp --permanent
firewall-cmd --zone=public --add-port=58138/tcp --permanent

> firewall-cmd --reload






### 4. 启动服务
启动RPC和nfs服务
> 注：如果不启动RPC服务，会出现下面问题一中的报错

`systemctl start rpcbind`

`systemctl start nfs-server`



 

### 5. 客户端挂载共享文件夹
linux端

客户端也需要安装相应软件，具体安装方法参见步骤1

将共享文件挂载到已经创建好了的文件夹/backupnfs中

`mount  -t  nfs  172.16.34.61:/nfsfiles/  /backupnfs/`

 

打开文件夹，验证挂载是否成功

 

以上结果表示访问成功



Windows端

> 注：鸟哥私房菜上说NFS只能在Linux操作系统之间共享文件，但是实际当中确实能在windows和Linux上共享，我在windows2012的系统中实验成功，但是windows10报错。。

在windows系统中需要先安装nfs服务器，安装方式是打开“控制面板”，选择“程序和功能”，点击“启动或关闭Windows功能”，勾选“NFS客户端”和“Telnet客户端”


 

如果是windows2012系统，则打开服务器管理工具，添加角色，添加“NFS客户端”和“Telnet客户端”两个角色即可
 

安装好后，打开cmd，输入命令
mount  \\172.16.34.61\nfsfiles   z:

 

 


> 注：貌似windows10的NFS支持不好，无法挂载，显示“不受支持的windows”，以上系统是windows2012操作系统

## 常见问题
### 问题一：nfs无法启动，rpc.nfsd	报错
 

### 解决方法：
因为nfs依赖于rpc服务，所以必须先启动rpc

systemctl start rpcbind

再启动nfs

systemctl start nfs-server



