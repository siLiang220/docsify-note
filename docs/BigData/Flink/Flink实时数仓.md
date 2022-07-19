## 普通实时计算与实时数仓比较 
**普通的实时计算**优先考虑时效性，所以从数据源采集经过实时计算直接得到结果。如此 做时效性更好，但是弊端是由于计算过程中的中间结果没有沉淀下来，所以当面对大量实时 需求的时候，计算的复用性较差，开发成本随着需求增加直线上升。 实时数仓基于一定的数据仓库理念，对数据处理流程进行规划、分层，目的是提高数据 的复用性。 

**实时数仓**基于一定的数据仓库理念，对数据处理流程进行规划、分层，目的是提高数据 的复用性。
![分层](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1655212523364.png)
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1655212523364.png)
## 实时数仓分层
**ODS**：原始数据，日志和业务数据
**DWD**：根据数据对象为单位进行分流，比如订单、页面访问等等 
**DIM**：维度数据 
**DWM**：对于部分数据对象进行进一步加工，比如独立访问、跳出行为，也可以和维度 进行关联，形成宽表，依旧是明细数据。 
**DWS**：根据某个主题将多个事实数据轻度聚合，形成主题宽表。 
**ADS**：把ClickHouse中的数据根据可视化需进行筛选聚合
## 离线计算与实时计算的比较
**离线计算**：就是在计算开始前已知所有输入数据，输入数据不会产生变化，一般计算量 级较大，计算时间也较长。例如今天早上一点，把昨天累积的日志，计算出所需结果。最经 典的就是 Hadoop 的 MapReduce 方式； 一般是根据前一日的数据生成报表，虽然统计指标、报表繁多，但是对时效性不敏感。 从技术操作的角度，这部分属于批处理的操作。即根据确定范围的数据一次性计算。
**实时计算**：输入数据是可以以序列化的方式一个个输入并进行处理的，也就是说在开始 的时候并不需要知道所有的输入数据。与离线计算相比，运行时间短，计算量级相对较小。 强调计算过程的时间要短，即所查当下给出结果。 主要侧重于对当日数据的实时监控，通常业务逻辑相对离线需求简单一下，统计指标也 少一些，但是更注重数据的时效性，以及用户的交互性。从技术操作的角度，这部分属于流 处理的操作。根据数据源源不断地到达进行实时的运算
## 实时需求的种类
- 日常统计报表或分析图中需要包含当日部分
- 实时数据大屏监控
- 数据预警或提示
- 实时推荐系统
## 实现Flink 动态分流
#### 动态分流
由于 FlinkCDC 是把全部数据统一写入一个 Topic 中, 这样显然不利于日后的数据处理。 所以需要把各个表拆开处理。但是由于每个表有不同的特点，有些表是维度表，有些表是事 实表。 在实时计算中一般把维度数据写入存储容器，一般是方便通过主键查询的数据库比如 HBase,Redis,MySQL 等。一般把事实数据写入流中，进行进一步处理，最终形成宽表。 
这样的配置不适合写在配置文件中，因为这样的话，业务端随着需求变化每增加一张表， 就要修改配置重启计算程序。所以这里需要一种动态配置方案，把这种配置长期保存起来， 一旦配置有变化，实时计算可以自动感知
#### 实现方案
1. 一种是用 mysql 数据库存储，周期性的同步；
2. 另一种是用 mysql 数据库存储，使用广播流。
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1655215159909.png)

- 创建配置表 与实体类
```sql
CREATE TABLE ` table_process ` (
    ` source_table ` VARCHAR(200) NOT NULL COMMENT '来源表',
    ` operate_type ` VARCHAR(200) NOT NULL COMMENT '操作类型 insert,update,delete',
    ` sink_type ` VARCHAR(200) DEFAULT NULL COMMENT '输出类型 hbase kafka',
    ` sink_table ` VARCHAR(200) DEFAULT NULL COMMENT '输出表(主题)',
    ` sink_columns ` VARCHAR(2000) DEFAULT NULL COMMENT '输出字段',
    ` sink_pk ` VARCHAR(200) DEFAULT NULL COMMENT '主键字段',
    ` sink_extend ` VARCHAR(200) DEFAULT NULL COMMENT '建表扩展',
    PRIMARY KEY (` source_table `, ` operate_type `)
) ENGINE = InnoDB DEFAULT CHARSET = utf8
```

