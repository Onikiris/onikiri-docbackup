title: Ubuntu安装JDK和Tomcat
date: 2018/04/16  20:53:25
tags: 潜龙在渊
categories: 聚沙成塔
---
#### Java Development Kit安装：
安装的jdk有很多选择，我选择的是Oracle JDK，通过ppa源方式安装，源方式可以通过 apt-get upgrade 方式方便获得jdk的升级

##### 添加ppa
```
$ sudo add-apt-repository ppa:webupd8team/java
```
当我执行这条命令的时候出现了问题
```
vi@vengao:/root$ sudo add-apt-repository ppa:webupd8team/java
The program 'add-apt-repository' is currently not installed. You can install it by typing:
apt install software-properties-common
```
<!--more-->
按照提示执行后解决问题：
```
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
```

##### 安装oracle-java-installer

##### JDK8
```
$ sudo apt-get install oracle-java8-installer
```
##### JDK7
```
$ sudo apt-get install oracle-java7-installer
```
安装器会提示你同意 oracle 的服务条款，选择 ok，选择yes ,然后坐等下载

##### 设置系统默认jdk

```
$ sudo update-java-alternatives -s java-8-oracle
```
##### 检查版本信息
```
vi@vengao:/root$ java -version
java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
```

#### Tomcat安装：
##### 创建tomcat用户

创建一个新tomcat组：
```
sudo groupadd tomcat
```
接下来，创建一个新`tomcat`用户。我们将使该用户成为该`tomcat`组的成员，并具有一个主目录`/opt/tomcat`（我们将在其中安装Tomcat）以及一个`/bin/false`（没有人可以登录该帐户）：
```
sudo useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat
```
##### tomcat下载
tomcat就不选择源方式安装了，从官网上查找下载：在 8.5.30->Binary Distributions->Core 分类中的 tar.gz包

链接：[apache-tomcat-8.5.30](http://tomcat.apache.org/download-80.cgi#8.5.30)

懒得在网页上去下载，使用curl下载
切换到/tmp服务器上的目录。这个目录来一般用于下载临时文件
```
$ cd /tmp
$ curl -O http://apache.mirrors.ionfish.org/tomcat/tomcat-8/v8.5.30/bin/apache-tomcat-8.5.30.tar.gz
```
等待下载完毕即可
将Tomcat安装到/opt/tomcat目录。
```
$ sudo mkdir /opt/tomcat
$ sudo tar xzvf apache-tomcat-8.5.30.tar.gz -C /opt/tomcat --strip-components=1
```
##### 更新权限

切换到解压Tomcat安装的目录：
```
$ cd /opt/tomcat
```
给tomcat整个安装目录组的所有权：
```
$ sudo chgrp -R tomcat /opt/tomcat
```
接下来，让该tomcat组读取对该conf目录及其所有内容的访问权限，并执行对该目录本身的访问：
```
$ sudo chmod -R g+r conf
$ sudo chmod g+x conf
```
让tomcat用户获取以下目录权限 webapps，work，temp，logs：
```
$ sudo chown -R tomcat webapps/ work/ temp/ logs/
```
##### 创建一个systemd服务文件
将Tomcat作为服务运行，配置systemd服务文件。
Tomcat需要知道Java的安装位置，也就是`JAVA_HOME`，查找该位置的最简单方法是运行以下命令：
```
$ sudo update-java-alternatives -l
```
```
java-8-oracle                  1081       /usr/lib/jvm/java-8-oracle
```
正确的`JAVA_HOME` 变量可以通过从最后一列输出追加`/jre`到末尾来构造
所以`JAVA_HOME`的配置如下：
```
/usr/lib/jvm/java-8-oracle/jre
```
备注：你的`JAVA_HOME`可能不一样

创建systemd服务文件，打开目录中调用的文件/etc/systemd/system，输入以下命令创建tomcat.service：
```
$ sudo vi /etc/systemd/system/tomcat.service
```
输入以下内容：
```
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/java-8-oracle/jre
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

接下来，重新加载systemd守护进程，刷新服务文件：
```
$ sudo systemctl daemon-reload
```
输入以下命令启动Tomcat服务：
```
$ sudo systemctl start tomcat
```
输入以下内容：
```
$ sudo systemctl status tomcat
```
会出现类似内容即可
```
tomcat.service - Apache Tomcat Web Application Container
   Loaded: loaded (/etc/systemd/system/tomcat.service; disabled; vendor preset: enabled)
   Active: active (running) since Thu 2018-04-26 15:53:35 CST; 9s ago
   Process: 23303 ExecStart=/opt/tomcat/bin/startup.sh (code=exited, status=0/SUCCESS)
   Main PID: 23313 (java)
   CGroup: /system.slice/tomcat.service
           └─23313 /usr/lib/jvm/java-8-oracle/jre/bin/java -Djava.util.logging.config.file=/opt/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apa

Apr 26 15:53:35 vengao systemd[1]: Starting Apache Tomcat Web Application Container...
Apr 26 15:53:35 vengao systemd[1]: Started Apache Tomcat Web Application Container.

```
##### 调整防火墙并测试Tomcat服务器

Tomcat默认使用端口8080来接受传统的请求。通过输入以下内容允许通信到该端口
```
$ sudo ufw allow 8080
```
通过 `ip_url:8080` 访问Tomcat 例如：`localhost:8080`

如果能够成功访问Tomcat，创建Tomcat在虚拟机上的开机启动文件：
```
$ sudo systemctl enable tomcat
```
出现以下内容表示设置成功
```
Created symlink from /etc/systemd/system/multi-user.target.wants/tomcat.service to /etc/systemd/system/tomcat.service.
```
