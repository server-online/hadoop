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
### 2、Hive建表SerDe支持json
* 下载第三方json的jar包、挂载
* 资源文档：https://code.google.com/p/hive-json-serde/wiki/GettingStarted

```
wget https://hive-json-serde.googlecode.com/files/hive-json-serde-0.2.jar
# 添加jar包
hive> add jar /home/heyuan.lhy/develop/wanke_http_test/hive-json-serde-0.2.jar;
hive> 
# 创建hive表
CREATE TABLE test_json
(
    id BIGINT,
    text STRING,
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.contrib.serde2.JsonSerde'
STORED AS TEXTFILE
;
LOAD DATA LOCAL INPATH "test.json" OVERWRITE INTO TABLE test_json;
```
* 加载jar包的方式：http://www.cnblogs.com/tangtianfly/archive/2012/11/06/2756745.html
* 自己编写jar包：http://blog.cloudera.com/blog/2012/12/how-to-use-a-serde-in-apache-hive/
### 3、第三方类库需要在beeline生效
* On the Beeline client machine, in /etc/hive/conf/hive-site.xml, set the hive.aux.jars.path property to a comma-separated list of the fully-qualified paths to the JAR file and any dependent libraries.

```
  <property>
    <name>hive.aux.jars.path</name>
    <value>file:///etc/hive/auxlib/hive-json-serde-0.2.jar</value>
  </property>
```
