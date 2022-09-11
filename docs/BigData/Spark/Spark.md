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

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662639295883.png)


### SparkSession
- SparkSession 主要用在`SparkSQL`，也可以用在其他场合，可代替`sparkContext`
- SparkSession 实际上封装了`SparkContext`，另外也封装了`SparkConf`，`sqlContext`
- SparkSession 可以拿到`SparkContext`和其他的`Context`

```python

object SparkSessionExample{
  
  def main(args: Array[String]){
   val spark = SparkSession
			.builder()
			.appName("parquet")
			.config("spark.yarn.maxAppAttempts", 1)
			.enableHiveSupport()     
			.getOrCreate()
			.master("local[*]")
	val df = sparkSession.read.option("header","true").csv("src/sales.csv")
 
    df.show()
   }
}
```

- appName() 设置的应用程序名
- config() 重载函数设置配置项
- enableHiveSupport 表示支持hive,包括链接持久化hive metasore ，支持hive serdes，hdf
- getOrCreate() 获取已经得到的sparkSession,不存在则依据builder选项创建新的SparkSession
- master() 设置spark master url 的连接，比如`local[*]`
- withExtensions(scala.Function1<SparkSessionExtensions,scala.runtime.BoxedUnit> f)
这允许用户添加Analyzer rules, Optimizer rules, Planning Strategies 或则customized parser.这一函不常见的。


Structured Streaming 的编程模型

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662277694133.png)

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662639448830.png)

Structured Streaming 处理不会体现整个表，从流数据源中读取最新的可用数据，以增量的方式对处理以更新结果，然后丢弃源数据，仅保留更新结果所需的最小中间状态

### 处理模型

#### 1\. 微批处理模型
Structured Streaming 默认使用微批处理模型，Spark流计算引擎会定期检查数据源，并对自上一批次结束后到达的新数据执行批量查询，`Derive`驱动程序通过将当前待处理数据的偏移量保存到预写入日志中，对数据处理进度设置检查点，以便以后重新启动或恢复查询。为获得确定性的重新执行和端到端语义，在下个微批处理之前，就要对该微批所要处理的数据的偏移范围保存到日志，当前到达的数据需要等待先前的批作业处理完成，且偏移范围被记入日志后才能在下一个微批作业中得到处理，这会导致数据到达和得到处理并输出结果之间存在延时，但可保证端到端的完全一致性

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662278553249.png)


#### 2\.持续处理模型
在每个任务的输入数据流中，一个特殊标记的记录被注入，当任务遇到标记时，任务把处理后的最后偏移量异步报告给引擎，引擎到接所有写入接收器的任务的偏移量后，写入预写日志。任务可以持续处理，降低延迟到毫秒级。但故障会导致数据流可能被处理超过一次以上，持续处理只能做到“至少一次” 的一致性。

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662279346398.png)

### Structured Streaming 和 Spark SQL、Spark Streaming 之间的区别
Structured Streaming 和Spark Streaming 都是处理原源源不断的流数据，Spark Streaming 采用的是`DStream `（本质上就是RDD），Structrued Streaming采用的是`DataFrame`，Structured Streaming可以采用Spark SQL的`DataFrame/DataSet`来处理数据。但Spark SQL 只能处理静态数据，Structured 可以处理流数据

### 输出操作

#### 启动流计算输出
DataFrame/DataSet 的writeStream()方法返回DataStremWriter接口，接口通过start()真正启动流计算，并将DataFrame/DataSet写入外部的输出接收器
- **DataFrameWriter接口包括的函数**
	 - **format**：接收器类型
	 - **outputMode**：数据模式，指定写入接收器的内容，`Append`、`Complete`、`Update` 模式
	 - **queryName**：查询的名字，可选，标识唯一的名称
	 - **trigger**：触发间隔，可选。如果未指定，则系统将在上一次处理完成之后立即检查新数据的可用性。如果由于先前的处理未完成导致超过触发间隔，则系统将在处理完成后立即触发新的查询
	 
