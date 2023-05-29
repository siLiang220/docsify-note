## 概述
`Aviator`是java实现的表达式引擎，主要用于表达式的动态求值`Aviator`是直接将表达式编译成java字节码，交给JVM执行

## 使用场景
- 规则判断
- 公式计算
- 动态脚本控制

## 基本类型
Aviator 支持类型，数字，布尔值、字符串，但没有byte/short/int等类型，统一整数类型都为long，整数也可使用十六进制表示，比如 0xFF(255)、0xAB(171)。

## 基本使用
-  公式计算
```java
public void test1() {
    Long sum = (Long)AviatorEvaluator.execute("1 + 2 + 3");
    System.out.println(sum);
}
//结果为： 6
```

- 往表达式传入值
```java
@Test
public void test3() {
	Map<String, Object> env = new HashMap<>();
	env.put("name", "ruilin.shao");
	String str = "'hello ' + name";
	String result = (String) AviatorEvaluator.execute(str, env);
	System.out.println(result);
	//结果为：hello ruilin.shao
	//写法二
	String result2 = (String)AviatorEvaluator.exec(str, "便利蜂");
	System.out.println(result2);
	//结果为：hello 便利蜂
}
```
- 自定义函数

`getName()` 返回方法名。一般来说都推荐继承 `com.googlecode.aviator.runtime.function.AbstractFunction`  ，并覆写对应参数个数的方法即可，例如上面例子中定义了 `add` 方法，它接受两个参数。`AbstractVariadicFunction` 可实可变参数函数
```java
public class TestAviator {
    public static void main(String[] args) {
            //注册函数
            AviatorEvaluator.addFunction(new AddFunction());
            System.out.println(AviatorEvaluator.execute("add(1, 2)"));           // 3.0
            System.out.println(AviatorEvaluator.execute("add(add(1, 2), 100)")); // 103.0
        }
    }
    class AddFunction extends AbstractFunction {
        @Override
        public AviatorObject call(Map<String, Object> env, 
                                  AviatorObject arg1, AviatorObject arg2) {
            Number left = FunctionUtils.getNumberValue(arg1, env);
            Number right = FunctionUtils.getNumberValue(arg2, env);
            return new AviatorDouble(left.doubleValue() + right.doubleValue());
        }
        public String getName() {
            return "add";
        }
    }
```
通过规则引擎，只需将配置的规则转换成一个规则的字符串表达式，在使用的时候只需要传递规则所需要的参数
```java
public class AviatorExampleTwo {
    //规则可以保存在数据库中，mysql或者redis等等
    Map<Integer, String> ruleMap = new HashMap<>();
    public AviatorExampleTwo() {
        //秒数计算公式
        ruleMap.put(1, "hour * 3600 + minute * 60 + second");
        //正方体体积计算公式
        ruleMap.put(2, "height * width * length");
        //判断一个人是不是资深顾客
        ruleMap.put(3, "age >= 18 && sumConsume > 2000 && vip");
        //资深顾客要求修改
        ruleMap.put(4, "age > 10 && sumConsume >= 8000 && vip && avgYearConsume >= 1000");
        //判断一个人的年龄是不是大于等于18岁
        ruleMap.put(5, "age  >= 18 ? 'yes' : 'no'");
    }
    public static void main(String[] args) {
        AviatorExampleTwo aviatorExample = new AviatorExampleTwo();
        //选择规则，传入规则所需要的参数
        System.out.println("公式1：" + aviatorExample.getResult(1, 1, 1, 1));
        System.out.println("公式2：" + aviatorExample.getResult(2, 3, 3, 3));
        System.out.println("公式3：" + aviatorExample.getResult(3, 20, 3000, false));
        System.out.println("公式4：" + aviatorExample.getResult(4, 23, 8000, true, 2000));
        System.out.println("公式5：" + aviatorExample.getResult(5, 12));
    }
    public Object getResult(int ruleId, Object... args) {
        String rule = ruleMap.get(ruleId);
        return AviatorEvaluator.exec(rule, args);
    }
}
//公式1：3661
//公式2：27
//公式3：false
//公式4：true
//公式5：no
```

- 工作中实际使用
```java
import com.googlecode.aviator.AviatorEvaluator;
import com.googlecode.aviator.AviatorEvaluatorInstance;
import com.googlecode.aviator.Options;
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.List;
import java.util.Map;
/**表达式工具类*/
public class ExpressionUtil {
    public static AviatorEvaluatorInstance aviatorEvaluator = AviatorEvaluator.getInstance();
    static {
        aviatorEvaluator.setOption(Options.ALWAYS_PARSE_FLOATING_POINT_NUMBER_INTO_DECIMAL, true);
        aviatorEvaluator.setCachedExpressionByDefault(true);
        AviatorEvaluator.addFunction(new ExpressionFunction());
    }
    /**
     * 表达式验证
     **/
    public static boolean verify(String expression) {
        try {
            aviatorEvaluator.validate(expression);
        } catch (Exception e) {
            return false;
        }
        return true;
    }
    /**
     * 获取表达式变量
     */
    public static List<String> getVariableNames(String expression) {
        return aviatorEvaluator.compile(expression).getVariableNames();
    }
    /**
     * 表达式计算
     * @param expression 表达式
     * @param params     需要替换的表达式参数
     * @return calculate result
     */
    public static BigDecimal calculate(String expression, Map<String, Object> params, int scale) {
        try {
            BigDecimal val = (BigDecimal) aviatorEvaluator.compile(expression).execute(params);
            return val.setScale(scale, RoundingMode.HALF_UP);
        } catch (ArithmeticException e) {
            return BigDecimal.ZERO;
        }
    }
    /**
     * 表达式计算
     * @param expression 表达式
     * @param params     需要替换的表达式参数
     * @return calculate result
     */
    public static String calculateAge(String expression, Map<String, Object> params) {
        String age = (String) AviatorEvaluator.execute(expression,params);
        return age;
    }
}
```

## Aviator 动态脚本
[AviatorScript 文档 ](https://www.yuque.com/boyan-avfmj/aviatorscript)

## 参考文章
[风控逻辑利器---规则引擎 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/552289273)
