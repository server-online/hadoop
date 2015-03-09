# Cloudera Manage 硬件配置方案

## 说明
如果集群有大量小文件，则要求NN内存至少要20GB，DN内存至少2GB。

几个组件中NN对内存最为敏感，它有单点问题，直接影响到集群的可用性；JT同样是单点，如果JT内存溢出则所有MapReduce Job都无法正常执行。

## 一、线下环境
| 节点 | 角色 | 配置 |
| ------ | ------------ | -----|
| CDH001 | CM-Server、Mysql(CM元数据存储)、NameNode Jobtracker、Hive-Metastore、ResourManager、 CM-agent | 8GB内存，1xT硬盘 |
| CDH002 | SecondaryNameNode、CM-agent | 16GB内存，1xT硬盘 |
| CDH003 | DataNode TaskTracker Zookeeper CM-agent | 8G内存，1xT硬盘 |
| CDH004 | DataNode TaskTracker Zookeeper CM-agent | 8G内存，1x1T硬盘 |



## 二、生产环境
| 节点 | 角色 | 配置 |
| ------ | ------------ | -----|
| CDH001 | CM-Server Mysql(CM元数据存储) | 1U,双路4核CPU,16GB内存，1xTB硬盘 |
| CDH002 | NameNode Jobtracker Hive-Metastore ResourManager CM-agent | 1U,双路4核CPU,32GB内存，4x1T(2T) |
| CDH003 | SecondaryNameNode CM-agent | 1U,双路4核CPU,16GB内存，4x1T(2T) |
| CDH004 | DataNode TaskTracker Zookeeper CM-agent | 1U,双路4核CPU,16GB内存，4x1T(2T) |
| CDH005 | DataNode TaskTracker Zookeeper CM-agent | 1U,双路4核CPU,16GB内存，4x1T(2T) |
