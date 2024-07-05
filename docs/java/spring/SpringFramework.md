
## spring基础

https://www.wolai.com/oacbJpH1wPzGNoMAVnoELR

- [动态实现配置类注入](https://juejin.cn/post/7342279484487270450)

## SpEL表达式
### 简介
Spring Expression Language是一个支持运行时查询和操作对象图的强大表达式语言

### SpEL常见用法
-  SpEL字面量
	- 整数：#{8}
	- 小数：#{8.8}
	- 科学计数法：#{1e4}
	- String：#{'string'}
	- Boolean：#{true}
- SpEl引用bean，属性和方法
	- 引用其他对象：#{car}
	- 引用其他对象的属性：#{car.brand}
	- 调用其他方法，还可以链式操作：#{car.toString()}
	- 调用静态方法静态属性：#{T(java.lang.Math).PI}
-   SpEL支持的运算符号：  
	-   算术运算符：+，-，*，/，%，^(加号还可以用作字符串连接)
	-   比较运算符：< , > , == , >= , <= , lt , gt , eg , le , ge
	-   逻辑运算符：and , or , not , |
	-   if-else 运算符(类似三目运算符)：？:(temary), ?:(Elvis)
	-   正则表达式：#{admin.email matches ‘[a-zA-Z0-9._%±]+@[a-zA-Z0-9.-]+.[a-zA-Z]{2,4}’}‘


### 入门

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1663504494027.png)

- 字符串
```java
public void demo() {
    // 1 定义解析器
    SpelExpressionParser parser = new SpelExpressionParser();
    // 2 使用解析器解析表达式
    Expression exp = parser.parseExpression("'xxx'.concat('yyy')");
    // 3 获取解析结果
    String value = (String) exp.getValue();
    System.out.println(value);//xxxyyy
    exp = parser.parseExpression("'xxx'.bytes");
    byte[] bytes = (byte[]) exp.getValue();
    exp = parser.parseExpression("'xxx'.bytes.length");
    int length = (Integer) exp.getValue();
    System.out.println("length: " + length);//length: 3
}
```
- 对象
```java
public void demo() {
    User user = new User();
    user.setName("xxx");
    User user2 = new User();
    user2.setName(user.getName());
    // 1 定义解析器
    ExpressionParser parser = new SpelExpressionParser();
    // 指定表达式
    Expression exp = parser.parseExpression("name");
    // 2 使用解析器解析表达式，获取对象的属性值
    String name = (String) exp.getValue(user2);
    // 3 获取解析结果
    System.out.println(name);//xxx

    // 2.1 使用解析器解析表达式，获取对象的属性值并进行运算 
    Expression exp2 = parser.parseExpression("name == 'xxx'");
    // 3.1 获取解析结果
    boolean result = exp2.getValue(user2, Boolean.class);
    System.out.println(result);//true
}
```

- EvaluationContext

`EvaluationContext`可以理解为parser 在这个环境里执行`parseExpression`解析操作，比如说我们现在往ctx（一个`EvaluationContext` ）中放入一个 对象`list` (注：假设list里面已经有数据，即list[0]=true).
```java
ctx.setVariable("list" , list);//可以理解为往ctx域 里放了一个list变量
```
接下来要想获取或设置list的值都要在ctx范围内才能找到：
```java
parser.parseExpression("#list[0]").getValue(ctx);//在ctx这个环境里解析出list[0]的值
parser.parseExpression("#list[0]").setValue(ctx , "false");//在ctx这个环境中奖 list[0]设为false

```
如果在ctx 中放入一个person对象，取对象里的name值
```java
ctx.setVariable("p", person);
parser.parseExpression("#p.name").getValue(ctx)；
//也可以不使用ctx直接获取到name
parser.parseExpression("name").getValue(person);//在person上解析name属性
```


