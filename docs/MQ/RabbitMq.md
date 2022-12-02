
## RabbitMq工作原理

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669948007980.png)


### 组成部分说明：

- **Broker**：消息队列服务进程，此进程包括两个部分：Exchange和Queue
- **Exchange**：消息队列交换机，按一定的规则将消息路由转发到某个队列，对消息进行过虑。
- **Queue**：消息队列，存储消息的队列，消息到达队列并转发给指定的
- **Producer**：消息生产者，即生产方客户端，生产方客户端将消息发送
- **Consumer**：消息消费者，即消费方客户端，接收MQ转发的消息。
- **Virtual Host**：虚拟主机，一个broker里可以设置多个vhost，实现用户权限分离
- **Channel**：消息通道，在客户端的每个连接里可以建立多个Channel，每个Channel代表一个会话任务
- **Binding**：把Exchange 和 Queue 按照路由规则绑定起来，交换机都需要与队列进行绑定
- **RoutingKey**：路由关键字，Exchange根据关键字进行投递

### 生产者发送消息流程：

1. 生产者和Broker建立TCP连接。

2. 生产者和Broker建立通道。

3. 生产者通过通道消息发送给Broker，由Exchange将消息进行转发。

4. Exchange将消息转发到指定的Queue（队列）

### 消费者接收消息流程：

1. 消费者和Broker建立TCP连接

2. 消费者和Broker建立通道

3. 消费者监听指定的Queue（队列）

4. 当有消息到达Queue时Broker默认将消息推送给消费者。

5. 消费者接收到消息。

6. ack回复


## 消息模式

1. 简单队列模式
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669950381554.png)


2. 工作队列模式
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669950423201.png)


3. 发布订阅
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669950472385.png)

4. 路由模式
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669950514635.png)


5. 通配符模式
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669950544203.png)

### 常用的三种交换机

- **direct**：当消息中的RoutingKey（路由键）和交换器与队列之间的BindingKey（绑定键）完全匹配时，会发送消息给相应的队列
- **fanout**：把当前路由器下所有消息发送到与该交换器绑定的队列中（不在意绑定键的有无与匹配），不需要路由键，即使绑了路由键也是无视的。
- **topic** ：将消息路由到路由键与绑定键相匹配的队列中。
匹配规则：
路由键一般：单词.单词.单词 构成.（单词为一个或者几个字母组成）
采用"*“和”#"，用于做模糊匹配,其中"#“用于匹配一个单词，”"用于匹配多个单词(0~n)个。
举例说明：路由键：.kkk.*可以匹配绑定键：rrr.kkk.lll。路由键：#.jjj.# 可以匹配绑定键：xxx.mmm.kkk

## 简单案例

### Hello world

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/image-20210626162052259.png)

- pom依赖
```xml
<!--指定 jdk 编译版本-->
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>8</source>
                <target>8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
<dependencies>
    <!--rabbitmq 依赖客户端-->
    <dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>amqp-client</artifactId>
        <version>5.8.0</version>
    </dependency>
    <!--操作文件流的一个依赖-->
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.6</version>
    </dependency>
</dependencies>
```

- 消息生产者

```java
package com.oddfar.one;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

/**
 * @author zhiyuan
 */
public class Producer {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        //创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("42.192.149.71");
        factory.setUsername("admin");
        factory.setPassword("123456");
        //channel 实现了自动 close 接口 自动关闭 不需要显示关闭
        //创建连接
        Connection connection = factory.newConnection();
        //获取信道
        Channel channel = connection.createChannel();
        /**
         * 生成一个队列
         * 1.队列名称
         * 2.队列里面的消息是否持久化 也就是是否用完就删除
         * 3.该队列是否只供一个消费者进行消费 是否进行共享 true 可以多个消费者消费
         * 4.是否自动删除 最后一个消费者端开连接以后 该队列是否自动删除 true 自动删除
         * 5.其他参数
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        String message = "hello world";
        /**
         * 发送一个消息
         * 1.发送到那个交换机
         * 2.路由的 key 是哪个
         * 3.其他的参数信息
         * 4.发送消息的消息体
         */
        channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
        System.out.println("消息发送完毕");

    }   
}

```

- 消息消费者

```java
package com.oddfar.one;

import com.rabbitmq.client.*;

/**
 * @author zhiyuan
 */
public class Consumer {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("42.192.149.71");
        factory.setUsername("admin");
        factory.setPassword("123456");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        System.out.println("等待接收消息.........");

        //推送的消息如何进行消费的接口回调
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody());
            System.out.println(message);
        };
        //取消消费的一个回调接口 如在消费的时候队列被删除掉了
        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println("消息消费被中断");
        };
        /**
         * 消费者消费消息 - 接受消息
         * 1.消费哪个队列
         * 2.消费成功之后是否要自动应答 true 代表自动应答 false 手动应答
         * 3.消费者未成功消费的回调
         * 4.消息被取消时的回调
         */
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
    }

}

```


### Work Queues
Work Queues——工作队列(又称任务队列)的主要思想是避免立即执行资源密集型任务，而不得不等待它完成。 相反我们安排任务在之后执行。我们把任务封装为消息并将其发送到队列。在后台运行的工作进 程将弹出任务并最终执行作业。当有多个工作线程时，这些工作线程将一起处理这些任务。

#### 轮询分发消息
在这个案例中我们会启动两个工作线程，一个消息发送线程，我们来看看他们两个工作线程是如何工作的。
##### 抽取工具类

```java
package com.oddfar.utils;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class RabbitMqUtils {
    //得到一个连接的 channel
    public static Channel getChannel() throws Exception {
        //创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("42.192.149.71");
        factory.setUsername("admin");
        factory.setPassword("123456");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        return channel;
    }
}

```

##### 启动两个消费者工作线程

