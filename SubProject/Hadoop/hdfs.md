# Hadoop HDFS

## 一、介绍

HDFS(Hadoop Distributed File System) 是 hadoop 运算框架中的分布式文件管理系统 .

### 1、端口介绍
| 属性 | 端口 | 说明 |
| ------ | ------------ | -----|
| mapred.job.tracker.http.address |  0.0.0.0:50030 | jobtracker 的 http 服务器地址和端口 |
| mapred.task.tracker.http.address |  0.0.0.0:50060 | tasktracker 的 http 服务器地址和端口 |
| dfs.http.address |  0.0.0.0:50070 | namenode 的 http 服务器地址和端口 |
| dfs.datanode.http.address |  0.0.0.0:50075 | datanode 的 http 服务器地址和端口 |
| dfs.secondary.http.address |  0.0.0.0:50090 | 辅助 namenode 的 http 服务器地址和端口 |
|  |  0.0.0.0:8020 | hdfs 文件端口 |

## 二、操作

### 1、shell 文件操作 (类似 linux 文件操作)
hadoop fs 查看所有命令

```
1) 列出文件列表
hadoop fs –ls [文件目录]
hadoop fs -ls /

2) 打开某个文件
hadoop fs –cat [file_path]
hadoop fs -cat /user/hive/warehouse/jason_test.db/student/student.txt

3) 将本地文件存储至 hadoop
hadoop fs –put [本地地址] [hadoop目录]
hadoop fs -put ./example.log /user/test/a.log

4) 将本地文件夹存储至 hadoop
hadoop fs –put [本地目录] [hadoop目录]
hadoop fs –put /home/t/dir_name /user/test/

5) 下载 hadoop 文件到我本地
hadoop fs -get [文件目录] [本地目录]
hadoop fs –get /user/test/a.log ./

6) 删除 hadoop 上指定文件
hadoop fs –rm [文件地址]
hadoop fs –rm /user/test/a.log

7) 删除 hadoop 上指定文件夹（包含子目录等）
hadoop fs –rmr [目录地址]
hadoop fs –rmr /user/test

8) 在 hadoop 指定目录内创建新目录
hadoop fs –mkdir /user/test/aaa

9) 在 hadoop 指定目录下新建一个空文件
hadoop fs -touchz  /user/test/new.txt

10) 将 hadoop 上某个文件重命名
hadoop fs –mv [原目录地址] [新目录地址]
hadoop fs –mv /user/test/new.txt  /user/test/new1.txt

11) 将hadoop指定目录下所有内容保存为一个文件，同时下载至本地
hadoop fs –getmerge [原目录地址] [本地目录地址]
hadoop fs –getmerge /user/test/ ./


```

### 2、shell job 操作
hadoop job 查看所有命令

```
1) 查看 job 任务
hadoop job -list

2) 将正在运行的 hadoop 作业kill掉
hadoop job –kill [job-id]

3) 查看 job 状态
hadoop job -status [job-id]
```

## 三、挂载hdfs目录到本地，实现云存储到 hadoop hdfs
安装 JDK 包，下载地址 http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html

```
$(lsb_release -cs)  linux内核版本

1、找到源
a) Ubuntu
wget  http://archive.cloudera.com/cdh5/one-click-install/$(lsb_release -cs)/amd64/cdh5-repository_1.0_all.deb

dpkg -i cdh5-repository_1.0_all.deb

sudo apt-get update

apt-get install hadoop-hdfs-fuse


b) Centos6
yum 包源
wget http://archive.cloudera.com/cdh5/redhat/6/x86_64/cdh/cloudera-cdh5.repo

cp ./cloudera-cdh5.repo /etc/yum.repos.d/

导入key
rpm --import http://archive.cloudera.com/cdh5/redhat/6/x86_64/cdh/RPM-GPG-KEY-cloudera

yum install hadoop-hdfs-fuse


2)、挂载
语法
hadoop-fuse-dfs dfs://<name_node_hostname>:<namenode_port> <mount_point>

mkdir -p /data/hdfs/
chown -R hdfs:hdfs /data/hdfs/

hadoop-fuse-dfs dfs://192.168.160.45:8020/ /data/hdfs/

umount /data/hdfs/ 卸载分区
```

### 2、shell.sh 操作 hadoop
```
1) 列出目录
/usr/bin/hadoop fs -ls hdfs://192.168.160.45:8020/user/

2) 写本地文件到 hdfs 中
/usr/bin/hadoop fs -put /data/soj_log/aaa.log hdfs://192.168.160.45:8020/user/test/

```