```java
package org.example.bean;  
  
/**  
 * author:zhaosiliang 
 * date:2022/6/18 14:34  
 * 描述：创建配置表实体类  
 **/  
public class TableProcess {  
  
    //动态分流SINK 常量  
    public static final String SINK_TYPE_HBASE ="hbase";  
    public static final String SINK_TYPE_KAFKA="kafka";  
    public static final String SINK_TYPE_CK="clickhouse";  
    //来源表  
    String sourceTable;  
    //操作类型  insert,update,delete    String operateType;  
    //输出类型，hbase kafka  
    String sinkType;  
    //输出表  
    String sinkTable;  
    //输出字段  
    String sinkColumns;  
    //主键字段  
    String sinkPk;  
    //建表扩展字段  
    String sinkExtend;  
  
    public String getSourceTable() {  
        return sourceTable;  
    }  
  
    public void setSourceTable(String sourceTable) {  
        this.sourceTable = sourceTable;  
    }  
  
    public String getOperateType() {  
        return operateType;  
    }  
  
    public void setOperateType(String operateType) {  
        this.operateType = operateType;  
    }  
  
    public String getSinkType() {  
        return sinkType;  
    }  
  
    public void setSinkType(String sinkType) {  
        this.sinkType = sinkType;  
    }  
  
    public String getSinkTable() {  
        return sinkTable;  
    }  
  
    public void setSinkTable(String sinkTable) {  
        this.sinkTable = sinkTable;  
    }  
  
    public String getSinkColumns() {  
        return sinkColumns;  
    }  
  
    public void setSinkColumns(String sinkColumns) {  
        this.sinkColumns = sinkColumns;  
    }  
  
    public String getSinkPk() {  
        return sinkPk;  
    }  
  
    public void setSinkPk(String sinkPk) {  
        this.sinkPk = sinkPk;  
    }  
  
    public String getSinkExtend() {  
        return sinkExtend;  
    }  
  
    public void setSinkExtend(String sinkExtend) {  
        this.sinkExtend = sinkExtend;  
    }  
}
```
kafkaUtil
```java
package org.example.utils;  
  
import com.ververica.cdc.connectors.shaded.org.apache.kafka.clients.producer.ProducerConfig;  
import org.apache.flink.api.common.eventtime.WatermarkStrategy;  
import org.apache.flink.api.common.serialization.SimpleStringSchema;  
import org.apache.flink.connector.base.DeliveryGuarantee;  
import org.apache.flink.connector.kafka.sink.KafkaRecordSerializationSchema;  
import org.apache.flink.connector.kafka.sink.KafkaSink;  
import org.apache.flink.connector.kafka.source.KafkaSource;  
import org.apache.flink.connector.kafka.source.enumerator.initializer.OffsetsInitializer;  
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;  
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer;  
import org.apache.flink.streaming.connectors.kafka.KafkaSerializationSchema;  
  
/**  
 * author:zhaosiliang * date:2022/6/18 10:10 * 描述：  
 **/  
public class KafkaUtil {  
  
    //1.15 版本中移除FlinkKafkaConsumer  
    public static KafkaSource <String> getKafkaSource(String topic, String groupId){  
        KafkaSource<String> source = KafkaSource.<String>builder()  
                .setBootstrapServers("brokers")  
                .setTopics(topic)  
                .setGroupId(groupId)  
                .setStartingOffsets(OffsetsInitializer.earliest())  
                .setValueOnlyDeserializer(new SimpleStringSchema())  
                .build();  
        return source;  
    }  
  
    public static <T> KafkaSink<String> getKafkaSinkBySchema() {  
  
        KafkaSink<String> sink = KafkaSink.<String>builder()  
                .setBootstrapServers("brokers")  
                .setRecordSerializer(KafkaRecordSerializationSchema.builder()  
                        .setTopic("topic-name")  
                        .setValueSerializationSchema(new SimpleStringSchema())  
                        .build()  
                )  
                .setDeliverGuarantee(DeliveryGuarantee.AT_LEAST_ONCE)  
                .build();  
        return sink;  
    }  
  
}
```
- 读取配置信息将配置信息广播

