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
1. 当一个spark应用被提交时，先为这个应用构建基本的运行环境，由Driver创建一个SparkContext对象，由SparkContext负责与ResourceManager通信以及资源的申请、任务的分配和监控等，SparkContext会向ResourceManger注册并申请运行Executor的资源，SparkContext可以看作应用连接集群的通道
2. ResourceManager为Executor分配资源，并启动Executor进程，Executor运行情况将随着心跳发送到ResourceManager
3. SparkContext根据RDD的依赖关系构建DAG图，并提交给DAG调度器进行解析，将DAG分解成多个阶段（每个阶段都是一个任务集），再把任务集提交给低层的任务调度器（TaskScheduler）进行处理；Executor向SparkContext申请任务，任务调度器将任务发送到Executor，同时将程序代码发送Executor
4. 任务在Executor上运行，把执行结果反馈给任务调度器，然后反馈给DAG调度器，运行完成释放资源

### RDD 

#### RDD的设计
RDD是分布式对象集合，每个RDD可以分为多个分区，每个分区就是一个数据集片段，并且一个RDD的 不同分区保存在不同的节点上，从而可以在集群中的不同节点上进行计算。RDD提供了一种高度受限的共享内存模型，RDD是只读的记录分区集合，不能直接修改，只能基于稳定的物理存储数据来创建RDD,或者通过其他RDD上执行转换操作（Map,Join）创建新的RDD,RDD的数据操作分为Action 和Transformation 两种类型。Action执行计算并指定数据的形式，Transformation 指定RDD之间的关系，转换操作（map,join ,group by,filter）接受并返回RDD，行动操作（count,collect）接受RDD返回非RDD（输出值或者结果）
RDD采用的是惰性调用，在Action之前所有的Transformation的操作都不会触发真正的计算，只会记录RDD生成的轨迹，通过血缘关系把RDD操作实现管道化，避免了多次转换操作之间数据的等待，一个操作的结果直接管道式的流入下一个操作进行处理

#### RDD的特性
1. 容错性，RDD设计中，数据制度不可修改，如果需要修改，则需从父RDD转换到子RDD
2. 中间结持久到内存，数据在内存中的多个RDD操作之间传递不需要落盘
3. 存放的数据可以是java对象，避免不必要的序列化和反序列化

#### RDD之间的依赖关系
RDD的依赖关系分为宽依赖和窄依赖

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/3ce343bbf9d88d8c3ad0e606eec0339.jpg)


- **宽依赖**表现为一个父RDD的分区对应一个子RDD分区 ，或多个父RDD分区对应一个子RDD分区。操作包括groupByKey、sortByKey等会操作shuffle操作
- **窄依赖**表现为一个父RDD的分区对应一个子RDD的多个分区。操作包括map,filter,union等不会shuffle操作

#### RDD操作
1. 转换操作
filter、map、flatmap、groupByKey、reduceBykey
2. 行动操作
行动操作时真正触发计算的地方
count、collect、first、take(以数组形式返回数据集中的前n个元素)、redeuce、foreach

#### RDD持久化
Spark 采用的时惰性机制，每次遇到Action操作都会从头执行计算，可以通过持久化来避免这种重复的计算，使用persist()方法将一个RDD标记为持久化，当遇到第一个行动操作的计算之后会把数据持久化，持久化的RDD会保留到计算节点的内存中，被后面的actionc操作重复使用
- **persist(MEMORY_ONLY)**:RDD作为反序列化对象存储在内存中，内存不够时使用LRU算法替换缓存中的内容
- **persist(MEMORY_AND_DISK)**:RDD作为反序列化对象存储在内存中,内存不足时超出的分区会存放在硬盘上
当不需要RDD时，使用`unpersist()`方法将持久化的RDD释放

#### RDD分区

##### 分区的作用
1. RDD是弹性分布式数据集，RDD会被分成多个分区，分布在不同的节点上，增加任务并行度
2. 减少网络开销，执行连接操作时会将两个数据集中的所有key的哈希值求出，将哈希值相同的记录发送到同一台机器上，在这台机器上对所有的Key相同的执行连接操作
未分区的时进行的连接操作,每次连接操作都会有数据混洗的操作，在这种情况下仍会造成网络开销

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662202174443.png)

选择对userData进行哈希分区后的连接操作

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662202504344.png)

##### 分区的原则