```java
package com.oddfar.two;

import com.oddfar.utils.RabbitMqUtils;
import com.rabbitmq.client.CancelCallback;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.DeliverCallback;

/**
 * 这是一个工作线程，相当于之前的消费者
 *
 * @author zhiyuan
 */
public class Worker01 {

    private static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {

        Channel channel = RabbitMqUtils.getChannel();

        //消息接受
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String receivedMessage = new String(delivery.getBody());
            System.out.println("接收到消息:" + receivedMessage);
        };
        //消息被取消
        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println(consumerTag + "消费者取消消费接口回调逻辑");

        };

        System.out.println("C1 消费者启动等待消费.................. ");
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);

    }
}

```

##### 启动生产者发送消息
```java
package com.oddfar.two;

import com.oddfar.utils.RabbitMqUtils;
import com.rabbitmq.client.CancelCallback;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.DeliverCallback;

/**
 * 这是一个工作线程，相当于之前的消费者
 *
 * @author zhiyuan
 */
public class Worker01 {

    private static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {

        Channel channel = RabbitMqUtils.getChannel();

        //消息接受
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String receivedMessage = new String(delivery.getBody());
            System.out.println("接收到消息:" + receivedMessage);
        };
        //消息被取消
        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println(consumerTag + "消费者取消消费接口回调逻辑");

        };

        System.out.println("C1 消费者启动等待消费.................. ");
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);

    }
}

```
### 打印结果
通过程序执行发现生产者总共发送 4 个消息，消费者 1 和消费者 2 分别分得两个消息，并且是按照有序的一个接收一次消息
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669980711630.png)

## 消息应答
消费者完成一个任务可能需要一段时间，如果其中一个消费者处理一个长的任务并仅只完成了部分突然它挂掉了，会发生什么情况。RabbitMQ 一旦向消费者传递了一条消息，便立即将该消息标记为删除。在这种情况下，突然有个消费者挂掉了，我们将丢失正在处理的消息。以及后续发送给该消费这的消息，因为它无法接收到。

为了保证消息在发送过程中不丢失，引入消息应答机制，消息应答就是：**消费者在接收到消息并且处理该消息之后，告诉 rabbitmq 它已经处理了，rabbitmq 可以把该消息删除了**

### 自动应答

**消息发送后立即被认为已经传送成功**，这种模式需要在**高吞吐量和数据传输安全性方面做权衡**,因为这种模式如果消息在接收到之前，消费者那边出现连接或者 channel 关闭，那么消息就丢失 了,当然另一方面这种模式消费者那边可以传递过载的消息，**没有对传递的消息数量进行限制**，当然这样有可能使得消费者这边由于接收太多还来不及处理的消息，导致这些消息的积压，最终使 得内存耗尽，最终这些消费者线程被操作系统杀死，**所以这种模式仅适用在消费者可以高效并以 某种速率能够处理这些消息的情况下使用。**

### 手动消息应答

-   Channel.basicAck(用于肯定确认)
    
    RabbitMQ 已知道该消息并且成功的处理消息，可以将其丢弃了
    
-   Channel.basicNack(用于否定确认)
    
-   Channel.basicReject(用于否定确认)
    
    与 Channel.basicNack 相比少一个参数，不处理该消息了直接拒绝，可以将其丢弃了

**Multiple 的解释：**

手动应答的好处是可以批量应答并且减少网络拥堵

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000017.png)


-   true 代表批量应答 channel 上未应答的消息
    
    比如说 channel 上有传送 tag 的消息 5,6,7,8 当前 tag 是8 那么此时5-8 的这些还未应答的消息都会被确认收到消息应答
    
-   false 同上面相比只会应答 tag=8 的消息 5,6,7 这三个消息依然不会被确认收到消息应答

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000018.png)


### 消息自动重新入队

如果消费者由于某些原因失去连接(其通道已关闭，连接已关闭或 TCP 连接丢失)，导致消息未发送 ACK 确认，RabbitMQ 将了解到消息未完全处理，并将对其重新排队。如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者。这样，即使某个消费者偶尔死亡，也可以确保不会丢失任何消息。

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000019.png)

### 消息手动应答代码

默认消息采用的是自动应答，所以我们要想实现消息消费过程中不丢失，需要把自动应答改为手动应答

消费者在上面代码的基础上增加了以下内容

```java
channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);

```
#### 消息生产者

```java
package com.oddfar.three;

import com.oddfar.utils.RabbitMqUtils;
import com.rabbitmq.client.Channel;

import java.util.Scanner;

/**
 * 消息生产者,消息在手动应答时是不丢失的，放回队列重新消费
 *
 * @author zhiyuan
 */
public class Task02 {
    private static final String TASK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        //声明队列
        channel.queueDeclare(TASK_QUEUE_NAME, false, false, false, null);
        Scanner sc = new Scanner(System.in);
        System.out.println("请输入信息");
        while (sc.hasNext()) {
            String message = sc.nextLine();
            //发布消息
            channel.basicPublish("", TASK_QUEUE_NAME, null, message.getBytes("UTF-8"));
            System.out.println("生产者发出消息" + message);
        }
    }

}
```

#### 消费者01
```java
package com.oddfar.three;

import com.oddfar.utils.RabbitMqUtils;
import com.rabbitmq.client.CancelCallback;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.DeliverCallback;

/**
 * 消费者01
 *
 * @author zhiyuan
 */
public class Work03 {
    private static final String TASK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        System.out.println("C1 等待接收消息处理时间较 短");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("接收到消息:" + message);
            /**
             * 1.消息标记 tag
             * 2.是否批量应答未应答消息
             */
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };

        CancelCallback cancelCallback = (s) -> {
            System.out.println(s + "消费者取消消费接口回调逻辑");
        };

        //采用手动应答
        boolean autoAck = false;
        channel.basicConsume(TASK_QUEUE_NAME, autoAck, deliverCallback, cancelCallback);


    }
}

```

#### 消费者02
```java
//把时间改为30秒
```


正常情况下消息发送方发送两个消息 C1 和 C2 分别接收到消息并进行处理
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000021.png)

在发送者发送消息 dd，发出消息之后的把 C2 消费者停掉，按理说该 C2 来处理该消息，但是由于它处理时间较长，在还未处理完，也就是说 C2 还没有执行 ack 代码的时候，C2 被停掉了， 此时会看到消息被 C1 接收到了，说明消息 dd 被重新入队，然后分配给能处理消息的 C1 处理了
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669981637501.png)


