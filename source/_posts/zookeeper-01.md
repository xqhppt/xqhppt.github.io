---
title: ZooKeeper介绍
date: 2020-01-31 22:17:00
author: xqh
tags:
- ZooKeeper
- 分布式
categories: ZooKeeper
---

## 简介

它是一个分布式服务框架，是Apache Hadoop 的一个子项目，它主要是用来解决分布式应用中经常遇到的一些数据管理问题如：数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、配置维护，名字服务、分布式同步、分布式锁和分布式队列
等功能。

- 一致性：数据一致性，数据按照顺序分批入库。
- 原子性：事务要么成功要么失败，不会局部化。
- 单一视图：客户端连接集群中任一zk节点，数据都是一致的。
- 可靠性:每次对zk的操作状态都会保存在服务端。
- 实时性：客户端可以读取到zk服务器的最新数据。

## 安装

### 单机

#### linux环境

##### 下载

wget http://mirror.bit.edu.cn/apache/zookeeper/stable/zookeeper-3.4.10.tar.gz

##### 解压

tar -zxvf zookeeper-3.4.10.tar.gz

##### 复制配置文件
cp zoo_sample.cfg zoo.cfg

##### 启动
bin/zkServer.sh start

##### 查看状态
bin/zkServer.sh status

##### 停止
bin/zkServer.sh stop

##### 连接
bin/zkCli.sh -server 127.0.0.1:2181

##### client常用命令
help

#### Mac下快捷安装

##### 安装
brew install zookeeper

##### 查看
brew info zookeeper

> 安装位置在/usr/local/Cellar
> 
> 配置文件在/usr/local/etc

### 集群

## 常用命令

### create

语法：

`create [-s] [-e] path data acl`
> -s为有序 -e为临时

例子：

```
//创建节点并写入数据
[zk: 127.0.0.1:2181(CONNECTED) 31] create /top "this is top node"
Created /top

//查看节点列表
[zk: 127.0.0.1:2181(CONNECTED) 32] ls /
[top]

//创建有序节点，节点名为/s+自增序号
[zk: 127.0.0.1:2181(CONNECTED) 35] create -s /s "s1"
Created /s0000000006

//创建临时节点（会话过期后会被删除）
[zk: 127.0.0.1:2181(CONNECTED) 43] create -e /tmp "t1"
Created /tmp
```

### get
语法：

`get path [watch]`

例子：
```
[zk: 127.0.0.1:2181(CONNECTED) 33] get /top
this is top node
cZxid = 0x30
ctime = Thu Jan 30 15:36:56 CST 2020
mZxid = 0x30
mtime = Thu Jan 30 15:36:56 CST 2020
pZxid = 0x30
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 16
numChildren = 0
```

含义

- cZxid：数据节点创建时的事务ID
- ctime：数据节点创建时的时间
- mZxid：数据节点最后一次更新时的事务ID
- mtime：数据节点最后一次更新时的时间
- pZxid：数据节点的子节点最后一次被修改时的事务 ID
- cversion：子节点的更改次数
- dataVersion：节点数据的更改次数
- aclVersion：节点的 ACL 的更改次数
- ephemeralOwner：如果节点是临时节点，则表示创建该节点的会话的
- dataLength：数据内容的长度
- numChildren：数据节点当前的子节点个数

### set

语法：

`set path data [version]`

例子：

```
//修改节点数据
[zk: 127.0.0.1:2181(CONNECTED) 56] set /top "this is top node 4"
cZxid = 0x30
ctime = Thu Jan 30 15:36:56 CST 2020
mZxid = 0x36
mtime = Thu Jan 30 16:01:36 CST 2020
pZxid = 0x32
cversion = 1
dataVersion = 3
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 18
numChildren = 1

//根据版本号修改
[zk: 127.0.0.1:2181(CONNECTED) 58] set /top "this is top node 4" 1
version No is not valid : /top
```

### delete

语法：

`delete path [version]`

例子：


```
//根据版本号删除
[zk: 127.0.0.1:2181(CONNECTED) 62] delete /top 1
version No is not valid : /top

//删除
[zk: 127.0.0.1:2181(CONNECTED) 68] delete /top
```

