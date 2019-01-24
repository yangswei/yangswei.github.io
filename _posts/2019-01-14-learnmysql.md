---
layout: post
title: "Mysql学习笔记-1:安装"
date: 2019-01-14  
description: "learn mysql 1"
tag: Mysql
---  

> 记录mysql读书日记

#### 安装mysql

二进制mysql[下载首页](https://dev.mysql.com/downloads/mysql/)  
选择下载版本,这里下载的是5.6.42的二进制包  
[5.6二进制包下载链接](wget https://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.42-linux-glibc2.12-x86_64.tar.gz)  

这里学习**使用源码安装mysql**[源码下载页](https://dev.mysql.com/downloads/mirrors/)，选择对应的链路下载，没有看到中国节点，可能需要翻墙或者去国内的开源镜像站下载[163开源镜像站](http://mirrors.163.com/)  
这里选择了个日本节点的5.6.40的源码包[5.6.40源码包](wget http://ftp.jaist.ac.jp/pub/mysql/Downloads/MySQL-5.6/mysql-5.6.40.tar.gz)  

##### 准备基础环境  

```bash
yum -y  install  gcc   gcc-c++  make
yum install ncurses-devel libaio-devel -y
#这里是centos7，其他平台去https://cmake.org/download/ 找编译好的二进制下载吧，我下的源码，装了几分钟
#源码 ./bootstrap --prefix=/usr;make;sudo make install
wget https://github.com/Kitware/CMake/releases/download/v3.13.2/cmake-3.13.2-Linux-x86_64.sh
wget https://github.com/Kitware/CMake/releases/download/v3.13.2/cmake-3.13.2-Linux-x86_64.tar.gz

```

##### 卸载系统自带数据库(如果有)

```x
rpm -qa|grep mysql*
```

##### 安装mysql

```
groupadd mysql
useradd -g mysql mysql -s /bin/false
mkdir /data/mysql
chown -R mysql:mysql /data/mysql
mkdir /usr/local/ysql  
#第一个参数指定安装的根目录，第二个指定数据存储目录，第三个指定配置文件目录
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/data/mysql \
-DSYSCONFDIR=/etc \
-DMYSQL_UNIX_ADDR=/usr/local/mysql/tmp/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_EXTRA_CHARSETS=all \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITHOUT_EXAMPLE_STORAGE_ENGINE=1 \
-DWITH_ZLIB=bundled \
-DWITH_SSL=bundled \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_EMBEDDED_SERVER=1 \
-DENABLE_DOWNLOADS=1 \
-DWITH_DEBUG=0
#编译后执行
make  
make install
```

安装mysql到安装根目录中`/usr/local/mysql/support-files`  

- 生成系统数据库./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql  
- 配置文件**my-default.cnf**,复制到指定的配置文件目录,修改配置参数  
- 系统启动文件**mysql.server**,添加到系统启动cp mysql.server /etc/init.d/mysqld(需要填入basedir和datadir分别对应编译安装时指定的安装根目录和数据存储目录)  
- 添加到系统环境变量vim /etc/profile; export PATH=$PATH:/usr/local/mysql/bin(或者把mysql/bin软链到/usr/local/bin中去)  

/etc/init.d/mysqld start  
报错：  
Segmentation fault  
编辑文件 cmd-line-utils/libedit/terminal.c  
把terminal_set方法中的 char buf[TC_BUFSIZE]; 这一行注释,再把 area = buf;改为 area = NULL;  