RDD分区的一个原则是使得分区个数尽量等于集群中CPU core 数，对于不同的部署模式可以通过设置`spark.default.parallelism`参数的值设置默认的分区数
- local模式：默认本地机器的CPU数，若设置`local[N]`,则默认N
- Standalone 或者 Yarn 模式：在集群中所有的CPU数总和与2比较取最大值
- Mesos模式：默认分区是8
**设置分区个数**
1. 创建RDD时手动指定分区的个数，在调用textFile()和parallelize()方法手动指定分区个数
	- parallelize()没有指定分区数，默认为`spark.default.parallelism`
	- textFile()没有指定分区，默认为min(defaultParallelism,2),defaultParallelism是`spark.default.parallelism`,如果从HDFS中读取文件，则分区数为文件的分片数
2. 使用`repartition`方法设置分区的个数，通过转换得到RDD时，直接调用`repartition`方法
3. 自定义分区方法
	待补充

## Spark SQL 简介
Spark SQL中所使用的数据抽象非RDD，而是DataFrame，RDD是java对象的集合，DataFrame是以RDD为基础的分布式数据集，提供详细的表结构信息，相当于关系数据库的一张表

### DataFrame创建

| 创建方式                                  | 描述                |
| ----------------------------------------- | ------------------- |
| read.text()                               | 读取文本文件中创建  |
| read.json()                               | 读取json文件创建    |
| read.parquet()                            | 读取parquet文件创建 |
| read.format('text').load("xx.text")       | 读取文本文件创建    |
| read.format('json').load("xx.json")       | 读取json文件创建    |
| read.fromat('parquet').load("xx.parquet") | 读取parquet文件创建    | 


### RDD转换为DataFrame
1. 利用反射方式推断RDD模式，使用数据结构已知的RDD转换
2. 使用编程方式定义RDD

## Spark Streaming

>Spark Streaming 主要是抽像的**DStream** (离散化数据流)，表示连续不断的数据流。在内部实现上，Spark Streaming的输入数据按照时间片拆分，然后将每一段数据转换为Spark 的RDD，并且对DStream的操作最终转变为对应的RDD操作。

### Spark Streaimg 与 Storm 区别

Spark Streaming 无法实现毫秒级运算，Storm 可以实现毫秒级响应
Spark Streaming 将流数据分解为一系列的批处理作业，在作业过程当中会产生多个Spark 作业，每个作业都要经过Spark DAG图分解，任务调度等过程，需要一定的时间开销。storm 采用的是以数据为元组

### Sprak Streaming 基本步骤
1. 创建`DStream` 来定义输入源。流计算处理的数据对象是来自输入源的数据，并发送给Spark Streaming，由`Receiver`组件接收交给自定义的Spark Streaming程序处理
2. 通过`DStream` 应用转换操作和输出操作定义流计算
3. 调用`StreamingContext` 对象`start()`方法开始接收数据和处理流程
4. 调用`StreamContext`对象的`awaitTermination()`方法等待流计算进程结束，或者调用`stop()`方法手动结束流计算进程
- pySpark中编写
```python
from pyspark.streaming import StreamingContext
# sc 是默认创建的SparkContext 对象,1表示每一秒数据切分一个片段
ssc = StreamingContext(sc,1)
```
- 独立程序中编写
```python
from pyspark import SparkContext,SparkConf
from pyspark.streaming import SreamingContext
conf = SparkConf()
conf.setAppName('test')
conf.setMaster('local[2]')
sc=SparkContext(conf = conf)
ssc = StreamingContext(sc,1)
```

### Streaming RDD队列流
用StreamingContext.queueStream(queueOfRDD)创建基于RDD的DStream,每个1s创建一个RDD,加入到队列中,每隔2s对DStream进处理
```python
from pyspark import SparkContext
from spark.streaming import StreamingContext
if__name__=“main”:
sc = SparkContext(appName=‘PythonStreamingQueueStream’)
ssc =StreamingContext(sc,2)
#下面创建一个RDD队列流加了5次
rddQueue = []
for i in range(5):
rddQueue += [ssc.saprkContext.parallelize([j for j in range(1,1001)],10)]#10是分区，每次生成一千个元素
time.sleep(1)#每隔1s筛一个RDD队列

Input = ssc.queueStream(rddQueue)
mappedStream = input.map(lamda x:(x%10,1))
reducedStream = mappedStream.reduceByKey(lambda a,b:a+b)
reducedStream.pprint()
ssc.start()
ssc.stop(stopSparkContext=True,stopGraceFully=True)
```

