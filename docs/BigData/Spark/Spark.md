## Spark概述
### spark 的特点
- Spark 处理数据时，可以将中间结果数据存储到内存中
- Spark提供了非常丰富的算子，可以做到复杂任务在一个Spark程序中完成
### Spark框架模块
- **Spark Core**：Spark Core以RDD 为数据抽象，提供Python Java、Scala、R语言的api
- **SparkSQL**：基于Sprirk Core 之上，提供结构化的数据处理模块，SprakSQL本省针对离线计算场景。同时基于
- **SparkSQL**，sprark提供了`StructredStreaming`模块，可以基于SparkSQL进行数据的流式计算。
- **SparkStreaming**：以Spark Core为基础，提供数据的流式计算。
- **MLlib**：以Spark为基础，进行机器学习。
- **Graphx**：以SparkCore为基础，进行图计算。
### Spark的运行模式
- **本地模式**
以一个独立的进程，通过其内部的多个线程来模拟整个Spark的运行环境
- **Standalone模式**
各个角色以独立的进程的形式存在
- **Hadoop yarn模式**
- **Kubernetes模式**
### Spark的架构
#### 基本概念
- **RDD**: 弹性分布式数据集
- **DAG**: 是向无环图，反映RDD之间的关系
- **Executor**：是运行在工作节点（work node）上的一个进程，负责运行任务，并为程序储存数据
- **应用（Application）**:编写的spark程序
- **任务（Task）**:运行在Executor上的工作单元
- **作业（Job）**:一个作业包含多个RDD及作用于相应RDD上的各种操作
- **阶段（Stage）**:是作业的基本调度单位，一个作业分为多组任务，每组任务被称为“阶段”，或者“业务集”
#### 架构设计
+ 资源管理层面
	+ 集群资源管理者：ResourceManager
	+ 单机资源管理者：NodeManager
+ 任务计算
	+ 单任务管理者：ApplicationMaster
	+ 单任务执行者：Task
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1661688995894.png)

Master进程负责整个集群的资源管理
Worker进程负责所在服务器的资源管理
Driver运行在Master进程内，负责任务的调度管理
Executor运行在Worker进程内，负责某任务的执行

一个应用（Application）有一个任务控制节点（Driver）和若干个作业（Job）构成，一个作业有 多个阶段（Stage）构成，一个阶段由多个任务（Task)组成。当执行一个应用时，任务控制节点会向集群管理器（Cluster Manager）申请资源，启动Executor，并向Executor发送应用程序和文件，然后在Executor上执行任务。运行结束后，执行结果会返回给任务控制点（Driver）,或写到HDFS或其他数据库中
与MapReduce相比，Spark 采用Executor 中的优点：利用多线程执行任务（MapReduce 使用的是进程），Executor中有一个BlockManager存储模块，会将内存和磁盘共同作为储存设备（默认使用内存，内存不够写到磁盘） 
#### 运行基本流程
（1）当一个spark应用被提交时，先为这个应用构建基本的运行环境，由Driver创建一个SparkContext对象，由SparkContext负责与ResourceManager通信以及资源的申请、任务的分配和监控等，SparkContext会向ResourceManger注册并申请运行Executor的资源，SparkContext可以看作应用连接集群的通道
（2）ResourceManager为Executor分配资源，并启动Executor进程，Executor运行情况将随着心跳发送到ResourceManager
（3）SparkContext根据RDD的依赖关系构建DAG图，并提交给DAG调度器进行解析，将DAG分解成多个阶段（每个阶段都是一个任务集），再把任务集提交给低层的任务调度器（TaskScheduler）进行处理；Executor向SparkContext申请任务，任务调度器将任务发送到Executor，同时将程序代码发送Executor
（4）任务在Executor上运行，把执行结果反馈给任务调度器，然后反馈给DAG调度器，运行完成释放资源
#### RDD 
##### RDD的设计
RDD是分布式对象集合，每个RDD可以分为多个分区，每个分区就是一个数据集片段，并且一个RDD的 不同分区保存在不同的节点上，从而可以在集群中的不同节点上进行计算。RDD提供了一种高度受限的共享内存模型，RDD是只读的记录分区集合，不能直接修改，只能基于稳定的物理存储数据来创建RDD,或者通过其他RDD上执行转换操作（Map,Join）创建新的RDD,RDD的数据操作分为Action 和Transformation 两种类型。Action执行计算并指定数据的形式，Transformation 指定RDD之间的关系，转换操作（map,join ,group by,filter）接受并返回RDD，行动操作（count,collect）接受RDD返回非RDD（输出值或者结果）
RDD采用的是惰性调用，在Action之前所有的Transformation的操作都不会触发真正的计算，只会记录RDD生成的轨迹，通过血缘关系把RDD操作实现管道化，避免了多次转换操作之间数据的等待，一个操作的结果直接管道式的流入下一个操作进行处理
##### RDD的特性
1. 容错性，RDD设计中，数据制度不可修改，如果需要修改，则需从父RDD转换到子RDD
2. 中间结持久到内存，数据在内存中的多个RDD操作之间传递不需要落盘
3. 存放的数据可以是java对象，避免不必要的序列化和反序列化
##### RDD之间的依赖关系
RDD的依赖关系分为宽依赖和窄依赖
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/3ce343bbf9d88d8c3ad0e606eec0339.jpg)

- **宽依赖**表现为一个父RDD的分区对应一个子RDD分区 ，或多个父RDD分区对应一个子RDD分区。操作包括groupByKey、sortByKey等会操作shuffle操作
- **窄依赖**表现为一个父RDD的分区对应一个子RDD的多个分区。操作包括map,filter,union等不会shuffle操作
##### RDD操作
1. 转换操作
filter、map、flatmap、groupByKey、reduceBykey
2. 行动操作
行动操作时真正触发计算的地方
count、collect、first、take、redeuce、foreach
##### RDD持久化
Spark 采用的时惰性机制，每次遇到Action操作都会从头执行计算，可以通过持久化来避免这种重复的计算，使用persist()方法将一个RDD标记为持久化，当遇到第一个行动操作的计算之后会把数据持久化，持久化的RDD会保留到计算节点的内存中，被后面的actionc操作重复使用
- **persist(MEMORY_ONLY)**:RDD作为反序列化对象存储在内存中，内存不够时使用LRU算法替换缓存中的内容
- **persist(MEMORY_AND_DISK)**:RDD作为反序列化对象存储在内存中,内存不足时超出的分区会存放在硬盘上