###### NVL
```
第一个参数为空那么显示第二个参数的值，如果第一个参数的值不为空，则显示第一个参数本来的值。
```
##### 窗口函数
 窗口函数order by
```
当为排序函数，如row_number(),rank()等时，over中的order by只起到窗口内排序作用。

当为聚合函数，如max，min，count等时，不加order by ，就是针对一整个分区进行sum求和。加上order by score后，就是根据当前行到之前所有行聚合
```
- lead 函数
```
LEAD (col, n, default)：用于统计窗口内往下滴 n 行值。第一个参数为列名，第二个参数为往下滴 n 行（默认为 1 ），第三个参数为默认值（当往下第n行为NULL时候，取默认值，如不指定，则为NULL）
```
- LAG 
```
与 lead 函数相反，用于统计窗口内网上第 n 行值。
```
FIRST_VALUE 
```
取分组内排序后，截止到当前行，第一个值

This takes at most two parameters. The first parameter is the column for which you want the first value, the second (optional) parameter must be a boolean which is false by default. If set to true it skips null values.
```
LAST_VALUE
```
与first_value 相反
取分组内排序后，截止到当前行，最后一个值

This takes at most two parameters. The first parameter is the column for which you want the last value, the second (optional) parameter must be a boolean which is false by default. If set to true it skips null values.

```

##### 窗口函数去重
```
INSERT OVERWRITE TABLE ecang_ods_order_detail 
SELECT  op_id,
        order_id,
        product_sku,
        sku,
        warehouse_sku,
        warehouse_sku_qty,
        unit_price,
        qty
FROM    (
            SELECT  *
                    ,row_number() OVER (PARTITION BY aa.op_id ORDER BY aa.op_id DESC ) AS rn
            FROM    ecang_ods_order_detail AS aa
        ) AS t
WHERE   t.rn = 1
;

INSERT OVERWRITE 清除表后重新插入
row_number() OVER (PARTITION BY aa.op_id ORDER BY aa.op_id DESC ) AS rn 

op_id 分组 分组后在组内以op_id 排序 生成每组内部排序后的顺序编号rank ,取结果数据rank= 1的数据

```
##### 修改数据
```
INSERT OVERWRITE TABLE ecang_order_data
SELECT  CASE    WHEN s.order_id IS NOT NULL THEN s.order_id 
                ELSE t.order_id 
        END AS order_id
        ,CASE    WHEN s.order_id IS NOT NULL THEN s.platform 
                 ELSE t.platform 
                 FROM    ecang_order_data AS t
FULL OUTER JOIN ecang_ods_order_data AS s
ON      t.order_id = s.order_id
;
源表 order_id 不存在取目标表数据,存在取源表
```
##### 列转行
```
INSERT OVERWRITE TABLE ecang_ods_order_detail

SELECT 

        GET_JSON_OBJECT( CONCAT('{', info, "}"),'$.op_id') AS op_id
        ,GET_JSON_OBJECT( CONCAT('{', info, "}"),'$.order_id') AS order_id
        ,GET_JSON_OBJECT(CONCAT('{', info, "}"),'$.product_sku') AS product_sku

FROM ( SELECT  GET_JSON_OBJECT(value,"$.value") as details FROM   ecang_analytics_transfer  WHERE GET_JSON_OBJECT(value,"$.type") = 'order_details')
LATERAL VIEW EXPLODE (split(regexp_replace(regexp_replace(details,'\\[\\{',''),'}]',''),'},\\{')) info as info;
```
##### 删除数据
```
INSERT OVERWRITE TABLE table1
SELECT t1.key1
      ,t1.key2
      ,t1.col1
      ,t1.col2
  FROM table1 t1
  LEFT OUTER JOIN table2 t2 ON t1.key1 = t2.key1 AND t1.key2 = t2.key2
  WHERE t2.key1 IS NULL
;                        
```
##### Merge（当日发生过删除操作）
```
INSERT OVERWRITE TABLE table1
SELECT *
  FROM(
//先把上日和今日都存在的记录从上日表中排除，再把今日删除的记录排除。剩下的就是今日没有更新的记录。
SELECT t1.key1
      ,t1.key2
      ,t1.col1
      ,t1.col2
  FROM table1 t1
  LEFT OUTER JOIN table2 t2 ON t1.key1 = t2.key1 AND t1.key2 = t2.key2
  LEFT OUTER JOIN table3 t3 ON t1.key1 = t3.key1 AND t1.key2 = t3.key2
  WHERE t2.key1 IS NULL OR t3.key1 IS NULL
  UNION ALL
//合并上今日增量，就是今日的全量。
SELECT t2.key1
      ,t2.key2
      ,t2.col1
      ,t2.col2
  FROM table2 t2)tt
;
```

##### Merge（当日没有发生过删除操作）
```
INSERT OVERWRITE TABLE table1
SELECT *
  FROM(
//先把上日存在、今日也存在的记录从上日表中排除，剩下的就是今日没有更新的记录。
SELECT t1.key1
      ,t1.key2
      ,t1.col1
      ,t1.col2
  FROM table1 t1
  LEFT OUTER JOIN table2 t2 ON t1.key1 = t2.key1 AND t1.key2 = t2.key2
  WHERE t2.key1 IS NULL
  UNION ALL
//再合并上今日增量，就是今天的全量。
SELECT t2.key1
      ,t2.key2
      ,t2.col1
      ,t2.col2
  FROM table2 t2)tt
;
```