## 消息分发机制

### 轮询分发
这个是RabbitMQ默认的消息分发机制，使用任务队列的优点之一就是可以轻易的并行工作。如果我们有很多要分发的消息，可以通过增加工作者（消费者）来解决这种状况，使得系统的伸缩性更加容易扩展。

在默认情况下，RabbitMQ不会顾虑消息者处理消息的能力，即使其中有的消费者闲置有的消费者高负荷。RabbitMQ会逐个发送消息到在序列中的下一个消费者(而不考虑每个任务的时长等等，且是提前一次性分配，并非一个一个分配)。平均每个消费者获得相同数量的消息，这种方式分发消息机制称为Round-Robin（轮询）。


### 公平分发

>[!tip]
> 使用公平分发，必须关闭自动应答，改为手动应答。

rabbitMQ 分发消息采用的轮训分发，但是在某种场景下这种策略并不是很好，比方说有两个消费者在处理任务，其中有个**消费者 1** 处理任务的速度非常快，而另外一个**消费者 2** 处理速度却很慢，这个时候我们还是采用轮训分发的化就会到这处理速度快的这个消费者很大一部分时间处于空闲状态，而处理慢的那个消费者一直在干活，这种分配方式在这种情况下其实就不太好，但是 RabbitMQ 并不知道这种情况它依然很公平的进行分发。

为了避免这种情况，**在消费者中消费之前**，我们可以设置参数 `channel.basicQos(1);`
```java
//公平分发
int prefetchCount = 1;
channel.basicQos(prefetchCount);
//采用手动应答
boolean autoAck = false;
channel.basicConsume(TASK_QUEUE_NAME, autoAck, deliverCallback, cancelCallback);
```

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669982329270.png)


意思就是如果这个任务我还没有处理完或者我还没有应答你，你先别分配给我，我目前只能处理一个 任务，然后 rabbitmq 就会把该任务分配给没有那么忙的那个空闲消费者，当然如果所有的消费者都没有完 成手上任务，队列还在不停的添加新任务，队列有可能就会遇到队列被撑满的情况，这个时候就只能添加 新的 worker 或者改变其他存储任务的策略.

>Spring Boot封装的RabbitMQ方法，默认ACK机制是使用手工应答机制，当@RabbitListener修饰的方法被调用且没有抛出异常时, Spring Boot会为我们自动应答。


### 预取值分发
带权的消息分发

本身消息的发送就是异步发送的，所以在任何时候，channel 上肯定不止只有一个消息另外来自消费 者的手动确认本质上也是异步的。因此这里就存在一个未确认的消息缓冲区，因此希望开发人员能**限制此缓冲区的大小**，**以避免缓冲区里面无限制的未确认消息问题**。这个时候就可以通过使用 basic.qos 方法设 置“预取计数”值来完成的。

该值定义通道上允许的未确认消息的最大数量。一旦数量达到配置的数量， RabbitMQ 将停止在通道上传递更多消息，除非至少有一个未处理的消息被确认，例如，假设在通道上有未确认的消息 5、6、7，8，并且通道的预取计数设置为 4，此时RabbitMQ 将不会在该通道上再传递任何消息，除非至少有一个未应答的消息被 ack。比方说 tag=6 这个消息刚刚被确认 ACK，RabbitMQ 将会感知 这个情况到并再发送一条消息。消息应答和 QoS 预取值对用户吞吐量有重大影响。

通常，增加预取将提高 向消费者传递消息的速度。**虽然自动应答传输消息速率是最佳的，但是，在这种情况下已传递但尚未处理的消息的数量也会增加，从而增加了消费者的 RAM 消耗**(随机存取存储器)应该小心使用具有无限预处理的自动确认模式或手动确认模式，消费者消费了大量的消息如果没有确认的话，会导致消费者连接节点的 内存消耗变大，所以找到合适的预取值是一个反复试验的过程，不同的负载该值取值也不同 100 到 300 范 围内的值通常可提供最佳的吞吐量，并且不会给消费者带来太大的风险。

预取值为 1 是最保守的。当然这将使吞吐量变得很低，特别是消费者连接延迟很严重的情况下，特别是在消费者连接等待时间较长的环境 中。对于大多数应用来说，稍微高一点的值将是最佳的。

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000032.png)


## RabbitMQ持久化

当 RabbitMQ 服务停掉以后，消息生产者发送过来的消息不丢失要如何保障？默认情况下 RabbitMQ 退出或由于某种原因崩溃时，它忽视队列和消息，除非告知它不要这样做。确保消息不会丢失需要做两件事：**我们需要将队列和消息都标记为持久化。**

### 队列实现持久化
```java
//让队列持久化
boolean durable = true;
//声明队列
channel.queueDeclare(TASK_QUEUE_NAME, durable, false, false, null);

```

>[!tip]
>如果之前声明的队列不是持久化的，需要把原先队列先删除，或者重新创建一个持久化的队列，不然就会出现错误
>![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000026.png)

以下为控制台中持久化与非持久化队列的 UI 显示区

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000027.png)

### 消息实现持久化
需要在消息**生产者**修改代码，`MessageProperties.PERSISTENT_TEXT_PLAIN` 添加这个属性。

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000028.png)

将消息标记为持久化并不能完全保证不会丢失消息。尽管它告诉 RabbitMQ 将消息保存到磁盘，但是这里依然存在当消息刚准备存储在磁盘的时候 但是还没有存储完，消息还在缓存的一个间隔点。此时并没 有真正写入磁盘。持久性保证并不强，但是对于我们的简单任务队列而言，这已经绰绰有余了。

## 消息发布确认

### 发布确认的的逻辑