```python
query = wordcount
		.outputMode("update")
		.format("console")
		.option("truncate","false")
		.trigger(processingTime="8 seconds" )
		.start()
```

#### 输出模式
- **Append**：只有结果表中上次触发间隔后增加的新行，才会写入外部储存
- **Complete**：已更新的完整结果表可被写入外部存储器
- **Update**：只有上次触发间隔后结果表中发生更新的行，才会被写入到储存器

流计算查询类型与输出类型的兼容性
  <div class="table-box"><table><tbody><tr><th>查询类型</th><th>&nbsp;</th><th>支持的数据模式</th><th>说明</th></tr><tr><td rowspan="2">带聚合的查询</td><td>带水印的基于事件时间的聚合</td><td>Append, Update, Complete</td><td> <p>Append模式使用水印来删除旧的聚合状态. 但是，窗口聚合输出根据withWatrmark()的定义延迟了指定的延迟阈值，行只能在最后确定之后(即在水印交叉之后)添加到结果表中一次<br><br> Update模式使用水印来删除旧的聚合状态</p> <p><br> Complete模式不会删除旧的聚合状态，因为根据定义，该模式保留结果表中的所有数据。</p> </td></tr><tr><td>其他聚合</td><td>Complete, Update</td><td>由于没有定义水印，所以不会删除旧的聚合状态。<br><br> 不支持Append模式，没有定义水印导致聚合可能会一直更新，因此会违反Append模式的语义。</td></tr><tr><td colspan="2"><code>使用了mapGroupsWithState的查询</code></td><td>Update</td><td>&nbsp;</td></tr><tr><td rowspan="2">使用了`flatMapGroupsWithState`的查询</td><td>Append 操作模式</td><td>Append</td><td>在调用`flatMapGroupsWithState`之后允许聚合。</td></tr><tr><td>Update 操作模式</td><td>Update</td><td>在调用`flatMapGroupsWithState`之后允许聚合。</td></tr><tr><td colspan="2">连接操作</td><td>Append</td><td>不支持Update和Complete模式</td></tr><tr><td colspan="2">其他查询</td><td>Append, Update</td><td>不支持Complete模式，因为在结果表中保留所有未聚合的数据是不可行的。</td></tr></tbody></table></div>

spark 内置的输出接收器

| 接收器      | 支持的输出模式           | 选项                                  | 容错                          |
| ----------- | ------------------------ | ------------------------------------- | ----------------------------- |
| File        | Append                   | 输出的路径必须指定                    | YES,数据只会被处理一次        |
| Kafka       | Append、Complete、Update | 参考kafka                             | YES,数据只会被处理一次        |
| Foreach     | Append、Complete、Update | 无                                    | 去决议ForeachWriter的实现     |
| FoeachBatch | Append、Complete、Update | 无                                    |                               |
| Console     | Append、Complete、Update | numRow ：默认20行，truncate：是否截断 | NO                            |
| Memory      | Append、Compelte         | 无                                    | NO,Compelte模式，查询会重建表 |

### 容错处理
Spark 输入源通过设置偏移量标记目前处理的位置，引擎通过检查点保存中间状态，接收器可以使用幂等的接收器保障输出的稳定性
假设每个流式源具有偏移量（kafka offset 或 kinesis 序列号） 以跟踪流中的读取位置，引擎使用检查点和预写日志记录每个触发器正在处理的数据偏移范围。接收器设计为具有处理和再处理的的幂等性，结合使用可重放源和幂等性，结构化流确保端到端的精确一致性语义

#### 从检查点恢复故障
正确的配置容错环境包括容错的输入源，记录数据源位置的偏移量，保存检查点和预写日志的中间状态，以及使用容器的接收器，记录输入源位置偏移量、检查点、预写日志由Spark引擎完成，只需要提供检查点的路径，Spark引擎就会保存恢复的必要数据

