# Hive

## 一、了解 Hadoop 生态的中的 Hive

### 1、作用
在分布式存储大型数据集的查询和管理，使用类似于 sql 的语言称为 HiveQL，通过 HiveQL 执行 MapReduce 查询数据。

把 sql 转换成一个 MapReduce 的程序运行放到后台运行，再返回结果

### 2、历史
起源自 facebook 由 Jeff Hammerbacher 领导的团队

构建在 Hadoop 上的数据仓库框架

设计的目的是让 Java 技能较弱的分析师可以通过类 SQL 查询海量数据

2008 年 facebook 把 hive 项目贡献给 Apache


## 二、Hive 组件与体系架构

### 1、组件
```
用户接口 : shell、thrift(自动生成Java代码)、web

Thrift 服务器

元数据库 (MetaStore) "Derby,Mysql" 等

解析器 : 输入 SQL 映射为 MapReduce 的 job

Hadoop : Hive 的数据都是放在 Hadoop 里面的
```

### 2、构架图
![image](./imgs/hive1.png)


## 三、Hive 安装

### 1、三种安装模式
```
1) 内嵌模式
元数据保持在内嵌的 Derby 数据库中，一次只能有一个连接会话。

2) 本地独立模式
在本地安装 Mysql , 把元数据放到 Mysql 中

3) 远程模式
元数据放到远程的 Mysql 数据库中
```