### 转换操作

#### 1\. DStream无状态操作
| 操作         | 含义                                                          |
| ------------ | ------------------------------------------------------------- |
| map          | 转换得到一个新的DStream                                       |
| flatmap      | 输出多个项                                                    |
| filter       | 返回满足要求的DStream                                         |
| reparitition | 创建更多或者更少的分区改变DStream的并行度                     |
| reduce       | 聚合DStream 的元素,返回一个单元素RDD的DStream                 |
| count        | 统计每个RDD元素数量                                           |
| union        | 返回一个包含源DStream和其他DStream                            |
| countByValue | 返回一个(K,V)的新DStream,键的值是在源DStream的每个RDD出现次数 |
| reduceByKey  | 返回一个新的(K,V)DStream,每个key的值均给定的reduce函数聚合    |
| join         |                                                               |
| cogroup      | 返回一个包含(K,seq(V),seq(W))的元组                           |
| transform    | 对源DStream的每个RDD应用RDD-to-RDD函数,创建一个新的DStream    | 

#### 2\.DStream 有状态操作
##### 1\. 滑动窗口转换操作
设定滑动窗口的实践间隔,让窗口按照在指定时间间隔在DStream上滑动

| 操作                  | 含义                                                                                                        |
| --------------------- | ----------------------------------------------------------------------------------------------------------- |
| window                | 窗口化的批数据                                                                                              |
| countByWindow         | 获取流元素中一个滑动窗口数                                                                                  |
| reduceByWindow        | 对窗口内的元素进行聚集,得到一个单元素流                                                                     |
| reduceByKeyAndWindow  | 返回一个(K,V)组成的新DStream,key的值是由自定义函数计算得到,还有一个可选参数对离开窗口的数据做逆向reduce操作 |
| countByValueAndWindow | 返回一个(K,V)组成的新DStream,key的值是在滑动窗口出现的频率                                                  |


![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662276749772.png)

##### 2\. updateStateByKey
滑动窗口操作只能对当前窗口内的数据进行计算,无法跨批次之间维护状态.如果要在跨批次之间维护状态,就需要使用`updateStateByKey` 。 例如在词频统计时，本批次的词频统计结果要在之前的统计结果的基础上累加。

## Structured Streaming
### 概述
Structured Streaming 关键思想时将实时数据流视为一张在不断添加数据的表。流计算等同于在一个静态表上的批处理查询，Spark不断添加数据到无界表上进行运行，并进行批量查询

Structured Streaming 的编程模型

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662277694133.png)


### 处理模型

#### 1\. 微批处理模型
Structured Streaming 默认使用微批处理模型，Spark流计算引擎会定期检查数据源，并对自上一批次结束后到达的新数据执行批量查询，`Derive`驱动程序通过将当前待处理数据的偏移量保存到预写入日志中，对数据处理进度设置检查点，以便以后重新启动或恢复查询。为获得确定性的重新执行和端到端语义，在下个微批处理之前，就要对该微批所要处理的数据的偏移范围保存到日志，当前到达的数据需要等待先前的批作业处理完成，且偏移范围被记入日志后才能在下一个微批作业中得到处理，这会导致数据到达和得到处理并输出结果之间存在延时，但可保证端到端的完全一致性

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662278553249.png)


#### 2\.持续处理模型
在每个任务的输入数据流中，一个特殊标记的记录被注入，当任务遇到标记时，任务把处理后的最后偏移量异步报告给引擎，引擎到接所有写入接收器的任务的偏移量后，写入预写日志。任务可以持续处理，降低延迟到毫秒级。但故障会导致数据流可能被处理超过一次以上，持续处理只能做到“至少一次” 的一致性。

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662279346398.png)

### Structured Streaming 和 Spark SQL、Spark Streaming 之间的区别
Structured Streaming 和Spark Streaming 都是处理原源源不断的流数据，Spark Streaming 采用的是DStream （本质上就是RDD），Structrued Streaming采用的是DataFrame，Structured Streaming可以采用Spark SQL的DataFrame/DataSet来处理数据。但Spark SQL 只能处理静态数据，Structured 可以处理结构化数据

