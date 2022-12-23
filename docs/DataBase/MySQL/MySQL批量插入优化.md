## Mybatis批量插入执行优化



### 一. 使用Mybatis foreach插入
```xml
<insert id="insertBatch">
	INSERT INTO  `demo`(  `id`, `name`,
	`key_word`,
	`punch_time`,
	 `salary_money`,
	 `bonus_money`,
	 `sex`, `age`, `birthday`,
	  `email`, `content`)
	VALUES
	<foreach collection ="list" item="demo" separator =",">
		(  #{demo.id},  #{demo.name}, #{demo.keyWord},
		 #{demo.punchTime},  #{demo.salaryMoney},  #{demo.bonusMoney},
		 #{demo.sex},  #{demo.age},  #{demo.birthday},
		  #{demo.email},  #{demo.content}  )
	</foreach >
</insert>
```

这种方式对插入的数据量过大还是会有很大的耗时，直接改用后两种实现方式

You can also read more about insert speed in the [MySQL Docs](http://dev.mysql.com/doc/refman/5.6/en/insert-speed.html). It clearly describs the following.

>To optimize insert speed, combine many small operations into a single large operation. Ideally, you make a single connection, send the data for many new rows at once, and delay all index updates and consistency checking until the very end.

在[Stack Overflow](https://stackoverflow.com/questions/19682414/how-can-mysql-insert-millions-records-faster) 中提到
Of course don't combine ALL of them, if the amount is HUGE. Say you have 1000 rows you need to insert, then don't do it one at a time. But you probably shouldn't equally try to have all 1000 rows in a single query. Instead break it into smaller sizes.

当插入数量很多时，不能一次性全放在一条语句里。[资料](https://stackoverflow.com/questions/32649759/using-foreach-to-do-batch-insert-with-mybatis/40608353)：

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1671796434258.png)

`Mybatis`的三种执行器

```java
public enum ExecutorType {
  SIMPLE, REUSE, BATCH
}
```

- **ExecutorType.SIMPLE**: 这个执行器类型不做特殊的事情。它为每个语句的执行创建一个新的预处理语句。
- **ExecutorType.REUSE**: 这个执行器类型会复用预处理语句。
- **ExecutorType.BATCH**:这个执行器会批量执行所有更新语句,如果 SELECT 在它们中间执行还会标定它们是 必须的,来保证一个简单并易于理解的行为。

`SqlSessionTemplate` 源码中使用的是 `SIMPLE`
```java
public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
this(sqlSessionFactory, sqlSessionFactory.getConfiguration().getDefaultExecutorType());
}
protected ExecutorType defaultExecutorType = ExecutorType.SIMPLE;
```

Simple执行器会为每个语句创建一个新的预处理语句，也就是创建一个`PreparedStatement`对象。在我们的项目中，会不停地使用批量插入这个方法，而因为MyBatis对于含有`<foreach>`的语句，无法采用缓存，那么在每次调用方法时，都会重新解析sql语句。

#### 参考文章
- [Mybatis 大数据量的批量insert解决方案_小灯光环的博客-CSDN博客_数据量太大 insert](https://blog.csdn.net/wlwlwlwl015/article/details/50246717)
- [MyBatis批量插入几千条数据慎用foreach_huanghanqian的博客-CSDN博客_mybatis批量插入100条](https://blog.csdn.net/huanghanqian/article/details/83177178)

### 二. 修改mybatis 执行器

```java
@Test
public void testMybatisInsertSqlSessionBatchSave() {
	List<Demo> jeecgDemoList = initDemos();
	System.out.println(("----- testMybatisInsert100000SqlSessionBatchSave method test ------start:" + System.currentTimeMillis()));
	SqlSession sqlSession = sqlSessionTemplate.getSqlSessionFactory().openSession(ExecutorType.BATCH.BATCH, false);
	DemoMapper demoMapper = sqlSession.getMapper(DemoMapper.class);
	DemoMapper.forEach(jeecgDemo -> {
		DemoMapper.insert(jeecgDemo);
	});
	sqlSession.commit();
	System.out.println(("----- testMybatisInsert100000SqlSessionBatchSave method test ------end: " + System.currentTimeMillis()));
}
```

### 三. Mybatis 官网推荐批量插入方式

[MyBatis Dynamic SQL – Insert Statements](https://mybatis.org/mybatis-dynamic-sql/docs/insert.html)
```java
SimpleTableRecord row = new SimpleTableRecord();
row.setId(100);
row.setFirstName("Joe");
row.setLastName("Jones");
row.setBirthDate(new Date());
row.setEmployed(true);
row.setOccupation("Developer");

InsertStatementProvider<SimpleTableRecord> insertStatement = insert(row)
		.into(simpleTable)
		.map(id).toProperty("id")
		.map(firstName).toProperty("firstName")
		.map(lastName).toProperty("lastName")
		.map(birthDate).toProperty("birthDate")
		.map(employed).toProperty("employed")
		.map(occupation).toProperty("occupation")
		.build()
		.render(RenderingStrategies.MYBATIS3);

int rows = mapper.insert(insertStatement);
```

## SpringJDBC方式批量保存

```java
@Test
public void testJdbcInsertBatchSave() {
	List<Object[]> demoList = initJDBCDemos();
	DruidDataSource dataSource = DynamicDBUtil.getDbSourceByDbKey("master");
	JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
	System.out.println(("----- testJdbcInsert100000BatchSave method test ------start:" + System.currentTimeMillis()));
	String sql ="INSERT INTO  `demo`(  `id`, `name`,\n" +
			"\t\t`key_word`,\n" +
			"\t\t`punch_time`,\n" +
			"\t\t `salary_money`,\n" +
			"\t\t `bonus_money`,\n" +
			"\t\t `sex`, `age`, `birthday`,\n" +
			"\t\t  `email`, `content`)\n" +
			"\t\tVALUES (?,?,?,?,?,?,?,?,?,?,?)";

	jdbcTemplate.batchUpdate(sql,demoList);
	System.out.println(("----- testJdbcInsert100000BatchSave method test ------end: " + System.currentTimeMillis()));
}
```

## SpringDataJpa

SpringDataJpa 调用saveAll方法默认也是一条一条的插入,可以通过开启Jpa日志统计信息和Druid执行数进行测试
```
spring.jpa.properties.hibernate.generate_statistics=true
```

1. Mysql数据库链接配置添加
```
$rewriteBatchedStatements=true
```
2.设置Jpa批量写
```
spring.jpa.properties.hibernate.jdbc.batch_size=500
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates =true
```