##### SEMI JOIN（半连接）
```
left semi join
当join条件成立时，返回左表中的数据。如果左表中满足指定条件的某行数据在右表中出现过，则此行保留在结果集中。
LEFT SEMI JOIN只会返回左表中能匹配到右表的数据

在MaxCompute中，与left semi join类似的操作为in subquery，请参见IN SUBQUERY。您可以自行选择其中一种方式。

left anti join
当join条件不成立时，返回左表中的数据。如果左表中满足指定条件的某行数据没有在右表中出现过，则此行保留在结果集中。 
LEFT ANTI JOIN只会返回左表中不能匹配到右表的数据

```
```
在MaxCompute中，与left anti join类似的操作为not in subquery，但并不完全相同，请参见NOT IN SUBQUERY。

semi join支持mapjoin hint，可以提高left semi join和left anti join的性能。更多mapjoin hint信息，请参见MAPJOIN HINT。

示例数据
为便于理解，本文为您提供源数据，基于源数据提供相关示例。创建表sale_detail和sale_detail_sj，并添加数据，命令示例如下：
--创建一张分区表sale_detail。
create table if not exists sale_detail
(
shop_name     string,
customer_id   string,
total_price   double
)
partitioned by (sale_date string, region string);

create table if not exists sale_detail_sj
(
shop_name     string,
customer_id   string,
total_price   double
)
partitioned by (sale_date string, region string);

--向源表增加分区。
alter table sale_detail add partition (sale_date='2013', region='china');
alter table sale_detail_sj add partition (sale_date='2013', region='china');

--向源表追加数据。
insert into sale_detail partition (sale_date='2013', region='china') values ('s1','c1',100.1),('s2','c2',100.2),('s3','c3',100.3);
insert into sale_detail_sj partition (sale_date='2013', region='china') values ('s1','c1',100.1),('s2','c2',100.2),('s5','c2',100.2),('s2','c2',100.2);
使用示例
示例1：查询sale_detail表中，total_price出现在sale_detail_sj表中的数据集。命令示例如下：
select * from sale_detail a left semi join sale_detail_sj b on a.total_price=b.total_price;
返回结果如下：
+------------+-------------+-------------+------------+------------+
| shop_name  | customer_id | total_price | sale_date  | region     |
+------------+-------------+-------------+------------+------------+
| s2         | c2          | 100.2       | 2013       | china      |
| s1         | c1          | 100.1       | 2013       | china      |
+------------+-------------+-------------+------------+------------+
只会返回sale_detail中的数据，只要sale_detail的total_price在sale_detail_sj的total_price中出现过。

示例2：查询sale_detail表中，total_price没有出现在sale_detail_sj表中的数据集。命令示例如下：
select * from sale_detail a left anti join sale_detail_sj b on a.total_price=b.total_price;
返回结果如下：
+------------+-------------+-------------+------------+------------+
| shop_name  | customer_id | total_price | sale_date  | region     |
+------------+-------------+-------------+------------+------------+
| null       | c5          | NULL        | 2014       | shanghai   |
| s3         | c3          | 100.3       | 2013       | china      |
| s6         | c6          | 100.4       | 2014       | shanghai   |
| s7         | c7          | 100.5       | 2014       | shanghai   |
+------------+-------------+-------------+------------+------------+
只会返回sale_detail中的数据，只要sale_detail的total_price在sale_detail_sj的total_price中没有出现过。

```
##### MAPJOIN HINT

当您对一个大表和一个或多个小表执行join操作时，可以在select语句中显式指定mapjoin Hint提示以提升查询性能。本文为您介绍如何通过mapjoin hint连接表。
###### 使用方法
- 需要在select语句中使用Hint提示/*+ mapjoin(<table_name>) */才会执行mapjoin。：

- 引用小表或子查询时，需要引用别名。
- mapjoin支持小表为子查询。
在mapjoin中，可以使用不等值连接或or连接多个条件。您可以通过不写on语句而通过mapjoin on 1 = 1的形式，实现笛卡尔乘积的计算。例如select /*+ mapjoin(a) */ a.id from shop a join table_name b on 1=1;，但此操作可能带来数据量膨胀问题。
mapjoin中多个小表用英文逗号（,）分隔，例如/*+ mapjoin(a,b,c)*/。

######  使用限制
- mapjoin操作的使用限制如下：
- mapjoin在Map阶段会将指定表的数据全部加载在内存中，因此指定的表仅能为小表，且表被加载到内存后占用的总内存不得超过512 MB。由于MaxCompute是压缩存储，因此小表在被加载到内存后，数据大小会急剧膨胀。此处的512 MB是指加载到内存后的空间大小。
- mapjoin中join操作的限制如下：
left outer join的左表必须是大表。
right outer join的右表必须是大表。
不支持full outer join。
inner join的左表或右表均可以是大表。


