---
layout:     post
title:      CentOS7使用解压的方式手动安装mysql5.7
date:       2019-07-04
author:     移影残风
catalog: true
tags:
    - mysql
    - CentOS
---

### 下载mysql5.7  
```
wget https://mirrors.huaweicloud.com/mysql/Downloads/MySQL-5.7/mysql-5.7.26-linux-glibc2.12-x86_64.tar.gz
```

### 解压至任意目录
```
tar -zxvf mysql-5.7.26-linux-glibc2.12-x86_64.tar.gz -C /opt/
```

### 重命名一下
```
cd /opt
mv mysql-5.7.26-linux-glibc2.12-x86_64 mysql-5.7.26
```

### 创建my.cnf文件
```
cd /opt/mysql-5.7.26
touch my.cnf
```
### 写入以下配置信息
```
[mysqld]
port=3307

character-set-server=utf8mb4 
collation-server=utf8mb4_general_ci

innodb_buffer_pool_size = 64M
performance_schema=Off
table_definition_cache=400

datadir=/opt/mysql-5.7.26/data
socket=/opt/mysql-5.7.26/mysql.sock

log-error=/opt/mysql-5.7.26/log/mysqld.log
pid-file=/opt/mysql-5.7.26/mysqld.pid

[client] 
default-character-set=utf8mb4
```
### 创建必要目录
```
mkdir data log
```
### 执行初始化命令
```
cd bin
./mysqld --defaults-file=/opt/mysql-5.7.26/my.cnf --initialize-insecure  
```

### 创建一个普通用户，用来运行mysql，mysql无法直接在root用户下运行  

```
useradd mysql
# 授予权限
chown -R mysql:mysql /opt/mysql-5.7.26
```
### 创建systemd文件
```
vim /usr/lib/systemd/system/mysqld.service

[Unit]
Description=mysqld5.7
After=syslog.target
After=network.target

[Service]

Type=simple
User=mysql
Group=mysql
ExecStart=/opt/mysql-5.7.26/bin/mysqld --defaults-file=/opt/mysql-5.7.26/my.cnf --user=mysql
ExecStop=/bin/kill -s QUIT $MAINPID
Restart=always

```
### 启动mysql
```
systemctl daemon-reload
systemctl start mysqld
```

### 连接mysql，并修改密码
```
# 连接方式1，如果本机已经有了其他的mysql实例，连接时需加上 -S 参数
mysql -uroot -S /opt/mysql-5.7.26/mysql.sock
# 连接方式2, 如果是默认端口号3306则不用加 -P 参数
mysql -uroot -h 127.0.0.1 -P 3307

# 由于初始化时使用的是 --initialize-insecure 参数，而不是 --initialize 参数
# 所以无需密码即可连接，
mysql -uroot

解压版的mysql默认没有启用validate_password插件，这一点和安装版的不一样

# 为了提高安全性，可以启用该插件
# 启用插件的方式1：
进入mysql交互环境，输入以下命令，该方式即刻生效
INSTALL PLUGIN validate_password SONAME 'validate_password.so';

# 启用插件的方式2：修改my.cnf在[mysqld]下加一行，然后需要重启mysql服务
# 比如：
[mysqld]
plugin-load-add=validate_password.so


# 启用该插件后，修改密码时必须满足默认的密码策略强度：必须含有数字、大小写、特殊字符， 至少包含8个字符，
# 如果不想设置太复杂的密码，可以降低密码策略强度，
# 进入mysql交互环境
set global validate_password_policy=0;
#0/LOW：只检查长度;
#1/MEDIUM：检查长度、数字、大小写、特殊字符;
#2/STRONG：检查长度、数字、大小写、特殊字符字典文件。
set global validate_password_length=4;   #密码最小长度
set global validate_password_mixed_case_count=0;
#密码至少要包含的小写字母个数和大写字母个数;
set global validate_password_number_count=0;
# 注意，mysql服务重启后，上面的这几个变量会恢复成默认值

# 修改密码
alter user 'root'@'localhost' identified by '1111';

# 开启远程访问 
update mysql.user set host='%' where user='root' and host='localhost';
flush privileges;

```