> 使用`rmr`命令可以递归删除节点

### wacth（监听器）

* 针对每个节点的操作都会有一个监督者：watcher
* 当监控的某个对象（znode）发生了变化，则触发watcher
* zookeeper中的watcher是一次性的，触发后立即销毁


```
//1.通过get注册监视器
[zk: 127.0.0.1:2181(CONNECTED) 42] get /top watch
//set delete操作当前节点都会触发
[zk: 127.0.0.1:2181(CONNECTED) 43] set /top "123"
cZxid = 0x43
WATCHER::


WatchedEvent state:SyncConnected type:NodeDataChanged path:/top

//2.通过stat注册监视器
[zk: 127.0.0.1:2181(CONNECTED) 46] stat /top watch
//set delete操作当前节点都会触发
[zk: 127.0.0.1:2181(CONNECTED) 47] set /top "this is"

WATCHER::
cZxid = 0x43
WatchedEvent state:SyncConnected type:NodeDataChanged path:/top

//3.通过ls或ls2注册监视器
[zk: 127.0.0.1:2181(CONNECTED) 48] ls /top watch
//create set delete子节点都会触发
[zk: 127.0.0.1:2181(CONNECTED) 50] create /top/top1 "1"

WATCHER::

WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/top
Created /top/top1
```

注：
> `get`和`stat`命令只能触发当前节点
> 
> `ls`和`ls2`命令只能触发节点下的所有子节点

### 四字命令

- conf打印服务配置的详细信息。
- cons列出连接到此服务器的所有客户端的完整连接/会话详细信息。包括接收/发送的数据包数量，会话 ID，操作延迟，上次执行的操作等信息。
- dump列出未完成的会话和临时节点。这只适用于 Leader 节点。
- envi打印服务环境的详细信息。
- ruok测试服务是否处于正确状态。如果正确则返回“imok”，否则不做任何相应。
- stat列出服务器和连接客户端的简要详细信息。
- wchs列出所有 watch 的简单信息。
- wchc按会话列出服务器 watch 的详细信息。
- wchp按路径列出服务器 watch 的详细信息。

例子：

```
$ echo conf | nc localhost 2181
clientPort=2181
dataDir=/usr/local/var/run/zookeeper/data/version-2
dataLogDir=/usr/local/var/run/zookeeper/data/version-2
tickTime=2000
maxClientCnxns=60
minSessionTimeout=4000
maxSessionTimeout=40000
serverId=0
```