#### 故障恢复的限制
在程序停止运行和故障发生后，可能修改部分程序并重启查询，这时仍要使用就的检查点数据恢复故障，但有写修改操作时不被允许的

- 输入源的类型和数量的更改
- 输入源的数据参数的更改：部分不会影响到检查点状态的参数可以修改，如kafka的`maxOffesetPerTrigge`等限速参数，而修改kafka的主题和文件路径时不被允许的
- 接收器的类型更改：Filed接收器可以改为Kafka接收器，但kafka只能接收到新的数据，kafka不能改为File接收器，kafka接收器和foreach接收器可以相互替换
- 接收器的参数更改：不可更改file接收器的路径，kafka的输出主题允许被更改，Foreach接收器自定义参数可以更改
- projcction、filter、map等类型操作更改：部分允许，如增加或者删除过滤条件等
- 有状态的操作更改：有些流计算查询操作需要保存状态到检查点，以便数据持续到来时更新查询结果，这种不被允许

### 事件时间在窗口上的操作
滑动窗口在10分钟的窗口内计算字数，每五分钟更新一次，也就是在10分钟的窗口内12：00 - 12：10、12：05 - 12：15、12：10 - 12：20 等之间收到的字数统计

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662642471255.png)

#### 迟到数据处理
如下，12:04（事件时间）生成的单词可以在12:11被应用程序接收，应用程序该使用时间是12:04 （12:00-12:10窗口）更新计算 而不是用处理时间12:11（12:05-12:10窗口）来更新计算
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662643507704.png)

在Spark 2.1 中引入`watermarking` ,让引擎自动跟踪数据中的evnet-time,并尝试清理旧状态，可以通过指定时间列来定义查询的`watermarking`，并根据事件时间指定数据延迟的阈值。对于在time结束的特定窗口T，引擎保持状态，并允许延迟数据 更新状态，直到引擎看到最大事件时间-迟到的最大阈值 >T（max event time seen by the engine - late threshold > T）。也就是阈值内的迟到数据将被聚合，超过阈值的数据将被丢弃

```python
words = ...  # streaming DataFrame of schema { timestamp: Timestamp, word: String }

# Group the data by window and word and compute the count of each group
windowedCounts = words 
    .withWatermark("timestamp", "10 minutes") 
    .groupBy(
        window(words.timestamp, "10 minutes", "5 minutes"),
        words.word) 
    .count()
```
在此示例中，在“timestamp”列上定义查询水印，并设置10分钟为允许数据延迟的阈值

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662644457605.png)
引擎跟踪的最大事件时间为 _蓝色虚线_，每次触发开始时设置的水印为`(max event time - '10 mins')` 红线。例如，当引擎观察到数据时 `(12:14, dog)`，它将下一次触发的水印设置为`12:04`。此水印让引擎将中间状态保持额外 10 分钟，以允许对迟到的数据进行计数。比如数据`(12:09, cat)`乱序和延迟，落入windows`12:00 - 12:10`和`12:05 - 12:15`. 由于它仍然领先`12:04`于触发器中的水印，因此引擎仍然将中间计数保持为状态并正确更新相关窗口的计数。但是，当水印更新为`12:11`，窗口的中间状态`(12:00 - 12:10)`被清除，所有后续数据（例如`(12:04, donkey)`）被认为“为时已晚”，因此被忽略。请注意，在每次触发之后，更新的计数（即紫色行）将作为触发输出写入接收器，如更新模式所指示。

>在非流数据集上使用`withWatermark` 是无效的

#### 时间窗口的类型
Spark 支持三种类型的时间窗口：滚动、滑动、会话
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662776412083.png)

会话窗口，
```python
events = ...  # streaming DataFrame of schema { timestamp: Timestamp, userId: String }

# Group the data by session window and userId, and compute the count of each group
sessionizedCounts = events \
    .withWatermark("timestamp", "10 minutes") \
    .groupBy(
        session_window(events.timestamp, "5 minutes"),
        events.userId) \
    .count()
```

