
## 1.为什么要用starter

-   现在我们就来回忆一下，在还没有Spring-boot框架的时候，我们使用Spring 开发项目，如果需要某一个框架，例如mybatis，我们的步骤一般都是：
-   到maven仓库去找需要引入的mybatis jar包，选取合适的版本（易发生冲突）
-   到maven仓库去找mybatis-spring整合的jar包，选取合适的版本（易发生冲突）
-   在spring的applicationContext.xml文件中配置dataSource和mybatis相关信息
-   假如所有工作都到位，一般可以一气呵成；但很多时候都会花一堆时间解决jar 冲突，配置项缺失，导致怎么都启动不起来等等，各种问题。

SpringBoot 设计的目标就是简化繁琐配置，快速建立Spring 应用，例如在pom 文件中引入了`spring-boot-starter-web、spring-boot-starter-data-redis` 依赖然后几乎不用配置就可以使用

## 2.命名规范
  
  ![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20230115110141.png)

**Srping官方命名格式为：spring-boot-starter-{name}**

**非Spring官方建议命名格式：{name}-spring-boot-starter**

## 3. 开发starter
参考京东示例开发一个日志记录的组件

### 3.1创建工程
创建一个maven项目

### 3.2 引入Pom依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.13</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>log-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>log-spring-boot-starter</name>
    <url>http://www.example.com</url>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>


    <dependencies>
        <!-- 提供了自动装配功能-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <!-- 在编译时会自动收集配置类的条件，写到一个META-INF/spring-autoconfigure-metadata.json中-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
        </dependency>
        <!--记录日志会用到切面，所以需要引入-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>2.2.1</version>
                <executions>
                    <execution>
                        <id>attach-sources</id>
                        <goals>
                            <goal>jar-no-fork</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```


`spring-boot-autoconfigure` :提供自动化装配功能，是为了Spring Boot 应用在各个模块提供自动化配置的作用；即加入对应 pom，就会有对应配置其作用；所以我们想要自动装配功能，就需要引入这个依赖。

`spring-boot-configuration-processor`：将自定义的配置类生成配置元数据，所以在引用自定义STARTER的工程的YML文件中，给自定义配置初始化时，会有属性名的提示；确保在使用`@ConfigurationProperties`注解时，可以优雅的读取配置信息，引入该依赖后，IDEA不会出现“spring boot configuration annotation processor not configured”的错误；编译之后会在META-INF 下生成一个spring-configuration-metadata.json 文件


### 3.3定义属性配置
```java
package com.example.config;  
  
import lombok.Data;  
import org.springframework.beans.factory.annotation.Value;  
import org.springframework.boot.context.properties.ConfigurationProperties;  
  
@Data  
@ConfigurationProperties(prefix = "log")  
public class LogProperties {  
  
    /**  
     * 开启日志  
     */  
    private boolean enable;  
  
    /**  
     * 区分不同使用平台  
     */  
    @Value("${spring.application.name:#{null}}")  
    private String platform;  
}
```

`@ConfigurationProperties`：该注解和`@Value` 注解作用类似，用于获取配置文件中属性定义并绑定到Java Bean 或者属性中；换句话来说就是将配置文件中的配置封装到JAVA 实体对象，方便使用和管理。

### 3.4定义自动配置类
```java
package com.example;  
  
import com.example.config.LogProperties;  
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;  
import org.springframework.boot.context.properties.EnableConfigurationProperties;  
import org.springframework.context.annotation.ComponentScan;  
import org.springframework.context.annotation.Configuration;  
  
