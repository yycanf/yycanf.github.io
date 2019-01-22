---
layout:     post
title:      统计gradle官网下载页面所有all.zip结尾文件的总大小
subtitle:   统计gradle官网下载页面所有all.zip结尾文件的总大小
date:       2019-01-22
author:     BY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - 正则表达式
    - Linux
    - Android
---

## 统计[gradle](https://services.gradle.org/distributions/) 下载页面所有类似 gradle-5.1-all.zip这样的all.zip结尾的文件的总大小
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-1.jpg)
#### 主要用到的工具及知识：
notepad++，正则表达式，cat，awk，管道，lftp  
##### 1.ctrl+a 复制网页上所有的文字
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-2.jpg)
##### 2.粘贴到notepad++里面
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-3.jpg)
###### 3.掐头去尾  
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-4.jpg)
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-5.jpg)

##### 4.ctrl+f 然后点替换，使用正则表达式  
```
.*sha256.*
```
将所有sha256的行去除  
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-6.jpg)
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-7.jpg)


##### 5.同理，使用正则表达式  

```
^\s
```

将所有空白行去除  
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-8.jpg)
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-9.jpg)


##### 6.同理，使用正则表达式  

```
.*src.*
```

将所有src文件的行去除  
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-10.jpg)
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-11.jpg)


##### 7.使用正则表达式  

```
.*bin.*
```

将所有bin文件的行去除  
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-12.jpg)
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-13.jpg)


##### 8.同理，使用正则表达式  

```
^\s
```
 
将所有空白行去除  
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-14.jpg)
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-15.jpg)



##### 9.同理，使用正则表达式  

```
M$
```

将文件大小后面的“M”字母去掉（最好勾上匹配大小写）  
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-16.jpg)
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-17.jpg)



##### 10.随便打开一个ubuntu或者centos或者其他任何linux终端(Mac OSX没试过，貌似也可以？)，将结果复制粘贴到一个文件里  
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-18.jpg)


##### 11.使用cat,awk命令配合管道就能统计出总大小了  

```
cat test.txt |awk '{print $5}'|awk '{sum += $1};END {print sum}'  
```
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-19.jpg)
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-20.jpg)
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-21.jpg)
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-22.jpg)

大概15.1个G，也不算很大，那就同步到本地吧，省的以后还得从官网下  
然后搭配linux下的lftp工具，就可以同步到本地了  
lftp各大linux发行版的软件源里应该都有  
ubuntu(wsl大法好!)：  
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-23.jpg)
centos：  
![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-22-24.jpg)
Mac OSX下的安装参考：http://macappstore.org/lftp/  

##### 同步命令:
```
lftp "https://services.gradle.org/distributions/" -e "mirror --verbose -P 5 --only-newer --exclude-glob *.sha256 --exclude-glob *src.zip  --exclude-glob *bin.zip   --exclude-glob  gradle-1*    --exclude-glob  gradle-0* ; bye"
```

部分参数说明：  
 --exclude-glob 排除符合条件的文件，正常Android Studio用的gradle都是all.zip结尾的，其他的就没必要同步了,当然，也可以把一些很老的版本剔除掉  
-P N 并行下载N个文件  
 --only-newer 仅下载较新的文件  

相关链接：  
[https://lftp.yar.ru/lftp-man.html](https://lftp.yar.ru/lftp-man.html)  
[https://services.gradle.org/distributions/](https://services.gradle.org/distributions/)