生产者将信道设置成 confirm 模式，一旦信道进入 confirm 模式，所有在该信道上面发布的消息都将会被指派一个唯一的 ID(从 1 开始)，一旦消息被投递到所有匹配的队列之后，broker 就会发送一个确认给生产者(包含消息的唯一 ID)，这就使得生产者知道消息已经正确到达目的队列了，如果消息和队列是可持久化的，那么确认消息会在将消息写入磁盘之后发出，broker 回传给生产者的确认消息中 delivery-tag 域包含了确认消息的序列号，此外 broker 也可以设置basic.ack 的 multiple 域，表示到这个序列号之前的所有消息都已经得到了处理。

confirm 模式最大的好处在于他是异步的，一旦发布一条消息，生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用便可以通过回调方法来处理该确认消息，如果RabbitMQ 因为自身内部错误导致消息丢失，就会发送一条 nack 消息， 生产者应用程序同样可以在回调方法中处理该 nack 消息

### 发布确认的策略
发布确认默认是没有开启的，如果要开启需要调用方法 confirmSelect，每当你要想使用发布确认，都需要在 channel 上调用该方法

#### 单个确认发布
这是一种简单的确认方式，它是一种**同步确认发布**的方式，也就是发布一个消息之后只有它被确认发布，后续的消息才能继续发布，`waitForConfirmsOrDie(long)` 这个方法只有在消息被确认的时候才返回，如果在指定时间范围内这个消息没有被确认那么它将抛出异常。

这种确认方式有一个最大的缺点就是：**发布速度特别的慢**，因为如果没有确认发布的消息就会阻塞所有后续消息的发布，这种方式最多提供每秒不超过数百条发布消息的吞吐量。当然对于某些应用程序来说这可能已经足够了。

#### 批量确认发布
上面那种方式非常慢，与单个等待确认消息相比，先发布一批消息然后一起确认可以极大地提高吞吐量，当然这种方式的缺点就是：当发生故障导致发布出现问题时，不知道是哪个消息出 问题了，我们必须将整个批处理保存在内存中，以记录重要的信息而后重新发布消息。当然这种方案仍然是同步的，也一样阻塞消息的发布。

#### 异步确认发布

## 发布确认高级

### 发布确认 SpringBoot 版本

在生产环境中由于一些不明原因，导致 RabbitMQ 重启，在 RabbitMQ 重启期间生产者消息投递失败， 导致消息丢失，需要手动处理和恢复。

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000068.png)

代码架构图
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000069.png)

在配置文件当中需要添加
```
spring.rabbitmq.publisher-confirm-type=correlated

```

-   `NONE` 值是禁用发布确认模式，是默认值
    
-   `CORRELATED` 值是发布消息成功到交换器后会触发回调方法
    
-   `SIMPLE` 值经测试有两种效果，其一效果和 CORRELATED 值一样会触发回调方法，其二在发布消息成功后使用 rabbitTemplate 调用 waitForConfirms 或 waitForConfirmsOrDie 方法等待 broker 节点返回发送结果，根据返回结果来判定下一步的逻辑，要注意的点是 waitForConfirmsOrDie 方法如果返回 false 则会关闭 channel，则接下来无法发送消息到 broker;

#### 配置类
```java
@Configuration
public class ConfirmConfig {
    public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";
    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";

    //声明业务 Exchange
    @Bean("confirmExchange")
    public DirectExchange confirmExchange() {
        return new DirectExchange(CONFIRM_EXCHANGE_NAME);
    }

    // 声明确认队列
    @Bean("confirmQueue")
    public Queue confirmQueue() {
        return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }

    // 声明确认队列绑定关系
    @Bean
    public Binding queueBinding(@Qualifier("confirmQueue") Queue queue,
                                @Qualifier("confirmExchange") DirectExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with("key1");
    }
}

```
#### 消息生产者回调接口
```java
@Component
@Slf4j
public class MyCallBack implements RabbitTemplate.ConfirmCallback {
    /**
     * 交换机不管是否收到消息的一个回调方法
     *
     * @param correlationData 消息相关数据
     * @param ack             交换机是否收到消息
     * @param cause           为收到消息的原因
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        String id = correlationData != null ? correlationData.getId() : "";
        if (ack) {
            log.info("交换机已经收到 id 为:{}的消息", id);
        } else {
            log.info("交换机还未收到 id 为:{}消息，原因:{}", id, cause);
        }
    }

}
```
#### 消息生产者
```java
@RestController
@RequestMapping("/confirm")
@Slf4j
public class ProducerController {
    public static final String CONFIRM_EXCHANGE_NAME = "confirm.exchange";
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Autowired
    private MyCallBack myCallBack;

    //依赖注入 rabbitTemplate 之后再设置它的回调对象
    @PostConstruct
    public void init() {
        rabbitTemplate.setConfirmCallback(myCallBack);
    }
    
    /**
     * 消息回调和退回
     *
     * @param message
     */
    @GetMapping("sendMessage/{message}")
    public void sendMessage(@PathVariable String message) {

        //指定消息 id 为 1
        CorrelationData correlationData1 = new CorrelationData("1");
        String routingKey = "key1";
        rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME, routingKey, message + routingKey, correlationData1);
        log.info(routingKey + "发送消息内容:{}", message + routingKey);

        CorrelationData correlationData2 = new CorrelationData("2");
        routingKey = "key2";
        rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME, routingKey, message + routingKey, correlationData2);
        log.info(routingKey + "发送消息内容:{}", message + routingKey);

    }

}

```

#### 消息消费者
```java
@Component
@Slf4j
public class ConfirmConsumer {
    public static final String CONFIRM_QUEUE_NAME = "confirm.queue";

    @RabbitListener(queues = CONFIRM_QUEUE_NAME)
    public void receiveMsg(Message message) {
        String msg = new String(message.getBody());
        log.info("接受到队列 confirm.queue 消息:{}", msg);
    }

}
```
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669966017423.png)

可以看到，发送了两条消息，第一条消息的 RoutingKey 为 "key1"，第二条消息的 RoutingKey 为 "key2"，两条消息都成功被交换机接收，也收到了交换机的确认回调，但消费者只收到了一条消息，因为第二条消息的 RoutingKey 与队列的 BindingKey 不一致，也没有其它队列能接收这个消息，所有第二条消息被直接丢弃了。

