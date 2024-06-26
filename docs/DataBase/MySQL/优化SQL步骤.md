## 查看SQL执行频率

MySQL 客户端连接成功后，通过 `show [session|global] status` 命令可以提供服务器状态信息。

show [session|global] status 可以根据需要加上参数“session”或者“global”来显示 session 级（当前连接）的计结果和 global 级（自数据库上次启动至今）的统计结果。如果不写，默认使用参数是“session”。

下面的命令显示了当前 session 中所有统计参数的值：
```sql
show status like 'Com_______';
```

Com_xxx 表示每个 xxx 语句执行的次数
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1670071026977.png)

```sql
show status like 'Innodb_rows_%';
```

我们通常比较关心的是以下几个统计参数

| 参数 | 含义 |
| --- | --- |
| Com\_select | 执行 select 操作的次数，一次查询只累加 1。 |
| Com\_insert | 执行 INSERT 操作的次数，对于批量插入的 INSERT 操作，只累加一次。 |
| Com\_update | 执行 UPDATE 操作的次数。 |
| Com\_delete | 执行 DELETE 操作的次数。 |
| Innodb\_rows\_read | select 查询返回的行数。 |
| Innodb\_rows\_inserted | 执行 INSERT 操作插入的行数。 |
| Innodb\_rows\_updated | 执行 UPDATE 操作更新的行数。 |
| Innodb\_rows\_deleted | 执行 DELETE 操作删除的行数。 |
| Connections | 试图连接 MySQL 服务器的次数。 |
| Uptime | 服务器工作时间。 |
| Slow\_queries | 慢查询的次数。 |

Com\_\*\*\* : 这些参数对于所有存储引擎的表操作都会进行累计。

Innodb\_\*\*\* : 这几个参数只是针对InnoDB 存储引擎的，累加的算法也略有不同。

## 定位低效率执行SQL
-   慢查询日志
    
    通过慢查询日志定位那些执行效率较低的 SQL 语句，用 log-slow-queries[file_name] 选项启动时，mysqld 写一个包含所有执行时间超过 long_query_time 秒的 SQL 语句的日志文件。具体可以查看日志管理的相关部分。
-   show processlist
    
    慢查询日志在查询结束以后才纪录，所以在应用反映执行效率出现问题的时候查询慢查询日志并不能定位问题，可以使用 `show processlist` 命令查看当前MySQL在进行的线程，包括线程的状态、是否锁表等，可以实时地查看 SQL 的执行情况，同时对一些锁表操作进行优化。
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/utools_1556098544349.png)

1） id列，用户登录mysql时，系统分配的"connection_id"，可以使用函数connection_id()查看

2） user列，显示当前用户。如果不是root，这个命令就只显示用户权限范围的sql语句

3） host列，显示这个语句是从哪个ip的哪个端口上发的，可以用来跟踪出现问题语句的用户

4） db列，显示这个进程目前连接的是哪个数据库

5） command列，显示当前连接的执行的命令，一般取值为休眠（sleep），查询（query），连接（connect）等

6） time列，显示这个状态持续的时间，单位是秒

7） state列，显示使用当前连接的sql语句的状态，很重要的列。state描述的是语句执行中的某一个状态。一个sql语句，以查询为例，可能需要经过copying to tmp table、sorting result、sending data等状态才可以完成

8） info列，显示这个sql语句，是判断问题语句的一个重要依据

## explain分析执行计划

通过 EXPLAIN 或者 DESC命令获取 MySQL 如何执行 SELECT 语句的信息，包括在 SELECT 语句执行过程中表如何连接和连接的顺序。

```sql
explain  select * from tb_item where id = 1;
```


![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/1552487489859.png)

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/1552487526919.png)