##### SPLIT_PART
```
命令说明
依照分隔符separator拆分字符串str，返回从start部分到end部分的子串（闭区间）。

参数说明
str：必填。STRING类型。待拆分的字符串。如果是BIGINT、DOUBLE、DECIMAL或DATETIME类型，则会隐式转换为STRING类型后参与运算，其他类型会返回报错。
separator：必填。STRING类型常量。拆分用的分隔符，可以是一个字符，也可以是一个字符串，其他类型返回报错。
start：必填。BIGINT类型常量，必须大于0。非常量或其他类型会返回异常报错。返回段的开始编号（从1开始），如果没有指定end，则返回start指定的段。
end：BIGINT类型常量，大于等于start，否则返回报错。返回段的截止编号，非常量或其他类型会返回报错。不指定时表示最后一部分。

返回值说明
返回STRING类型。
如果start的值大于切分后实际的分段数，例如字符串拆分完有6个片段，start大于6，返回空串。
如果separator不存在于str中，且start指定为1，返回整个str。如果str为空串，则输出空串。
如果separator为空串，则返回原字符串str。
如果end大于片段个数，返回从start开始的子串。
除separator外，如果任一输入参数为NULL，则返回NULL。
--返回a。
select split_part('a,b,c,d', ',', 1);
--返回a,b。
select split_part('a,b,c,d', ',', 1, 2);
```
##### 增量历史拉链表(增量表与历史表历史表中会保存重复没有变化的数据)
```
创建历史表
CREATE TABLE IF NOT EXISTS dw_order_his
(
    orderid       STRING,
    status        STRING,
    createtime    STRING,
    modifiedtime  STRING,
    dw_start_date STRING,
    dw_end_date   STRING
) ;

增量表 以日期为分区
CREATE TABLE IF NOT EXISTS ods_orders_inc
(
    orderid      STRING,
    status       STRING,
    createtime   STRING,
    modifiedtime STRING
) 
PARTITIONED BY
(
    day          STRING
)
LIFECYCLE 1;

全量初始数据
INSERT INTO TABLE `ods_orders_inc` PARTITION (day= '2019-03-20') VALUES ('1', '创建', '2019-03-20 10:19:53', '2019-03-20 10:20:33');
INSERT INTO TABLE `ods_orders_inc` PARTITION (day= '2019-03-20') VALUES ('2', '创建', '2019-03-20 10:20:42', '2019-03-20 10:20:42');
INSERT INTO TABLE `ods_orders_inc` PARTITION (day= '2019-03-20') VALUES ('3', '创建', '2019-03-20 10:20:55', '2019-03-20 10:20:55');
INSERT INTO TABLE `ods_orders_inc` PARTITION (day= '2019-03-20') VALUES ('4', '创建', '2019-03-20 10:21:00', '2019-03-20 10:21:00');
INSERT INTO TABLE `ods_orders_inc` PARTITION (day= '2019-03-20') VALUES ('5', '创建', '2019-03-20 10:21:05', '2019-03-20 10:21:05');

INSERT OVERWRITE  TABLE dw_order_his
	SELECT orderid,
		   status,
	       createtime,
	       modifiedtime,
	       to_date(createtime) AS dw_start_date,
	       '9999-12-31' AS dw_end_date
	FROM ods_orders_inc WHERE  day='2019-03-20'
;
+------------+-------------+-------------+------------+------------+------------+
orderid     	createtime	   modifiedtime	  status	dw_start_date	dw_end_date
+------------+-------------+-------------+------------+------------+------------+    
|   4	2019-03-20 10:21:00	2019-03-20 10:21:00	创建	2019-03-20	9999-12-31
|   2	2019-03-20 10:20:42	2019-03-20 10:20:42	创建	2019-03-20	9999-12-31
|   5	2019-03-20 10:21:05	2019-03-20 10:21:05	创建	2019-03-20	2019-03-20
|   3	2019-03-20 10:20:55	2019-03-20 10:20:55	创建	2019-03-20	2019-03-20
|   1	2019-03-20 10:19:53	2019-03-20 10:20:33	创建	2019-03-20	2019-03-20
+------------+-------------+-------------+------------+------------+------------+    


新增数据
INSERT INTO `ods_orders_inc` PARTITION (day= '2019-03-21') VALUES ('1', '支付', '2019-03-20 10:19:53', '2019-03-21 13:12:46');
INSERT INTO `ods_orders_inc` PARTITION (day= '2019-03-21')  VALUES ('3', '完成', '2019-03-20 10:20:55', '2019-03-21 13:13:55');
INSERT INTO `ods_orders_inc` PARTITION (day= '2019-03-21') VALUES ('5', '支付', '2019-03-20 10:21:05', '2019-03-21 13:14:05');
INSERT INTO `ods_orders_inc` PARTITION (day= '2019-03-21') VALUES ('6', '创建', '2019-03-21 10:21:05', '2019-03-21 13:15:05');

orderid	createtime	modifiedtime	status	dw_start_date	dw_end_date
1	2019-03-20 10:19:53	2019-03-21 13:12:46	支付	2019-03-21	9999-12-31
6	2019-03-21 10:21:05	2019-03-21 13:15:05	创建	2019-03-21	9999-12-31
5	2019-03-20 10:21:05	2019-03-21 13:14:05	支付	2019-03-21	9999-12-31
3	2019-03-20 10:20:55	2019-03-21 13:13:55	完成	2019-03-21	9999-12-31



创建临时表 增量刷新历史数据
set odps.sql.validate.orderby.limit=false;
-- set odps.sql.type.system.odps2=true;
DROP TABLE IF EXISTS dw_orders_his_tmp;	
CREATE TABLE dw_orders_his_tmp AS
SELECT orderid,
       createtime,
       modifiedtime,
       status,
       dw_start_date,
       dw_end_date
FROM
  (SELECT a.orderid,
          a.createtime,
          a.modifiedtime,
          a.status,
          a.dw_start_date,
          CASE
              WHEN b.orderid IS NOT NULL
                   AND a.dw_end_date > '2019-03-21' THEN '2019-03-20'
              ELSE a.dw_end_date
          END AS dw_end_date
   FROM dw_order_his a
   LEFT OUTER JOIN
     (SELECT *
      FROM ods_orders_inc
      WHERE day = '2019-03-21') b ON (a.orderid = b.orderid)
   UNION ALL SELECT orderid,
                    createtime,
                    modifiedtime,
                    status,
                    TO_CHAR(modifiedtime,'yyyy-mm-dd')AS dw_start_date,
                    '9999-12-31' AS dw_end_date
   FROM ods_orders_inc
   WHERE day = '2019-03-21' ) x
ORDER BY orderid,
         dw_start_date;
         
orderid	createtime	modifiedtime	status	dw_start_date	dw_end_date
1	2019-03-20 10:19:53	2019-03-20 10:20:33	创建	2019-03-20	2019-03-20
1	2019-03-20 10:19:53	2019-03-21 13:12:46	支付	2019-03-21	9999-12-31
2	2019-03-20 10:20:42	2019-03-20 10:20:42	创建	2019-03-20	9999-12-31
3	2019-03-20 10:20:55	2019-03-20 10:20:55	创建	2019-03-20	2019-03-20
3	2019-03-20 10:20:55	2019-03-21 13:13:55	完成	2019-03-21	9999-12-31
4	2019-03-20 10:21:00	2019-03-20 10:21:00	创建	2019-03-20	9999-12-31
5	2019-03-20 10:21:05	2019-03-20 10:21:05	创建	2019-03-20	2019-03-20
5	2019-03-20 10:21:05	2019-03-21 13:14:05	支付	2019-03-21	9999-12-31
6	2019-03-21 10:21:05	2019-03-21 13:15:05	创建	2019-03-21	9999-12-31




UNION ALL的两个结果集中，
第一个是用历史拉链表left outer join 日期为2019-03-21的增量,能关联上并且dw_end_date > ‘2019-03-21’，说明状态有变化，则把原来的dw_end_date置为2019-03-20,
关联不上的，说明状态无变化,dw_end_date则不变。
第二个结果集是直接将日期为2019-03-21代表最新的状态的增量数据插入历史拉链表。


将临时表数据插入历史表

INSERT overwrite TABLE dw_order_his
	SELECT orderid as orderid,
		   status as status,
	       createtime AS createtime,
	       modifiedtime AS modifiedtime,
	        dw_start_date,
	       dw_end_date
	FROM dw_orders_his_tmp;

————————————————
原文链接：https://blog.csdn.net/weixin_43215250/article/details/88709006
https://developer.aliyun.com/article/542146
```
```
--通过DW历史数据和ODS增量数据刷新DW表
insert overwrite table dw_orders_his_d 
SELECT a0.orderid, a0.createtime, a0.modifiedtime, a0.o_status, a0.dw_start_date, a0.dw_end_date
FROM (
	-- 对orderid进行开窗然后按照生命周期结束时间倒序排，支持重跑
	SELECT a1.orderid, a1.createtime, a1.modifiedtime, a1.o_status, a1.dw_start_date, a1.dw_end_date
	, ROW_NUMBER() OVER (distribute BY a1.orderid,a1.createtime, a1.modifiedtime,a1.o_status sort BY a1.dw_end_date DESC) AS nums
	FROM (
		-- 用历史数据与增量22日的数据进行匹配，当发现在22日新增数据中存在且end_date > 当前日期的就表示数据状态发生过变化，然后修改生命周期
		-- 修改昨日已经生命截止的数据并union最新增量数据到DW
		SELECT a.orderid, a.createtime, a.modifiedtime, a.o_status, a.dw_start_date  
			, CASE 
				WHEN b.orderid IS NOT NULL AND a.dw_end_date > ${bdp.system.bizdate} THEN ${yesterday}
				ELSE a.dw_end_date
			END AS dw_end_date
		FROM dw_orders_his_d a
		LEFT OUTER JOIN (
			SELECT *
			FROM ods_orders_inc_d
			WHERE dt = ${bdp.system.bizdate}
		) b
		ON a.orderid = b.orderid
		UNION ALL
		--2015-08-22的增量数据刷新到DW
		SELECT orderid, createtime, modifiedtime, o_status, modifiedtime AS dw_start_date
			, '99991231' AS dw_end_date
		FROM ods_orders_inc_d
		WHERE dt = ${bdp.system.bizdate}
	) a1
) a0 
-- 开窗口后对某个订单中生命周期为'9999-12-31'的取值并写入，防止重跑数据情况。
WHERE a0.nums = 1
order by a0.orderid,a0.dw_start_date;
```
##### 如何使用拉链表
```
- 查看某一天的全量历史快照数据。
SELECT *
FROM dw_orders_his_d
WHERE dw_start_date <= '20150822'
	AND dw_end_date >= '20150822'
ORDER BY orderid
LIMIT 10000;

- 取一段时间的变化记录集合，如在20150822-20150823变化的记录。
SELECT *
FROM dw_orders_his_d
WHERE dw_start_date <= '20150823'
	AND dw_end_date >= '20150822'
ORDER BY orderid
LIMIT 10000;
- 查看某一订单历史变化情况。
SELECT *
FROM dw_orders_his_d
WHERE orderid = 8
ORDER BY dw_start_date;
- 取最新的数据。
SELECT *
FROM dw_orders_his_d
WHERE dw_end_date = '99991231'
```
##### 基于maxcompute的拉链表实现，支持回滚、重跑
```
先看这一篇：阿里云社区一篇关于拉链表的文章
基于MaxCompute的拉链表设计
背景：
使用拉链表记录用户登录的最终状态，有cty,os,uid,pn,idfa,ip… 具体含义不需要理解，就是用户标签，用拉链表记录用户标签的变化过程来替代一天一张全量表，节省存储资源
历史拉链表：etl_puid_user_lst
每日增量表：etl_puid_user_tmp

-- status 分区，值为1的表示增量数据，0的表示旧数据，用处在于将maxcompute数据同步到adb的时候只需增量更新而不需要全量更新，节省资源
insert overwrite table etl_puid_user_lst PARTITION(status)
select x.recorddate,x.cgi,x.puid,x.lst_cty,x.lst_os,x.lst_uid,x.lst_pn,
    x.lst_idfa,x.lst_ip,x.lst_serverid,x.start_date,x.end_date,
    case when
     x.end_date = to_char(DATEADD(to_date('${parttime}','yyyy-mm-dd'),-1,'dd'), 'yyyy-mm-dd') or 
     (x.start_date='${parttime}' and x.end_date = '9999-12-31') 
    then 1 else 0 end  
from (
    select t.*,ROW_NUMBER() OVER(PARTITION BY t.cgi,t.puid,lst_cty,lst_os,lst_uid,lst_pn,lst_idfa,lst_ip,lst_serverid,end_date ORDER BY t.recorddate asc ) AS rn  -- 重跑时需要 ，重跑时候会产生类似如下数据： 02 - 9999 03 - 9999 保留一条即可 02 - 9999... 
    from (
        select a.recorddate,a.cgi,a.puid,a.lst_cty,a.lst_os,a.lst_uid,a.lst_pn,
        a.lst_idfa,a.lst_ip,a.lst_serverid,a.start_date
        ,case when b.puid is not null and a.end_date>b.parttime 
            then to_char(DATEADD(to_date(b.parttime,'yyyy-mm-dd'),-1,'dd'), 'yyyy-mm-dd') 
            else a.end_date end as end_date
        from (select recorddate,cgi,puid,lst_cty,lst_os,lst_uid,lst_pn,lst_idfa,lst_ip,lst_serverid,start_date,
                case when end_date >=  '${parttime}' then '9999-12-31'   --重 跑 07 改 05 - 08 为 05 - 9999 即回到6号的状态
                     when end_date = to_char(DATEADD(to_date('${parttime}','yyyy-mm-dd'),-1,'dd'), 'yyyy-mm-dd') then '9999-12-31' -- 重跑 07 改 06 - 06 为 06 - 9999 即回到6号的状态
                    else end_date end as end_date
                from etl_puid_user_lst where status = 0 or status = 1 and recorddate<'${parttime}'
            ) a  -- a表 这么一大串做的工作是对对数据预处理，比如跑 7号数据，需要先将拉链表回到 6 号的状态，这是支持重跑的关键 
        left join 
            (select * from etl_puid_user_tmp where parttime='${parttime}' and lst_logon_time is not null and lst_logon_time !='') b
            on a.cgi=b.cgi and a.puid=b.puid and ( a.lst_cty != b.lst_cty or a.lst_os != b.lst_os or  a.lst_uid != b.lst_uid or a.lst_pn != b.lst_pn or a.lst_idfa != b.lst_idfa  or a.lst_ip != b.lst_ip  or a.lst_serverid != b.lst_serverid ) 
        union all 
            select parttime,c.cgi,c.puid,c.lst_cty,c.lst_os,c.lst_uid,c.lst_pn,c.lst_idfa,c.lst_ip,c.lst_serverid
             ,parttime as start_date,'9999-12-31' as end_date --,1 as status
            from 
                (select * from etl_puid_user_tmp where parttime='${parttime}' and lst_logon_time is not null and lst_logon_time !='') c
    ) t
) x
where rn = 1 
;
————————————————
版权声明：本文为CSDN博主「linweicong1」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/linweicong1/article/details/94441355
```
##### 历史拉链表动态分区 
**新数据存放9999-12-30分区，过期数据放入前一天分区**
1. 使用rank 取同一个账号id下最后一条状态的数据
2. 9999-12-30分区full join 增量表ods_inc 表
3. if 判断增量的数据id。 是否存在，存在使用增量表的数据，否者使用历史表数据，生成9999-12-30分区最新的数据。
4. 历史表与新表都存在的id,说明该idde 数据已经被更新了，将该条数据做为过期数据保存在前一天分区。
5. 使用union all 将新数据和过期的历史关联在一起，dt字段做为insert动态分区

