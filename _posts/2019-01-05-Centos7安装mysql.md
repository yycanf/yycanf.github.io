---
layout:     post
title:      Centos7安装mysql
subtitle:   
date:       2019-01-05
author:     移影残风
header-img: img/post-bg-debug.png
catalog: true
tags:
    - MySQL
    - Linux
---

[MySQL 5.7参考手册(官方)](https://dev.mysql.com/doc/refman/5.7/en/)
# 一、使用rpm包安装
## 更换阿里yum（个人习惯）
步骤：  
1）下载wget  
yum install -y wget  
2）备份默认的yum  
mv /etc/yum.repos.d /etc/yum.repos.d.backup  
3）设置新的yum目录  
mkdir /etc/yum.repos.d  
4）下载阿里yum配置到该目录中  
wget -O /etc/yum.repos.d/CentOS-Base.repo   http://mirrors.aliyun.com/repo/Centos-7.repo  
5）重建缓存  

```
yum clean all
yum makecache  
```
6）升级所有包（改变软件设置和系统设置，系统版本内核都升级，故需要几分钟耐心等待）  

```
yum update -y
```
以[mysql-5.7.23-1.el7.x86_64.rpm-bundle.tar](https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.23-1.el7.x86_64.rpm-bundle.tar)为例  
1.下载   
下载地址：  
官方：  
[https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.23-1.el7.x86_64.rpm-bundle.tar](https://note.youdao.com/)  
网易:  
[http://mirrors.163.com/mysql/Downloads/MySQL-5.7/mysql-5.7.23-1.el7.x86_64.rpm-bundle.tar](http://mirrors.163.com/mysql/Downloads/MySQL-5.7/mysql-5.7.23-1.el7.x86_64.rpm-bundle.tar)  
可以直接在centos里面用wget命令下载，或者在自己电脑上下完再传上去  
2.解开tar包  

```
tar -xvf mysql-5.7.23-1.el7.x86_64.rpm-bundle.tar
```
3.移除centos7自带的mariadb-libs包，不移除的话安装会出错  

```
yum remove -y mariadb-libs
```
4.使用yum localinstall命令进行本地安装，按照顺序执行  
```
yum localinstall -y mysql-community-common-5.7.23-1.el7.x86_64.rpm 
yum localinstall -y mysql-community-libs-5.7.23-1.el7.x86_64.rpm 
yum localinstall -y mysql-community-client-5.7.23-1.el7.x86_64.rpm 
yum localinstall -y mysql-community-server-5.7.23-1.el7.x86_64.rpm 
yum localinstall -y mysql-community-libs-compat-5.7.23-1.el7.x86_64.rpm
```
期间yum会自动为我们安装所有的依赖包  

5.启动mysql服务并允许开机自启  

```
systemctl start mysqld
systemctl enable mysqld
```
6.查询mysql生成的临时的管理员账号密码（12位）  
```
grep 'temporary password'  /var/log/mysqld.log
```
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-06-1.png)  

7.使用生成的临时密码登录mysql  

```
mysql -uroot -p
#(输入上面的初始密码)
```
(1)修改密码  

```
alter user  root@localhost identified by 'MyNewPass4!';
```
#### 注意
MySQL的 validate_password 插件是默认安装的。要求密码至少包含一个大写字母，一个小写字母，一个数字和一个特殊字符，并且总密码长度至少为8个字符。如果设置的密码强度不符合要求会报错

**(2)降低密码策略强度，然后将密码修改简单一点，便于记忆(不建议在生产环境下这么做)**


```
set global validate_password_policy=0;
#0/LOW：只检查长度;
#1/MEDIUM：检查长度、数字、大小写、特殊字符;
#2/STRONG：检查长度、数字、大小写、特殊字符字典文件。
set global validate_password_length=4;   #密码最小长度
set global validate_password_mixed_case_count=0;
#密码至少要包含的小写字母个数和大写字母个数;
set global validate_password_number_count=0;
#密码至少要包含的数字个数

#修改密码：
set password=password('1111');

```
(3)设置允许远程访问  
```
update mysql.user set host='%' where user='root';
```
8.设置用户或更改密码后需用flush   privileges刷新MySQL的系统权限相关表，否则会出现拒绝访问  

```
flush privileges;
```

或者exit然后  

```
systemctl restart mysqld;
```
重启mysql服务也行  
官方文档：[https://dev.mysql.com/doc/refman/5.7/en/linux-installation-yum-repo.html](https://dev.mysql.com/doc/refman/5.7/en/linux-installation-yum-repo.html)
# 二、使用MySQL Yum Repository在centos上安装MySQL
#### (确保网络畅通)
下载地址：[https://repo.mysql.com//mysql57-community-release-el7-11.noarch.rpm](https://repo.mysql.com//mysql57-community-release-el7-11.noarch.rpm)  
1.下载发行包  

```
wget https://repo.mysql.com//mysql57-community-release-el7-11.noarch.rpm
```


2.安装MySQL Yum Repository  

```
yum install -y https://repo.mysql.com//mysql57-community-release-el7-11.noarch.rpm
```


3.安装（期间要联网下载）  

```
yum install mysql-community-server
```

之后同上第五步及之后的步骤  
一键安装：  

```
wget https://repo.mysql.com//mysql57-community-release-el7-11.noarch.rpm 
yum localinstall -y mysql57-community-release-el7-11.noarch.rpm
yum install mysql-community-server -y
```
也可以使用清华大学的mysql源  

```
#配置清华大学的mysql yum源
touch /etc/yum.repos.d/mysql-community.repo
cat > /etc/yum.repos.d/mysql-community.repo <<- "EOF"
[mysql-connectors-community]
name=MySQL Connectors Community
baseurl=http://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-connectors-community-el7
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql-tools-community]
name=MySQL Tools Community
baseurl=http://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-tools-community-el7
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql57-community-el7
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=http://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

EOF
yum makecache
yum install mysql-community-server -y
```

#### 一键安装配置shell脚本：  

```
yum install -y wget vim
#更换阿里的源(非必要)
mv /etc/yum.repos.d /etc/yum.repos.d.backup
mkdir /etc/yum.repos.d
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache
yum update -y

#配置清华大学的mysql yum源
touch /etc/yum.repos.d/mysql-community.repo
cat > /etc/yum.repos.d/mysql-community.repo <<- "EOF"
[mysql-connectors-community]
name=MySQL Connectors Community
baseurl=http://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-connectors-community-el7
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql-tools-community]
name=MySQL Tools Community
baseurl=http://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-tools-community-el7
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql57-community-el7
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=http://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

EOF
yum makecache
yum install mysql-community-server -y

systemctl start mysqld
systemctl enable mysqld
passlog=$(grep 'temporary password'  /var/log/mysqld.log)
pass=${passlog:${#passlog}-12:${#passlog}}
mysql -uroot -p"${pass}" -e"alter user root@localhost identified by 'QQQqqq111...' " --connect-expired-password
pass=QQQqqq111...
mysql -uroot -p"${pass}" -e"set global validate_password_policy=0;" --connect-expired-password
mysql -uroot -p"${pass}" -e"set global validate_password_length=4;" --connect-expired-password
mysql -uroot -p"${pass}" -e"set global validate_password_mixed_case_count=0;" --connect-expired-password
mysql -uroot -p"${pass}" -e"set global validate_password_number_count=0;" --connect-expired-password
#echo 'enter your mysql password'
#read password
password=1111
mysql -uroot -p"${pass}" -e"set password=password('${password}');" --connect-expired-password
mysql -uroot -p"${password}" -e"update mysql.user set host='%' where user='root';" --connect-expired-password
mysql -uroot -p"${password}" -e"flush privileges;" --connect-expired-password
```
