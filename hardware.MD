# Cloudera Manage 硬件配置方案

## 一、硬件分配

### 1、建立分配
NN: 15-25 GB  
JT：2-4GB  
DN：1-4 GB  
TT：1-2 GB，Child VM 1-2 GB


如果集群有大量小文件，则要求NN内存至少要20GB，DN内存至少2GB。


几个组件中NN对内存最为敏感，它有单点问题，直接影响到集群的可用性；JT同样是单点，如果JT内存溢出则所有MapReduce Job都无法正常执行。


## 二、实际部署
| 节点 | 角色 | 详细 |
| ------ | ------------ | -----|
| dev001 | CM-Server Mysql(CM元数据存储) | - |
| dev002 | NameNode Jobtracker Hive-Metastore ResourManager CM-agent | - |
| dev003 | SecondaryNameNode CM-agent | - |
| dev004 | DataNode TaskTracker Zookeeper CM-agent | - |
| dev005 | DataNode TaskTracker Zookeeper CM-agent | - |
