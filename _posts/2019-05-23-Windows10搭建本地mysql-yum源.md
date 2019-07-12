---
layout:     post
title:      Windows10搭建本地mysql-yum源
date:       2019-05-23
author:     移影残风
catalog: true
tags:
    - mysql
    - shell
    - CentOS
---

### 以下操作可在wsl中进行

1. 同步索引信息

- mysql 8 

```bash
mkdir -p mysql8/repodata
cd mysql8/repodata
lftp "https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/repodata/" -e "mirror --verbose -P 5 --delete --only-missing; bye"
```
- mysql 5.7
```bash
mkdir -p mysql57/repodata
cd mysql57/repodata
lftp "https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql57-community-el7/repodata/" -e "mirror --verbose -P 5 --delete --only-missing; bye"
```

2. 获取最新的版本号  

- mysql 8

```bash
mysql8_version=$(lftp https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/ -e "cls;bye" | grep -e "mysql-community-client.*.el7.x86_64.rpm" | sed -r 's/mysql-community-client-(.*).el7.x86_64.rpm/\1/g' | sort -rV |  xargs | awk -F ' ' '{print $1}')
```

- mysql 5.7

```bash
mysql57_version=$(lftp https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql57-community-el7/ -e "cls;bye" | grep -e "mysql-community-client.*.el7.x86_64.rpm" | sed -r 's/mysql-community-client-(.*).el7.x86_64.rpm/\1/g' | sort -rV |  xargs | awk -F ' ' '{print $1}')
```

<!-- more -->

3. 在与repodata同级的目录下，下载各个依赖的rpm包
- mysql 8
```bash
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-client-${mysql8_version}.el7.x86_64.rpm
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-common-${mysql8_version}.el7.x86_64.rpm
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-devel-${mysql8_version}.el7.x86_64.rpm
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-embedded-compat-${mysql8_version}.el7.x86_64.rpm
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-libs-${mysql8_version}.el7.x86_64.rpm
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-libs-compat-${mysql8_version}.el7.x86_64.rpm
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-server-${mysql8_version}.el7.x86_64.rpm
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-test-${mysql8_version}.el7.x86_64.rpm
```
- mysql 5.7
```bash
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-client-${mysql57_version}.el7.x86_64.rpm
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-common-${mysql57_version}.el7.x86_64.rpm
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-devel-${mysql57_version}.el7.x86_64.rpm
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-embedded-compat-${mysql57_version}.el7.x86_64.rpm
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-libs-${mysql57_version}.el7.x86_64.rpm
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-libs-compat-${mysql57_version}.el7.x86_64.rpm
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-server-${mysql57_version}.el7.x86_64.rpm
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-test-${mysql57_version}.el7.x86_64.rpm
```


4. 在本地的windows上配置一个http服务器，将http路径指向刚刚操作的目录，

   我用的是Apache，参考配置：

   ```xml
   Alias /mirrors/mysql8 "E:/linux/repo/mysql8"
   <Directory E:/linux/repo/mysql8>
       Options Indexes
   	IndexOptions NameWidth=25 Charset=UTF-8 
       AllowOverride All
       Require all granted
   </Directory>
   
   Alias /mirrors/mysql57 "E:/linux/repo/mysql57"
   <Directory E:/linux/repo/mysql57>
       Options Indexes
   	IndexOptions NameWidth=25 Charset=UTF-8 
       AllowOverride All
       Require all granted
   </Directory>
   ```

### CentOS7虚拟机上yum源的配置

```bash
vim /etc/yum.repos.d/mysql-community.repo
# 粘贴如下内容
[mysql-connectors-community]
name=MySQL Connectors Community
baseurl=http://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-connectors-community-el7
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql-tools-community]
name=MySQL Tools Community
baseurl=http://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql-tools-community-el7
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql57-community]
name=MySQL 5.7 Community Server
#baseurl=http://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql57-community-el7
baseurl=http://192.168.5.1/mirrors/mysql57/
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql80-community]
name=MySQL 8.0 Community Server
#baseurl=http://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7
baseurl=http://192.168.5.1/mirrors/mysql8/
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```



### 自动同步脚本

```bash
#!/bin/bash
work_directory=$(pwd)

mysql_components=("client" "common" "devel" "embedded-compat" "libs" "libs-compat" "server" "test" )

# mysql8
mkdir -p ${work_directory}/mysql8/repodata
cd ${work_directory}/mysql8/repodata
lftp "https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/repodata/" -e "mirror --verbose -P 5 --delete --only-missing; bye"
cd ${work_directory}/mysql8
mysql8_version=$(lftp https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/ -e "cls;bye" | grep -e "mysql-community-client.*.el7.x86_64.rpm" | sed -r 's/mysql-community-client-(.*).el7.x86_64.rpm/\1/g' | sort -rV |  xargs | awk -F ' ' '{print $1}')

for mysql_component in ${mysql_components[@]};do

if [ -a "mysql-community-${mysql_component}-${mysql8_version}.el7.x86_64.rpm" ];then
echo "mysql-community-${mysql_component}-${mysql8_version}.el7.x86_64.rpm 已存在,跳过";
else
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql80-community-el7/mysql-community-${mysql_component}-${mysql8_version}.el7.x86_64.rpm
fi

done


# mysql5.7
mkdir -p ${work_directory}/mysql57/repodata
cd ${work_directory}/mysql57/repodata
lftp "https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql57-community-el7/repodata/" -e "mirror --verbose -P 5 --delete --only-missing; bye"
cd ${work_directory}/mysql57
mysql57_version=$(lftp https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql57-community-el7/ -e "cls;bye" | grep -e "mysql-community-client.*.el7.x86_64.rpm" | sed -r 's/mysql-community-client-(.*).el7.x86_64.rpm/\1/g' | sort -rV |  xargs | awk -F ' ' '{print $1}')


for mysql_component in ${mysql_components[@]};do

if [ -a "mysql-community-${mysql_component}-${mysql57_version}.el7.x86_64.rpm" ];then
echo "mysql-community-${mysql_component}-${mysql57_version}.el7.x86_64.rpm 已存在,跳过";
else
curl -O https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/mysql57-community-el7/mysql-community-${mysql_component}-${mysql57_version}.el7.x86_64.rpm
fi

done

```