```sql

with
tmp as(
    select
        old.id old_id,
        old.login_name old_login_name,
        old.nick_name old_nick_name,
        old.name old_name,
        old.phone_num old_phone_num,
        old.email old_email,
        old.user_level old_user_level,
        old.birthday old_birthday,
        old.gender old_gender,
        old.create_time old_create_time,
        old.operate_time old_operate_time,
        old.start_date old_start_date,
        old.end_date old_end_date,
        new.id new_id,
        new.login_name new_login_name,
        new.nick_name new_nick_name,
        new.name new_name,
        new.phone_num new_phone_num,
        new.email new_email,
        new.user_level new_user_level,
        new.birthday new_birthday,
        new.gender new_gender,
        new.create_time new_create_time,
        new.operate_time new_operate_time,
        new.start_date new_start_date,
        new.end_date new_end_date
    from
    (
        select
            id,
            login_name,
            nick_name,
            name,
            phone_num,
            email,
            user_level,
            birthday,
            gender,
            create_time,
            operate_time,
            start_date,
            end_date
        from dim_user_zip
        where dt='9999-12-31'
    )old
    full outer join
    (
        select
            id,
            login_name,
            nick_name,
            md5(name) name,
            md5(phone_num) phone_num,
            md5(email) email,
            user_level,
            birthday,
            gender,
            create_time,
            operate_time,
            '2020-06-15' start_date,
            '9999-12-31' end_date
        from
        (
            select
                data.id,
                data.login_name,
                data.nick_name,
                data.name,
                data.phone_num,
                data.email,
                data.user_level,
                data.birthday,
                data.gender,
                data.create_time,
                data.operate_time,
                -- 取同一个账号id最长一个状态的数据                row_number() over (partition by data.id order by ts desc) rn
            from ods_user_info_inc
            where dt='2020-06-15'
        )t1
        where rn=1
    )new
    on old.id=new.id
)
```