@Configuration  
@ComponentScan("com.example")  
@ConditionalOnProperty(prefix = "log",name = "enable",havingValue = "true",matchIfMissing = false)  
@EnableConfigurationProperties({LogProperties.class})  
public class LogAutoConfiguration {  
}
```
这个类最关键了，它是整个starter 最重要的类，它就是将配置自动装载进spring-boot的；具体是怎么实现的，下面在讲解原理的时候会再详细说说，这里先完成示例。
@Configuration ：这个就是声明这个类是一个配置类

@ConditionalOnProperty：作用是可以指定prefix.name 配置文件中的[属性值](https://www.zhihu.com/search?q=%E5%B1%9E%E6%80%A7%E5%80%BC&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A2827791274%7D)来判定configuration是否被注入到Spring,就拿上面代码的来说，会根据配置文件中是否配置jd.enable 来判断是否需要加载JdLogAutoConfiguration 类，如果配置文件中不存在或者配置的是等于false 都不会进行加载，如果配置成true 则会加载；指定了havingValue，要把配置项的值与havingValue对比，一致则加载Bean;配置文件缺少配置，但配置了matchIfMissing = true，加载Bean，否则不加载。

@EnableConfigurationProperties使@ConfigurationProperties 注解的类生效。

### 3.5 配置EnableAutoConfiguration
在resources/META-INF/ 目录新建spring.factories 文件，配置内容如下；
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.example.LogAutoConfiguration
```

### 3.6 业务功能实现
```java
package com.example.annotation;  
  
import java.lang.annotation.ElementType;  
import java.lang.annotation.Retention;  
import java.lang.annotation.RetentionPolicy;  
import java.lang.annotation.Target;  
  
@Target({ElementType.METHOD})  
@Retention(RetentionPolicy.RUNTIME)  
public @interface Log {  
  
}
```

实现简单的aop业务逻辑
```java
package com.example.aop;  
  
import com.example.config.LogProperties;  
import com.example.annotation.Log;  
import lombok.AllArgsConstructor;  
import lombok.extern.slf4j.Slf4j;  
import org.aspectj.lang.ProceedingJoinPoint;  
import org.aspectj.lang.annotation.Around;  
import org.aspectj.lang.annotation.Aspect;  
import org.aspectj.lang.annotation.Pointcut;  
import org.springframework.stereotype.Component;  
  
@Aspect  
@Component  
@Slf4j  
@AllArgsConstructor  
public class LogAspectProcess {  
  
    LogProperties logProperties;  
  
    /**  
     * 定义切点  
     */  
    @Pointcut("@annotation(com.example.annotation.Log)")  
    public void pointCut() {  
    }  
  
    /**  
     * 环绕通知  
     *  
     * @return  
     */  
    @Around("pointCut() && @annotation(logs)")  
    public Object around(ProceedingJoinPoint thisJoinPoint, Log logs) {  
        //执行方法名称  
        String taskName = thisJoinPoint.getSignature()  
                .toString().substring(  
                        thisJoinPoint.getSignature()  
                                .toString().indexOf(" "),  
                        thisJoinPoint.getSignature().toString().indexOf("("));  
        taskName = taskName.trim();  
        long time = System.currentTimeMillis();  
        Object result = null;  
        try {  
            result = thisJoinPoint.proceed();  
        } catch (Throwable throwable) {  
            throwable.printStackTrace();  
        }  
        log.info("{} -- method:{} run :{} ms", logProperties.getPlatform(), taskName,  
                (System.currentTimeMillis() - time));  
        return result;  
    }  
}
```

### 3.7 整体项目结构
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1673768845644.png)

mvn打包后就可以测试

### 3.8 测试

创建一个新的web项目在pom中引入自定义的依赖
```xml
<!--  引入自定义log      -->  
<dependency>  
    <groupId>com.example</groupId>  
    <artifactId>log-spring-boot-starter</artifactId>  
    <version>1.0-SNAPSHOT</version>  
</dependency>
```

创建一个接口

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1673769091217.png)

在yml 中开启日志

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1673769120927.png)

访问[127.0.0.1:8089/api/v1/logs](http://127.0.0.1:8089/api/v1/logs) 看控制台打印

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20230115155514.png)

## 4.原理讲解


## 5.参考文章

- [Spring Boot 中的 starter 到底是什么 ?](https://www.zhihu.com/question/377025265/answer/2827791274)