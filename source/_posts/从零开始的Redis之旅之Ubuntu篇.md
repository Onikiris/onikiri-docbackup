title: 从零开始的Redis之旅—Ubuntu篇
date: 2018/01/15  20:46:25
tags: 潜龙在渊
categories: 小试牛刀
---
最近接触到的Redis安装都是在CentOS上的操作，奈何自己的笔记本上只有Ubuntu系统（版本：Ubuntu 16.04.3 LTS），翻阅网上各类关于Ubuntu上安装Redis的教程，记录自己不断踩坑的过程<!--more-->
### Redis安装
在执行所有操作前，先使用命令更新软件列表
```
$ sudo apt-get update
```
通过以下步骤下载，解压，编译：
```
$ wget http://download.redis.io/releases/redis-4.0.6.tar.gz      ---下载
$ tar -zxvf redis-4.0.6.tar.gz        				 ---解压
$ sudo chmod -R 777 /usr/local  			   	 ---获取文件夹修改权限
$ mv redis-4.0.6    /usr/local                                   ---将解压后的redis移动至/usr/local下
$ cd /usr/local/redis-4.0.6                                      ---进入解压缩后的文件夹
$ make                                                           ---编译
```
备注：Redis2.x版本前不支持集群环境搭建，需要Redis3.x版本以上
### Redis服务启动
make(编译)完后 redis-4.0.6目录下会出现编译后的redis服务程序redis-server，还有用于测试的客户端程序redis-cli，两个程序位于安装目录 src 目录下：
```
$ cd src
$ ./redis-server
```
这种方式启动redis 使用的是默认配置，也可以通过启动参数告诉redis使用指定配置文件使用下面命令启动：
```
$ cd src
$ ./redis-server ../redis.conf
```
备注：redis.conf是一个默认的配置文件，我们可以根据需要使用自己的配置文件

启动redis服务进程后，就可以使用测试客户端程序redis-cli和redis服务交互了
如下例子：
```
$ src/redis-cli
redis> set foo FuckRedis
OK
redis> get foo
"FuckRedis"
```
### Redis集群安装
要让集群正常工作至少需要3个主节点，我们创建6个redis节点，其中三个为主节点，三个为从节点，对应的redis节点的ip和端口对应关系如下：
```
192.168.169.129:7000
192.168.169.129:7001
192.168.169.129:7002
192.168.169.129:7003
192.168.169.129:7004
192.168.169.129:7005
```
#### 第一步：创建集群环境所需的目录
在解压缩后的redis-4.0.6同根目录下新建文件夹cluster
```
$ mkdir -p /usr/local/cluster
```
同时在cluster下新建6个文件夹
```
$ cd /usr/local/cluster
$ mkdir  7000 7001 7002 7003 7004 7005
```
#### 第二步：修改配置文件redis.conf
先将redis-4.0.6文件夹下的redis.conf复制到cluster
```
$ cp /usr/local/redis-4.0.6/redis.conf  /usr/local/cluster
```
使用vi编辑修改:
```
$ cd /usr/local/cluster
$ vi redis.conf
```
vi命令参考此篇文章，感觉vi命令麻烦的话，使用：
```
$ sudo gedit redis.conf
```
修改配置文件内容如下（修改配置时注意去掉前面的注释# ）:
```
port  7000    			      ---端口7000，7001，7002，7003，7004，7005        
bind 192.168.169.129        	      ---默认ip为192.168.169.129，需要改为其他节点机器可访问的ip，否则创建集群时无法访问对应的端口，无法创建集群
daemonize    yes                      ---redis后台运行
cluster-enabled  yes    	      ---开启集群  把注释#去掉
cluster-config-file  nodes_7000.conf  ---集群的配置  配置文件首次启动自动生成 7000，7001，7002，7003，7004，7005
cluster-node-timeout  15000           ---请求超时  默认15秒，可自行设置
appendonly  yes                       ---aof日志开启  有需要就开启，它会每次写操作都记录一条日志
```

备注：（我的本机ip是192.168.169.129）可通过`$ ifconfig`查看本机ip
再将修改后的redis.conf复制到cluster下的6个文件夹中
```
$ cp /usr/local/cluster/redis.conf  /usr/local/cluster/7000/
$ cp /usr/local/cluster/redis.conf  /usr/local/cluster/7001/
$ cp /usr/local/cluster/redis.conf  /usr/local/cluster/7002/
$ cp /usr/local/cluster/redis.conf  /usr/local/cluster/7003/
$ cp /usr/local/cluster/redis.conf  /usr/local/cluster/7004/
$ cp /usr/local/cluster/redis.conf  /usr/local/cluster/7005/
```
分别进入到上面的六个新建的文件夹中， 按照上面的配置介绍中的样例进行对应修改