丢弃的消息交换机是不知道的，需要解决告诉生产者消息传送失败

### 回退消息

Mandatory 参数

```java
rabbitTemplate.setReturnsCallback(myCallBack);
```

在仅开启了生产者确认机制的情况下，交换机接收到消息后，会直接给消息生产者发送确认消息，**如果发现该消息不可路由，那么消息会被直接丢弃，此时生产者是不知道消息被丢弃这个事件的**。

那么如何让无法被路由的消息帮我想办法处理一下？最起码通知我一声，我好自己处理啊。通过设置 mandatory 参数可以在当消息传递过程中不可达目的地时将消息返回给生产者。

#### 修改配置

```
#消息退回
spring.rabbitmq.publisher-returns=true
```

#### 修改回退接口
```java
@Component
@Slf4j
public class MyCallBack implements RabbitTemplate.ConfirmCallback, RabbitTemplate.ReturnsCallback {

    /**
     * 交换机不管是否收到消息的一个回调方法
     *
     * @param correlationData 消息相关数据
     * @param ack             交换机是否收到消息
     * @param cause           为收到消息的原因
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        String id = correlationData != null ? correlationData.getId() : "";
        if (ack) {
            log.info("交换机已经收到 id 为:{}的消息", id);
        } else {
            log.info("交换机还未收到 id 为:{}消息，原因:{}", id, cause);
        }
    }

    //当消息无法路由的时候的回调方法
    @Override
    public void returnedMessage(ReturnedMessage returned) {

        log.error("消息：{}，被交换机 {} 退回，原因：{}，路由key：{},code:{}",
                new String(returned.getMessage().getBody()), returned.getExchange(),
                returned.getReplyText(), returned.getRoutingKey(),
                returned.getReplyCode());

    }
}
```

#### 修改生产者
```java
//依赖注入 rabbitTemplate 之后再设置它的回调对象
@PostConstruct
public void init() {
    //消息回调
    rabbitTemplate.setConfirmCallback(myCallBack);
    /**
     * true：交换机无法将消息进行路由时，会将该消息返回给生产者
     * false：如果发现消息无法进行路由，则直接丢弃
     */
    rabbitTemplate.setMandatory(true);
    //设置回退消息交给谁处理
    rabbitTemplate.setReturnsCallback(myCallBack);
  	//RabbitMQ版本低的是 rabbitTemplate.setReturnCallback(myCallBack);

}
```

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669966743635.png)


## 死信队列
### 死信概念
一般来说，producer 将消息投递到 broker 或者直接到queue 里了，consumer 从 queue 取出消息 进行消费，但某些时候由于特定的原因**导致 queue 中的某些消息无法被消费**，这样的消息如果没有后续的处理，就变成了死信，有死信自然就有了死信队列。

应用场景：为了保证订单业务的消息数据不丢失，需要使用到 RabbitMQ 的死信队列机制，当消息消费发生异常时，将消息投入死信队列中。还有比如说：用户在商城下单成功并点击去支付后在指定时间未支付时自动失效

### 死信的来源

-   消息 TTL 过期
    
    TTL是Time To Live的缩写, 也就是生存时间
    
-   队列达到最大长度
    
    队列满了，无法再添加数据到 mq 中
    
-   消息被拒绝
    
    (basic.reject 或 basic.nack) 并且 requeue=false.

### 死信实践
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000048.png)


## 延迟队列

### 延迟队列介绍

#### 延迟队列的概念
延时队列,队列内部是有序的，最重要的特性就体现在它的延时属性上，延时队列中的元素是希望 在指定时间到了以后或之前取出和处理，简单来说，延时队列就是用来存放需要在指定时间被处理的 元素的队列。

#### 延迟队列使用场景

1.订单在十分钟之内未支付则自动取消 2.新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒。 3.用户注册成功后，如果三天内没有登陆则进行短信提醒。 4.用户发起退款，如果三天内没有得到处理则通知相关运营人员。 5.预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议

这些场景都有一个特点，需要在某个事件发生之后或者之前的指定时间点完成某一项任务，如： 发生订单生成事件，在十分钟之后检查该订单支付状态，然后将未支付的订单进行关闭；那我们一直轮询数据，每秒查一次，取出需要被处理的数据，然后处理不就完事了吗？

如果数据量比较少，确实可以这样做，比如：对于“如果账单一周内未支付则进行自动结算”这样的需求， 如果对于时间不是严格限制，而是宽松意义上的一周，那么每天晚上跑个定时任务检查一下所有未支付的账单，确实也是一个可行的方案。但对于数据量比较大，并且时效性较强的场景，如：“订单十分钟内未支付则关闭“，短期内未支付的订单数据可能会有很多，活动期间甚至会达到百万甚至千万级别，对这么庞大的数据量仍旧使用轮询的方式显然是不可取的，很可能在一秒内无法完成所有订单的检查，同时会给数据库带来很大压力，无法满足业务要求而且性能低下。

#### TTL

TTL 是什么呢？TTL 是 RabbitMQ 中一个**消息或者队列**的属性，表明一条消息或者该队列中的所有消息的最大存活时间，单位是毫秒。

-   **队列设置TTL**

在创建队列的时候设置队列的“x-message-ttl”属性

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000057.png)


-   **消息设置TTL**

是针对每条消息设置TTL
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000056.png)


两者的区别
如果设置了队列的 TTL 属性，那么一旦消息过期，就会被队列丢弃(如果配置了死信队列被丢到死信队列中)，而第二种方式，消息即使过期，也不一定会被马上丢弃，因为消息是否过期是在即将投递到消费者之前判定的，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间；

>[!tip]
>另外，还需要注意的一点是，如果不设置 TTL，表示消息永远不会过期，如果将 TTL 设置为 0，则表示除非此时可以直接投递该消息到消费者，否则该消息将会被丢弃。

### 整合SpringBoot

延时队列，不就是想要消息延迟多久被处理吗，TTL 则刚好能让消息在延迟多久之后成为死信，另一方面， 成为死信的消息都会被投递到死信队列里，这样只需要消费者一直消费死信队列里的消息就完事了，因为里面的消息都是希望被立即处理的消息。