```sql
insert overwrite table dim_user_zip partition(dt)
select
    if(new_id is not null,new_id,old_id),
    if(new_id is not null,new_login_name,old_login_name),
    if(new_id is not null,new_nick_name,old_nick_name),
    if(new_id is not null,new_name,old_name),
    if(new_id is not null,new_phone_num,old_phone_num),
    if(new_id is not null,new_email,old_email),
    if(new_id is not null,new_user_level,old_user_level),
    if(new_id is not null,new_birthday,old_birthday),
    if(new_id is not null,new_gender,old_gender),
    if(new_id is not null,new_create_time,old_create_time),
    if(new_id is not null,new_operate_time,old_operate_time),
    if(new_id is not null,new_start_date,old_start_date),
    if(new_id is not null,new_end_date,old_end_date),
    if(new_id is not null,new_end_date,old_end_date) dt
    -- 获取新增即变化和未发生变化的数据放入9999分区
    -- 这里使用的是动态分区，dt作为动态分区的字段
from tmp
union all
select
    old_id,
    old_login_name,
    old_nick_name,
    old_name,
    old_phone_num,
    old_email,
    old_user_level,
    old_birthday,
    old_gender,
    old_create_time,
    old_operate_time,
    old_start_date,
    cast(date_add('2020-06-15',-1) as string) old_end_date,
    cast(date_add('2020-06-15',-1) as string) dt
from tmp
-- 老数据与新数据都含有的id说明这条数据是过期的数据
where old_id is not null
and new_id is not null;

```

