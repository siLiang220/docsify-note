
## Config

1. 维护性
2. 时效性
3. 安全性
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20221209100016.png)

### springcloud config 对比 三大优势： 
- springcloud config大部分场景结合git 使用, 动态变更还需要依赖Spring Cloud Bus 消息总线来通过所有的客户端变 化. 
- springcloud config不提供可视化界面 
- nacos config使用长轮询更新配置, 一旦配置有变动后，通知Provider的过程非常的迅速, 从速度上秒杀springcloud 原来的config几条街,
Spring Cloud Alibaba Nacos Config 目前提供了三种配置能力从 Nacos 拉取相关的配置。

- A: 通过 spring.cloud.nacos.config.shared-configs 支持多个共享 Data Id 的配置
- B: 通过 spring.cloud.nacos.config.ext-config[n].data-id 的方式支持多个扩展 Data Id 的配置
- C: 通过内部相关规则(应用名、应用名+ Profile )自动生成相关的 Data Id 配置当三种方式共同使用时，他们的一个优先级关系是:A < B < C

优先级从高到低：

1)  nacos-config-product.yaml 精准配置

2)  nacos-config.yaml 同工程不同环境的通用配置

3)  ext-config: 不同工程 扩展配置

4)  shared-dataids 不同工程通用配置

nacos 每10ms 拉取一次

[Nacos config · alibaba/spring-cloud-alibaba Wiki (github.com)](https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config)

