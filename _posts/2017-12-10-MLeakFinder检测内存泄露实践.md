---
layout:     post
title:      MLeakFinder检测内存泄露实践
date:       2017-12-10 08:08:13
summary:    Wellcome To My Blog.
categories: study
thumbnail: cogs
tag:
 - 内存泄露
 - 优化

# MLeakFinder检测内存泄露实践
---

ARC确实减轻了程序员许多的内存管理工作，但是也为项目埋下了不少的隐患，没有怎么经历过MRC项目的程序员就会写下很多内存泄露的代码，Xcode自带工具Leaks是MRC时期产物，对于ARC环境经常出现的```Abandoned memory(Memory still referenced by your application that has no useful purpose)```是无法检测到的。更多的内存管理知识可以参读这篇[文章](http://blog.csdn.net/linyousong/article/details/51816140)。本文就是借助微信的一个团队WeRead开源的工具[MLeakFinder](https://github.com/Tencent/MLeaksFinder)来对实际项目做内存泄露的检测及优化，[官方博客](http://wereadteam.github.io/2016/02/22/MLeaksFinder/)自取，源码分析参读这篇[文章](http://www.jianshu.com/p/ccf3014ca6a6)。


##安装

MLeaksFinder是默认在Debug模式下运行的


###安装 JDK 和 MySQL

####安装 JDK

```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

执行上述命令后，在终端里面输入```java -version```，如能正常显示 JDK 版本则说明安装成功。

####安装 MySQL

```
sudo apt-get update
sudo apt-get install mysql-server
sudo mysql_secure_installation
```

安装完毕后，在终端中输入 ```systemctl status mysql.service```，如能有如下输出则说明安装成功：

<img src="http://upload-images.jianshu.io/upload_images/721001-0242e8a42dbfc1f3.png?imageMogr2/auto-orient/strip" width="400" height="130" alt="photos"/>

##安装 SonarQube

###1.解压安装包至安装目录

安装包[下载地址](https://www.sonarqube.org/downloads/)

###2.创建数据库及相应用户

####新建数据库

```
CREATE DATABASE sonar;
```

####创建用户并赋予权限

```
CREATE USER 'sonar' IDENTIFIED BY 'sonar’;
GRANT ALL ON sonar.* TO 'sonar'@'%' IDENTIFIED BY 'sonar’;
GRANT ALL ON sonar.* TO 'sonar'@'localhost' IDENTIFIED BY 'sonar’;
FLUSH PRIVILEGES;
```

####配置数据库访问参数

编辑 ```<install_directory>/conf/sonar.properties```，根据使用的数据库类型配置相应的参数，这里选用了 MySQL 数据库，因而需要配置其用户名、密码以及 jdbc url。

<img src="http://upload-images.jianshu.io/upload_images/721001-cfe0acae263be37f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="400" height="200" alt="photos"/>

####配置 Web Server

同样编辑 ```<install_directory>/conf/sonar.properties```，设置 Web Server host 主机的 ip、端口以及访问的上下文。

```
sonar.web.host=192.0.0.1
sonar.web.port=80
sonar.web.context=/sonar
```

上述配置完成后即可启动 SonarQube：

```
On Linux/Mac OS: bin/<YOUR OS>/sonar.sh start
On Windows: bin/windows-x86-XX/StartSonar.bat
```

在实际使用中会出现MySQL 数据库无法访问的情况，通常有如下几种问题：
1.用户没有权限，加权限即可

```
mysql>GRANT ALL PRIVILEGES ON *.* TO 'sonar'@'%' IDENTIFIED BY 'sonar' WITH GRANT OPTION;
```

2.默认3306端口只允许本地访问，修改 ```/etc/mysql/mysql.conf.d/mysqld.cnf```
注掉 ```bind-address = 127.0.0.1```

上述软件安装完毕后，SonarQube 就搭建完成，在浏览器中输入 ```http://192.0.0.1/sonar``` 即可使用SonarQube 来检查代码质量。

SonarQube 默认并不支持对 Objective-C 代码质量扫描，官方提供的插件每年需要5000欧，对于普通开发者来说太贵，而网上的一些先前的jar包已经过时，本君给大家一个当下能用的[jar插件]()，下载好后将jar包放置```<install_directory>/extensions/plugins``` 目录中，重启 SonarQube后在系统控制面板中如能看到如下内容则说明安装成功。

<img src="http://upload-images.jianshu.io/upload_images/721001-a03a9f36c4bba994.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="300" height="20" alt="photos"/>
 
##配置

sonar-project.properties

在项目根目录添加 sonar-project.properties，必要配置内容如下：

```
//数据库数据配置

# DATABASE
#
# IMPORTANT: the embedded H2 database is used by default. It is recommended for tests but not for
# production use. Supported databases are MySQL, Oracle, PostgreSQL and Microsoft SQLServer.

# User credentials.
# Permissions to create tables, indices and triggers must be granted to JDBC user.
# The schema must be created first.
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar

#----- Embedded Database (default)
# H2 embedded database server listening port, defaults to 9092
#sonar.embeddedDatabase.port=9092
#----- MySQL 5.6 or greater
# Only InnoDB storage engine is supported (not myISAM).
# Only the bundled driver is supported. It can not be changed.
sonar.jdbc.url=jdbc:mysql://172.16.0.2:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
```

```
//web配置
# Binding IP address. For servers with more than one IP address, this property specifies which
# address will be used for listening on the specified ports.
# By default, ports will be used on all IP addresses associated with the server.
sonar.web.host=172.16.0.2()

# Web context. When set, it must start with forward slash (for example /sonarqube).
# The default value is root context (empty value).
sonar.web.context=/sonar

# TCP port for incoming HTTP connections. Default value is 9000.
sonar.web.port=9000
```

##分析项目

###1.项目新建
运行SonarQube成功后，打开http://172.16.0.23:9007/sonar/，自定义需要扫描项目的一些信息


###2.环境配置
上面步骤完成后，网页会提示有两步，第一步是下载sonar-scanner，注意：下载后需要在个人.bash_profile文件进行配置

```
SONAR_PATH=/*你的sonar-scanner路径*/
PATH=$SONAR_PATH/bin:$PATH
export PATH
```


打开要进行代码分析的项目根目录，新建```sonar-project.properties```文件

输入以下信息

```
#----- Default SonarQube server（IP和端口）
sonar.host.url=172.16.0.2:9000

#----- Default source code encoding
#sonar.sourceEncoding=UTF-8

# must be unique in a given SonarQube instance
sonar.projectKey=joshpell2
# this is the name and version displayed in the SonarQube UI. Was mandatory prior to SonarQube 6.1.
sonar.projectName=WeiZi
sonar.projectVersion=2.6.7
 
# Path is relative to the sonar-project.properties file. Replace "\" by "/" on Windows.
# This property is optional if sonar.modules is set. 
sonar.sources=./Users/Joshpell/Desktop/WITH8-1的副本\ 2
 
# Encoding of the source code. Default is default system encoding
#sonar.sourceEncoding=UTF-8

sonar.login=admin
sonar.password=admin
```


然后再启动sonar-scanner文件夹下/bin/sonar-scanner这个命令行工具（直接拖到命令行工具，回车），分析完后结果如下：

<img src="http://pic.yupoo.com/joshpell/GRLfh2Zc/VcEK9.png" width="400" height="100" alt="photos"/>

报告上传后，SonarQube 会根据报告内容生成可视化的界面，描述每一项存在的问题，如可能存在的 Bug、代码缺陷、需要重构的代码块等等。