动态间隙时间
```python
from pyspark.sql import functions as F

events = ...  # streaming DataFrame of schema { timestamp: Timestamp, userId: String }

session_window = session_window(events.timestamp, \
    F.when(events.userId == "user1", "5 seconds") \
    .when(events.userId == "user2", "20 seconds").otherwise("5 minutes"))

# Group the data by session window and userId, and compute the count of each group
sessionizedCounts = events \
    .withWatermark("timestamp", "10 minutes") \
    .groupBy(
        session_window,
        events.userId) \
    .count()
```
- 在使用会话窗口的限制
	- 不支持“Update mode” 为输出模式
	- 除了session_window分组键之外，至少还有列
默认情况下，Spark不会对会话窗口执行部分聚合，因为它需要在分组之前在本地进行额外的排序。
`spark.sql.streaming.sessionWindow.merge.sessions.in.local.partition`以指示 Spark 执行部分聚合。

#### 清除水印聚合状态的条件
- 输出模式必须为“追加”或“更新”。完整模式要求保留所有聚合数据，因此不能使用水印删除中间状态
- 聚合必须具在 event-time 列，或者`window`在 event-time 列上有一个
- `withWatermark` 必须在与聚合中使用的时间戳列调用。例如， `df.withWatermark("time", "1 min").groupBy("time2").count()`在追加输出模式下无效，因为水印是在与聚合列不同的列上定义的。
- `withWatermark`必须在聚合之前调用才能使用watermark 详细信息。例如，`df.groupBy("time").count().withWatermark("time", "1 min")`在追加输出模式下无效。

#### 带有水印的聚合语义保证
-  watermark 延迟 (set with `withWatermark`) “2 小时“，确保引擎永远不会丢弃延迟少于2小时的数据。
- 但是，这种只在一个方向上严格。延迟2小时以上的数据不保证被丢弃；他可能被聚合，也可能不会聚合。数据延迟，引擎处理的可能性越小

### Spark Structured Streaming join
#### join操作
`Structured Streaming`支持将DataSet/DataFrame 与Static DataSet/DataFreame 与另一个Stream DataSet/DataFreame join 在一起，Stream join 的结果是增量生成的

##### 流静态 join

Spark 2.0开始`Structured Streaming` 支持流和 static Dataset/DataFrame之间连接
```python
staticDf = spark.read. ...
streamingDf = spark.readStream. ...
streamingDf.join(staticDf, "type")  # inner equi-join with a static DF
streamingDf.join(staticDf, "type", "left_outer")  # left outer join with a static DF
```

静态流是没有状态，因此不需要状态管理

##### 流流join
Spark 2.3开始支持了stream-stream join的支持，可以连接两个stream DataSet/DataFrame。在任何时间点，两侧数据集视图都不完整spark 过去的输入流状态进行缓存，以便每个将来输入与过去的输入进行匹配，并相应地生成合并的结果。与流式聚合类似，Spark可以自动处理迟到的乱数据，并可以使用watermark 限制状态

##### 可选水印的 Inner Joins
为了避免无界状态，需定义额外的连接条件，以便无期限的旧输入与未来的输入匹配
1. 在两个输入上定义watermark 延迟，以便引擎知道输入可以延迟多长时间
2. 定义两个输入之间的事件时间约束，以便引擎可以确定何时不需要一个输入的旧行（即不满足时间约束）与另一个输入匹配
	- 时间范围加入条件（JOIN ON leftTime BETWEEN rightTime AND rightTime + INTERVAL 1 HOUR）。
	- 加入事件时间窗口（JOIN ON leftTimeWindow = rightTimeWindow）。

示例：广告展示与另一个用户点击广告流结合，以关联广告何时获利的点击
1. 水印延迟：展示次数和相应的点击在事件时间中可能分别延迟/无序最多 2 小时和 3 小时。
2.  事件时间范围条件：比如说，点击可以发生在相应展示后0秒到1小时的时间范围内。

