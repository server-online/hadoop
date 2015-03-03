# Cloudera 实战

## 一、准备工作 (这里的是公众的准备步骤)

### * 系统服务
```
关闭防火墙
service iptables stop

关闭selinux
setenforce 0

需要重启
vim /etc/selinux/config
SELINUX=disabled
```

### 1、设置所有集群机器的 hostname
```
设置 HOSTNAME 与 域名统一

master
vi /etc/sysconfig/network
HOSTNAME=dev1.jsonlin.cn

slave1
vi /etc/sysconfig/network
HOSTNAME=dev1.jsonlin.cn

slave2
vi /etc/sysconfig/network
HOSTNAME=dev2.jsonlin.cn

slave3
vi /etc/sysconfig/network
HOSTNAME=dev3.jsonlin.cn

重启后生效
```

### 2、创建执行用户（每台服务器）
注：决定用哪个用户执行，就创建哪个用户，如果是root用户则这一步略过
```
groupadd -r hadoop
useradd -g hadoop hadoop
```

### 3、设置免ssh登陆(每台服务器)
```
每台服务器生成key
ssh-keygen -t rsa -P ""  
```

### 4、master 分发 key 免密码访问

#### 4.1、说明 authorized_keys
```
master 需要对所有的 slave 免密码访问

master 需要对 master 免密码访问

master 要分发给所有的 slave
```

#### 4.2、具体实现 authorized_keys
```
a) 设置authorized_keys
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
 (OR)
cp id_rsa.pub authorized_keys

b) 开始分发给slave
scp ~/.ssh/authorized_keys hadoop@dev1.jsonlin.cn:~/.ssh/

scp ~/.ssh/authorized_keys hadoop@dev2.jsonlin.cn:~/.ssh/

scp ~/.ssh/authorized_keys hadoop@dev3.jsonlin.cn:~/.ssh/
```

#### 4.3、安装 Java 7

具体百度啦，这个怎么可能难道你呢，出现的结果这样就对了
```
[root@dev1 hadoop]# java -version
java version "1.7.0_75"
Java(TM) SE Runtime Environment (build 1.7.0_75-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.75-b04, mixed mode)
```

#### 4.4、所有节点配置 NTP 服务

##### 4.4.1、为什么要安装 NTP

集群中所有主机必须保持时间同步，如果时间相差较大会引起各种问题。

master  节点作为 ntp 服务器与外界对时中心同步时间，随后对所有 datanode 节点提供时间同步服务。

所有 datanode 节点以 master 节点为基础同步时间。


##### 4.4.2、NTP 时间服务器安装

详细
http://www.cnblogs.com/liuyou/archive/2012/07/29/2614338.html

```
1) 所有节点都要安装 NTP
Centos 下安装
yum install ntp 安装
chkconfig ntpd on 开机启动
chkconfig --list ntpd


***master 节点操作。slave 节点走 2.2 的步骤***
先安装 NTP 服务
2.1) master (master 节点作为局域网内的时间服务器,  )

a) 手动同步下系统时间，以 ntp.sjtu.edu.cn 时间为准
ntpdate -u ntp.sjtu.edu.cn

b) 修改配置文件 (看见没有的就加一条)
vim /etc/ntp.conf
#允许本机的所有操作
restrict 127.0.0.1
restrict -6 ::1
#拒绝客户端的所有操作
restrict default kod nomodify notrap nopeer noquery
#设置常用的同步服务器(master专属) ntp.sjtu.edu.cn
server ntp.sjtu.edu.cn prefer

c) 启动服务，开机启动操作。


***slave 节点操作***
2.2) slave 节点 (slave 以 master 节点作为时间服务器)
a) 先手动同步一下时间,已主节点为准
ntpdate -u dev1.jsonlin.cn

b) 配置ntp客户端 slave（所有datanode节点）
vim /etc/ntp.conf
restrict 127.0.0.1
restrict -6 ::1
restrict default kod nomodify notrap nopeer noquery
#设置同步服务器(这里用的是 master 的域名地址或者IP地址)
server dev1.jsonlin.cn prefer

c) 启动服务，开机启动操作。


3) 启动操作 master slave 通用
service ntpd start

表示 启动成功
[root@dev1 hadoop]# ntpstat
synchronised to NTP server (202.112.31.197) at stratum 3
time correct to within 82 ms
polling server every 128 s

--------------OR----------------

[root@dev1 hadoop]# netstat -tunlp | grep ntp
udp        0      0 192.168.1.109:123           0.0.0.0:*                               2805/ntpd
udp        0      0 127.0.0.1:123               0.0.0.0:*                               2805/ntpd
udp        0      0 0.0.0.0:123                 0.0.0.0:*                               2805/ntpd
udp        0      0 ::1:123                     :::*                                    2805/ntpd
udp        0      0 fe80::20c:29ff:fe44:3f09:123 :::*                                    2805/ntpd
udp        0      0 :::123                      :::*                                    2805/ntpd

```