```java
package org.example.app;
import com.alibaba.fastjson.JSONObject;
import com.ververica.cdc.connectors.mysql.source.MySqlSource;
import com.ververica.cdc.connectors.shaded.org.apache.kafka.connect.data.Field;
import com.ververica.cdc.connectors.shaded.org.apache.kafka.connect.data.Schema;
import com.ververica.cdc.connectors.shaded.org.apache.kafka.connect.data.Struct;
import com.ververica.cdc.connectors.shaded.org.apache.kafka.connect.source.SourceRecord;
import com.ververica.cdc.debezium.DebeziumDeserializationSchema;
import io.debezium.data.Envelope;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.FilterFunction;
import org.apache.flink.api.common.state.MapStateDescriptor;
import org.apache.flink.api.common.typeinfo.TypeInformation;
import org.apache.flink.connector.kafka.sink.KafkaSink;
import org.apache.flink.connector.kafka.source.KafkaSource;
import org.apache.flink.runtime.state.hashmap.HashMapStateBackend;
import org.apache.flink.streaming.api.datastream.*;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;
import org.apache.flink.util.OutputTag;
import org.example.bean.TableProcess;
import org.example.fun.DimSink;
import org.example.fun.TableProcessFunction;
import org.example.utils.KafkaUtil;
/**
 * author:zhaosiliang
 * date:2022/6/18 14:49
 * 描述：把分好的流保存的课对应的表、主题中
 **/
public class BaseDBApp {

    public static void main(String[] args) {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.setStateBackend(new HashMapStateBackend());
        env.getCheckpointConfig().setCheckpointStorage("file:///d:/checkpoint/");
        env.getCheckpointConfig().setCheckpointTimeout(3000);
        
        //读取kafka
        String topic ="";
        String groupId ="";
        KafkaSource<String> kafkaSource = KafkaUtil.getKafkaSource(topic, groupId);
        DataStreamSource<String> kafkaDs = env.fromSource(kafkaSource, WatermarkStrategy.noWatermarks(), "my kafka");
        SingleOutputStreamOperator<JSONObject> jsonObjDS = kafkaDs.map(JSONObject::parseObject);

        //过滤
        SingleOutputStreamOperator<JSONObject> filterDS = jsonObjDS.filter(new FilterFunction<JSONObject>() {
            @Override
            public boolean filter(JSONObject value) throws Exception {
                String data = value.getString("data");
                return data != null && data.length() > 0;
            }
        });

        //读取数据库配置表形成广播流
        MySqlSource<String> mysqlSourceFunction = MySqlSource.<String>builder()
                .hostname("127.0.0.1")
                .port(3306)
                .username("root")
                .password("root")
                .tableList("test")
                .tableList("test.table_process")
                .deserializer(new DebeziumDeserializationSchema<String>() {
                    @Override
                    public void deserialize(SourceRecord sourceRecord, Collector<String> collector) throws Exception {
                        String topic = sourceRecord.topic();
                        String[] split = topic.split("\\.");
                        String db = split[1];
                        String table = split[2];
                        //获取数据 
                        Struct value = (Struct) sourceRecord.value();
                        Struct after = value.getStruct("after");
                        JSONObject data = new JSONObject();
                        if (after != null) {
                            Schema schema = after.schema();
                            for (Field field : schema.fields()) {
                                data.put(field.name(), after.get(field.name()));
                            }
                        }
                        //获取操作类型
                        Envelope.Operation operation = Envelope.operationFor(sourceRecord);
                        //存放数据
                        JSONObject result = new JSONObject();
                        result.put("database", db);
                        result.put("table", table);
                        result.put("type", operation.toString().toLowerCase());
                        result.put("data", data);
                    }

                    //定义数据类型
                    @Override
                    public TypeInformation<String> getProducedType() {
                        return TypeInformation.of(String.class);
                    }
                }).build();

        //读取mysql 数据
        DataStreamSource<String> tableProcessDS = env.fromSource(mysqlSourceFunction,WatermarkStrategy.noWatermarks(),"my mysql");

        //将配置信息作为广播流
        MapStateDescriptor<String, TableProcess> mapStateDescriptor = new MapStateDescriptor<>("table-process-state",String.class,TableProcess.class);
        BroadcastStream<String> broadcastStream = tableProcessDS.broadcast(mapStateDescriptor);

        //将主流数据和广播流进行连接
        BroadcastConnectedStream<JSONObject, String> connectStream = filterDS.connect(broadcastStream);

        //分流 处理数据广播流数据，主流数据
        OutputTag<JSONObject> hbaseTag = new OutputTag<>("hbase-tag");
        //提取kafka流数据和HBase 数据
        SingleOutputStreamOperator<JSONObject> kafka = connectStream.process(new TableProcessFunction(hbaseTag, mapStateDescriptor));

        DataStream<JSONObject> hbaseJsonDS = kafka.getSideOutput(hbaseTag);
        hbaseJsonDS.addSink(new DimSink());


//        DataStreamSink<JSONObject> jsonObjectDataStreamSink = kafka.addSink(KafkaUtil.getKafkaSinkBySchema());
    }
    
}

```

- 分流 处理数据，广播流数据，主流数据（根据广播流数据处理）
processBroadCastElement与processElement 执行时没有先后执行顺序
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/Pasted%20image%2020220618153758.png)

