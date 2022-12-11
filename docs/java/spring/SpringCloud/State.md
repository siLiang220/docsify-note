---
created: 2022-12-10T16:22:58 (UTC +08:00)
tags: [SpringCloudAlibaba---Seata学习篇]
source: https://blog.csdn.net/qq_44692851/article/details/127850038?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EOPENSEARCH%7ERate-3-127850038-blog-127236721.pc_relevant_recovery_v2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EOPENSEARCH%7ERate-3-127850038-blog-127236721.pc_relevant_recovery_v2&utm_relevant_index=3
author: 
---



> ## Excerpt
> SpringCloudAlibaba---Seata学习篇

---
## [Seata官方文档](http://seata.io/zh-cn/docs/overview/what-is-seata.html)

## Seata是什么？

> Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案。AT模式是阿里首推的模式。

## Seata的三大角色

### 1\. TC（Transcation Coordinator）-- 事务协调者

> 维护全局和分布式事务的状态，驱动全局事务提交或回滚

### 2\. TM（Transcation Manager）-- 事务管理器

> 定义全局事务范围：开始全局事务、提交或回滚全局事务

### 3\. RM（Resource Manager）-- 资源管理器

> 管理分支事务处理的资源，与TC交谈以及注册分支事务和报告分支事务的状态，并驱动分支事务的提交或回滚

![在这里插入图片描述](https://img-blog.csdnimg.cn/6bb360d8a25e4fd78042b4f04d58ad33.png)

## 分布式模式

### 1\. AT模式（Auto transcation 自动事务）

> 阿里的Seata就实现了该模式，属于AT的模板  
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/1c463955e55746f2b869cd49e3b0bf93.png)

### AT模式如何做到对业务的无侵入：

#### 1\. 一阶段：

> 在第一阶段，Seata会拦截业务SQL，首先解析SQL语义，找到业务SQL要更新的业务数据，在业务数据被更新前保存为`before image`快照，然后执行业务SQL更新业务数据，在业务数据更新以后保存成`after image`快照，最后生成行锁。以上操作全部在一个数据库事务内完成，保证一阶段事务的操作的原子性

![在这里插入图片描述](https://img-blog.csdnimg.cn/84d04e5fdf7c494285238823ed51263b.png)

#### 2\. 二阶段：

##### 2.1 二阶段提交：

> 如果第一阶段没有问题则在第二阶段提交，因为业务SQL在第一阶段已经提交到数据库，所以Seata框架只需要将第一阶段保存的快照数据与行锁删除即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/ad5707dfdf264284b86d8fc37ba1f3a7.png)

##### 2.2 二阶段回滚：

> 如果第一阶段出现问题则在第二阶段进行回滚。Seata需要回滚第一阶段已执行的业务SQL还原业务数据。回滚的方式就是使用`before image`快照还原业务数据。在还原数据之前首先回进行`脏写校验`,对比业务数据与`after image`快照中的数据是否一致，一致则说明没有出现脏写，可以还原业务数据。出现脏写则需要人工处理，但有行锁其实不会出现脏锁