比较麻烦的方式启动服务：
```
$ cd /usr/local/redis-4.0.6/src/
$ redis-server ../../cluster/7000/redis.conf
$ redis-server ../../cluster/7001/redis.conf
$ redis-server ../../cluster/7002/redis.conf
$ redis-server ../../cluster/7003/redis.conf
$ redis-server ../../cluster/7004/redis.conf
$ redis-server ../../cluster/7005/redis.conf
```
我个人使用的方式，不喜欢的可不用
脚本快速启动服务：
```
$ cd /usr/loacl/cluster
$ vi redis-start.sh

#填入以下命令：
cd /usr/local/cluster/7000/
redis-server redis.conf
cd /usr/local/cluster/7001/
redis-server redis.conf
cd /usr/local/cluster/7002/
redis-server redis.conf
cd /usr/local/cluster/7003/
redis-server redis.conf
cd /usr/local/cluster/7004/
redis-server redis.conf
cd /usr/local/cluster/7005/
redis-server redis.conf
```
启动服务：
```
$ ./redis-start.sh
```
使用脚本快速关闭服务：
```
$ cd /usr/local/cluster
$ vi redis-shutdown.sh

# 填写如下命令：
redis-cli -h 192.168.169.129 -p 7000 shutdown
redis-cli -h 192.168.169.129 -p 7001 shutdown
redis-cli -h 192.168.169.129 -p 7002 shutdown
redis-cli -h 192.168.169.129 -p 7003 shutdown
redis-cli -h 192.168.169.129 -p 7004 shutdown
redis-cli -h 192.168.169.129 -p 7005 shutdown
```
关闭服务：
```
$ ./redis-shutdown.sh
```
查看服务：
```
$ ps -ef|grep redis-server
```
出现以下类似信息表示服务成功启动：
```
redis      3062   1947  0 00:26 ?        00:00:00 redis-server 192.168.169.129:7000 [cluster]
redis      3075   1947  0 00:26 ?        00:00:00 redis-server 192.168.169.129:7001 [cluster]
redis      3080   1947  0 00:28 ?        00:00:00 redis-server 192.168.169.129:7002 [cluster]
redis      3087   1947  0 00:28 ?        00:00:00 redis-server 192.168.169.129:7003 [cluster]
redis      3092   1947  0 00:28 ?        00:00:00 redis-server 192.168.169.129:7004 [cluster]
redis      3099   1947  0 00:28 ?        00:00:00 redis-server 192.168.169.129:7005 [cluster]
redis      3132   3022  0 00:32 pts/4    00:00:00 grep --color=auto redis-server
```
虽然六个服务都起来了， 但这时他们之间还不知道彼此的存在， 无法组成集群提供服务

#### 第三步：连接服务
使用工具 ‘redis-trib.rb’ 将上面的六个节点连接起来， redis-trib.rb 是需要ruby环境
先安装ruby环境
```
$ sudo apt-get install ruby
```
为了执行 redis-trib.rb 需要安装gem redis
```
$ sudo gem install redis
```
安装时可能会被墙，可以在RubyGems下载最新版
使用命令关联集群（ip地址和对应的端口号）
```
$ cd /usr/local/redis-4.0.6/src
$ ./redis-trib.rb create --replicas 1 192.168.169.129:7000 192.168.169.129:7001 192.168.169.129:7002 192.168.169.129:7003 192.168.169.129:7004 192.168.169.129:7005
```
备注：参数1表示为每个主节点创建一个从节点，其他参数是实例的地址集合
出现以下信息表示集群启动成功：
```
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.169.129:7000
192.168.169.129:7001
192.168.169.129:7002
Adding replica 192.168.169.129:7003 to 192.168.169.129:7000
Adding replica 192.168.169.129:7004 to 192.168.169.129:7001
Adding replica 192.168.169.129:7005 to 192.168.169.129:7002
M: ef06386f81ea6ebcfd1750ce3f28dac6c8d1787d 192.168.169.129:7000
   slots:0-5460 (5461 slots) master
M: 73fd9b7f608dd31ad1dd793b79ea3b6991bf14f9 192.168.169.129:7001
   slots:5461-10922 (5462 slots) master
M: 8ad9a9283c0fc34ffdce686d0cf28426b843dbef 192.168.169.129:7002
   slots:10923-16383 (5461 slots) master
S: e08c6b78c4b6b866c4fa2154c3d4ea790cc3c2b1 192.168.169.129:7003
   replicates ef06386f81ea6ebcfd1750ce3f28dac6c8d1787d
S: fb7eab7cff0d5c06c500bdd6f138e4af970ca67b 192.168.169.129:7004
   replicates 73fd9b7f608dd31ad1dd793b79ea3b6991bf14f9
S: 0e889f689fcdf99061adcc807f6ed8815d330c14 192.168.169.129:7005
   replicates 8ad9a9283c0fc34ffdce686d0cf28426b843dbef
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join...
>>> Performing Cluster Check (using node 192.168.169.129:7000)
M: ef06386f81ea6ebcfd1750ce3f28dac6c8d1787d 192.168.169.129:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: 73fd9b7f608dd31ad1dd793b79ea3b6991bf14f9 192.168.169.129:7001
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
M: 8ad9a9283c0fc34ffdce686d0cf28426b843dbef 192.168.169.129:7002
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: e08c6b78c4b6b866c4fa2154c3d4ea790cc3c2b1 192.168.169.129:7003
   slots: (0 slots) slave
   replicates ef06386f81ea6ebcfd1750ce3f28dac6c8d1787d
S: fb7eab7cff0d5c06c500bdd6f138e4af970ca67b 192.168.169.129:7004
   slots: (0 slots) slave
   replicates 73fd9b7f608dd31ad1dd793b79ea3b6991bf14f9
S: 0e889f689fcdf99061adcc807f6ed8815d330c14 192.168.169.129:7005
   slots: (0 slots) slave
   replicates 8ad9a9283c0fc34ffdce686d0cf28426b843dbef
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
使用redis-cli命令进入集群环境
```
$ redis-cli -h 192.168.169.129 -p 7000 -c
```
备注：-h 指定IP地址 -p 指定端口
能进行如下测试表示成功
```
redis 192.168.169.129:7000> set foo bar
-> Redirected to slot [12182] located at 192.168.169.129:7002
OK
redis 192.168.169.129:7002> set hello world
-> Redirected to slot [866] located at 192.168.169.129:7000
OK
redis 192.168.169.129:7000> get foo
-> Redirected to slot [12182] located at 192.168.169.129:7002
"bar"
redis 192.168.169.129:7000> get hello
-> Redirected to slot [866] located at 192.168.169.129:7000
"world"
```
至此，Redis相关安装全部结束