```java
package org.example.fun;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import org.apache.flink.api.common.state.BroadcastState;
import org.apache.flink.api.common.state.MapStateDescriptor;
import org.apache.flink.api.common.state.ReadOnlyBroadcastState;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.functions.co.BroadcastProcessFunction;
import org.apache.flink.util.Collector;
import org.apache.flink.util.OutputTag;
import org.example.bean.TableProcess;
import org.example.common.GmallConfig;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.Set;
/**
 * author:zhaosiliang
 * date:2022/6/18 16:21
 * 描述：自定义函数
 **/
public class TableProcessFunction extends BroadcastProcessFunction<JSONObject,String, JSONObject> {
    private OutputTag<JSONObject> hbaseTag;
    private MapStateDescriptor<String,TableProcess> mapStateDescriptor;
    Connection connection = null;

    public TableProcessFunction(OutputTag<JSONObject> hbaseTag, MapStateDescriptor<String, TableProcess> mapStateDescriptor) {
        this.hbaseTag = hbaseTag;
        this.mapStateDescriptor = mapStateDescriptor;
    }

    @Override
    public void open(Configuration parameters) throws Exception {
         //初始化phoenix
        Class.forName(GmallConfig.PHOENIX_DRIVER);
        connection = DriverManager.getConnection(GmallConfig.PHOEXIN_SERVER);
    }
    // processElement 处理主流数据
    //1.读取状态
    //2.过滤数据
    //3.分流
    @Override
    public void processElement(JSONObject value, BroadcastProcessFunction<JSONObject, String, JSONObject>.ReadOnlyContext ctx, Collector<JSONObject> out) throws Exception {
        ReadOnlyBroadcastState<String, TableProcess> broadcastState = ctx.getBroadcastState(mapStateDescriptor);
        //获取表名+操作类型
        String table = value.getString("table");
        String type = value.getString("type");
        //获取对应的配置信息
        TableProcess tableProcess = broadcastState.get(table + ":" + type);

        if (tableProcess != null){
            // 设置要写入的目标表或者topic
            value.put("sink_table",tableProcess.getSinkTable());
            //根据配置信息中提供的字段做数据过滤
            filterColumn(value.getJSONObject("data"), tableProcess.getSinkColumns());
            //判断当前数据应该写往 HBASE 还是 Kafka
            if (TableProcess.SINK_TYPE_KAFKA.equals(tableProcess.getSinkType())) {
                //Kafka 数据,将数据输出到主流
                out.collect(value);
            } else if (TableProcess.SINK_TYPE_HBASE.equals(tableProcess.getSinkType())) {
                //HBase 数据,将数据输出到侧输出流
                ctx.output(hbaseTag, value);
            }

        }
    }

    //根据配置信息中提供的字段做数据过滤
    private void filterColumn(JSONObject data, String sinkColumns) {
        //保留的数据字段
        String[] fields = sinkColumns.split(",");
        List<String> fieldList = Arrays.asList(fields);
        Set<Map.Entry<String, Object>> entries = data.entrySet();
        // while (iterator.hasNext()) {
        // Map.Entry<String, Object> next = iterator.next();
        // if (!fieldList.contains(next.getKey())) {
        // iterator.remove();
        // }
        // }
        entries.removeIf(next -> !fieldList.contains(next.getKey()));
    }

    //processBroadcastElement 处理广播流数据
    //1.间隙string -> tableProcess
    //2. 检查hbase 表是否存在并建表
    //3.写入状态
    @Override
    public void processBroadcastElement(String value, BroadcastProcessFunction<JSONObject, String, JSONObject>.Context ctx, Collector<JSONObject> out) throws Exception {
        //获取状态
        BroadcastState<String, TableProcess> broadcastState =
                ctx.getBroadcastState(mapStateDescriptor);
        //{"database":"","table":"","type","","data":{"":""}}
        JSONObject jsonObject = JSON.parseObject(value);
        //取出数据中的表名以及操作类型封装 key
        JSONObject data = jsonObject.getJSONObject("data");
        String table = data.getString("source_table");
        String type = data.getString("operate_type");
        String key = table + ":" + type;
        //取出 Value 数据封装为 TableProcess 对象
        TableProcess tableProcess = JSON.parseObject(data.toString(), TableProcess.class);
        //HBASE 建表
        if (TableProcess.SINK_TYPE_HBASE.equals(tableProcess.getSinkType())) {
            checkTable(tableProcess.getSinkTable(), tableProcess.getSinkColumns(), tableProcess.getSinkPk(),
                    tableProcess.getSinkExtend());
        }
        System.out.println("Key:" + key + "," + tableProcess);
        //广播出去 无需关系如何广播出去的
        broadcastState.put(key, tableProcess);

    }

    private void checkTable(String sinkTable, String sinkColumns, String sinkPk, String sinkExtend) {
        //给主键以及扩展字段赋默认值
        if (sinkPk == null) {
            sinkPk = "id";
        }
        if (sinkExtend == null) {
            sinkExtend = "";
        }
        //封装建表 SQL
        StringBuilder createSql = new StringBuilder("create table if not exists ").append(GmallConfig.HBASE_SCHEMA).append(".").append(sinkTable).append("(");
        //遍历添加字段信息
        String[] fields = sinkColumns.split(",");
        for (int i = 0; i < fields.length; i++) {
            //取出字段
            String field = fields[i];
            //判断当前字段是否为主键
            if (sinkPk.equals(field)) {
                createSql.append(field).append(" varchar primary key ");
            } else {
                createSql.append(field).append(" varchar ");
            }
            //如果当前字段不是最后一个字段,则追加","
            if (i < fields.length - 1) {
                createSql.append(",");
            }
        }
        createSql.append(")");
        createSql.append(sinkExtend);
        System.out.println(createSql);
        //执行建表 SQL
        PreparedStatement preparedStatement = null;
        try {
            preparedStatement = connection.prepareStatement(createSql.toString());
            preparedStatement.execute();
        } catch (SQLException e) {
            e.printStackTrace();
            throw new RuntimeException("创建 Phoenix 表" + sinkTable + "失败！");
        } finally {
            if (preparedStatement != null) {
                try {
                    preparedStatement.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }

}
```
- 分流sink 维度保存Hbase
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1655560139481.png)

