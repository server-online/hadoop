## 一、学习路线
### 1、学习hive基本语法

### 2、关于gateway

```
为了安全，单独弄台很小的机器做 gateway
gateway 就是客户端的意思，BI可以在这个上面执行 hive 命令
但不跑 hadoop 的datanode，mr 这些服务
```


## 二、过程中遇到的问题
### 1、报错找不到类
* 例如：

```
ClassNotFoundException: Class org.apache.hadoop.hive.contrib.serde2.RegexSerDe not found
```
*解决方案：

```
参考：
http://blog.csdn.net/sptoor/article/details/9838691
http://blog.csdn.net/xiao_jun_0820/article/details/38302451

命令：
sudo ln -s /opt/cloudera/parcels/CDH-5.3.2-1.cdh5.3.2.p0.10/lib/hive/lib /etc/hive/auxlib

CM配置：
Hive 辅助 JAR 目录：
	/etc/hive/auxlib
hive-env.sh 的 Gateway 客户端环境高级配置代码段（安全阀）：
	HIVE_AUX_JARS_PATH=/etc/hive/auxlib
```