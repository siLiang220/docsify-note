## Ribbon
### Ribbon 本地负载均衡客户端 VS Nginx 服务端负载均衡区别

-   Nginx 是服务器负载均衡，客户端所有请求都会交给nginx， 然后 nginx 实现转发请求。即负载均衡是由服务端实现的。
    
-   Ribbon 本地负载均衡，在调用微服务接口的时候，会在注册中心上获取注册信息服务列表后缓存到JVM 本地，从而在本地实现RPC远程 服务调用技术。


### Ribbon 集成
source: [6. Client Side Load Balancer: Ribbon (spring.io)](https://cloud.spring.io/spring-cloud-netflix/multi/multi_spring-cloud-ribbon.html)

- 配置类方式配置
```java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RoundRobinRule;


@Configuration
public class ServiceHiRibbonConfiguration {
	
	@Bean
	public IRule iRule() {
		return new RoundRobinRule();
	}
}
```

```java
@Configuration
@RibbonClient(name = "user-service",configuration = ServiceHiRibbonConfiguration.class)
public class RibbonConfig {
}
```

>[!Attention]
自定义的负载均衡配置类不能放在 `@ComponentScan` 所扫描的当前包下及其子包下，否则自定义的这个配置类就会被所有的Ribbon客户端所共享，也就是说我们达不到特殊化定制的目的了；

- yml方式配置
```yaml
#指定为某个服务配置负载规则
test1:
  ribbon:
    #配置自定义策略
    NFLoadBalancerRuleClassName: com.example.springcloud.testdemo.ribbonConfig.MyRule
 
 
ribbon-java:
  ribbon:
    #配置随机策略
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
 
---
#为所有服务配置统一规则
ribbon:
  #所有服务都使用随机策略
  #NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
  NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule  #指定使用Nacos提供的负载均衡策略（优先调用同一集群的实例，基于随机&权重）
```

Ribbon 于RestTemplate整合
```java
@Bean
@LoadBalanced
RestTemplate restTemplate() {
	return new RestTemplate();
}
```


## 服务降级：系统有限的资源的合理协调

服务降级是对系统整体资源的合理分配。区分核心服务和非核心服务。对某个服务的访问延迟时间、异常等情况做出预估并给出兜底方法。这是一种全局性的考量，对系统整体负荷进行管理。

对服务提供者超时，down机等或客户端自身的故障客户端通过自身fallback，返回一个缺省值。


## 熔断机制：应对雪崩效应的链路自我保护机制

熔断机制是应对雪崩的一种微服务链路保护机制，当扇出链路的某个微服务出现不可用或者响应时间过长，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。当检查到该节点微服务调用响应正常后，恢复调用链路。


[(225条消息) Sentinel集成Nacos数据源_sermonlizhi的博客-CSDN博客_sentinel-datasource-nacos](https://blog.csdn.net/sermonlizhi/article/details/123009182)