```python
from pyspark.sql.functions import expr
impressions = spark.readStream. ...
clicks = spark.readStream. ...
# Apply watermarks on event-time columns
impressionsWithWatermark = impressions.withWatermark("impressionTime", "2 hours")
clicksWithWatermark = clicks.withWatermark("clickTime", "3 hours")
# Join with event-time constraints
impressionsWithWatermark.join(
  clicksWithWatermark,
  expr("""
    clickAdId = impressionAdId AND
    clickTime >= impressionTime AND
    clickTime <= impressionTime + interval 1 hour
    """)
)
```

###### 带水印的流-流内连接语义保证
与带有水印的聚合类似，“2 小时”的水印延迟保证引擎永远不会丢弃任何延迟少于 2 小时的数据。但是延迟超过 2 小时的数据可能会也可能不会被处理。

##### 带水印的Outer Join
>水印 + 事件时间约束对于内部连接是可选的，但对于外部连接必须指定，**为了在外连接中产生NULL结果**，引擎必须知道将来什么时候输入行不匹配任何内容。因此，必须指定水印+事件时间约束

```python
impressionsWithWatermark.join(
  clicksWithWatermark,
  expr("""
    clickAdId = impressionAdId AND
    clickTime >= impressionTime AND
    clickTime <= impressionTime + interval 1 hour
    """),
  "leftOuter"                 # can be "inner", "leftOuter", "rightOuter", "fullOuter", "leftSemi"
)
```

###### 带水印的流外连接语义保证
与内连接的水印延迟和是否丢弃数据相同

###### 注意事项
-   _外连接 NULL 结果生成，延迟取决于指定的水印延迟和时间范围条件。_ 因为引擎必须等待很长时间才能确保没有匹配项，并且将来不会有更多匹配项。

-   在微批处理引擎中的当前实现中，水印是在一个微批处理结束时被推进，下一个微批处理使用更新后的水印来清理状态并输出外部结果。由于我们仅在有新数据要处理时才触发微批处理，因此如果流中没有接收到新数据，则外部结果的生成可能会延迟。 _简而言之，如果正在连接的两个输入流中的任何一个在一段时间内没有接收到数据，则外部（左或右）输出可能会延迟。_

##### 带水印的Semi Joins
>右表只用于过滤左表的数据且不出现在结果集中，**半连接必须指定watermark和eventTime约束**
- **left semi joins**：结果集仅仅保留左表的行，这些行的joinKey出现在右表中，这种join是会出重的，当左边表join到一个之后便返回不在继续join。
- **left anti joins**：结果集joinKey不在右表中

###### 带水印的流-流半连接语义保证
半联接与内联接在水印延迟和是否丢弃数据相同。

##### 支持流查询的连接矩阵
<table><tbody><tr><td><strong>左输入</strong></td><td><strong>右输入</strong></td><td><strong>连接类型</strong></td><td>&nbsp;</td></tr><tr><td>Static</td><td>Static</td><td>所有类型</td><td>支持，因为它不在流数据上，即使它可以存在于流式查询中</td></tr><tr><td colspan="1" rowspan="4">Stream</td><td colspan="1" rowspan="4">Static</td><td>内连接</td><td>支持，无状态</td></tr><tr><td>左外连接</td><td>支持，无状态</td></tr><tr><td>右外连接</td><td>不支持</td></tr><tr><td>全外连接</td><td>不支持</td></tr><tr><td colspan="1" rowspan="4">Static</td><td colspan="1" rowspan="4">Stream</td><td>内连接</td><td>支持，无状态</td></tr><tr><td>左外连接</td><td>不支持</td></tr><tr><td>右外连接</td><td>支持无状态</td></tr><tr><td>全外连接</td><td>不支持</td></tr><tr><td colspan="1" rowspan="4">Stream</td><td colspan="1" rowspan="4">Stream</td><td>内连接</td><td>支持，可选择在两侧指定水印+状态清理的时间限制</td></tr><tr><td>左外连接</td><td>有条件支持，必须在右侧指定水印 + 时间约束上以获得正确的结果，可选择在左侧指定水印以进行所有状态清理</td></tr><tr><td>右外连接</td><td>有条件支持，必须在左侧指定水印 + 时间约束上以获得正确的结果，可选择在右侧指定水印以进行所有状态清理</td></tr><tr><td>全外连接</td><td>不支持</td></tr></tbody></table>

