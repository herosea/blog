---
title: WebHDFS_AND_HttpFS
date: 2016-01-02 23:29:59
tags:
- hadoop
category:
- 大数据
---

引自<http://blog.cheyo.net/90.html>

# WebHDFS与HttpFS的使用
## WebHDFS
### 介绍
提供HDFS的RESTFul接口，可通过此接口进行HDFS文件操作
### 安装
WebHDFS 服务内置在HDFS中，不需二外安装、启动
### 配置
需要在hdfs-site.xml打开WebHDFS开关，此开关默认打开

```
<property>
  <name>dfs.webhdfs.enabled</name>
  <value>true</value>
</property>
```
### 使用
连接NameNode的50070端口进行文件操作

比如：

```
curl "http://ctrl:50070/webhdfs/v1/?op=liststatus&user.name=root" | python -mjson.tool
```
### 更多操作
参考文档：[官方WebHDFS REST API](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-hdfs/WebHDFS.html)

## HttpFS[Hadoop HDFS over HTTP]
### 介绍
HttpFS is a server that provides a REST HTTP gateway supporting all HDFS File System operations [read and write]. And it is interoperable with the webhdfs REST HTTP API.
**HttpFS和WebHDFS兼容**
### 安装
Hadoop自带，不需要额外安装。服务默认情况没有启动，需要手工启动。
### 配置
* httpfs-site.xml
  此配置一般用hadoop默认的即可，无需修改。
* hdfs-site.xml

需要增加如下配置，其中root代表启动hdfs服务的os用户，根据实际情况修改。

```
<property>
  <name>hadoop.proxyuser.root.hosts</name>
  <value>*</value>
</property>
<property>
  <name>hadoop.proxyuser.root.groups</name>
  <value>*</value>
</property>
```

### 启动
```
sbin/httpfs.sh start
sbin/httpfs.sh stop
```
启动后，默认监听14000端口：

```
netstat -antp | grep 14000
```
使用

```
curl "http://ctrl:14000/webhdfs/v1/?op=liststatus&user.name=root" | python -mjson.tool
```
### 更多操作
### 参考文档

1. [官方WebHDFS REST API](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-hdfs/WebHDFS.html)
2. [HttpFS官方文档](http://hadoop.apache.org/docs/r2.4.1/hadoop-hdfs-httpfs/index.html)

## WebHDFS和HttpFS的关系
WebHDFS vs HttpFs Mojor difference between WebHDFS and HttpFs: WebHDFS needs access to all nodes of the cluster and where some data is read it is transmitted from that node directly, whereas in HttpFs, a signe node will act similar to a "gateway" and will be a single point of data transfer to the client node . So, HttpFs could be choked during a large file transfer but the good thing is that we are minimizing the footprint required to access HDFS.