## 二、Cloudera Manager

### 1、Cloudera Manager 介绍

Cloudera Manager 可以使集群中主机自动安装Hadoop，Hive，zookeeper，impalal等组件，无需做复杂的一系列配置。

Cloudera Express 版本是免费的

Cloudera Enterprise 是需要购买注册码的

### 2、Cloudera Manager 下载地址
```
2.1) 下载地址

Cloudera Manager 管理地址
http://archive-primary.cloudera.com/cm5/cm/5/

CDH安装包地址 (Cloudera Manager 是通过这种方式管理各个组件的)
http://archive.cloudera.com/cdh5/parcels/latest/
Centos6下载：
CDH-5.3.1-1.cdh5.3.1.p0.5-el6.parcel
CDH-5.3.1-1.cdh5.3.1.p0.5-el6.parcel.sha1
manifest.json


CM4、CM5包汇总
http://archive.cloudera.com/cm4/
http://archive.cloudera.com/cm5/

2.2) 版本对应  
a) Centos 对应
   el*
   如el5、el6

b) Linux ubuntu对应
   lucid(debian)
   precise（ubuntu）

c) debian对应
   squeeze  
   wheezy

d) redhat对应
   sles*

c) 安装版本、下载详细说明汇总
http://www.aboutyun.com/thread-8908-1-1.html

```

### 3、Cloudera Manager 安装介绍
详细指导文章(请查看指导文章安装)：
http://www.aboutyun.com/thread-9219-1-1.html

#### 3.1、 CM (Cloudera Manager) 有三种安装方式

```
1) cloudera-manager-installer.bin 安装
只要从官网下载cloudera-manager-installer.bin，然后执行这个bin文件

参考文章:
http://www.aboutyun.com/thread-9303-1-1.html
http://www.aboutyun.com/thread-9075-1-1.html

2) rpm、yum、apt-get方式在线安装
因为 rpm 对包依赖的关系不好，所以产生了yum
参考文章:
http://www.aboutyun.com/thread-9107-1-1.html

3) Tarballs (本文安装方式)
tar 包管理源码安装
```

### 4、Cloudera Manager 安装

记得前面的准备工作要做哦

详细地址：http://www.cnblogs.com/jasondan/p/4011153.html

其中 /opt/cloudera-manager/cm-5.3.1/ 作为一个自定义目录

#### 4.1、安装命令 (master 节点)
```
1) 目录规划
mv cloudera-manager-el6-cm5.3.1_x86_64.tar.gz /opt/
tar -zxvf cloudera-manager-el6-cm5.3.1_x86_64.tar.gz
mkdir /opt/cloudera-manager
mv /opt/cm-5.3.1/ /opt/cloudera-manager/

2) 为 Cloudera Manager Server 建立数据库
#添加 java mysql 驱动类
cp -arip  mysql-connector-java-5.1.31-bin.jar /opt/cloudera-manager/cm-5.3.1/share/cmf/lib/

a) mysql 上建立scm数据库
/opt/cloudera-manager/cm-5.3.1/share/cmf/schema/scm_prepare_database.sh mysql -h127.0.0.1 -uroot -p514591 --scm-host localhost scm scm scm
#---格式是:  scm_prepare_database.sh 数据库类型  数据库 服务器 用户名 密码  --scm-host  Cloudera_Manager_Server所在的机器

#---出现如下表示成功
ully connected to database.
All done, your SCM database is configured correctly!

b) 启动 Cloudera Manager Server 服务端口
/opt/cloudera-manager/cm-5.3.1/etc/init.d/cloudera-scm-server start [--restart重启 --stop 关闭]

c) 设置成开机启动
vim /etc/rc.d/rc.local
#---mysql开机启动
/usr/local/mysql-5.6.21/support-files/mysql.server start

#---cloudera-scm-server 服务端口开机启动
/opt/cloudera-manager/cm-5.3.1/etc/init.d/cloudera-scm-server start


3) Cloudera Manager Agents 服务端口配置
a) 先修改 Agents 配置文件 (master 上操作)
vim /opt/cloudera-manager/cm-5.3.1/etc/cloudera-scm-agent/config.ini
#这里写 master 域名或者Ip
server_host=dev1.jsonlin.cn
server_port=7182 #服务端口

b) 把整个目录分发给 slave (master 上操作)
scp -r /opt/cloudera-manager root@dev2.jsonlin.cn:/opt/
scp -r /opt/cloudera-manager root@dev3.jsonlin.cn:/opt/
scp -r /opt/cloudera-manager root@dev4.jsonlin.cn:/opt/

c) 添加 cloudera-scm 用户 (所有节点)
useradd --system --home=/opt/cloudera-manager/cm-5.3.1/run/cloudera-scm-server --no-create-home --shell=/bin/false --comment "Cloudera SCM User" cloudera-scm

d) 为所有的节点都启动 (所有节点)
/opt/cloudera-manager/cm-5.3.1/etc/init.d/cloudera-scm-agent start

#---所有节点开机启动
vim /etc/rc.d/rc.local

#cloudera-scm-agent
/opt/cloudera-manager/cm-5.3.1/etc/init.d/cloudera-scm-agent start
```