```text
TO_JSON
命令格式
to_json(<expr>)
命令说明
将给定的复杂类型expr，以JSON字符串格式输出。

参数说明
expr：必填。ARRAY、MAP、STRUCT复杂类型。
说明 如果输入为STRUCT类型（struct<key1:value1, key2:value2）：
转换为JSON字符串时，key会全部转为小写。
value如果为NULL，则不输出value本组的数据。例如value2为NULL，则key2:value2不会输出到JSON字符串。
示例
示例1：将指定复杂类型以指定格式输出。命令示例如下。
--返回{"a":1,"b":2}。
select to_json(named_struct('a', 1, 'b', 2));
--返回{"time":"26/08/2015"}。
select to_json(named_struct('time', "26/08/2015"));
--返回[{"a":1,"b":2}]。
select to_json(array(named_struct('a', 1, 'b', 2)));
--返回{"a":{"b":1}}。
select to_json(map('a', named_struct('b', 1)));
--返回{"a":1}。
select to_json(map('a', 1));
--返回[{"a":1}]。
select to_json(array((map('a', 1))));
示例2：输入为STRUCT类型的特殊情况。命令示例如下。
--返回{"a":"B"}。STRUCT类型转换为JSON字符串时，key会全部转为小写。
select to_json(named_struct("A", "B"));
--返回{"k2":"v2"}。NULL值所在组的数据，不会输出到JSON字符串。
select to_json(named_struct("k1", cast(null as string), "k2", "v2"));
```
##### 交集、并集和补集
您可以通过MaxCompute对查询结果数据集执行取交集、并集或补集操作。本文为您介绍交集（intersect、intersect all、intersect distinct）、并集（union、union all、union distinct）和补集（except、except all、except distinct、minus、minus all、minus distinct）的使用方法。
-  交集

```text
--取交集不去重。
<select_statement1> intersect all <select_statement2>;
--取交集并去重。intersect效果等同于intersect distinct。
<select_statement1> intersect [distinct] <select_statement2>;
对两个数据集取交集，不去重
select * from values (1, 2), (1, 2), (3, 4), (5, 6) t(a, b) 
intersect all 
select * from values (1, 2), (1, 2), (3, 4), (5, 7) t(a, b);

+------------+------------+
| a          | b          |
+------------+------------+
| 1          | 2          |
| 1          | 2          |
| 3          | 4          |
+------------+------------+
对两个查询结果取交集并去重
select * from values (1, 2), (1, 2), (3, 4), (5, 6) t(a, b) 
intersect distinct 
select * from values (1, 2), (1, 2), (3, 4), (5, 7) t(a, b);
--等效于如下语句。
select distinct * from 
(select * from values (1, 2), (1, 2), (3, 4), (5, 6) t(a, b) 
intersect all 
select * from values (1, 2), (1, 2), (3, 4), (5, 7) t(a, b)) t;

+------------+------------+
| a          | b          |
+------------+------------+
| 1          | 2          |
| 3          | 4          |
+------------+------------+
```

- 并集

```text
-- 取并集不去重。
select_statement1 union all <select_statement2>;
-- 取并集并去重。
<select_statement1> union [distinct] <select_statement2>;
-- 存在多个union all时，支持通过括号指定union all的优先级。
-- union后如果有cluster by、distribute by、sort by、order by或limit子句时，如果设置set odps.sql.type.system.odps2=false;，其作用于union的最后一个select_statement；如果设置set odps.sql.type.system.odps2=true;时，作用于前面所有union的结果。

-- 对两个数据集取并集，不去重。
select * from values (1, 2), (1, 2), (3, 4) t(a, b)
union all
select * from values (1, 2), (1, 4) t(a, b);
+------------+------------+
| a          | b          |
+------------+------------+
| 1          | 2          |
| 1          | 2          |
| 3          | 4          |
| 1          | 2          |
| 1          | 4          |
+------------+------------+
-- 对两个数据集取并集并去重
select * from values (1, 2), (1, 2), (3, 4) t(a, b)
union distinct 
select * from values (1, 2), (1, 4) t(a, b);
-- 等效于如下语句。
select distinct * from (
select * from values (1, 2), (1, 2), (3, 4) t(a, b) 
union all 
select * from values (1, 2), (1, 4) t(a, b));
+------------+------------+
| a          | b          |
+------------+------------+
| 1          | 2          |
| 1          | 4          |
| 3          | 4          |
+------------+------------+
-- 通过括号指定union all的优先级
select * from values (1, 2), (1, 2), (5, 6) t(a, b)
union all
(select * from values (1, 2), (1, 2), (3, 4) t(a, b)
union all
select * from values (1, 2), (1, 4) t(a, b));
+------------+------------+
| a          | b          |
+------------+------------+
| 1          | 2          |
| 1          | 2          |
| 5          | 6          |
| 1          | 2          |
| 1          | 2          |
| 3          | 4          |
| 1          | 2          |
| 1          | 4          |
+------------+------------+

```
- 补集