| 字段           | 含义                                                                                                                                                                                             |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| id             | select查询的序列号，是一组数字，表示的是查询中执行select子句或者是操作表的顺序。                                                                                                                 |
| select\_type   | 表示 SELECT 的类型，常见的取值有 SIMPLE（简单表，即不使用表连接或者子查询）、PRIMARY（主查询，即外层的查询）、UNION（UNION 中的第二个或者后面的查询语句）、SUBQUERY（子查询中的第一个 SELECT）等 |
| table          | 输出结果集的表                                                                                                                                                                                   |
| type           | 表示表的连接类型，性能由好到差的连接类型为( system ---> const -----> eq\_ref ------> ref -------> ref\_or\_null----> index\_merge ---> index\_subquery -----> range -----> index ------> all )   |
| possible\_keys | 表示查询时，可能使用的索引                                                                                                                                                                       |
| key            | 表示实际使用的索引                                                                                                                                                                               |
| key\_len       | 索引字段的长度                                                                                                                                                                                   |
| rows           | 扫描行的数量                                                                                                                                                                                     |
| extra          | 执行情况的说明和描述                                                                                                                                                                             |

### 环境准备

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/1556122799330.png)

```sql
CREATE TABLE `t_role` (
  `id` VARCHAR(32) NOT NULL,
  `role_name` VARCHAR(255) DEFAULT NULL,
  `role_code` VARCHAR(255) DEFAULT NULL,
  `description` VARCHAR(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `unique_role_name` (`role_name`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;

CREATE TABLE `t_user` (
  `id` VARCHAR(32) NOT NULL,
  `username` VARCHAR(45) NOT NULL,
  `password` VARCHAR(96) NOT NULL,
  `name` VARCHAR(45) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `unique_user_username` (`username`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;

CREATE TABLE `user_role` (
  `id` INT(11) NOT NULL AUTO_INCREMENT ,
  `user_id` VARCHAR(32) DEFAULT NULL,
  `role_id` VARCHAR(32) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fk_ur_user_id` (`user_id`),
  KEY `fk_ur_role_id` (`role_id`),
  CONSTRAINT `fk_ur_role_id` FOREIGN KEY (`role_id`) REFERENCES `t_role` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION,
  CONSTRAINT `fk_ur_user_id` FOREIGN KEY (`user_id`) REFERENCES `t_user` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION
) ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `t_user` (`id`, `username`, `password`, `name`) VALUES('1','super','$2a$10$TJ4TmCdK.X4wv/tCqHW14.w70U3CC33CeVncD3SLmyMXMknstqKRe','超级管理员');
INSERT INTO `t_user` (`id`, `username`, `password`, `name`) VALUES('2','admin','$2a$10$TJ4TmCdK.X4wv/tCqHW14.w70U3CC33CeVncD3SLmyMXMknstqKRe','系统管理员');
INSERT INTO `t_user` (`id`, `username`, `password`, `name`) VALUES('3','itcast','$2a$10$8qmaHgUFUAmPR5pOuWhYWOr291WJYjHelUlYn07k5ELF8ZCrW0Cui','test02');
INSERT INTO `t_user` (`id`, `username`, `password`, `name`) VALUES('4','stu1','$2a$10$pLtt2KDAFpwTWLjNsmTEi.oU1yOZyIn9XkziK/y/spH5rftCpUMZa','学生1');
INSERT INTO `t_user` (`id`, `username`, `password`, `name`) VALUES('5','stu2','$2a$10$nxPKkYSez7uz2YQYUnwhR.z57km3yqKn3Hr/p1FR6ZKgc18u.Tvqm','学生2');
INSERT INTO `t_user` (`id`, `username`, `password`, `name`) VALUES('6','t1','$2a$10$TJ4TmCdK.X4wv/tCqHW14.w70U3CC33CeVncD3SLmyMXMknstqKRe','老师1');