#### 4.2、准备Parcels，用以安装CDH5

##### 4.2.1、下载地址
```
 http://archive.cloudera.com/cdh5/parcels/latest/

Centos6 :
CDH-5.3.1-1.cdh5.3.1.p0.5-el6.parcel
CDH-5.3.1-1.cdh5.3.1.p0.5-el6.parcel.sha1
manifest.json

其他系统欢迎补充.....
```

##### 4.2.2、存放目录

```
1) 存放目录 (master)
#---没有则创建一个 (存放到 master 的 /opt/cloudera/parcel-repo)
ll /opt/cloudera/parcel-repo
mkdir -p /opt/cloudera/parcel-repo

2) 存放文件到 /opt/cloudera/parcel-repo/ (master)
CDH-5.3.1-1.cdh5.3.1.p0.5-el6.parcel
CDH-5.3.1-1.cdh5.3.1.p0.5-el6.parcel.sha1
manifest.json

3) *.sha1 文件改名为 *.sha
mv /opt/cloudera/parcel-repo/CDH-5.3.1-1.cdh5.3.1.p0.5-el6.parcel.sha1  /opt/cloudera/parcel-repo/CDH-5.3.1-1.cdh5.3.1.p0.5-el6.parcel.sha
```

#### 4.3 准备浏览器跑起来，辛苦这么久嘿嘿

```
1) 访问 master 的服务 7180 端口
http://dev1.jsonlin.cn:7180

#启动 cloudera-scm-server
/opt/cloudera-manager/cm-5.3.1/etc/init.d/cloudera-scm-server restart

#启动 cloudera-scm-agent
/opt/cloudera-manager/cm-5.3.1/etc/init.d/cloudera-scm-agent restart

先让 cloudera-scm-server 启动成功
netstat -tunlp | grep jave --- 7180

再启动 cloudera-scm-agent jave --- 7180
netstat -tunlp | grep jave --- 7182

一般需要耐心等待几分钟

```


## ---------华丽的分割线------------


## 三、CDH --- (敬请期待)

### 1、CDH 介绍

CDH (Cloudera Distribution Hadoop) Cloudera 发行的 Hadoop 稳定版本，更稳定，包含了企业以及个人的修复补丁。

CDH 不同于 Cloudera Manager , CDH 是需要配置一堆参数来安装集群的，喜欢折腾，深入研究可以使用此方法安装，更好的理解原理和集群。

CDH4 在 Apache Hadoop2.0.0 版本基础上演化而来的

CDH5 在 Apache hadoop2.5.0 版本基础上演化而来的

### 2、CDH 下载地址

```
CDH4 http://archive.cloudera.com/cdh4/cdh/4/
CDH5 http://archive.cloudera.com/cdh5/cdh/5/

以 5.1.3 版本为例子

下载对应的安装包即可，最新版本
hadoop : hadoop-2.5.0-cdh5.3.1.tar.gz

下载对应的生态包，找到如下规律的对应安装包即可
Hbase : hbase-0.98.1-cdh5.3.1.tar.gz
Hive : hive-0.12.0-cdh5.3.1.tar.gz
Hue : hue-3.6.0-cdh5.3.1.tar.gz

```

### 3、CDH 安装介绍

详细指导文章(请查看指导文章安装)： http://www.aboutyun.com/thread-9219-1-1.html

```
1) Yum/Apt 包

2) RPM 包

3) CM (Cloudera Manager ) 管理安装

4) Tar包 (本文的方式)
```


### 其他信息
java进程统计管理，可以查看有多少hadoop进程启动了
/usr/local/jdk1.7.0_75/bin/jps

持续不定期更新，地址
https://github.com/server-online/hadoop