### 实际应用
author：[转转SpEL快速上手及实践 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/539163585)

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/v2-cee73e6f614615e76666b31507cd39b5_r.jpg)
具体方式：
-   定义一个注解：

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Monitor {

    /**
     * 类型
     * @return
     */
    MonitorScenesTypeEnum scenes() default MonitorScenesTypeEnum.DEFAULT;

    /**
     * 表达式
     * @return
     */
    String monitorSpEL() default "";

}
```

-   注解处理类：

```java
public void process(Monitor monitor, JoinPoint joinPoint, Object result, Throwable ex) throws ClassNotFoundException {
    MonitorScenesTypeEnum scenes = monitor.scenes();
    Integer code = scenes.getCode();
    //获取Apollo配置
    Map<Integer, MonitorConfig> monitorConfigMap = apolloConfigService.getMonitorConfigMap();
    if (MapUtils.isEmpty(monitorConfigMap) || !monitorConfigMap.containsKey(code)) {
        return;
    }
    MonitorConfig monitorConfig = monitorConfigMap.get(code);

    //当有返回值时需要校验一下结果是否符合预期
    String resultType = monitorConfig.getResultType();
    if (checkResult(result, resultType)) {
        return;
    }
    
    //获取入参的SpEL表达式进行解析
    String monitorSpEL = monitor.monitorSpEL();
    String monitorTrace = String uuid = StringUtils.isNotBlank(monitorSpEL) ? 
            SpelParseUtil.generateKeyBySpEL(monitorSpEL, joinPoint) : StringUtils.EMPTY_STRING;

    //截取一下异常信息
    String otherParamsJson = "";
    if (Objects.nonNull(ex)) {
        Map<String, String> otherParams = Maps.newHashMap();
        String stackTraceAsString = Throwables.getStackTraceAsString(ex);
        String errMsg = stackTraceAsString;
        if (stackTraceAsString.length() > NumberConstant.NUMBER_512) {
            errMsg = stackTraceAsString.substring(0,NumberConstant.NUMBER_512) + "...";
        }
        otherParams.put("异常信息", errMsg);
        otherParamsJson = JsonUtil.silentObject2String(otherParams);
    }
    // 异步发送报警信息
    asyncSendMonitor(scenes, monitorTrace, otherParamsJson);
    }
}


private static SpelExpressionParser parser = new SpelExpressionParser();
private static DefaultParameterNameDiscoverer nameDiscoverer = new DefaultParameterNameDiscoverer();