-  添加依赖
```xml
<dependencies>
   <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <!--RabbitMQ 依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.47</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    <!--swagger-->
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>3.0.0</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>3.0.0</version>
    </dependency>
    <!--RabbitMQ 测试依赖-->
    <dependency>
        <groupId>org.springframework.amqp</groupId>
        <artifactId>spring-rabbit-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```


-  修改配置文件
```properites
spring.rabbitmq.host=42.192.149.71
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=123456
```

-  添加Swagger

```java
package com.oddfar.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;


@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket webApiConfig() {
        return new Docket(DocumentationType.SWAGGER_2)
                .groupName("webApi")
                .apiInfo(webApiInfo())
                .select()
                .build();
    }

    private ApiInfo webApiInfo() {
        return new ApiInfoBuilder()
                .title("rabbitmq 接口文档")
                .description("本文档描述了 rabbitmq 微服务接口定义")
                .version("1.0")
                .contact(new Contact("zhiyuan", "http://oddfar.com", "test@qq.com"))
                .build();
    }
}
```

#### 队列TTL

##### 队列架构图

创建两个队列 QA 和 QB，两者队列 TTL 分别设置为 10S 和 40S，然后在创建一个交换机 X 和死信交 换机 Y，它们的类型都是direct，创建一个死信队列 QD，它们的绑定关系如下：
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000060.png)
##### 配置文件代码
```java
package com.oddfar.config;

import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

/**
 * @author zhiyuan
 */
@Configuration
public class TtlQueueConfig {
    public static final String X_EXCHANGE = "X";
    public static final String QUEUE_A = "QA";
    public static final String QUEUE_B = "QB";
    //死信交换机
    public static final String Y_DEAD_LETTER_EXCHANGE = "Y";
    //死信队列
    public static final String DEAD_LETTER_QUEUE = "QD";

    // 声明 xExchange
    @Bean("xExchange")
    public DirectExchange xExchange() {
        return new DirectExchange(X_EXCHANGE);
    }

    // 声明 死信队列交换机
    @Bean("yExchange")
    public DirectExchange yExchange() {
        return new DirectExchange(Y_DEAD_LETTER_EXCHANGE);
    }

    //声明队列 A ttl 为 10s 并绑定到对应的死信交换机
    @Bean("queueA")
    public Queue queueA() {
        Map<String, Object> args = new HashMap<>(3);
        //声明当前队列绑定的死信交换机
        args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
        //声明当前队列的死信路由 key
        args.put("x-dead-letter-routing-key", "YD");
        //声明队列的 TTL
        args.put("x-message-ttl", 10000);
        return QueueBuilder.durable(QUEUE_A).withArguments(args).build();
    }

    // 声明队列 A 绑定 X 交换机
    @Bean
    public Binding queueaBindingX(@Qualifier("queueA") Queue queueA,
                                  @Qualifier("xExchange") DirectExchange xExchange) {
        return BindingBuilder.bind(queueA).to(xExchange).with("XA");
    }

    //声明队列 B ttl 为 40s 并绑定到对应的死信交换机
    @Bean("queueB")
    public Queue queueB() {
        Map<String, Object> args = new HashMap<>(3);
        //声明当前队列绑定的死信交换机
        args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
        //声明当前队列的死信路由 key
        args.put("x-dead-letter-routing-key", "YD");
        //声明队列的 TTL
        args.put("x-message-ttl", 40000);
        return QueueBuilder.durable(QUEUE_B).withArguments(args).build();
    }

    //声明队列 B 绑定 X 交换机
    @Bean
    public Binding queuebBindingX(@Qualifier("queueB") Queue queue1B,
                                  @Qualifier("xExchange") DirectExchange xExchange) {
        return BindingBuilder.bind(queue1B).to(xExchange).with("XB");
    }

    //声明死信队列 QD
    @Bean("queueD")
    public Queue queueD() {
        return new Queue(DEAD_LETTER_QUEUE);
    }

    //声明死信队列 QD 绑定关系
    @Bean
    public Binding deadLetterBindingQAD(@Qualifier("queueD") Queue queueD,
                                        @Qualifier("yExchange") DirectExchange yExchange) {
        return BindingBuilder.bind(queueD).to(yExchange).with("YD");
    }

}

```

##### 消费者代码
```java
package com.oddfar.consumer;

import com.rabbitmq.client.Channel;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.Date;

/**
 * 消费者 - 死信队列
 * @author zhiyuan
 */
@Slf4j
@Component
public class DeadLetterQueueConsumer {

    @RabbitListener(queues = "QD")
    public void receiveD(Message message, Channel channel) throws IOException {
        String msg = new String(message.getBody());
        log.info("当前时间：{},收到死信队列信息{}", new Date().toString(), msg);
    }

}

```

##### 生产者代码
```java
package com.oddfar.contorller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Date;

/**
 * @author zhiyuan
 */
@Slf4j
@RequestMapping("ttl")
@RestController
public class SendMsgController {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("sendMsg/{message}")
    public void sendMsg(@PathVariable String message) {
        log.info("当前时间：{},发送一条信息给两个 TTL 队列:{}", new Date(), message);
        rabbitTemplate.convertAndSend("X", "XA", "消息来自 ttl 为 10S 的队列: " + message);
        rabbitTemplate.convertAndSend("X", "XB", "消息来自 ttl 为 40S 的队列: " + message);
    }
    
}

```

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669961699044.png)


第一条消息在 10S 后变成了死信消息，然后被消费者消费掉，第二条消息在 40S 之后变成了死信消息， 然后被消费掉，这样一个延时队列就打造完成了。

不过，如果这样使用的话，岂不是每增加一个新的时间需求，就要新增一个队列，这里只有 10S 和 40S 两个时间选项，如果需要一个小时后处理，那么就需要增加TTL 为一个小时的队列，如果是预定会议室然后提前通知这样的场景，岂不是要增加无数个队列才能满足需求

#### 延时队列优化

新增了一个队列 QC,绑定关系如下,该队列不设置TTL 时间
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000062.png)