DimSink 继承了 `RickSinkFunction`，这个 function 得分两条时间线。 
一条是任务启动时执行 open 操作（图中紫线），我们可以把连接的初始化工作放 在此处一次性执行。 
一条是随着每条数据的到达反复执行 invoke()（图中黑线）,在这里面我们要实 现数据的保存，主要策略就是根据数据组合成 sql 提交给 hbase。
```java
package org.example.fun;
import com.alibaba.fastjson.JSONObject;
import org.apache.commons.lang3.StringUtils;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.functions.sink.RichSinkFunction;
import org.example.common.GmallConfig;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.Collection;
import java.util.Set;
/**
 * author:zhaosiliang
 * date:2022/6/18 21:55
 * 描述：
 **/
public class DimSink  extends RichSinkFunction<JSONObject> {
    private Connection connection = null;
    @Override
    public void open(Configuration parameters) throws Exception {
        //初始化 Phoenix 连接
        Class.forName(GmallConfig.PHOENIX_DRIVER);
        connection = DriverManager.getConnection(GmallConfig.PHOENIX_DRIVER);
    }

    //将数据写入 Phoenix：upsert into t(id,name,sex) values(...,...,...)
    @Override
    public void invoke(JSONObject jsonObject, Context context) throws Exception {
        PreparedStatement preparedStatement = null;
        try {
            //获取数据中的 Key 以及 Value
            JSONObject data = jsonObject.getJSONObject("data");
            Set<String> keys = data.keySet();
            Collection<Object> values = data.values();
            //获取表名
            String tableName = jsonObject.getString("sink_table");
            //创建插入数据的 SQL
            String upsertSql = genUpsertSql(tableName, keys, values);
            System.out.println(upsertSql);
            //编译 SQL
            preparedStatement = connection.prepareStatement(upsertSql);
            //执行
            preparedStatement.executeUpdate();
            //提交
            connection.commit();
        } catch (SQLException e) {
            e.printStackTrace();
            System.out.println("插入 Phoenix 数据失败！");
        } finally {
            if (preparedStatement != null) {
                preparedStatement.close();
            }

        }
    }
    //创建插入数据的 SQL upsert into t(id,name,sex) values('...','...','...')
    private String genUpsertSql(String tableName, Set<String> keys, Collection<Object> values) {
        return "upsert into " + GmallConfig.HBASE_SCHEMA + "." +
                tableName + "(" + StringUtils.join(keys, ",") + ")" +
                " values('" + StringUtils.join(values, "','") + "')";
    }

}


```

## DWD 与DWS层
#### 访客UV计算
识别到当日的访客
1. 识别出该访客打开的第一个页面，表示这个访客开始进入
2. 对一天范围内的访客去重

```java
package org.example.app;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONAware;
import com.alibaba.fastjson.JSONObject;
import org.apache.flink.api.common.JobExecutionResult;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.FilterFunction;
import org.apache.flink.api.common.functions.RichFilterFunction;
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.connector.kafka.source.KafkaSource;
import org.apache.flink.runtime.state.hashmap.HashMapStateBackend;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.example.utils.KafkaUtil;
import java.text.SimpleDateFormat;

/**
 * author:zhaosiliang
 * date:2022/6/19 17:12
 * 描述：访客uv计算
 **/
public class UniqueVisitApp {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.setStateBackend(new HashMapStateBackend());
        env.getCheckpointConfig().setCheckpointStorage("file:///d:/checkpoint/");
        env.getCheckpointConfig().setCheckpointTimeout(3000);

        //读取kafka
        String groupId = "unique_visit_app";
        String sourceTopic = "dwd_page_log";
        String sinkTopic = "dwm_unique_visit";
        KafkaSource<String> kafkaSource = KafkaUtil.getKafkaSource(sourceTopic, groupId);
        DataStreamSource<String> kafkaDs = env.fromSource(kafkaSource, WatermarkStrategy.noWatermarks(), "my kafka");

        //将数据转为json
        SingleOutputStreamOperator<JSONObject> jsonObjDs = kafkaDs.map(JSON::parseObject);

        //过滤数据，状态编程 值保留每个mid第一次登录数据
        KeyedStream<JSONObject, String> keyByStream = jsonObjDs.keyBy(jsonObject -> jsonObject.getJSONObject("common").getString("mid"));

        SingleOutputStreamOperator<JSONObject> uvDS = keyByStream.filter(new RichFilterFunction<JSONObject>() {

            private ValueState<String> dateState;
            private SimpleDateFormat simpleDateFormat;

            @Override
            public void open(Configuration parameters) throws Exception {
                ValueStateDescriptor<String> valueStateDescriptor = new ValueStateDescriptor<>("date-state", String.class);
                //设置TTL 及更新方式  
				StateTtlConfig stateTtlConfig = StateTtlConfig.newBuilder(Time.days(1)).setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite).build();  
valueStateDescriptor.enableTimeToLive(stateTtlConfig);
                dateState = getRuntimeContext().getState(valueStateDescriptor);
                simpleDateFormat= new SimpleDateFormat("yyyy-MM-dd");
            }

            @Override
            public boolean filter(JSONObject value) throws Exception {

                //1.判断 上一跳页面，判断是否为null,是 取出状态，判断状态是否为null，如果 null 返回true，变更状态
                String lastPageId = value.getJSONObject("page").getString("last_page_id");
                if (lastPageId == null || lastPageId.length() <= 0) {
                    //取当前状态
                    String lastDate = dateState.value();
                    String currentDate = simpleDateFormat.format(value.getLong("ts"));
                    //判断两个日期是否相同
                    if (!currentDate.equals(lastDate)) {
                        dateState.update(currentDate);
                        return true;
                    }
                }
                return false;
            }

        });

        uvDS.print();
        uvDS.map(JSONAware::toJSONString);
        env.execute();
    }
}
```

#### 跳出明细计算
用户成功访问一个页面后就退出，不在继续访问网站的其他页面。而跳出率就是跳出次数除以访问次数
- 该页面是用户访问的第一个页面
- 首次访问后一段事件，用户没有继续访问其他的页面

## DWM 层
#### 订单宽表
 双流join 时，下流的watermark 取的是上流最小的watermark
- 实数据和事实数据关联，其实就是流与流之间的关联。 
- 事实数据与维度数据关联，其实就是流计算中查询外部数据源。
##### 旁路缓存模式优化
旁路缓存模式：任何请求先访问缓存，缓存命中直接获的数据返回结果。如果未命中，查询数据库，同时将数据写入缓存中
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1656835221442.png)