public static String generateKeyBySpEL(String spelString, JoinPoint joinPoint) {
    // 通过joinPoint获取被注解方法
    MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
    Method method = methodSignature.getMethod();
    // 使用spring的DefaultParameterNameDiscoverer获取方法形参名数组
    String[] paramNames = nameDiscoverer.getParameterNames(method);
    // 解析过后的Spring表达式对象
    Expression expression = parser.parseExpression(spelString);
    // spring的表达式上下文对象
    EvaluationContext context = new StandardEvaluationContext();
    // 通过joinPoint获取被注解方法的形参
    Object[] args = joinPoint.getArgs();
    // 给上下文赋值
    for (int i = 0; i < args.length; i++) {
        context.setVariable(paramNames[i], args[i]);
    }
    //在上下文中解析获取指定的sqel值
    return Objects.requireNonNull(expression.getValue(context)).toString();
}
```

-   具体使用：

```java
@Monitor(scenes = MonitorScenesTypeEnum.CHANGE_PRICE_FAIL, spelStr = "#request?.orderId")
public ChangePriceResponse changePrice(ChangePriceRequest request) {
    BizOrderContext<ChangePriceRequest, ChangePriceResponse> bizOrderContext = BizOrderContext.create(OrderEventEnum.C1_CHANGE_JM_PRICE, request);
    ZzAssert.isTrue(stateMachine.isCanFire(bizOrderContext), BizErrorCode.ORDER_STATUS_CHANGED);
    stateMachine.fire(bizOrderContext);
    return bizOrderContext.getResponse();
```

- 示例效果

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1663507898966.png)

## Spring State Machine(状态机)


状态机可以看做可以是表驱动的一种，其实就是当前状态和事件两者组合与处理函数的一种对应关系。当然，处理成功之后还会有一个状态转移处理。

[使用JAVA状态机实现订单状态控制功能-云社区-华为云 (huaweicloud.com)](https://bbs.huaweicloud.com/blogs/228467)

[spring statemachine 多个状态机实践_chenpei1990的博客-CSDN博客_statemachine 多状态](https://blog.csdn.net/chenpei1990/article/details/81636897)

[spring statemachine-多个状态机共存 - 简书 (jianshu.com)](https://www.jianshu.com/p/ee8ecfacf6ed)

[Spring Statemachine - Reference Documentation](https://docs.spring.io/spring-statemachine/docs/3.2.0/reference/#statemachine)

[使用枚举实现状态机来优雅你的状态变更逻辑-云社区-华为云 (huaweicloud.com)](https://bbs.huaweicloud.com/blogs/344617)

[状态机引擎在vivo营销自动化中的深度实践](https://zhuanlan.zhihu.com/p/496396933)

---
## 事务的传播特性

### 什么是事务的传播特性?
A,B两个方法都有事务，在A方法中去调用B方法，此时事务方法应该如何进行处理。
比如 A方法里面调用B方法。
情况一：A方法出现异常，A方法需要回滚，那么B方法需不需要回滚呢？
情况二：B方法出现异常，B方法需要回滚，那么A方法需不需要回滚呢？


### spring总共给出了7种事务隔离级别:
1. PROPAGATION_REQUIRED:默认事务类型，如果没有，就新建一个事务;如果有，就加入当前事务。适合绝大多数情况。

解释：方法A中调用方法B，B方法的事务传播设置为PROPAGATION_REQUIRED，则如果A方法上有事务，B就用A的事务，没有的话，B就自己新建一个事务。
2. PROPAGATION_REQUIRES_NEW:如果没有，就新建一个事务;如果有，就将当前事务挂起。

解释：方法A中调用方法B，B方法的事务传播设置为PROPAGATION_REQUIRES_NEW，则如果A方法上有事务，则将A方法的事务挂起，B用自己的事务。A方法没有事务的话，B自己就新建一个事务。
3. PROPAGATION_NESTED:如果没有，就新建一个事务;如果有，就在当前事务中嵌套其他事务。

解释：方法A中调用方法B，B方法的事务传播设置为PROPAGATION_NESTED，则如果A方法上有事务，则B方法的事务嵌套在A方法事务中。A方法没有事务的话，B自己就新建一个事务。
嵌套导致的结果是 如果A回滚了，会导致B回滚，B回滚，则不会导致A回滚。
4. PROPAGATION_SUPPORTS:如果没有，就以非事务方式执行;如果有，就使用当前事务。

解释：方法A中调用方法B，B方法的事务传播设置为PROPAGATION_SUPPORTS，则如果A方法上有事务，B就用A的事务，没有的话，B就以非事务方式执行。
5. PROPAGATION_NOT_SUPPORTED:如果没有，就以非事务方式执行;如果有，就将当前事务挂起。即无论如何不支持事务。

解释：方法A中调用方法B，B方法的事务传播设置为PROPAGATION_NOT_SUPPORTED，则如果A方法上有事务，则将A方法的事务挂起(因为A,B方法是在同一个线程中执行的)以非事务方式执行，以非事务方式执行。A没有事务，就以非事务方式执行.
6. PROPAGATION_NEVER:如果没有，就以非事务方式执行;如果有，就抛出异常。

解释：方法A中调用方法B，B方法的事务传播设置为PROPAGATION_NEVER，则要求A方法上不能有事务，有的话，就报错。A没有事务，就以非事务方式执行，B始终是以非事务方式执行。
7. PROPAGATION_MANDATORY:如果没有，就抛出异常;如果有，就使用当前事务。

解释：方法A中调用方法B，B方法的事务传播设置为PROPAGATION_MANDATORY，则要求A方法上有事务，有的话，B用A的事务。A没有的话，就报错。