---
layout: post
title: "Mysql最佳实践"
date: 2019-01-14  
description: "learn mysql 1"
tag: Mysql
---  

> 记录mysql读书日记

#### 安装mysql


`$ curl -LO http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm`
安装 mysql 源

`$ sudo yum localinstall mysql57-community-release-el7-11.noarch.rpm`
检查 yum 源是否安装成功

```a

$ sudo yum repolist enabled | grep "mysql.*-community.*"
mysql-connectors-community           MySQL Connectors Community              21
mysql-tools-community                MySQL Tools Community                   38
mysql57-community                    MySQL 5.7 Community Server             130
```

如上所示，找到了 mysql 的安装包

1. 安装
```
$ sudo yum install mysql-community-server
$ sudo systemctl enable mysqld
$ sudo systemctl start mysqld
$ sudo systemctl status mysqld
```
2. 修改root默认密码

MySQL 5.7 启动后，在`/var/log/mysqld.log`文件中给 root 生成了一个默认密码,通过下面的方式找到 root 默认密码,然后登录 mysql 进行修改:

```b
$ grep 'temporary password' /var/log/mysqld.log
[Note] A temporary password is generated for root@localhost: **********
```

登录 MySQL 并修改密码  

```c
$ mysql -u root -p
Enter password: 
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
注意：MySQL 5.7 默认安装了密码安全检查插件（validate_password），默认密码检查策略要求密码必须包含：大小写字母、数字和特殊符号，并且长度不能少于 8 位。
```

通过 MySQL 环境变量可以查看密码策略的相关信息:  

```e

mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_check_user_name    | OFF    |
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.01 sec)
具体修改，参见 http://dev.mysql.com/doc/refman/5.7/en/validate-password-options-variables.html#sysvar_validate_password_policy
```

指定密码校验策略  

```f
$ sudo vi /etc/my.cnf

[mysqld]
# 添加如下键值对, 0=LOW, 1=MEDIUM, 2=STRONG
validate_password_policy=0
禁用密码策略

$ sudo vi /etc/my.cnf
	
[mysqld]
# 禁用密码校验策略
validate_password = off
重启 MySQL 服务，使配置生效

$ sudo systemctl restart mysqld
```

3. 添加远程登录用户  

MySQL默认只允许 root 帐户在本地登录,如果要在其它机器上连接 MySQL,必须修改 root 允许远程连接,或者添加一个允许远程连接的帐户,为了安全起见,本例添加一个新的帐户:  

`mysql> GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' IDENTIFIED BY 'secret' WITH GRANT OPTION;`

4. 配置默认编码为 utf8  

MySQL 默认为 latin1, 一般修改为 UTF-8 

```
$ vi /etc/my.cnf
[mysqld]
# 在myslqd下添加如下键值对
character_set_server=utf8
init_connect='SET NAMES utf8'
```
重启 MySQL 服务，使配置生效  
`$ sudo systemctl restart mysqld`  

查看字符集  

```
mysql> SHOW VARIABLES LIKE 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec
```

5. 开启端口  

```
$ sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
$ sudo firewall-cmd --reload
```

### 简易方式快速启动

宿主机上执行  
```
curl -LO http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
yum install mysql-community-client -y 
yum install  docker docker-compose -y
mkdir -p /mysql
mkdir -p /mysql/data
mkdir -p /mysql/config
```

将my.conf复制到/mysql/config下  
将docker-compose.yml拷贝到/mysql下  
my.cnf配置如下（.cnf结尾）  

```my.cnf
# The MySQL database server configuration file.
#
# You can copy this to one of:
# - "/etc/mysql/my.cnf" to set global options,
# - "~/.my.cnf" to set user-specific options.
#
# One can use all long options that the program supports.
# Run program with --help to get a list of available options and with
# --print-defaults to see which it would actually understand and use.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

# This will be passed to all mysql clients
# It has been reported that passwords should be enclosed with ticks/quotes
# escpecially if they contain "#" chars...
# Remember to edit /etc/mysql/debian.cnf when changing the socket location.

# Here is entries for some specific programs
# The following values assume you have at least 32M ram

[mysqld_safe]
socket		= /var/run/mysqld/mysqld.sock
nice		= 0

[mysqld]
#
# * Basic Settings
#
user		= mysql
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
port		= 3306
basedir		= /usr
datadir		= /var/lib/mysql
tmpdir		= /tmp
lc-messages-dir	= /usr/share/mysql
skip-external-locking
skip-grant-tables
#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address		= 0.0.0.0
#
# * Fine Tuning
#
key_buffer_size		= 16M
max_allowed_packet	= 16M
thread_stack		= 192K
thread_cache_size       = 8
# This replaces the startup script and checks MyISAM tables if needed
# the first time they are touched
myisam-recover-options  = BACKUP
#max_connections        = 100
#table_cache            = 64
#thread_concurrency     = 10
#
# * Query Cache Configuration
#
query_cache_limit	= 1M
query_cache_size        = 16M
#
# * Logging and Replication
#
# Both location gets rotated by the cronjob.
# Be aware that this log type is a performance killer.
# As of 5.1 you can enable the log at runtime!
#general_log_file        = /var/log/mysql/mysql.log
#general_log             = 1
#
# Error log - should be very few entries.
#
log_error = /var/log/mysql/error.log
#
# Here you can see queries with especially long duration
#log_slow_queries	= /var/log/mysql/mysql-slow.log
#long_query_time = 2
#log-queries-not-using-indexes
#
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
#server-id		= 1
#log_bin			= /var/log/mysql/mysql-bin.log
expire_logs_days	= 10
max_binlog_size   = 100M
#binlog_do_db		= include_database_name
#binlog_ignore_db	= include_database_name
#
# * InnoDB
#
# InnoDB is enabled by default with a 10MB datafile in /var/lib/mysql/.
# Read the manual for more InnoDB related options. There are many!
#
# * Security Features
#
# Read the manual, too, if you want chroot!
# chroot = /var/lib/mysql/
#
# For generating SSL certificates I recommend the OpenSSL GUI "tinyca".
#
# ssl-ca=/etc/mysql/cacert.pem
# ssl-cert=/etc/mysql/server-cert.pem
# ssl-key=/etc/mysql/server-key.pem
```

docker-compose文件   

```compose
version: '3'
services:
  mysql:
    image: mysql:5.7
    container_name: mysql
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci #设置utf8字符集
    restart: always
    ports:
      - 6606:3306
    volumes:
      - "/mysql/data:/var/lib/mysql"
      - "/mysql/config:/etc/mysql/conf.d"
    environment:
      MYSQL_ROOT_PASSWORD: "123456"   #root管理员用户密码
      MYSQL_USER: test   #创建test用户
      MYSQL_PASSWORD: test  #设置test用户的密码
```
  
docker-compose up -d 启动  
宿主机登录mysql: mysql -uroot -P6606 -p -h127.0.0.1  

报错`Access denied for user 'root'@'localhost' (using password: YES)`  
配置文件添加:skip-grant-tables  
进入数据量,更新密码`update mysql.user set authentication_string=password('*******') where user='*******'`

