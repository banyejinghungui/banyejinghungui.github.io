---
layout: post
title: FTP部署文档
---

# FTP部署文档
> 创建时间：2016.10.13
 

## 文档目的
使用vsftp服务实现ftp文件传输服务
基础知识
FTP 是文件传输协议 (File Transfer Protocol) 的简写，主要的功能是进
行服务器与客户端的档案管理、传输等事项；

命令通道的 ftp (默认为 port 21) 
数据传输的 ftp-data (默认为 port 20)。

vsftpd程序提供的FTP服务可选认证方式，分别为匿名访问、本地用户和虚拟用户：

匿名访问:任何人无需验证口令即可登入FTP服务端。
本地用户:使用FTP服务器中的用户、密码信息。
虚拟用户:创建独立的FTP帐号资料。

本地用户与虚拟用户则需要用户提供帐号及口令后才能登入FTP服务，更加的安全，而虚拟用户则是最安全的。

## 常用命令

`modprobe  `

加载ip_conntrack_ftp及ip_nat_ftp等模块，这几个模块会主动分析“目标是port21的联机”信息（链接的目标FTP服务器的命令通道必须是21，否则无法解析）
涉及软件

`vsftpd   yum install vsftpd -y`

`ftp      yum install ftp -y`







## 配置文件
主程序	  /usr/sbin/vsftpd
这就是 vsftpd 的主要执行档咯！不要怀疑， vsftpd 只有这一个执行档而已啊！

用户禁止登陆列表	/etc/vsftpd/ftpusers
                    /etc/vsftpd/user_list

/var/ftp/
这个是 vsftpd 的预设匿名者登入的根目录喔！其实与 ftp 这个账号的家目录
有关啦！

主配置文件 /etc/vsftpd/vsftpd.conf
严格来说，整个 vsftpd 的配置文件就只有这个档案！这个档案的设定是以 bash
的变量设定相同的方式来处理的， 也就是『参数=设定值』来设定的，注意， 等
号两边不能有空白喔！至于详细的 vsftpd.conf 可以使用 『 man 5
vsftpd.conf 』来详查。


vsftpd程序配置文件参数的作用：
参数	作用
listen=[YES|NO]	是否以独立运行的方式监听服务。
listen_address=IP地址	设置要监听的IP地址。
listen_port=21	设置FTP服务的监听端口。
download_enable＝[YES|NO]	是否允许下载文件。
userlist_enable=[YES|NO]
userlist_deny=[YES|NO]	是否启用“禁止登陆用户名单”。
max_clients=0	最大客户端连接数，0为不限制。
max_per_ip=0	同一IP地址最大连接数，0位不限制。
anonymous_enable=[YES|NO]	是否允许匿名用户访问。
anon_upload_enable=[YES|NO]	是否允许匿名用户上传文件。
anon_umask=022	匿名用户上传文件的umask值。
anon_root=/var/ftp	匿名用户的FTP根目录。
anon_mkdir_write_enable=[YES|NO]	是否允许匿名用户创建目录。
anon_other_write_enable=[YES|NO]	是否开放匿名用户其他写入权限。
anon_max_rate=0	匿名用户最大传输速率(字节)，0为不限制。
local_enable=[YES|NO]	是否允许本地用户登陆FTP。
local_umask=022	本地用户上传文件的umask值。
local_root=/var/ftp	本地用户的FTP根目录。
chroot_local_user=[YES|NO]	是否将用户权限禁锢在FTP目录，更加的安全。
local_max_rate=0	本地用户最大传输速率(字节)，0为不限制。


/etc/pam.d/vsftpd
这个是 vsftpd 使用 PAM 模块时的相关配置文件。主要用来作为身份认证之用，
还有一些用户身份的抵挡功能， 也是透过这个档案来达成的。你可以察看一下
该档案：

/etc/vsftpd/ftpusers
与上一个档案有关系，也就是 PAM 模块 (/etc/pam.d/vsftpd) 所指定的那个无
法登入的用户配置文件啊！ 这个档案的设定很简单，你只要将『不想让他登入
FTP 的账号』写入这个档案即可。一行一个账号。

/etc/vsftpd/user_list
这个档案是否能够生效与 vsftpd.conf 内的两个参数有关，分别是
『 userlist_enable, userlist_deny 』。 如果说 /etc/vsftpd/ftpusers 是
PAM 模块的抵挡设定项目，那么这个 /etc/vsftpd/user_list 则是 vsftpd 自
定义的抵挡项目。事实上这个档案与 /etc/vsftpd/ftpusers 几乎一模一样， 在
预设的情况下，你可以将不希望可登入 vsftpd 的账号写入这里。不过这个档案
的功能会依据 vsftpd.conf 配置文件内的 userlist_deny={YES/NO} 而不同，
这得要特别留意喔！

/etc/vsftpd/chroot_list
这个档案预设是不存在的，所以你必须要手动自行建立。这个档案的主要功能是
可以将某些账号的使用者 chroot 在他们的家目录下！但这个档案要生效与
vsftpd.conf 内的『 chroot_list_enable, chroot_list_file 』两个参数有关。
如果你想要将某些实体用户限制在他们的家目录下而不许到其他目录去，可以启
动这个设定项目喔！


## 系统环境
服务器操作系统：Linux version 3.10.0-229.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.2 20140120 (Red Hat 4.8.2-16) (GCC) )



## 操作步骤
### 1.安装vsftpd服务程序
`yum install vsftpd -y`

 

> 注：有些系统中ftp命令默认没有安装，需要自行安装，安装命令是
yum install ftp -y

 

