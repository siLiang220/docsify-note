>定义实时数仓的前提：定义什么叫实时。实时的定义，应当由业务方的最大数据可见性容忍度为基准，确定时效性后再确定使用什么技术实现


![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20231124191042.png)
基于Flink CDC 异步Lookup join Doris ：将单条的查询追加watcher到队列中，由后台线程监听实时监听队列，将队列中的数据实时 push 到工作线程池中，工作线程池中将收到的数据拼接成union all SQL，向Doris 发起查询请求
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20231226192150.png)

Flink  Doris Connector 是如何保证数据的一致性 


Doris 2.0 Light Schema Change 
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20231226204047.png)
 