```text
--取补集不去重。
<select_statement1> except all <select_statement2>;
<select_statement1> minus all <select_statement2>;
--取补集并去重。
<select_statement1> except [distinct] <select_statement2>;
<select_statement1> minus [distinct] <select_statement2>;
-- 求数据集的补集，不去重。
select * from values (1, 2), (1, 2), (3, 4), (3, 4), (5, 6), (7, 8) t(a, b)
except all
select * from values (3, 4), (5, 6), (5, 6), (9, 10) t(a, b);
-- 等效于如下语句。
select * from values (1, 2), (1, 2), (3, 4), (3, 4), (5, 6), (7, 8) t(a, b)
minus all 
select * from values (3, 4), (5, 6), (5, 6), (9, 10) t(a, b);

+------------+------------+
| a          | b          |
+------------+------------+
| 1          | 2          |
| 1          | 2          |
| 3          | 4          |
| 7          | 8          |
+------------+------------+

求数据集的补集并去重

select * from values (1, 2), (1, 2), (3, 4), (3, 4), (5, 6), (7, 8) t(a, b)
except distinct 
select * from values (3, 4), (5, 6), (5, 6), (9, 10) t(a, b);
等效于如下语句。
select * from values (1, 2), (1, 2), (3, 4), (3, 4), (5, 6), (7, 8) t(a, b)
minus distinct 
select * from values (3, 4), (5, 6), (5, 6), (9, 10) t(a, b);
等效于如下语句。
select distinct * from values (1, 2), (1, 2), (3, 4), (3, 4), (5, 6), (7, 8) t(a, b) except all select * from values (3, 4), (5, 6), (5, 6), (9, 10) t(a, b);

+------------+------------+
| a          | b          |
+------------+------------+
| 1          | 2          |
| 7          | 8          |
+------------+------------+
```
###### 只合并新增数据到全量表

```
left anti join相当于not in，增量not in全量,过滤后只剩下完全新增的id，对全量中已有的id不修改：

查询完全新增的id
select * from a left anti join b on a.id=b.id ;
结果如下
+------------+------+
| id         | name |
+------------+------+
| 7          | 777  |
+------------+------+
完全新增的合并全量表
select * from  a --增量表
left anti join b on a.id=b.id  
union all 
select * from b  --全量表
--结果如下
+------------+------+
| id         | name |
+------------+------+
| 1          |      |
| 2          | 222  |
| 3          | 333  |
| 4          | 444  |
| 7          | 777  |
+------------+------+
```
###### 合并新增数据到全量表，且更新历史数据
```
全量not in增量,过滤后只剩下历史的id，然后union all增量，既新增也修改

查询历史全量数据
select * from b left anti join a on a.id=b.id;
结果如下
+------------+------+
| id         | name |
+------------+------+
| 3          | 333  |
| 4          | 444  |
+------------+------+
合并新增数据到全量表，且更新历史数据
select * from  b --全量表
left anti join a on a.id=b.id
union all 
select * from a ; --增量表
--结果如下
+------------+------+
| id         | name |
+------------+------+
| 1          | 111  |
| 2          | two  |
| 7          | 777  |
| 3          | 333  |
| 4          | 444  |
+------------+------+
```
###### 判断是否为中文
```
select '我' rlike '[\\x{4e00}-\\x{9fa5}]+'
```
###### Merge Into
命令格式
```sql
merge into <target_table> as <alias_name_t> using <source expression|table_name> as <alias_name_s>
--从on开始对源表和目标表的数据进行关联判断。
on <boolean expression1>
--when matched…then指定on的结果为True的行为。多个when matched…then之间的数据无交集。
when matched [and <boolean expression2>] then update set <set_clause_list>
when matched [and <boolean expression3>] then delete 
--when not matched…then指定on的结果为False的行为。
when not matched [and <boolean expression4>] then insert values <value_list>
```

```
target_table：必填。目标表名称，必须是实际存在的表。
- alias_name_t：必填。目标表的别名。
- source expression|table_name：必填。关联的源表名称、视图或子查询。
- alias_name_s：必填。关联的源表、视图或子查询的别名。
- boolean expression1：必填。BOOLEAN类型判断条件，判断结果必须为True或False。
- boolean expression2、boolean expression3、boolean  - expression4：可选。update、delete、insert操作相应的BOOLEAN类型判断条件。需要注意的是
1. 当出现三个WHEN子句时，update、delete、insert都只能出现一次。
2. 如果update和delete同时出现，出现在前的操作必须包括[and <boolean expression>]。
3. when not matched只能出现在最后一个WHEN子句中，并且只支持insert操作。
4.  set_clause_list：当出现update操作时必填。待更新数据信息。更多update信息，请参见更新数据（UPDATE）。
5. value_list：当出现insert操作时必填。待插入数据信息。更多values信息，请参见VALUES。

- 使用示例

创建目标表acid_address_book_base1及源表exists tmp_table1，并插入数据。执行merge into操作，对符合on条件的数据用源表的数据对目标表进行更新操作，对不符合on条件并且源表中满足event_type为I的数据插入目标表。命令示例如下：
```