### 2.关闭防火墙设置
`firewall-cmd --zone=public --add-port=21/tcp --permanent`

`firewall-cmd --zone=public --add-port=20/tcp --permanent`

`firewall-cmd --reload`

 
### 3.不同模式设置

a）匿名访问模式设置

复制原配置文件备份
cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak
vim /etc/vsftpd/vsftpd.conf
 


删除所有内容，写入以下语句
anonymous_enable=YES
anon_umask=022
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
write_enable=YES
 
> 注：确保语句的结尾没有多余的空格或者tab，否则可能导致服务无法启动
注：write_enable=YES 很重要，要加上（Linux就该这么学中的教程没有），否则会报550 权限决绝错误

 


设置SELinux

`setsebool -P ftpd_full_access=on`

 
> 注：如果不设置SELinux，在新建目录的时候会出现错误“550 Create directory operation failed.”


开启vsftpd服务

`systemctl start vsftpd`

客户端登入：
ftp 172.16.34.68
名字输入： anonymous
密码直接按回车




b）本地用户模式设置

复制原配置文件备份
`cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak`

`vim /etc/vsftpd/vsftpd.conf`
 


删除所有内容，写入以下语句
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
userlist_deny=YES
userlist_enable=YES
pam_service_name=vsftpd
 

> 注：一定要加上 pam_service_name=vsftpd ，否则可能出现客户端登入时无法登入，报错“530 Login incorrect.”

设置SELinux
`setsebool -P ftpd_full_access=on`


创建本地用户
`useradd ftpuser1`
`passwd P@$$word`


客户端登入：
ftp 172.16.34.68
名字输入:[服务器上系统useradd 创建的用户名]
密码:[服务器上系统创建的对应密码]
 




c）虚拟用户模式
创建生产FTP用户数据库的原始账号和密码文件

`vim /etc/vsftpd/vuser.list`
 
写入以下内容：

`redhat`  
`P@$$word`  
`centos`  
`P@$$word`  

 
> 注：单数行为账号，双数行为密码


使用db_load命令使用HASH算法生成FTP用户数据库文件vuser.db

`db_load  -T  -t  hash  -f  /etc/vsftpd/vuser.list  /etc/vsftpd/vuser.db`


查看数据库文件  
`file /etc/vsftpd/vuser.db `

 


设置文件权限    
`chmod 600 /etc/vsftpd/vuser.db`
 


删除原始文件  
`rm -f /etc/vsftpd/vuser.list`


创建FTP根目录及虚拟用户映射的系统用户  
`useradd -d /var/ftproot -s /sbin/nologin virtualftp`
 


修改虚拟用户家目录权限  
`chmod -Rf 755 /var/ftproot`

 
建立支持虚拟用户的PAM认证文件  
`vim /etc/pam.d/vsftpd.vu`

写入以下语句  
`auth       required     pam_userdb.so db=/etc/vsftpd/vuser
account    required     pam_userdb.so db=/etc/vsftpd/vuser`

 

> 注：参数db用于指向刚刚生成的vuser.db文件，但是不有写后缀


修改vsftpd.conf文件，添加支持虚拟模式配置  
`vim /etc/vsftpd/vsftpd.conf`

写入以下语句：  
`anonymous_enable=NO`  
`local_enable=YES`  
`guest_enable=YES`  
`guest_username=virtualftp`  
`pam_service_name=vsftpd.vu` 
`allow_writeable_chroot=YES`  
`write_enable=YES`  


 

为虚拟用户设置不同权限，此时redhat 和 centos 用户权限相同，默认不能上传、创建、修改文件，如果希望redhat能够拥有完整的管理文件权限，需要配置FTP程序独立用户权限

`vim /etc/vsftpd/vsftpd.conf`  
在最后加入以下语句  
`user_config_dir=/etc/vsftpd/vusers_dir`

 


创建用户独立的权限配置文件存放目录  
`mkdir /etc/vsftpd/vusers_dir/`


创建空白的redhat配置文件,指定centos用户具体权限  
`touch /etc/vsftpd/vusers_dir/redhat`

`vim /etc/vsftpd/vusers_dir/centos`
写入以下语句  
`anon_upload_enable=YES`  
`anon_mkdir_write_enable=YES`  
`anon_other_write_enable=YES`  

 


设置SELinux  
`setsebool -P ftpd_full_access=on`


重启服务  
`systemctl restart vsftpd`  
`systemctl enable vsftpd`  






## 常见问题
### 问题一：修改vsftpd.conf 后vsftpd无法启动

### 解决方法：
检查写入的权限字段后面有没有空格或者tab多余的输入。如果有，删掉。

### 问题二：无法在ftp下新建文件夹，显示“550 Permission denied.”

### 解决方法：  
1) 确认防火墙21和20端口已关闭（firewall-cmd --list-all）,没有关闭请参考步骤2；
2) 确认SELinux设置正确；
3) 确认匿名用户有足够权限的牧区  chmod 777 pub
4) 确认/etc/vsftp/vsftp.conf 配置中有语句 write_enable=YES


### 问题三：无法在ftp下新建目录，显示“550 Create directory operation failed.”

### 解决方法：
设置SELinux
setsebool -P ftpd_full_access=on

### 问题四：在本地用户模式下，客户端始终无法登入，报错“530 Login incorrect.”
 

### 解决方法： 
检查/etc/vsftpd/user_list和/etc/ftpusers，是这个文件/etc/vsftpd/vsftpd.conf 是否少了一行：
pam_service_name=vsftpd




## 参考文献
http://www.linuxprobe.com/chapter-11.html

http://bbs.csdn.net/topics/360014866
