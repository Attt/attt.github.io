---
title: Class签名信息不匹配引起的依赖冲突
date: 2022-07-14 13:45:15
tags:
    - Java
    - maven
    - dependencies confliction
    - class load
    - class security check
categories:
    - Java
---

## 现象
线上某个服务升级zk到`3.4.14`, 引入依赖后无法启动。

报错信息：
```java
Caused by: java.lang.SecurityException: class "javax.annotation.ManagedBean"'s signer information does not match signer information of other classes in the same package
	at java.lang.ClassLoader.checkCerts(ClassLoader.java:891)
	at java.lang.ClassLoader.preDefineClass(ClassLoader.java:661)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:754)
	at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:142)
	at org.apache.catalina.loader.WebappClassLoaderBase.findClassInternal(WebappClassLoaderBase.java:2408)
	at org.apache.catalina.loader.WebappClassLoaderBase.findClass(WebappClassLoaderBase.java:855)
	at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1327)
	at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1180)
	at java.lang.Class.forName0(Native Method)
	at java.lang.Class.forName(Class.java:348)
```
## 解决
原因如错误信息所说，同package下有不同签名信息，属于依赖冲突。`javax.annotation.ManagedBean`的package是`javax.annotation`，找一下还有哪个jar里面有这个package：
![搜索图](https://cdn.jsdelivr.net/gh/attt/attt.github.io.pics@master/static/img/scrshot0.png)

然后找一下这两个包是什么依赖引入的（有一个肯定是zk😊):
```bash
mvn dependency:tree -Dverbose -Dincludes=com.google.code.findbugs,org.eclipse.jetty.orbit
```

```java
+- org.apache.zookeeper:zookeeper:jar:3.4.14:compile
|  \- com.github.spotbugs:spotbugs-annotations:jar:3.1.9:compile
|     \- com.google.code.findbugs:jsr305:jar:3.0.2:compile
\- com.dangdang:elastic-job-lite-lifecycle:jar:2.1.5:compile
   \- com.dangdang:elastic-job-common-restful:jar:2.1.5:compile
      \- org.eclipse.jetty.aggregate:jetty-all-server:jar:8.1.19.v20160209:compile
         +- org.eclipse.jetty.orbit:javax.servlet:jar:3.0.0.v201112011016:compile
         +- org.eclipse.jetty.orbit:javax.security.auth.message:jar:1.0.0.v201108011116:compile
         +- org.eclipse.jetty.orbit:javax.mail.glassfish:jar:1.4.1.v201005082020:compile
         |  \- (org.eclipse.jetty.orbit:javax.activation:jar:1.1.0.v201105071233:compile - omitted for duplicate)
         +- org.eclipse.jetty.orbit:javax.activation:jar:1.1.0.v201105071233:compile
         \- org.eclipse.jetty.orbit:javax.annotation:jar:1.1.0.v201108011116:compile
```

排掉任意一个就OK了，大概率是jetty做了签名，毕竟是个server（可执行代码）。

## 参考
> [Java SecurityException: signer information does not match](https://stackoverflow.com/questions/2877262/java-securityexception-signer-information-does-not-match)

```text
This happens when classes belonging to the same package are loaded from different JAR files, and those JAR files have signatures signed with different certificates - or, perhaps more often, at least one is signed and one or more others are not (which includes classes loaded from directories since those AFAIK cannot be signed).

So either make sure all JARs (or at least those which contain classes from the same packages) are signed using the same certificate, or remove the signatures from the manifest of JAR files with overlapping packages.
```