INSERT INTO `t_role` (`id`, `role_name`, `role_code`, `description`) VALUES('5','学生','student','学生');
INSERT INTO `t_role` (`id`, `role_name`, `role_code`, `description`) VALUES('7','老师','teacher','老师');
INSERT INTO `t_role` (`id`, `role_name`, `role_code`, `description`) VALUES('8','教学管理员','teachmanager','教学管理员');
INSERT INTO `t_role` (`id`, `role_name`, `role_code`, `description`) VALUES('9','管理员','admin','管理员');
INSERT INTO `t_role` (`id`, `role_name`, `role_code`, `description`) VALUES('10','超级管理员','super','超级管理员');

INSERT INTO user_role(id,user_id,role_id) VALUES(NULL, '1', '5'),(NULL, '1', '7'),(NULL, '2', '8'),(NULL, '3', '9'),(NULL, '4', '8'),(NULL, '5', '10') ;

```

### explain 之 id

id 字段是 select查询的序列号，是一组数字，表示的是查询中执行select子句或者是操作表的顺序。id 情况有三种

1. id 相同表示加载表的顺序是从上到下。
```sql
explain select * from t_role r, t_user u, user_role ur where r.id = ur.role_id and u.id = ur.user_id ;
```

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/1556102471304.png)

2.  id 不同id值越大，优先级越高，越先被执行。
```sql
EXPLAIN SELECT * FROM t_role WHERE id = (SELECT role_id FROM user_role WHERE user_id = (SELECT id FROM t_user WHERE username = 'stu1'))
```

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/1556103009534.png)

3. id 有相同，也有不同，同时存在。id相同的可以认为是一组，从上往下顺序执行；在所有的组中，id的值越大，优先级越高，越先执行。
```sql
EXPLAIN SELECT * FROM t_role r , (SELECT * FROM user_role ur WHERE ur.`user_id` = '2') a WHERE r.id = a.role_id ; 
```

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/1556103294182.png)

### explain 之 select_type
表示 SELECT 的类型，常见的取值，如下表所示：

| select\_type | 含义                                                                                                             |
| ------------ | ---------------------------------------------------------------------------------------------------------------- |
| SIMPLE       | 简单的select查询，查询中不包含子查询或者UNION                                                                    |
| PRIMARY      | 查询中若包含任何复杂的子查询，最外层查询标记为该标识                                                             |
| SUBQUERY     | 在SELECT 或 WHERE 列表中包含了子查询                                                                             |
| DERIVED      | 在FROM 列表中包含的子查询，被标记为 DERIVED（衍生） MYSQL会递归执行这些子查询，把结果放在临时表中                |
| UNION        | 若第二个SELECT出现在UNION之后，则标记为UNION ； 若UNION包含在FROM子句的子查询中，外层SELECT将被标记为 ： DERIVED |
| UNION RESULT | 从UNION表获取结果的SELECT                                                                                        |

- SIMPLE

SIMPLE 代表单表查询

```sql
EXPLAIN SELECT * FROM t_user;
```

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/image-20210522183120093.png)

- PRIMARY、SUBQUERY

在 SELECT 或 WHERE 列表中包含了子查询。最外层查询则被标记为 Primary。
```sql
explain select * from t_user where id = (select id from user_role where role_id='9' );
```

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/image-20210522183137975.png)

- DERIVED

在 FROM 列表中包含的子查询被标记为 DERIVED(衍生)，MySQL 会递归执行这些子查询, 把结果放在临时表里。
```sql
explain select a.* from (select * from t_user where id in('1','2') ) a;
```

mysql 5.7 中为 `simple`
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/mysql_explan-20210522180244911.png)

mysql 5.6 中：
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/mysql_image-20210522182540547.png)

- union
```sql
explain select * from t_user where id='1' union select * from  t_user where id='2';
```
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/mysql_explanimage-20210522183718254.png)

### explain 之 table

展示这一行的数据是关于哪一张表的

没有与之关系的表为 NULL
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/mysql_explan_image-20210522190653729.png)

###  explain 之 type
type 显示的是访问类型，是较为重要的一个指标，可取值为：

| type | 含义 |
| --- | --- |
| NULL | MySQL不访问任何表，索引，直接返回结果 |
| system | 表只有一行记录(等于系统表)，这是const类型的特例，一般不会出现 |
| const | 表示通过索引一次就找到了，const 用于比较primary key 或者 unique 索引。因为只匹配一行数据，所以很快。如将主键置于where列表中，MySQL 就能将该查询转换为一个常亮。const于将 "主键" 或 "唯一" 索引的所有部分与常量值进行比较 |
| eq\_ref | 类似ref，区别在于使用的是唯一索引，使用主键的关联查询，关联查询出的记录只有一条。常见于主键或唯一索引扫描 |
| ref | 非唯一性索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，返回所有匹配某个单独值的所有行（多个） |
| range | 只检索给定返回的行，使用一个索引来选择行。 where 之后出现 between ， < , > , in 等操作。 |
| index | index 与 ALL的区别为 index 类型只是遍历了索引树， 通常比ALL 快， ALL 是遍历数据文件。 |
| all | 将遍历全表以找到匹配的行 |

结果值从最好到最坏以此是：

-   NULL > system > const > eq\_ref > ref > fulltext > ref\_or\_null > index\_merge > unique\_subquery > index\_subquery > range > index > ALL
    
-   system > const > eq\_ref > ref > range > index > ALL

**一般来说， 我们需要保证查询至少达到 range 级别， 最好达到ref**

### explain 之 key
-   possible_keys :
    
    显示可能应用在这张表的索引， 一个或多个。
    
-   key
    
    实际使用的索引， 如果为NULL， 则没有使用索引。
    
-   key_len
    
    表示索引中使用的字节数， 该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前提下， 长度越短越好

### explain 之 rows

扫描行的数量。

### explain 之 extra

其他的额外的执行计划信息，在该列展示 。

| extra | 含义 |
| --- | --- |
| using filesort | 说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取， 称为 “文件排序”, 效率低。 |
| using temporary | 使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于 order by 和 group by； 效率低 |
| using index | 表示相应的select操作使用了覆盖索引， 避免访问表的数据行， 效率不错。 |

### show profile分析SQL

Mysql从5.0.37版本开始增加了对 show profiles 和 show profile 语句的支持。

show profiles 能够在做SQL优化时帮助我们了解时间都耗费到哪里去了。

通过 have_profiling 参数，能够看到当前MySQL是否支持profile：
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/mysql_profile_1552488401999.png)

默认profiling是关闭的，可以通过set语句在Session级别开启profiling：
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/mysql_profile1552488372405.png)

```sql
set profiling=1; //开启profiling 开关；
```
通过profile，我们能够更清楚地了解SQL执行的过程。

我们可以执行一系列的操作：
```sql
show databases;

use db01;

show tables;

select * from tb_item where id < 5;

select count(*) from tb_item;
```

执行完上述命令之后，再执行 `show profiles` 指令， 来查看SQL语句执行的耗时：
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/mysql_profile_1552489017940.png)

通过 `show profile for query query_id` 语句查看该SQL执行过程中每个线程的状态和消耗的时间
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/mysql_profile_1552489053763.png)

>[!TIP]
>Sending data 状态表示 MySQL 线程开始访问数据行并把结果返回给客户端，而不仅仅是返回个客户端。由于在 Sending data 状态下，MySQL 线程往往需要做大量的磁盘读取操作，所以经常是整各查询中耗时最长的状态。

在获取到最消耗时间的线程状态后，MySQL支持进一步选择all、cpu、block io 、context switch、page faults等明细类型类查看MySQL在使用什么资源上耗费了过高的时间。例如，选择查看CPU的耗费时间 ：
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/mysql_profile_1552489671119.png)

| 字段 | 含义 |
| --- | --- |
| Status | sql 语句执行的状态 |
| Duration | sql 执行过程中每一个步骤的耗时 |
| CPU\_user | 当前用户占有的cpu |
| CPU\_system | 系统占有的cpu |

