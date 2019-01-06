---
layout:     post
title:      Spring Boot 2.1.1配置HTTPS后启动时出现java.lang.UnsatisfiedLinkError解决方法
subtitle:   
date:       2019-01-06
author:     移影残风
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - Java
    - Spring Boot
    - Tomcat
---


###### 问题复现：
```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.1.RELEASE)

2019-01-06 19:13:18.455  INFO 6407 --- [           main] c.y.g.GoogleTranslateProxyApplication    : Starting GoogleTranslateProxyApplication v1 on iZ018i7q11hufgZ with PID 6407 (/root/springboot/google-translate-proxy-1-tomcat-9.0.13.jar started by root in /root/springboot)
2019-01-06 19:13:18.459  INFO 6407 --- [           main] c.y.g.GoogleTranslateProxyApplication    : No active profile set, falling back to default profiles: default
2019-01-06 19:13:21.608  INFO 6407 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8082 (https)
2019-01-06 19:13:21.648  INFO 6407 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2019-01-06 19:13:21.655  INFO 6407 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/9.0.13
2019-01-06 19:13:21.677  INFO 6407 --- [           main] o.a.catalina.core.AprLifecycleListener   : An older version [1.2.17] of the APR based Apache Tomcat Native library is installed, while Tomcat recommends a minimum version of [1.2.18]
2019-01-06 19:13:21.678  INFO 6407 --- [           main] o.a.catalina.core.AprLifecycleListener   : Loaded APR based Apache Tomcat Native library [1.2.17] using APR version [1.4.8].
2019-01-06 19:13:21.678  INFO 6407 --- [           main] o.a.catalina.core.AprLifecycleListener   : APR capabilities: IPv6 [true], sendfile [true], accept filters [false], random [true].
2019-01-06 19:13:21.678  INFO 6407 --- [           main] o.a.catalina.core.AprLifecycleListener   : APR/OpenSSL configuration: useAprConnector [false], useOpenSSL [true]
2019-01-06 19:13:21.685  INFO 6407 --- [           main] o.a.catalina.core.AprLifecycleListener   : OpenSSL successfully initialized [OpenSSL 1.0.2k-fips  26 Jan 2017]
2019-01-06 19:13:21.914  INFO 6407 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-01-06 19:13:21.914  INFO 6407 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 3271 ms
2019-01-06 19:13:22.434  INFO 6407 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-01-06 19:13:23.642  INFO 6407 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8082 (https) with context path ''
2019-01-06 19:13:23.656  INFO 6407 --- [           main] c.y.g.GoogleTranslateProxyApplication    : Started GoogleTranslateProxyApplication in 6.267 seconds (JVM running for 7.169)
2019-01-06 19:13:47.615 ERROR 6407 --- [nio-8082-exec-3] org.apache.tomcat.util.net.NioEndpoint   : 

java.lang.UnsatisfiedLinkError: org.apache.tomcat.jni.SSL.renegotiatePending(J)I
        at org.apache.tomcat.jni.SSL.renegotiatePending(Native Method) ~[tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at org.apache.tomcat.util.net.openssl.OpenSSLEngine.getHandshakeStatus(OpenSSLEngine.java:1021) ~[tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at org.apache.tomcat.util.net.openssl.OpenSSLEngine.wrap(OpenSSLEngine.java:457) ~[tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at javax.net.ssl.SSLEngine.wrap(SSLEngine.java:469) ~[na:1.8.0_192]
        at org.apache.tomcat.util.net.SecureNioChannel.handshakeWrap(SecureNioChannel.java:440) ~[tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at org.apache.tomcat.util.net.SecureNioChannel.handshake(SecureNioChannel.java:211) ~[tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1394) ~[tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_192]
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_192]
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at java.lang.Thread.run(Thread.java:748) [na:1.8.0_192]

2019-01-06 19:13:47.618 ERROR 6407 --- [nio-8082-exec-4] org.apache.tomcat.util.net.NioEndpoint   : 

java.lang.UnsatisfiedLinkError: org.apache.tomcat.jni.SSL.renegotiatePending(J)I
        at org.apache.tomcat.jni.SSL.renegotiatePending(Native Method) ~[tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at org.apache.tomcat.util.net.openssl.OpenSSLEngine.getHandshakeStatus(OpenSSLEngine.java:1021) ~[tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at org.apache.tomcat.util.net.openssl.OpenSSLEngine.wrap(OpenSSLEngine.java:457) ~[tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at javax.net.ssl.SSLEngine.wrap(SSLEngine.java:469) ~[na:1.8.0_192]
        at org.apache.tomcat.util.net.SecureNioChannel.handshakeWrap(SecureNioChannel.java:440) ~[tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at org.apache.tomcat.util.net.SecureNioChannel.handshake(SecureNioChannel.java:211) ~[tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1394) ~[tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_192]
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_192]
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-9.0.13.jar!/:9.0.13]
        at java.lang.Thread.run(Thread.java:748) [na:1.8.0_192]

^C2019-01-06 19:42:19.810  INFO 6407 --- [       Thread-3] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'


```
Spring Boot 2.1.1版本，配置完HTTPS后启动，启动过程中没有出现什么异常信息，但是浏览器无法访问并且一访问立马报了上面日志中的第二和第三段异常信息。  
于是二话不说，直接谷歌搜索“springboot java.lang.UnsatisfiedLinkError:org.apache.tomcat.jni.SSL.renegotiatePending(J)I”，从[第一个stackoverflow里面的回答](https://stackoverflow.com/questions/53596134/spring-boot-2-1-1-unsatisfiedlinkerror-org-apache-tomcat-jni-ssl-renegotiatepe) 就找到了解决方案（果然还是谷歌大法好！），主要是两种：  

###### 1.在pom.xml里指定tomcat的版本，覆盖springboot的默认配置  
Spring Boot 2.1.1集成的Tomcat 从9.0.12升级到了9.0.13。所以想继续使用9.0.12版本的话直接在properties下面指定tomcat.version为9.0.12就行了

```
    <properties>
        <tomcat.version>9.0.12</tomcat.version>
    </properties>
```
或者用Spring Boot 2.1.0的版本也行,但这毕竟不是长久之计，我们总不能都一直用着旧版本的吧  

###### 2.手动编译安装tomcat-native  
 如果你用的是CentOS你会发现epel源里面有已经有现成的tomcat-native包，但它的版本是1.2.17的，Spring Boot 2.1.1版本如果配置HTTPS的话，就必须至少是1.2.18版本的，而epel源目前还没有更新至1.2.18版本，所以只能手动编译安装了  
 
去[官网](http://tomcat.apache.org/download-native.cgi)下载源码，目前最新的版本是1.2.19  

如果已经通过yum源安装了把就先卸载安装的旧版本  

```
yum remove -y tomcat-native
```
按照[官方的手册](http://tomcat.apache.org/native-doc/)，先安装必要的依赖：

```
yum install apr-devel openssl-devel -y
```
解压，进入相应目录
```
cd /tmp
tar -zxvf tomcat-native-1.2.19-src.tar.gz
cd tomcat-native-1.2.19-src/native
mkdir ../output
```
  
configure
```
./configure --with-apr=/usr/bin/apr-1-config \
            --with-java-home=/usr/jdk1.8.0_192/ \
            --with-ssl=yes \
            --prefix=/tmp/tomcat-native-1.2.19-src/output
```
其中--with-java-home后面指定JDK的路径。理论上所有JDK都可以，但建议使用与Tomcat一起使用的JVM相同的JVM版本。  
--prefix指定编译后生成的文件的位置  

编译安装
```
make && make install
```
编译完成后生成文件的目录结构大概是这样的：  

![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-06-2.jpg)  
我们只需要把libtcnative-1.a  libtcnative-1.la  libtcnative-1.so  libtcnative-1.so.0  libtcnative-1.so.0.2.19这几个文件拷到/usr/lib64就可以了  

```
cd /tmp/tomcat-native-1.2.19-src/output/lib
ls
cp libtcnative-1.a  libtcnative-1.la  libtcnative-1.so  libtcnative-1.so.0  libtcnative-1.so.0.2.19 /usr/lib64
```
再次启动，可以看到springboot成功的加载了最新版本的tomcat-native   

![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-06-03.jpg)  

浏览器端也能正常访问了  

![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-06-4.jpg)


###### 附：
springboot配置HTTPS：  

```
#pem生成jks
openssl pkcs12 -export -in fullchain.pem -inkey privkey.pem -out fullchain_and_key.p12 -name test -passout pass:123456
keytool -importkeystore -deststorepass 123456 -destkeystore test.jks -srckeystore fullchain_and_key.p12 -srcstorepass 123456  -alias test  -srcstoretype PKCS12 
```
fullchain.pem和privkey.pem，正常配置apache的HTTPS会用到的2个文件，一个私钥文件和一个证书文件，可以在[letsencrypt](https://github.com/certbot/certbot)免费申请

在application.properties中配置：  
```
security.require-ssl = true
server.ssl.key-store=/root/test.jks
server.ssl.key-store-password=test
server.ssl.key-store-type=jks
server.ssl.key-alias:test
```  
也可以直接用第一步生成的pkcs12文件  
```
security.require-ssl = true
server.ssl.key-store=/root/fullchain_and_key.p12
server.ssl.key-store-password=test
server.ssl.key-store-type=PKCS12
server.ssl.key-alias:test
```
###### 小技巧：
vim 可以直接修改修改jar包的文本文件  

需要在Linux下安装zip，unzip

```
yum install -y unzip zip
```
就像这样：  

![image](https://yiyingcanfeng-blog-1253518847.cos.ap-shanghai.myqcloud.com/2019-01-06-01.gif)  
这样就可以在服务器里直接修改jar包里的一些配置文件或者是静态资源，省的本地再重新打包上传了，很方便啊有木有?
