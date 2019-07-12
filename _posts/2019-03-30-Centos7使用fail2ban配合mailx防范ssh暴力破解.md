---
layout:     post
title:      Centos7使用fail2ban配合mailx防范ssh暴力破解
date:       2019-03-30
author:     移影残风
catalog: true
tags:
    - CentOS
    - Linux
---

###  一、安装mailx

```bash
yum install -y mailx  
```

配置smtp，修改/etc/mail.rc  
```bash
vim /etc/mail.rc  
```
在最下面加入如下内容  
```bash
set from=你的邮箱
#以QQ邮箱为例，如果是163就换成smtps://smtp.qq.com:465，gmail暂时没试过
set smtp=smtps://smtp.qq.com:465
set smtp-auth-user=你的邮箱
set smtp-auth-password=你的授权码
set smtp-auth=login
set ssl-verify=ignore
set nss-config-dir=/etc/pki/nssdb
```

### 二、安装fail2ban
####  (1)使用yum安装  
```bash
yum install -y epel-release  
yum install -y fail2ban  
```
<!-- more -->
启动服务  
```bash
systemctl start fail2ban  
```
允许开机自启  
```bash
systemctl enable fail2ban  
```
#### (2)源码安装

最好先update一下
```bash
yum update 
git clone https://github.com/fail2ban/fail2ban.git
cd fail2ban
python setup.py install 
```
如果没有git，安装git
```bash
yum install -y git
```
安装完后将fail2ban添加到systemd服务
```bash
cp build/fail2ban.service /usr/lib/systemd/system/fail2ban.service
```
对于新创建的unit文件，或者修改了的unit文件 需要重载配置文件
```bash
systemctl daemon-reload
```
启动服务
```bash
systemctl start fail2ban
```
允许开机自启
```bash
systemctl enable fail2ban
```
### 三、配置jail.conf文件
```bash
vim /etc/fail2ban/jail.conf
```
在文件开头加入如下内容
```bash
#针对各服务的检查配置，如设置bantime、findtime、maxretry和全局冲突，服务优先级大于全局设置
[ssh-iptables]  
#是否激活此项（true/false）
enabled  = true                          
#过滤规则filter的名字，对应filter.d目录下的sshd.conf 
filter   = sshd                          
#动作的相关参数
action   = iptables[name=SSH, port=ssh, protocol=tcp] 
                mail-whois[name=SSH, dest=你的邮箱] 
#                                   触发报警的收件人  
#检测的系统的登陆日志文件 
logpath  = /var/log/secure                
#最大尝试次数
maxretry = 2      
```
![](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-03-30-01.png)  

在[DEFAULT]子项里有一些属性也建议修改一下  

```bash
#ip 被封禁的时间，单位：s(秒)，m(分钟)。
bantime  = 86400s  
#单位：s(秒)，m(分钟)。如果8000s内达到maxretry(即最大尝试次数)，ip就会被封禁。
findtime  = 8000s
#最大尝试次数，如果达到这个次数，ip就会被封禁
maxretry = 2
```
![](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-03-30-02.png)

查找mailx命令的位置

```bash
whereis mailx
```
![](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-03-30-03.png)

**/usr/bin/mailx**  就是mailx所在的绝对路径

编辑 /etc/fail2ban/action.d/mail-whois.conf
如果是通过yum安装的，可能会不存在此配置文件，那么就用git将源代码克隆下来

```bash
git clone https://github.com/fail2ban/fail2ban.git
```
配置文件都在fail2ban/config/action.d/下面，  

![](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-03-30-04.png)

将缺少的配置文件复制到/etc/fail2ban/action.d/下面即可   
```bash
cp config/action.d/mail-whois.conf  /etc/fail2ban/action.d/mail-whois.conf

vim /etc/fail2ban/action.d/mail-whois.conf
```
![](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-03-30-05.png)

将图中的三处mail替换成mailx所在的绝对路径，如图

![](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-03-30-06.png)

重启fail2ban
```bash
systemctl restart fail2ban
```
用另一台虚拟机测试，可以看到在2次尝试登陆失败后，fail2ban立刻就生效了，再次尝试连接时就直接被拒绝了

![](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-03-30-07.png)
![](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-03-30-08.png)
![](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-03-30-09.png)

查看fail2ban的运行状态，可以看到一个jail(监狱)正在运行中
```bash
fail2ban-client status 
```
![](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-03-30-10.png)

查看jail的详细信息，可以看到被封禁的ip
```bash
fail2ban-client status ssh-iptables
```
![](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-03-30-11.png)

解封ip：
```bash
fail2ban-client set ssh-iptables unbanip 192.168.5.10
```
![](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-03-30-12.png)

**注意！如果修改过系统时间的时区，日志的记录时间还是以之前的时区进行记录日志，/var/log/secure记录的时间跟系统时间不对的话，fail2ban的核心功能就会完全失效，也就是说封禁不了ip。所以修改过系统时间的时区之后，一定要重启一下系统日志服务(rsyslog)**
``` bash
systemctl restart rsyslog
```

参考资料：
[https://my.oschina.net/plutonji/blog/191683](https://my.oschina.net/plutonji/blog/191683)
[https://www.fail2ban.org/wiki/index.php/MANUAL_0_8](https://www.fail2ban.org/wiki/index.php/MANUAL_0_8)
[https://linux.cn/article-5067-1.html](https://linux.cn/article-5067-1.html)
[https://www.cnblogs.com/wangxiaoqiangs/p/5630325.html](https://www.cnblogs.com/wangxiaoqiangs/p/5630325.html)	
[https://www.cnblogs.com/jasmine-Jobs/p/5927968.html](https://www.cnblogs.com/jasmine-Jobs/p/5927968.html)
[http://www.cszhi.com/20131101/fail2ban.html](http://www.cszhi.com/20131101/fail2ban.html)