##### 配置代码
```java
@Configuration
public class MsgTtlQueueConfig {
    public static final String Y_DEAD_LETTER_EXCHANGE = "Y";
    public static final String QUEUE_C = "QC";

    //声明队列 C 死信交换机
    @Bean("queueC")
    public Queue queueB() {
        Map<String, Object> args = new HashMap<>(3);
        //声明当前队列绑定的死信交换机
        args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
        //声明当前队列的死信路由 key
        args.put("x-dead-letter-routing-key", "YD");
        //没有声明 TTL 属性
        return QueueBuilder.durable(QUEUE_C).withArguments(args).build();
    }

    //声明队列 B 绑定 X 交换机
    @Bean
    public Binding queuecBindingX(@Qualifier("queueC") Queue queueC,
                                  @Qualifier("xExchange") DirectExchange xExchange) {
        return BindingBuilder.bind(queueC).to(xExchange).with("XC");
    }
}

```

##### 生产者代码

```java
/**
 * 延时队列优化
 * @param message 消息
 * @param ttlTime 延时的毫秒
 */
@GetMapping("sendExpirationMsg/{message}/{ttlTime}")
public void sendMsg(@PathVariable String message, @PathVariable String ttlTime) {
    rabbitTemplate.convertAndSend("X", "XC", message, correlationData -> {
        correlationData.getMessageProperties().setExpiration(ttlTime);
        return correlationData;
    });
    log.info("当前时间：{},发送一条时长{}毫秒 TTL 信息给队列 C:{}", new Date(), ttlTime, message);
}
```

##### 消费者代码
```java
/**
 * 延时队列优化
 * @param message 消息
 * @param ttlTime 延时的毫秒
 */
@GetMapping("sendExpirationMsg/{message}/{ttlTime}")
public void sendMsg(@PathVariable String message, @PathVariable String ttlTime) {
    rabbitTemplate.convertAndSend("X", "XC", message, correlationData -> {
        correlationData.getMessageProperties().setExpiration(ttlTime);
        return correlationData;
    });
    log.info("当前时间：{},发送一条时长{}毫秒 TTL 信息给队列 C:{}", new Date(), ttlTime, message);
}
```

发送请求
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669956158092.png)

看起来似乎没什么问题，但是在最开始的时候，就介绍过如果使用在消息属性上设置 TTL 的方式，消息可能并不会按时“死亡“

因为 RabbitMQ 只会检查第一个消息是否过期，如果过期则丢到死信队列， 如果第一个消息的延时时长很长，而第二个消息的延时时长很短，第二个消息并不会优先得到执行。

这也就是为什么第二个延时2秒，却后执行。

### Rabbitmq 插件实现延迟队列
**如果不能实现在消息粒度上的 TTL，并使其在设置的TTL 时间及时死亡，就无法设计成一个通用的延时队列。那如何解决呢，接下来我们就去解决该问题。**

可去[官网下载 (opens new window)](https://www.rabbitmq.com/community-plugins.html)**rabbitmq_delayed_message_exchange** 插件，放置到 RabbitMQ 的插件目录。

进入 RabbitMQ 的安装目录下的 plgins 目录，执行下面命令让该插件生效，然后重启 RabbitMQ

```sh
[root@VM-0-6-centos software]# ls
erlang-21.3.8.21-1.el7.x86_64.rpm  rabbitmq_delayed_message_exchange-3.8.0.ez  rabbitmq-server-3.8.8-1.el7.noarch.rpm
#移动
cp rabbitmq_delayed_message_exchange-3.8.0.ez /usr/lib/rabbitmq/lib/rabbitmq_server-3.8.8/plugins
#安装
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
#重启服务
systemctl restart rabbitmq-server

```


#### 配置文件类代码

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000065.png)

在这里新增了一个队列delayed.queue,一个自定义交换机 delayed.exchange，绑定关系如下:


![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/RabbitMQ-00000066.png)


在我们自定义的交换机中，这是一种新的交换类型，该类型消息支持延迟投递机制消息传递后并不会立即投递到目标队列中，而是存储在 mnesia(一个分布式数据系统)表中，当达到投递时间时，才投递到目标队列中。

```java
@Configuration
public class DelayedQueueConfig {
    public static final String DELAYED_QUEUE_NAME = "delayed.queue";
    public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
    public static final String DELAYED_ROUTING_KEY = "delayed.routingkey";

    @Bean
    public Queue delayedQueue() {
        return new Queue(DELAYED_QUEUE_NAME);
    }

    //自定义交换机 我们在这里定义的是一个延迟交换机
    @Bean
    public CustomExchange delayedExchange() {
        Map<String, Object> args = new HashMap<>();
        //自定义交换机的类型
        args.put("x-delayed-type", "direct");
        return new CustomExchange(DELAYED_EXCHANGE_NAME, "x-delayed-message", true, false, args);
    }

    @Bean
    public Binding bindingDelayedQueue(@Qualifier("delayedQueue") Queue queue,
                                       @Qualifier("delayedExchange") CustomExchange delayedExchange) {
        return BindingBuilder.bind(queue).to(delayedExchange).with(DELAYED_ROUTING_KEY).noargs();
    }

}
```

#### 生产者代码 
```java
@GetMapping("sendDelayMsg/{message}/{delayTime}")
public void sendMsg(@PathVariable String message, @PathVariable Integer delayTime) {
    rabbitTemplate.convertAndSend(DELAYED_EXCHANGE_NAME, DELAYED_ROUTING_KEY, message,
            correlationData -> {
                correlationData.getMessageProperties().setDelay(delayTime);
                return correlationData;
            });
    log.info(" 当 前 时 间 ： {}, 发 送 一 条 延 迟 {} 毫秒的信息给队列 delayed.queue:{}", new Date(), delayTime, message);
}
```