JDBCUtil
```xml
<dependency>  
    <groupId>commons-beanutils</groupId>  
    <artifactId>commons-beanutils</artifactId>  
    <version>1.9.3</version>  
</dependency>  
<!--Guava 工程包含了若干被 Google 的 Java 项目广泛依赖的核心库,方便开发-->  
<dependency>  
    <groupId>com.google.guava</groupId>  
    <artifactId>guava</artifactId>  
    <version>29.0-jre</version>  
</dependency>
```

```java
package org.example.utils;  
  
import com.google.common.base.CaseFormat;  
import org.apache.commons.beanutils.BeanUtils;  
  
import java.lang.reflect.InvocationTargetException;  
import java.sql.*;  
import java.util.ArrayList;  
import java.util.List;  
  
/**  
 * author:zhaosiliang 
 * date:2022/7/3 13:03 
 * 描述：  
 **/  
public class JdbcUtil {  
  
    public static <T> List<T> queryList(Connection connection,String sql,Class<T> clz,Boolean underScoreToCamel) throws SQLException, InstantiationException, IllegalAccessException, InvocationTargetException {  
  
        List<T> resultList = new ArrayList<>();  
        //预编译sql  
        PreparedStatement preparedStatement = connection.prepareStatement(sql);  
        ResultSet resultSet = preparedStatement.executeQuery();  
        ResultSetMetaData metaData = resultSet.getMetaData();  
        int columnCount = metaData.getColumnCount();  
        while (resultSet.next()){  
            T t = clz.newInstance();  
            for (int i= 1 ; i< columnCount+1; i++){  
                String columnName = metaData.getColumnName(i);  
                Object value = resultSet.getObject(columnName);  
                //判断是否驼峰命名  
                if (underScoreToCamel) {  
                    columnName = CaseFormat.LOWER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL,columnName);  
                }  
                //泛型对象赋值  
                BeanUtils.setProperty(t,columnName,value);  
            }  
            resultList.add(t);  
        }  
        return resultList;  
    }  
}
```
DIMUtil
```java
package org.example.utils;

import com.alibaba.fastjson.JSONObject;
import org.example.common.GmallConfig;
import redis.clients.jedis.Jedis;

import java.lang.reflect.InvocationTargetException;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.List;

/**
 * author:zhaosiliang
 * date:2022/7/3 16:09
 * 描述：
 **/
public class DimUtil {
    public static JSONObject getDimInfo(Connection connection,String tableName,String id) throws SQLException, InvocationTargetException, InstantiationException, IllegalAccessException {
        //查询数据前先查询redis
        Jedis jedis = RedisUtil.getJedis();
        String redisKey = "DIM:"+tableName +":"+id;
        String dimInfoJsonStr = jedis.get(redisKey);
        if (dimInfoJsonStr!=null){
            
            //重置过期时间
            jedis.expire(redisKey,24*60*60);
            jedis.close();
            return JSONObject.parseObject(dimInfoJsonStr);
        }

        String querySql = "SELECT * FROM "+ GmallConfig.HBASE_SCHEMA+"."+tableName+"WHERE id = '"+ id+"'";
        List<JSONObject> queryList = JdbcUtil.queryList(connection, querySql, JSONObject.class, false);
        JSONObject jsonObject = queryList.get(0);

        //保存redis
        jedis.set(redisKey,jsonObject.toJSONString());
        jedis.expire(redisKey,24*60*60);
        return jsonObject;
    }
    public static void deleteCached(String tableName,String id) {
        try {
            String redisKey = "DIM:"+tableName +":"+id;
            Jedis jedis = RedisUtil.getJedis();
            // 通过 key 清除缓存
            jedis.del(redisKey);
            jedis.close();
        } catch (Exception e) {
            System.out.println("缓存异常！");
            e.printStackTrace();
        }
    }
}

```

redisUtil
```java
package org.example.utils;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

/**
 * author:zhaosiliang
 * date:2022/7/3 17:10
 * 描述：
 **/
public class RedisUtil {
    public static JedisPool jedisPool = null;
    public static Jedis getJedis() {
        if (jedisPool == null) {
            JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
            jedisPoolConfig.setMaxTotal(100); //最大可用连接数
            jedisPoolConfig.setBlockWhenExhausted(true); //连接耗尽是否等待
            jedisPoolConfig.setMaxWaitMillis(2000); //等待时间
            jedisPoolConfig.setMaxIdle(5); //最大闲置连接数
            jedisPoolConfig.setMinIdle(5); //最小闲置连接数
            jedisPoolConfig.setTestOnBorrow(true); //取连接的时候进行一下测试 ping pong
                    jedisPool = new JedisPool(jedisPoolConfig, "127.0.0.1", 6379, 1000);
            System.out.println("开辟连接池");
            return jedisPool.getResource();
        } else {
        // System.out.println(" 连接池:" + jedisPool.getNumActive());
            return jedisPool.getResource();
        }
    }
}

```
##### 异步查询查询优化
在 Flink 流处理过程中，经常需要和外部系统进行交互，用维度表补全事实表中的字段。
例如：在电商场景中，需要一个商品的 skuid 去关联商品的一些属性，例如商品所属行业、商品的生产厂家、生产厂家的一些情况；在物流场景中，知道包裹 id，需要去关联包裹的行业属性、发货信息、收货信息等等。
默认情况下，在 Flink 的 MapFunction 中，单个并行只能用同步方式去交互: 将请求发送到外部存储，IO 阻塞，等待请求返回，然后继续发送下一个请求。
Flink 在 1.2 中引入了 Async I/O，在异步模式下，将 IO 操作异步化，单个并行可以连续发送多个请求，哪个请求先返回就先处理，从而在连续的请求间不需要阻塞式等待，大大提高了流处理效率。
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1657373595417.png)
ThreadPoolUtil
```java
package org.example.utils;  
  
import java.util.concurrent.Executors;  
import java.util.concurrent.LinkedBlockingQueue;  
import java.util.concurrent.ThreadPoolExecutor;  
import java.util.concurrent.TimeUnit;  
  
public class ThreadPoolUtil {  
  
    static ThreadPoolExecutor threadPoolExecutor;  
  
    private ThreadPoolUtil() {  
    }  
  
    public static ThreadPoolExecutor getThreadPool(){  
        if (threadPoolExecutor == null){  
            synchronized (ThreadPoolUtil.class) {  
                if (threadPoolExecutor == null){  
                    threadPoolExecutor = new ThreadPoolExecutor(2,20,1, TimeUnit.MINUTES,new LinkedBlockingQueue<>());  
                }  
            }  
        }  
        return threadPoolExecutor;  
    }  
}
```

