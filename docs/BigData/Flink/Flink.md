## Watermark

### 基本概念
- **Window**：Window 是处理无界流的关键，Window将流拆分为一个个有限大小的buckets
- **start_time,end_time**：window时间窗口的开始结束时间（前闭后开）
- **Watermark**：等于`eventTime - delay` , 一旦`Watermeak`大于某个window 的end_time时，就会触发window的计算。只要到达该时间，表示这个时间戳前的所有数据都应该已经到达了，窗口就可以计算了

#### 推迟窗口的计算时间实现方式
通过当前窗口的**最大**的`eventTime - 延迟时间`所得到的Watermark与原始窗口的触发事件对比，当Watermark 大于窗口触发的原始时间则触发窗口执行

#### 迟到和乱序数据处理

>乱序数据的后果
>对于时间窗口来说，如果按照处理时间来闭合窗口，就会导致迟到的数据丢失。  
比如：系统时间为9:05的时候关闭窗口，那么9:04分产生的数据迟到了，就会丢失

#### waterMark要解决的问题
- **问题1：** 按照处理时间关闭窗口会丢失数据
- ~~**问题2：** 按照事件时间关闭窗口会导致窗口闭合时间不确定。  比如：9:05分产生的数据迟迟未到，那么[9:00,9:05)窗口永远不会闭合（不是很理解，时间窗口是左闭右开，为什么还会无法触发窗口关闭 待测试）~~

>必须要引入一个机制，保证一个特定的时间后，必须触发窗口闭合，进行运算输出。这个特别的机制，就是Watermark。

在流处理时间中，流经过source 再到operator ，大部分情况下在operator的数据都是按照事件时间顺序的，但中间的可能由于网络、分布式的原因导致乱序数据，operator 处理时数据并不是按照`eventTime`顺序排序的.在这种情况下如果用eventTime决定window 的运行，并不能保证数据全都到齐，但也不能通过延迟让window无期限的等下去

Watermark是一种衡量Event Time进展的机制。 Watermark是用于处理乱序事件的，而正确的处理乱序事件，通常用Watermark机制结合window来实现。

在有序的数据中，watermark 在流中只是一个周期性的标记
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662907309586.png)


在乱序中watermark是至关重要的，表示到流中的该点，在某个时间戳之前的所有事件都应该已经到达了
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1662907533967.png)

当Flink接收到数据时，会按照一定的规则去生成Watermark，这条Watermark就等于当前所有到达数据中的maxEventTime - 延迟时长，也就是说，Watermark是由数据携带的，一旦数据携带的Watermark比当前未触发的窗口的停止时间要晚，那么就会触发相应窗口的执行。_由于Watermark是由数据携带的，因此，如果运行过程中无法获取新的数据，那么没有被触发的窗口将永远都不被触发。_

#### watermark 作用

-  水位线（WaterMark）是一个时间戳，等于当前到达的消息最大时间戳减去配置的延迟时间，水位线是单调递增的，如果有晚到达的早消息也不会更新水位线，因为消息最大时间戳没变 `水位线 = 消息最大时间戳 - 配置的INTERVAl(offset)时间`

-  新消息到达时，才计算新的水位线，如果水位线大于等于窗口的endTime（左闭右开）则触发窗口计算，反之继续接收后续消息；消息的EventTime大于等于窗口beginTime则保留，反之被丢弃水位线 >= 窗口的endTime，则触发窗口计算

- 消息的EventTime小于水位线时不一定被丢弃；消息的EventTime小于窗口beginTime时才会被丢弃

-  与window一起使用，_可以对乱序到达的消息 排序后再处理_

-  **引入水位线机制的目的是延迟窗口触发计算的时间**，使晚到达的早的消息尽可能也能被保留，用于窗口计算，提高数据准确性

- 只要没有达到水位那么不管现实中的时间推进了多久都不会触发关窗。

#### 参考文档
[Timely Stream Processing | Apache Flink](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/concepts/time/)

[flink学习笔记-如何利用waterMark解决乱序、延迟问题_陈同学：的博客-CSDN博客](https://blog.csdn.net/qq_26719997/article/details/105063444)

[ Flink 水位线机制WaterMark实践 处理乱序消息_二十六画生的博客的博客-CSDN博客_flink水位机制](https://programskills.blog.csdn.net/article/details/106876623)

[ Flink窗口机制详解_最佳第六六六人的博客-CSDN博客_flink窗口触发机制](https://blog.csdn.net/qq_43523503/article/details/114958569)