有关支持的连接大的其他详细信息：
-   连接可以级联，也就是说，你可以做`df1.join(df2, ...).join(df3, ...).join(df4, ....)`.
-   从 Spark 2.4 开始，您只能在查询处于附加输出模式时使用连接。尚不支持其他输出模式。
-   从 Spark 2.4 开始，您不能在连接之前使用其他非地图类操作。以下是一些不能使用的示例。
    -   在加入之前不能使用流式聚合。
    -   加入前不能在更新模式下使用 mapGroupsWithState 和 flatMapGroupsWithState。

### 流式重复数据删除
数据的采集到最终的处理可能存在一条数据在某一个点被重复处理的情况。如`kafkfa`的至少一次语义，写数据到`kafaka`的时候，有些记录可能是重复的，例如消息已经被broker接收并写入到文件但是并没有应答，这时生产者重发一条消息。

Spark可以使用事件中的唯一标识符对数据流中的记录进行重复数据删除。与使用唯一标识符列的静态删除重复数据完全相同。
- **使用watermark：如果重复记录可能到达的**时间有上限**，则可以在事件时间列上定义watermark，并使用唯一标识符和事件时间列进行重复数据删除。
- **不使用watermark**：由于重复记录可能到达时间没有界限，所以查询过去所有的记录的数据

### 处理多个水印策略
流式查询中可以有多个流联接在一起，每个流中都有不同的延迟数据阈值，对与有状态的操作，这些阈值是需要接纳的
```java
inputStream1.withWatermark("eventTime1", "1 hour")
  .join(
    inputStream2.withWatermark("eventTime2", "2 hours"),
    joinCondition)
```
Structured Streaming 单独跟踪每个输入流中最大的事件时间，根据相应的延迟计算watermark ，**Spark还会选择一个全局watermark，默认情况下，会选择最小的作为全局watermark**， 因为它可以确保，如果其中一个流落后于其他流(例如，其中一个流由于上游故障而停止接收数据)，不会因为太晚而意外地删除数据。换句话说，全局水印将以最慢的流的速度安全地移动，查询输出将相应地延迟。也可以通过SQL配置`spark.sql.streaming.multipleWatermarkPolicy=max(默认值是min)`来设置最大值来作为全局水印，但这种会可能导致来的较慢的数据被删除

### 流式DataSet/DataFrame不支持的操作
流式 DataFrames/Datasets 不支持一些 DataFrame/Dataset 操作

-   流数据集尚不支持多个流聚合（即流 DF 上的聚合链）group by 之类操作。
    
-   流式数据集不支持limit和take操作
    
-   不支持对流数据集进行Distinct操作。
    
-   在流数据集上聚合后不支持重复数据删除操作。
    
-   只有在聚合后和完整输出模式下，流数据集才支持排序操作。
    
-   不支持流数据集上的几种外连接类型。参考[[#支持流查询的连接矩阵]]

### Spark 异步API
通过附加 `StreamingQueryListener` 异步监听`SaprkSession`  相关的查询
```java
SparkSession spark = ...

spark.streams().addListener(new StreamingQueryListener() {
    @Override
    public void onQueryStarted(QueryStartedEvent queryStarted) {
        System.out.println("Query started: " + queryStarted.id());
    }
    @Override
    public void onQueryTerminated(QueryTerminatedEvent queryTerminated) {
        System.out.println("Query terminated: " + queryTerminated.id());
    }
    @Override
    public void onQueryProgress(QueryProgressEvent queryProgress) {
        System.out.println("Query made progress: " + queryProgress.progress());
    }
});
```