RichAsyncFunction 异步IO
```java
package org.example.fun;  
  
import com.alibaba.fastjson.JSONObject;  
  
import java.text.ParseException;  
  
public interface DimAsyncJoinFunction<T> {  
      
    void join(T input, JSONObject dimInfo) throws ParseException;  
  
    String getKey(T input);  
}
```

```JAVA
package org.example.fun;

import com.alibaba.fastjson.JSONObject;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.functions.async.AsyncFunction;


import org.apache.flink.streaming.api.functions.async.ResultFuture;
import org.apache.flink.streaming.api.functions.async.RichAsyncFunction;
import org.example.common.GmallConfig;
import org.example.utils.DimUtil;
import org.example.utils.ThreadPoolUtil;

import java.lang.reflect.InvocationTargetException;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.text.ParseException;
import java.util.Collections;
import java.util.concurrent.ThreadPoolExecutor;

public abstract class DimAsyncFunction<IN> extends RichAsyncFunction<IN,IN> implements DimAsyncJoinFunction<IN> {


    private String tableName;

    public DimAsyncFunction(String tableName) {
        this.tableName = tableName;
    }

    public DimAsyncFunction() {
    }

    private Connection connection;
    private ThreadPoolExecutor threadPoolExecutor;
    //初始化连接
    @Override
    public void open(Configuration parameters) throws Exception {
        Class.forName(GmallConfig.PHOENIX_DRIVER);
        connection = DriverManager.getConnection(GmallConfig.PHOEXIN_SERVER);
        threadPoolExecutor = ThreadPoolUtil.getThreadPool();
    }


    @Override
    public void asyncInvoke(IN input, ResultFuture<IN> resultFuture) throws Exception {

        threadPoolExecutor.submit(new Runnable() {
            @Override
            public void run() {

                try {
                    // 查询维度信息，补存维度信息，将数据输出
                    String id  = getKey(input);
                    JSONObject dimInfo = DimUtil.getDimInfo(connection, tableName, id);
                    if (dimInfo != null){
                        join(input,dimInfo);
                    }
                    resultFuture.complete(Collections.singletonList(input));
                } catch (SQLException e) {
                    throw new RuntimeException(e);
                } catch (InvocationTargetException e) {
                    throw new RuntimeException(e);
                } catch (InstantiationException e) {
                    throw new RuntimeException(e);
                } catch (IllegalAccessException | ParseException e) {
                    throw new RuntimeException(e);
                }
            }
        });

    }



    @Override
    public void timeout(IN input, ResultFuture<IN> resultFuture) throws Exception {
        System.out.println("请求超时重新发送请求");
    }
}

```
OrderWideApp
```java
package org.example.app;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.connector.kafka.source.KafkaSource;
import org.apache.flink.runtime.state.hashmap.HashMapStateBackend;
import org.apache.flink.streaming.api.datastream.AsyncDataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.co.ProcessJoinFunction;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;
import org.example.bean.OrderDetail;
import org.example.bean.OrderInfo;
import org.example.bean.OrderWide;
import org.example.fun.DimAsyncFunction;
import org.example.utils.KafkaUtil;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.concurrent.TimeUnit;

/**
 * author:zhaosiliang
 * date:2022/7/3 11:04
 * 描述：
 **/
public class OrderWideApp {
    public static void main(String[] args) {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        //设置状态后端
        env.setStateBackend(new HashMapStateBackend());
        env.getCheckpointConfig().setCheckpointStorage("file:///D:/checkPoint/eff7bb8bcb4046ac95cf5c0d86dd3115");
        env.enableCheckpointing(3000);

        //读取kafka订单和订单主题明细数据
        String orderInfoSourceTopic = "dwd_order_info";
        String orderDetailSourceTopic ="dwd_order_detail";
        String orderWideSinkTopic="dwm_order_wide";
        String groupId ="order_wide_group";

        KafkaSource<String> orderInfoKafkaSource = KafkaUtil.getKafkaSource(orderInfoSourceTopic, groupId);
        KafkaSource<String> orderDetailKafkaSource = KafkaUtil.getKafkaSource(orderDetailSourceTopic, groupId);
        DataStreamSource<String> orderInfoKafkaDS = env.fromSource(orderInfoKafkaSource, WatermarkStrategy.noWatermarks(),"order_info_kafka");
        DataStreamSource<String> orderDetailKafkaDS = env.fromSource(orderDetailKafkaSource, WatermarkStrategy.noWatermarks(), "order_detail_kafka");

        //将数据转为java bean 提取时间戳生成watermark
        WatermarkStrategy<OrderInfo> orderInfoWatermarkStrategy = WatermarkStrategy.<OrderInfo>forMonotonousTimestamps()
                .withTimestampAssigner(new SerializableTimestampAssigner<OrderInfo>() {
                    @Override
                    public long extractTimestamp(OrderInfo element, long recordTimestamp) {
                        return element.getCreate_ts();
                    }
                });

        WatermarkStrategy<OrderDetail> orderDetailWatermarkStrategy = WatermarkStrategy.<OrderDetail>forMonotonousTimestamps()
                .withTimestampAssigner(new SerializableTimestampAssigner<OrderDetail>() {
                    @Override
                    public long extractTimestamp(OrderDetail element, long recordTimestamp) {
                        return element.getCreate_ts();
                    }
                });

        KeyedStream<OrderInfo, Long> orderInfoWithIdKeyedStream = orderDetailKafkaDS.map(jsonStr -> {
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            //将 JSON 字符串转换为 JavaBean
            OrderInfo orderInfo = JSON.parseObject(jsonStr, OrderInfo.class);
            //取出创建时间字段
            String create_time = orderInfo.getCreate_time();
            //按照空格分割
            String[] createTimeArr = create_time.split(" ");
            orderInfo.setCreate_date(createTimeArr[0]);
            orderInfo.setCreate_hour(createTimeArr[1]);
            orderInfo.setCreate_ts(sdf.parse(create_time).getTime());
            return orderInfo;
        }).assignTimestampsAndWatermarks(orderInfoWatermarkStrategy)
                .keyBy(OrderInfo::getId);

        KeyedStream<OrderDetail, Long> orderDetailWithOrderIdKeyedStream =
                orderDetailKafkaDS.map(jsonStr -> {
                            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                            OrderDetail orderDetail = JSON.parseObject(jsonStr, OrderDetail.class);
                            orderDetail.setCreate_ts(sdf.parse(orderDetail.getCreate_time()).getTime());
                            return orderDetail;
                        }).assignTimestampsAndWatermarks(orderDetailWatermarkStrategy)
                        .keyBy(OrderDetail::getOrder_id);
        //双流 JOIN
        SingleOutputStreamOperator<OrderWide> orderWideNoDimDS =
                orderInfoWithIdKeyedStream.intervalJoin(orderDetailWithOrderIdKeyedStream)
                        .between(Time.seconds(-5), Time.seconds(5))
                //生产环境,为了不丢数据,设置时间为最大网络延迟
                .process(new ProcessJoinFunction<OrderInfo, OrderDetail, OrderWide>() {
                    @Override
                    public void processElement(OrderInfo orderInfo, OrderDetail orderDetail, Context
                            context, Collector<OrderWide> collector) throws Exception {
                        collector.collect(new OrderWide(orderInfo, orderDetail));
                    }
                });

        //关联维度表信息
        //orderWideDS.map(orderWide->{
            //关联用户维度 DimUtil.getDimInfo

        // })

        //FLINK 异步IO
        //关联用户维度
        SingleOutputStreamOperator<OrderWide> orderWideWithUserDS = AsyncDataStream.unorderedWait(orderWideNoDimDS, new DimAsyncFunction<OrderWide>("DIM_USER_INFO") {

            public String getKey(OrderWide orderWide) {
                return orderWide.getUser_id().toString();
            }

            public void join(OrderWide orderWide, JSONObject dimInfo) throws ParseException {
                orderWide.setUser_gender(dimInfo.getString("GENDER"));
                String birthday = dimInfo.getString("BIRTHDAY");
                SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
                long currentTime = System.currentTimeMillis();
                long ts = simpleDateFormat.parse(birthday).getTime();
                Long age = (currentTime - ts) / (1000 * 60 * 60 * 24 * 365);
                orderWide.setUser_age(age.intValue());
            }
        }, 65, TimeUnit.SECONDS);

        //关联地区维度
        SingleOutputStreamOperator<OrderWide> orderWithArea = AsyncDataStream.unorderedWait(orderWideWithUserDS, new DimAsyncFunction<OrderWide>("DIM_BASE_PROVINCE") {
            @Override
            public void join(OrderWide orderWide, JSONObject dimInfo) throws ParseException {
                orderWide.setProvince_name(dimInfo.getString("name"));
                orderWide.setProvince_area_code(dimInfo.getString("area_code"));
                orderWide.setProvince_iso_code(dimInfo.getString("ISO_CODE"));
                orderWide.setProvince_3166_2_code(dimInfo.getString("ISO_S166_CODE"));
            }

            @Override
            public String getKey(OrderWide orderWide) {
                return orderWide.getProvince_id().toString();
            }
        }, 60, TimeUnit.SECONDS);
    }
    //关联其他维度信息
}


```