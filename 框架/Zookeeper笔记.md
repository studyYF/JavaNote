### 1.什么是zookeeper

> Apache ZooKeeper is an effort to develop and maintain an open-source server which enables highly reliable distributed coordination.

**zookeeper是一个开源的、高度可靠的分布式协调框架**

> ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services

**zookeeper可以用来做配置中心、命名服务、分布式锁、服务集群管理**

**zookeeper诞生之初是为了解决分布式系统中的一致性问题，多个节点就某一个提议如何达成一致性的问题**（拜占庭将军）

### 2.zookeeper基本使用及相关概念

#### 数据模型

zookeeper的数据模型类似于文件系统，每一个节点称为ZNode，每一个节点都可以保存数据和挂载子节点。

##### 节点类型：

- 持久节点(PERSISTENT)：磁盘中持久化
- 持久有序节点(PERSISTENT_SEQUENTIAL): 节点的子节点会有一个顺序
- 临时节点(EPHEMERAL): 当客户端与zookeeper的链接会话断开后会删除
- 临时有序节点(EPHEMERAL_SEQUENTIAL): 同上
- 容器节点(CONTAINER): 当该节点的所有子节点被删除后，该节点也被删除
- TTL: 创建持久节点和持久有序节点的时候可以设置一个TTL时间，当这个节点在这个时间内没有被修改或者没有子节点的时候，将会被删除

##### 节点存储信息 get stat

-![image-20200327111635367](/Users/yangf/Personal/Note/Resources/image-20200327111635367.png)



##### Watcher监听

zookeeper为每个节点提供了监听机制，我们可以监听一个节点的数据变化、子节点变化等，每个watch只能监听一次。

**Curator框架封装了对zookeeper的各种操作，可以实现注册中心、分布式锁、leader选举等功能**

### 3.zookeeper怎么实现的





### 4.哪些地方用到了zookeeper

很多中间件的集群使用到了zookeeper做leader选举