```
--创建目标表acid_address_book_base1。
create table if not exists acid_address_book_base1 
(id bigint,first_name string,last_name string,phone string) 
partitioned by(year string, month string, day string, hour string) 
tblproperties ("transactional"="true"); 

--创建源表exists tmp_table1。
create table if not exists tmp_table1 
(id bigint, first_name string, last_name string, phone string, _event_type_ string);

--向目标表acid_address_book_base1插入测试数据。
insert overwrite table acid_address_book_base1 
partition(year='2020', month='08', day='20', hour='16') 
values (4, 'nihaho', 'li', '222'), (5, 'tahao', 'ha', '333'), 
(7, 'djh', 'hahh', '555');

--向源表exists tmp_table1插入测试数据。
insert overwrite table tmp_table1 values 
(1, 'hh', 'liu', '999', 'I'), (2, 'cc', 'zhang', '888', 'I'),
(3, 'cy', 'zhang', '666', 'I'),(4, 'hh', 'liu', '999', 'U'),
(5, 'cc', 'zhang', '888', 'U'),(6, 'cy', 'zhang', '666', 'U');

--执行merge into操作。
merge into acid_address_book_base1 as t using tmp_table1 as s 
on s.id = t.id and t.year='2020' and t.month='08' and t.day='20' and t.hour='16' 
when matched then update set t.first_name = s.first_name, t.last_name = s.last_name, t.phone = s.phone 
when not matched and (s._event_type_='I') then insert values(s.id, s.first_name, s.last_name,s.phone,'2020','08','20','16');

--查询目标表的数据确认merge into操作结果。
select * from acid_address_book_base1;

+------------+------------+------------+------------+------------+------------+------------+------------+
| id         | first_name | last_name  | phone      | year       | month      | day        | hour       |
+------------+------------+------------+------------+------------+------------+------------+------------+
| 4          | hh         | liu        | 999        | 2020       | 08         | 20         | 16         |
| 5          | cc         | zhang      | 888        | 2020       | 08         | 20         | 16         |
| 7          | djh        | hahh       | 555        | 2020       | 08         | 20         | 16         |
| 1          | hh         | liu        | 999        | 2020       | 08         | 20         | 16         |
| 2          | cc         | zhang      | 888        | 2020       | 08         | 20         | 16         |
| 3          | cy         | zhang      | 666        | 2020       | 08         | 20         | 16         |
+------------+------------+------------+------------+------------+------------+------------+------------+
```
###### 计算上下两条记录的时间差
```
在mysql，数据如下：
#查询某一用户该日抽奖时间
select draw_time from user_draw_log where user_id = 1 and draw_date='2016-03-09' order by id;
+---------------------+
| draw_time           |
+---------------------+
| 2016-03-09 13:52:46 |
| 2016-03-09 13:52:53 |
| 2016-03-09 13:53:01 |
| 2016-03-09 13:53:13 |
| 2016-03-09 13:53:25 |
```
想计算每次抽奖时间之间的间隔 以便判断是否是并发插入 我的方法如下使用一个临时变量记录前一次的抽奖时间
```
select draw_time, timediff(draw_time,@prev_time) diff,(@prev_time:=draw_time) from user_draw_log where user_id = 1 and draw_date='2016-03-09' order by id;
+---------------------+------------------+-------------------------+
| draw_time           | diff             | (@prev_time:=draw_time) |
+---------------------+------------------+-------------------------+
| 2016-03-09 13:52:46 | -00:08:28.000000 | 2016-03-09 13:52:46     |
| 2016-03-09 13:52:53 | 00:00:07.000000  | 2016-03-09 13:52:53     |
| 2016-03-09 13:53:01 | 00:00:08.000000  | 2016-03-09 13:53:01     |
| 2016-03-09 13:53:13 | 00:00:12.000000  | 2016-03-09 13:53:13     |
| 2016-03-09 13:53:25 | 00:00:12.000000  | 2016-03-09 13:53:25     |
| 2016-03-09 13:53:32 | 00:00:07.000000  | 2016-03-09 13:53:32     |
| 2016-03-09 13:53:38 | 00:00:06.000000  | 2016-03-09 13:53:38     |
...
有没更方便的方法实现这一功能呢？对所有用户都求相邻记录时间差该如何操作？

hive做法如下：

1.Hive row_number() 函数的高级用法 row_num 按照某个字段分区显示第几条数据

select imei,ts,fuel_instant,gps_longitude,gps_latitude,row_number() over (PARTITION BY imei ORDER BY ts ASC) as row_num from sample_data_2

2.row_num 是相互连续的，join 自身，然后时间相减可求差
create table obd_20140101 as

　　select a.imei,a.row_num,a.ts,COALESCE(unix_timestamp(a.ts, 'yyyy-MM-dd HH:mm:ss.S'), 0) - unix_timestamp(b.ts, 'yyyy-MM-dd HH:mm:ss.S') as intervel ,a.fuel_instant,a.gps_speed as obd_speed,a.gps_status,a.gps_longitude,a.gps_latitude,a.direct_angle,a.obdspeed from obddata_20140101 a join obddata_20140101 b on a.imei = b.imei and a.row_num = b.row_num +1

事实上该方法有更加简便的方法，那就是hive的分析窗口函数：

create table obd_20140101 as

select imei,ts as ts1,fuel_instant,gps_longitude,gps_latitude,lead(ts,1,ts) over  (PARTITION BY imei ORDER BY ts ASC)  as ts2 from sample_data_2;

这样，数据会按imei分组，并按时间排序。接下来的时间相减就简单了。

select a.imei,a.row_num,a.ts,COALESCE(unix_timestamp(a.ts1, 'yyyy-MM-dd HH:mm:ss.S'), 0) - unix_timestamp(a.ts2, 'yyyy-MM-dd HH:mm:ss.S') as intervel ,a.fuel_instant,a.gps_speed as obd_speed,a.gps_status,a.gps_longitude,a.gps_latitude,a.direct_angle,a.obdspeed from obddata_20140101 a;

```
#### 使用炸裂函数lateral view explode统计最近1天7天30天的数据
使用炸裂函数生成1天7天30天3组数据并按 recent_days 进行分组
```sql
insert overwrite table ads_traffic_stats_by_channel
select * from ads_traffic_stats_by_channel
union
select
    '2020-06-14' dt,
    recent_days,
    channel,
    cast(count(distinct(mid_id)) as bigint) uv_count,
    cast(avg(during_time_1d)/1000 as bigint) avg_duration_sec,
    cast(avg(page_count_1d) as bigint) avg_page_count,
    cast(count(*) as bigint) sv_count,
    cast(sum(if(page_count_1d=1,1,0))/count(*) as decimal(16,2)) bounce_rate
from dws_traffic_session_page_view_1d lateral view explode(array(1,7,30)) tmp as recent_days
where dt>=date_add('2020-06-14',-recent_days+1)
group by recent_days,channel;
```