#### 消费者代码
```java
/**
 * 消费者 - 基于插件的延时队列
 *
 * @author zhiyuan
 */
@Slf4j
@ComponentScan
public class DelayQueueConsumer {

    public static final String DELAYED_QUEUE_NAME = "delayed.queue";

    @RabbitListener(queues = DELAYED_QUEUE_NAME)
    public void receiveDelayedQueue(Message message) {
        String msg = new String(message.getBody());
        log.info("当前时间：{},收到延时队列的消息：{}", new Date().toString(), msg);
    }
}
```


![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669961602124.png)

### 总结
延时队列在需要延时处理的场景下非常有用，使用 RabbitMQ 来实现延时队列可以很好的利用 RabbitMQ 的特性，如：消息可靠发送、消息可靠投递、死信队列来保障消息至少被消费一次以及未被正确处理的消息不会被丢弃。另外，通过 RabbitMQ 集群的特性，可以很好的解决单点故障问题，不会因为 单个节点挂掉导致延时队列不可用或者消息丢失。

当然，延时队列还有很多其它选择，比如利用 Java 的 DelayQueue，利用 Redis 的 zset，利用 Quartz 或者利用 kafka 的时间轮，这些方式各有特点,看需要适用的场景


## 幂等性、优先级、惰性

### 幂等性

#### 概念
用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用。 举个最简单的例子，那就是支付，用户购买商品后支付，支付扣款成功，但是返回结果的时候网络异常， 此时钱已经扣了，用户再次点击按钮，此时会进行第二次扣款，返回结果成功，用户查询余额发现多扣钱 了，流水记录也变成了两条。在以前的单应用系统中，我们只需要把数据操作放入事务中即可，发生错误立即回滚，但是再响应客户端的时候也有可能出现网络中断或者异常等等

#### 消息重复消费
消费者在消费 MQ 中的消息时，MQ 已把消息发送给消费者，消费者在给 MQ 返回 ack 时网络中断， 故 MQ 未收到确认信息，该条消息会重新发给其他的消费者，或者在网络重连后再次发送给该消费者，但实际上该消费者已成功消费了该条消息，造成消费者消费了重复的消息。

#### 解决思路
MQ 消费者的幂等性的解决一般使用全局 ID 或者写个唯一标识比如时间戳 或者 UUID 或者订单消费者消费 MQ 中的消息也可利用 MQ 的该 id 来判断，或者可按自己的规则生成一个全局唯一 id，每次消费消息时用该 id 先判断该消息是否已消费过。

#### 消费者幂等性保障
在海量订单生成的业务高峰期，生产端有可能就会重复发生了消息，这时候消费端就要实现幂等性， 这就意味着我们的消息永远不会被消费多次，即使我们收到了一样的消息。

业界主流的幂等性有两种操作:a. 唯一 ID+指纹码机制,利用数据库主键去重, b.利用 redis 的原子性去实现

-   唯一ID+指纹码机制

指纹码：我们的一些规则或者时间戳加别的服务给到的唯一信息码,它并不一定是我们系统生成的，基本都是由我们的业务规则拼接而来，但是一定要保证唯一性，然后就利用查询语句进行判断这个 id 是否存在数据库中，优势就是实现简单就一个拼接，然后查询判断是否重复；劣势就是在高并发时，如果是单个数据库就会有写入性能瓶颈当然也可以采用分库分表提升性能，但也不是我们最推荐的方式。

-   Redis 原子性

利用 redis 执行 setnx 命令，天然具有幂等性。从而实现不重复消费


## SpringBoot + RabbitMq 创建队列的三种方式

```java
package com.macs.order.message;
 
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;
 
/**
 * 接收mq消息
 * @author antma
 */
@Slf4j
@Component
public class MqReceiver {
 
    //1. 手动创建，需在RabbitMQ中手动创建myQueue1 队列，否则报错
    @RabbitListener(queues = "myQueue1")
    public void process1(String message){
        log.info("MqReceiver1: {}", message);
    }
 
    //2. 自动创建队列
    @RabbitListener(queuesToDeclare = @Queue("myQueue2"))
    public void process2(String message){
        log.info("MqReceiver2: {}", message);
 
    }
 
    //3. 自动创建队列，Exchange 与 Queue绑定
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue("myQueue3"),
            exchange = @Exchange("testExChange")
    ))
    public void process3(String message){
        log.info("MqReceiver3: {}", message);
    }
 
 
//    @RabbitListener(bindings = @QueueBinding(
//            exchange = @Exchange("myOrder"),
//            key = "computer",
//            value = @Queue("computerOrder")
//    ))
//    public void processComputer(String message){
//        log.info("computer MqReceiver: {}", message);
//
//    }
//
//    @RabbitListener(bindings = @QueueBinding(
//            exchange = @Exchange("myOrder"),
//            key = "fruit",
//            value = @Queue("fruitOrder")
//    ))
//    public void processFruit(String message){
//        log.info("fruit MqReceiver: {}", message);
//
//    }
 
}
```


## 参考文章
[(222条消息) 玩转 Spring Boot 集成篇（RabbitMQ）_一猿小讲的博客-CSDN博客](https://blog.csdn.net/javaforwork/article/details/122773576)

[(222条消息) Springboot 整合RabbitMq ，用心看完这一篇就够了_小目标青年的博客-CSDN博客_springboot rabbitmq](https://blog.csdn.net/qq_35387940/article/details/100514134)

[(222条消息) SpringBoot + RabbitMQ 简单实现创建队列的三种方式_程序员老油条的博客-CSDN博客_springboot创建rabbitmq队列](https://blog.csdn.net/antma/article/details/84311601?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2-84311601-blog-115024847.pc_relevant_landingrelevant&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2-84311601-blog-115024847.pc_relevant_landingrelevant&utm_relevant_index=5)

[(222条消息) RasbbitMQ 路由器使用 绑定键 路由键 详解 路由器与队列绑定_吴顺达的博客-CSDN博客_路由键[hs-mq-postback-topic-u6q8qj8atkkbsz84j1ekhg],是](https://blog.csdn.net/wsdfym/article/details/101726677)

[RabbitMQ - 知识体系 | OddFar's Notes](https://note.oddfar.com/rabbitmq/#%E8%A7%86%E9%A2%91%E6%95%99%E7%A8%8B)