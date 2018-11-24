title: Ubuntu上搭建Maven私服
date: 2018/05/01  20:46:25
tags: 潜龙在渊
categories: 聚沙成塔
---
关于为嘛要搭建Maven私服，套用大部分人的说法：
有些公司都不提供外网给项目组人员，因此就不能使用maven访问远程的仓库地址，所以很有必要在局域网里找一台有外网权限的机器，搭建nexus私服，然后开发人员连到这台私服上，这样的话就可以通过这台搭建了nexus私服的电脑访问maven的远程仓库。

下载Nexus开源版本，免费：[Nexus Repository OSS](https://www.sonatype.com/nexus-repository-oss)

#### 检查Java是否已经安装
nexus虽然只需要jre环境支持，但是方便以后的环境需要，所以直接配置好jdk环境
<!--more-->

终端上输入一下命令：
`$ java -version `

如果出现以下内容证明没有安装
```
root@vengao:~# java -version
 The program 'java' can be found in the following packages:
 * default-jre
 * gcj-5-jre-headless
 * openjdk-8-jre-headless
 * gcj-4.8-jre-headless
 * gcj-4.9-jre-headless
 * openjdk-9-jre-headless
Try: apt install <selected package>
```
附：[Jdk安装教程](https://visuper.cn/visuper-blog/2018/04/16/Ubuntu安装JDK8和Tomcat/)

#### 检查Maven是否安装
```
$ mvn --version
  Apache Maven 3.3.9
  Maven home: /usr/share/maven
  Java version: 1.8.0_151, vendor: Oracle Corporation
  Java home: /usr/lib/jvm/java-8-oracle/jre
  Default locale: en_US, platform encoding: ANSI_X3.4-1968
  OS name: "linux", version: "2.6.32-042stab127.2", arch: "amd64", family: "unix"
```
我的目前是安装了Maven的，但是官网推荐使用3.52版本，所以还是继续安装
```
$ sudo apt-get install maven
```
emmmm，再次检查版本，结果很不幸，这种安装并没有覆盖Ubuntu自带的Maven，进入`/usr/local/src` 目录下，获取maven安装文件
```
$ sudo cd /usr/local/src
$ sudo wget http://www-us.apache.org/dist/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz
```
提取maven.tar.gz文件，然后删除压缩文件
```
$ sudo tar -xf apache-maven-3.5.2-bin.tar.gz
$ sudo rm -f apache-maven-3.5.2-bin.tar.gz
```
你会得到新的目录apache-maven-3.5.2
#### 配置Maven环境变量
```
$ cd /etc/profile.d/ 
$ sudo vi maven.sh
```
粘贴以下内容
```
 export JAVA_HOME=/usr/lib/jvm/java-8-oracle
 export M2_HOME=/usr/local/src/apache-maven-3.5.2
 export MAVEN_HOME=/usr/local/src/apache-maven-3.5.2
 export PATH=${M2_HOME}/bin:${PATH}
 ```
 现在让`maven.sh`脚本可执行，然后通过运行`source`命令来应用配置
 ```
 $ sudo chmod +x maven.sh
 $ source maven.sh
 ```
 再次检查版本
 ```
 $ mvn --version
   Apache Maven 3.5.2 (138edd61fd100ec658bfa2d307c43b76940a5d7d; 2017-10-18T15:58:13+08:00)
   Maven home: /usr/local/src/apache-maven-3.5.2
   Java home: /usr/lib/jvm/java-8-oracle/jre
   Default locale: en_US, platform encoding: UTF-8
   OS name: "linux", version: "4.4.0-105-generic", arch: "amd64", family: "unix"
 ```