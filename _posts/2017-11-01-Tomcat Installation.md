---
layout: post
title: Tomcat Installation
---

# Tomcat部署及发布文档
# Tomcat部署及发布文档
### 创建时间：2016-06-14
### 更新时间：2017.02.10

## 文档目的
在Linux上安装tomcat，并使用tomcat进行发布。

## 基础知识
Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP 程序的首选。

以下是tomcat下各目录的功能介绍
bin：存放启动和关闭Tomcat的可执行脚本。
conf：Tomcat的配置文件，如server.xml(Tomcat服务器配置文件)和web.xml(被所有webapps共享的配置文件)，密码忘了看 tomcat-users.xml。
webapps：存放web applications，用户自己需要部署的应用程序也放到此目录。
work：tomcat运行时生成的临时文件，包括jsp编译后产生的class文件等。
logs：存放日志文件。
temp：JVM用于存放临时文件的目录(java.io.tmpdir)。  

## 常用命令
开启tomcat(一定要进入到bin文件夹才能执行脚本，否则日志会出问题!）	cd tomcat/bin/
./startup.sh
关闭tomcat	cd tomcat/bin/
./shutdown.sh
查看tomcat日志	tail  -f  tomcat/logs/catalina.out
查看端口	netstat  -ntlp
关闭防火墙端口	firewall-cmd --zone=public --add-port=端口号/tcp --permanent

firewall-cmd --reload
访问系统心跳（判断发布是否成功,code=0表示发布成功）	curl http:// [IP address]/commons/ctrl/status



## 系统环境
操作系统：Linux version 3.10.0-229.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 		  4.8.2 20140120 (Red Hat 4.8.2-16) (GCC) ) centos 7


Java版本：Java(TM) SE Runtime Environment (build 1.8.0_45-b14)
          Java HotSpot(TM) 64-Bit Server VM (build 25.45-b02, mixed mode)

下载地址：172.16.34.21/Tools/java/jdk-8u45-linux-x64.gz
上传到服务器路径： /soft/src


tomcat版本：Tomcat-8.5.0

下载地址：172.16.34.21/Tools/Tomcat/apache-tomcat-8.5.0.tar.gz
上传到服务器路径：/soft/src


注意：在安装Tomcat之前必须先安装JDK（其中包含Java虚拟机（JVM），只要安装了JDK，就可以利用JVM解释这些字节码文件，从而保证了Java的跨平台性）。
JDK的版本和Tomcat版本兼容很重要，以上是我使用成功的两个版本

## 操作步骤

### 1. 安装JDK
#### 1）解压jdk压缩包

``` tar -zxvf src/jdk-8u45-linux-x64.gz ```
 

#### 2）重命名jdk文件夹

``` mv jdk1.8.0_45 jdk8```

 

#### 3)修改环境变量

在profile 文件最尾部插入以下变量
``` vi /etc/profile```

# JDK CONFIG

> export JAVA_HOME=/soft/jdk8

> export JAVA_BIN=/soft/jdk8/bin

> export PATH=$PATH:$JAVA_HOME/bin

> export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

> export JAVA_HOME JAVA_BIN PATH CLASSPATH

 

 
#### 4)执行更新

为了让刚修改的环境变量生效，需要执行以下语句

``` source /etc/profile ```

 


#### 5)查看环境变量

查看环境变量，检查刚刚修改的环境变量是否已经生效



``` echo $PATH```

 
### 2. 安装tomcat压缩包
#### 1）解压tomcat压缩包

```tar -zxvf src/apache-tomcat-8.0.5.tar.gz```

 

#### 2）重命名tomcat

```mv apache-tomcat-8.0.5 tomcat8_8081```

 


### 3.配置tomcat
假设新增的应用需要使用8081端口，默认的端口是8080，这时候需要进入tomcat的配置文件中，修改相应的参数。

```vi /soft/tomcat8_8081/conf/server.xml```

分别修改以下三个参数的端口号
SHUTDOWN的8005 改为 8181
HTTP/1.1 的8080 改为 8081
AJP/1.2 的8009 改为 8881

注：以上修改后的端口号可以按照自己的使用情况，选择没有被占用的端口号即可，但是这里建议尽量让三个端口号之间有关联，方便之后监控管理。







修改之前
 
 


修改之后

 
 



### 4.开启、关闭tomcat并设置防火墙
tomcat的开启和关闭很简单，在tomcat/bin的文件夹下面都有开启（startup.sh）和关闭（shutdown.sh）脚本。只要执行这两个脚本即可实现开启和关闭的效果。但是这里要注意的是，运行这两个脚本的时候，必须cd到bin这个文件夹下运行，否则日志会出现问题，具体原因不明。。。


开启tomcat

cd /soft/tomcat8_8081/bin
./startup.sh

 



关闭tomcat

cd /soft/tomcat8_8081/bin
./shutdown.sh
 

至此，tomcat的基本操作已经完成，但是如果此时你用外网连这台机器的8081端口，会出现错误页面，显示无法连接。原因是该服务器上的8081端口的防火墙没有关闭，导致外网无法访问。需要关闭防火墙的8081端口，才能连接。


关闭防火墙

firewall-cmd --zone=public --add-port=8081/tcp --permanent
firewall-cmd --reload

 

注：tomcat8支持软连接（修改conf下的content.xml配置文件）

### 5.测试tomcat是否成功发布

首先可以看端口是否开启
netstat -nltp
 
8881和8081已被监听，说明tomcat开启成功


再查看logs，看看有没有报错，输入以下语句
````tail -f /soft/tomcat8_8081/logs/catalina.out```
 
 
最后一段消息表示tomcat开启成功


用浏览器访问系统心跳，code=0，表示发布成功
http://172.16.34.101:8084/commons/ctrl/status

 
### 6.tomcat上发布war包
1. 在/soft 下新建一个文件夹war，用来存放上传的war包
mkdir /soft/war

2. 将打包好的war包上传到/soft/war文件目录下

3. 删除tomcat/webapps/ROOT 下的所有文件和文件夹

cd /soft/tomcat8_8081/webapps/ROOT
rm -rf *

 

4. 解压war包到ROOT目录下
cd /soft/tomcat8_8081/webapps/ROOT
unzip /soft/war/zoe-ijhealth-doctor-web.war
 

5.按照上面的方式开启tomcat（如果已经开启，先关闭tomcat，再开启），然后检测tomcat是否运行正常


## 常见问题
### 问题一：使用软链接发布的网站无法访问？
解答：建立软链接前需要修改以下配置
vim tomcat/conf/context.xml

 
 

/soft/tomcat9/conf/Catalina/localhost/ROOT.xml 

<?xml version="1.0" encoding="UTF-8"?>
<Context docBase="/home/www/zoe-rus-api" path="" reloadable="false">
    <Resources allowLinking="true"/>
</Context>