![在这里插入图片描述](https://img-blog.csdnimg.cn/7641388617344912bcebf7bdaf31f707.png)

### 2\. TCC模式

> TCC模式需要用户根据自己的业务场景实现`·Try、Confirm和Cancel`三个操作。事务发起方在一阶段执行Try方法，二阶段提交执行Confirm方法，二阶段回滚执行Cancel方法

> 缺点：TCC侵入性强，需要用户自己去实现事务控制逻辑  
> 优点：对比Seata的AT模式，在整个过程中没有锁，性能更强，使用上更灵活

![在这里插入图片描述](https://img-blog.csdnimg.cn/ef5f9b5e1fb3449382e266e7f472aa74.png)

## Seata-Server环境搭建(TC)

### 1.1 [Seata下载](https://github.com/seata/seata/releases)

### 1.2 Seata-Server端存储模式支持三种：

> 1.  `File`：（默认）单机模式，全局事务绘画信息内存中读写并持久化本地本舰`root.data`，性能较高
> 2.  `DB`：（数据库版本5.7+）高可用模式，全局事务会话信息通过DB共享，性能较差些
> 3.  Redis：Seata-Server 1.3及以上版本支持，性能较高，存在事务信息丢失风险，需要提前配置适当场景的redis持久化

### 1.3 修改`file.conf`文件，修改mode=‘’为自己使用的模式即可。根据[Seata部署指南](http://seata.io/zh-cn/docs/ops/deploy-guide-beginner.html)的资源目录下找到对应的SQL脚本

### 1.4 Seata+Nacos高可用集群部署

> 修改`registry,conf`配置文件，这里使用的是nacos，修改后配置文件如下

```
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"

  nacos {
    application = "seata-server"
    serverAddr = "127.0.0.1:8848"
    group = "SEATA_GROUP"
    namespace = ""
    cluster = "default"
    username = "nacos"
    password = "nacos"
  }
  eureka {
    serviceUrl = "http://localhost:8761/eureka"
    application = "default"
    weight = "1"
  }
  redis {
    serverAddr = "localhost:6379"
    db = 0
    password = ""
    cluster = "default"
    timeout = 0
  }
  zk {
    cluster = "default"
    serverAddr = "127.0.0.1:2181"
    sessionTimeout = 6000
    connectTimeout = 2000
    username = ""
    password = ""
  }
  consul {
    cluster = "default"
    serverAddr = "127.0.0.1:8500"
  }
  etcd3 {
    cluster = "default"
    serverAddr = "http://localhost:2379"
  }
  sofa {
    serverAddr = "127.0.0.1:9603"
    application = "default"
    region = "DEFAULT_ZONE"
    datacenter = "DefaultDataCenter"
    cluster = "default"
    group = "SEATA_GROUP"
    addressWaitTime = "3000"
  }
  file {
    name = "file.conf"
  }
}

config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "nacos"

  nacos {
    serverAddr = "127.0.0.1:8848"
    namespace = ""
    group = "SEATA_GROUP"
    username = "nacos"
    password = "nacos"
  }
  consul {
    serverAddr = "127.0.0.1:8500"
  }
  apollo {
    appId = "seata-server"
    apolloMeta = "http://192.168.1.204:8801"
    namespace = "application"
  }
  zk {
    serverAddr = "127.0.0.1:2181"
    sessionTimeout = 6000
    connectTimeout = 2000
    username = ""
    password = ""
  }
  etcd3 {
    serverAddr = "http://localhost:2379"
  }
  file {
    name = "file.conf"
  }
}

```

### 1.5 事务分组

#### 1.5.1 事务分组的作用：

> 事务分组主要是为了高可用异地机房容错，在一个机房突发情况下可以迅速切换到另外一个机房。配置事务分组需要与客户端配置的事务分组一致

#### 1.5.2 如何实现事务分组：

> 这个需要在部署指南中下载script文件夹，并存放到Seata目录下 `script/config-center/config.txt`
> 
> #default\_tx\_group需要与客户端保持一致 default需要跟客户端和registry,conf中的cluster保持一致
> 
> #客户端中的配置：spring.cloud.alibaba.seata.tx-service-group=default\_tx\_group
> 
> default\_tx\_group是事务分组名词，可以自定义

##### config.txt主要修改事项分组与数据库信息。config.txt下存放的是全局配置信息

```
#Transaction routing rules configuration, only for the client
# 事务分组
service.vgroupMapping.my_test_tx_group=default
#If you use a registry, you can ignore it
service.default.grouplist=127.0.0.1:8091
service.enableDegrade=false
service.disableGlobalTransaction=false

#Transaction storage configuration, only for the server. The file, DB, and redis configuration values are optional.
store.mode=db
#These configurations are required if the `store mode` is `db`. If `store.mode,store.lock.mode,store.session.mode` are not equal to `db`, you can remove the configuration block.
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.jdbc.Driver
store.db.url=jdbc:mysql://127.0.0.1:3306/seata?useUnicode=true&rewriteBatchedStatements=true
store.db.user=root
store.db.password=123456
```

### 1.6 快速把配置信息配置到nacos配置中心：只需要在`script/config-center/nacos`下运行`nacos-comfig.sh`脚本即可

![](https://img-blog.csdnimg.cn/ba8bf961a38f4532839108d6e2b5b11c.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/b6a09ce7f00343b38e497ade0db1aa67.png)

## 使用Seata加入微服务项目中

### 1\. 添加依赖

```
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
```

### 1.2 在各微服务数据中添加数据表`undo_log`：作用于回滚记录日志

```
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

```

### 1.3 在各微服务配置文件中添加`事务分组`配置信息及`其他相关配置信息`

```
spring:
  cloud:
    alibaba:
      seata:
        tx-service-group: my_test_tx_group#事务分组 需与config.txt事务分组中一致
seata:
  registry:
    # 配置seata的注册中心 告诉Seata client怎么去访问seata server 这里通过nacos去实现
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      application: seata-server     #注册到nnacos上seata-server服务名称 默认可省略
      username: nacos
      password: nacos
      group: SEATA_GROUP      #seata server所属分组 默认可省略
      #seata的配置中心
  config:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      username: nacos
      password: nacos
```

### 1.4 添加seata事务注解`@GlobalTransactional`

> 调用接口可以发现微服务事务已经生效，并且可以通过打断点查询到每次操作数据中`undo_log`表都会存储数据作为异常回滚

![在这里插入图片描述](https://img-blog.csdnimg.cn/28dbafd7949144dfa90914e4aa7cc0a4.png)