更多参考[官网](https://zookeeper.apache.org/doc/current/zookeeperAdmin.html) The Four Letter Words这节内容

## 概念

### 集群角色
Zookeeper 集群中的机器分为以下三种角色：

- Leader ：为客户端提供读写服务，并维护集群状态，它是由集群选举所产生的；
- Follower ：为客户端提供读写服务，并定期向 Leader 汇报自己的节点状态。同时也参与写操作“过半写成功”的策略和 Leader 的选举；
- Observer ：为客户端提供读写服务，并定期向 Leader 汇报自己的节点状态，但不参与写操作“过半写成功”的策略和 Leader 的选举，因此 Observer 可以在不影响写性能的情况下提升集群的读性能。

### 会话
Zookeeper 客户端通过 TCP 长连接连接到服务集群，会话 (Session) 从第一次连接开始就已经建立，之后通过心跳检测机制来保持有效的会话状态。通过这个连接，客户端可以发送请求并接收响应，同时也可以接收到 Watch 事件的通知。

关于会话中另外一个核心的概念是 sessionTimeOut(会话超时时间)，当由于网络故障或者客户端主动断开等原因，导致连接断开，此时只要在会话超时时间之内重新建立连接，则之前创建的会话依然有效。

### 数据节点
Zookeeper 数据模型是由一系列基本数据单元 Znode(数据节点) 组成的节点树，其中根节点为 /。每个节点上都会保存自己的数据和节点信息。Zookeeper 中节点可以分为两大类：

持久节点 ：节点一旦创建，除非被主动删除，否则一直存在；
临时节点 ：一旦创建该节点的客户端会话失效，则所有该客户端创建的临时节点都会被删除。
临时节点和持久节点都可以添加一个特殊的属性：SEQUENTIAL，代表该节点是否具有递增属性。如果指定该属性，那么在这个节点创建时，Zookeeper 会自动在其节点名称后面追加一个由父节点维护的递增数字。

### 节点信息
每个 ZNode 节点在存储数据的同时，都会维护一个叫做 Stat 的数据结构，里面存储了关于该节点的全部状态信息，详细参考上面`get`命令那节


### Watcher
Zookeeper 中一个常用的功能是 Watcher(事件监听器)，它允许用户在指定节点上针对感兴趣的事件注册监听，当事件发生时，监听器会被触发，并将事件信息推送到客户端。该机制是 Zookeeper 实现分布式协调服务的重要特性。

### ACL
Zookeeper 采用 ACL(Access Control Lists) 策略来进行权限控制，类似于 UNIX 文件系统的权限控制。它定义了如下五种权限：

- CREATE：允许创建子节点；
- READ：允许从节点获取数据并列出其子节点；
- WRITE：允许为节点设置数据；
- DELETE：允许删除子节点；
- ADMIN：允许为节点设置权限。


## 应用场景

### 数据的发布/订阅
数据的发布/订阅系统，通常也用作配置中心。在分布式系统中，你可能有成千上万个服务节点，如果想要对所有服务的某项配置进行更改，由于数据节点过多，你不可逐台进行修改，而应该在设计时采用统一的配置中心。之后发布者只需要将新的配置发送到配置中心，所有服务节点即可自动下载并进行更新，从而实现配置的集中管理和动态更新。

Zookeeper 通过 Watcher 机制可以实现数据的发布和订阅。分布式系统的所有的服务节点可以对某个 ZNode 注册监听，之后只需要将新的配置写入该 ZNode，所有服务节点都会收到该事件。

### 命名服务
在分布式系统中，通常需要一个全局唯一的名字，如生成全局唯一的订单号等，Zookeeper 可以通过顺序节点的特性来生成全局唯一 ID，从而可以对分布式系统提供命名服务。

### Master选举
分布式系统一个重要的模式就是主从模式 (Master/Salves)，Zookeeper 可以用于该模式下的 Matser 选举。可以让所有服务节点去竞争性地创建同一个 ZNode，由于 Zookeeper 不能有路径相同的 ZNode，必然只有一个服务节点能够创建成功，这样该服务节点就可以成为 Master 节点。

### 分布式锁
可以通过 Zookeeper 的临时节点和 Watcher 机制来实现分布式锁，这里以排它锁为例进行说明：

分布式系统的所有服务节点可以竞争性地去创建同一个临时 ZNode，由于 Zookeeper 不能有路径相同的 ZNode，必然只有一个服务节点能够创建成功，此时可以认为该节点获得了锁。其他没有获得锁的服务节点通过在该 ZNode 上注册监听，从而当锁释放时再去竞争获得锁。锁的释放情况有以下两种：

当正常执行完业务逻辑后，客户端主动将临时 ZNode 删除，此时锁被释放；
当获得锁的客户端发生宕机时，临时 ZNode 会被自动删除，此时认为锁已经释放。
当锁被释放后，其他服务节点则再次去竞争性地进行创建，但每次都只有一个服务节点能够获取到锁，这就是排他锁。

### 集群管理
Zookeeper 还能解决大多数分布式系统中的问题：

如可以通过创建临时节点来建立心跳检测机制。如果分布式系统的某个服务节点宕机了，则其持有的会话会超时，此时该临时节点会被删除，相应的监听事件就会被触发。
分布式系统的每个服务节点还可以将自己的节点状态写入临时节点，从而完成状态报告或节点工作进度汇报。
通过数据的订阅和发布功能，Zookeeper 还能对分布式系统进行模块的解耦和任务的调度。
通过监听机制，还能对分布式系统的服务节点进行动态上下线，从而实现服务的动态扩容。


## 参考资料
[ZooKeeper官网](https://zookeeper.apache.org/doc/current/index.html)

[ZooKeeper 简介及核心概念](https://www.cnblogs.com/lyc88/articles/11361960.html)