## DeferredResult异步处理+长轮询

Servlet API3.0版本开始，引入异步处理机制。具体做法是Servlet容器的主线程池的工作线程接受到一个HTTP请求后，把他交给一个专门的任务处理子线程来处理该请求，工作线程又被释放回 主线程池，用于接收其他的HTTP请求，任务处理结果可通过在字线程中返回，这样既能实现线程隔离，又能同步返回处理结果给客户端。
- 异步servlet 示例
```java
@WebServlet(URLPatterns = "/async",asyncSupported = true)
public class AsyncServlet extends HttpServlet {

    ExecutorService executorService =Executors.newSingleThreadExecutor();

    @Override
     protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.开启异步,获取异步上下文
        final AsyncContext ctx = req.startAsync();
        //2.提交线程池异步执行
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    //3.模拟任务并输出
                    Thread.sleep(10000L);
                    ServletResponse response = ctx.getResponse();
                    outputStream = response.getOutputStream();
                    outputStream.write("task complete".getBytes(StandardCharsets.UTF_8));
                    outputStream.flush();
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                //4.执行完成后完成回调
                ctx.complete();
            }
        });
    }
}
```

-  spring mvc 线程池+`DeferredResult`实现文件上传 异步处理和同步返回

```java
@RestController
public class PictureUploadController {
    private ExecutorService threadPool = new ThreadPoolExecutor(16, 16, 30, TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(5000), new ThreadFactory() {
        private final AtomicInteger uploadUrlPicThreadNum = new AtomicInteger(1);
        @Override
        public Thread newThread(Runnable runnable) {
            Thread thread = Executors.defaultThreadFactory().newThread(runnable);
            thread.setName("uploadUrlPicThread-" + uploadUrlPicThreadNum.getAndIncrement());
            return thread;
        }
    }, new ThreadPoolExecutor.DiscardOldestPolicy());

    @Autowired
	PictureUploadService pictureUploadService;

    @PostMapping("/asyncUpload")
    public DeferredResult<ApiResult> asyncUploadPicture(@Valid PictureUploadDTO pictureUploadDTO) {    
        //1. 构建DeferredResult对象，设置超时时间
        DeferredResult<ApiResult> deferredResult = new DeferredResult<>(5000);
        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                //2.异步上传图片
                apiResult = pictureUploadService.upload(pictureUploadDTO);
                //3.设置返回值
                deferredResult.setResult(apiResult);
            }
        });
        return deferredResult;
    }
}
```


## 简单demo

```java
public class DeferredResultController {  
    private static Logger LOGGER = LoggerFactory.getLogger(DeferredResultController.class);  
    private static final Map<String, DeferredResult<String>> taskMap = new ConcurrentHashMap<>();  
    //创建任务  
    @RequestMapping("/createTask")  
    public DeferredResult<String> createTask(String uuid) {  
        LOGGER.info("ID[{}]任务开始", uuid);  
        StopWatch stopWatch = new StopWatch(" DeferredResult test 容器线程");  
        stopWatch.start("容器线程");  
        LOGGER.info("返回值DeferredResult 异步请求开始");  
        //超时时间100s  
        DeferredResult<String> deferredResult = new DeferredResult<>(100000L);  
        StopWatch t = new StopWatch(" DeferredResult test 工作线程");  
        t.start("工作线程");  
        deferredResult.onCompletion(() -> {  
            LOGGER.info("返回值DeferredResult onCompletion 工作线程处理完毕");  
            t.stop();  
            LOGGER.info(String.format("%s秒", t.getTotalTimeSeconds()));  
        });  
        taskMap.put(uuid, deferredResult);  
        LOGGER.info("返回值DeferredResult 异步请求结束");  
        stopWatch.stop();  
        LOGGER.info(String.format("%s秒", stopWatch.getTotalTimeSeconds()));  
        return deferredResult;  
    }  
  
    //查询任务状态  
    @RequestMapping("/queryTaskState")  
    public String queryTaskState(String uuid) {  
        DeferredResult<String> deferredResult = taskMap.get(uuid);  
        if (deferredResult == null) {  
            return "未查询到任务,uid:" + uuid;  
        }  
        if (deferredResult.hasResult()) {  
            return deferredResult.getResult().toString();  
        } else {  
            LOGGER.info("ID[{}]任务进行中", uuid);  
            return "进行中";  
        }  
    }  
  
    //模拟第三方调用通知任务结束  
    @RequestMapping("/changeTaskState")  
    public String changeTaskState(String uuid) {  
        DeferredResult<String> deferredResult = taskMap.remove(uuid);  
        if (deferredResult == null) {  
            return "未查到到任务";  
        }  
        if (deferredResult.hasResult()) {  
            return "已完成，无需再次设置";  
        } else {  
            //未完成设置为完成  
            deferredResult.setResult("已完成");  
            LOGGER.info("将任务ID{},设置为处理完成", uuid);  
            return "已完成";  
        }  
    }
```
在createTask创建任务并且设置超时时间。将任务保存到map中在合适的时候通过另一个请求设置任务结束，createTask接口在未收到完成通知或超时之前不会结束请求。

在多实例的情况下可以使用redis作为消息中间件，参考：
利用redis的发布订阅功能，实例可以构造一个有相同前缀的队列ID，比如createbranch-request-084Tkjfh和createbranch-response-084Tkjfh,然后消费方根据createbranch-request这个前缀获取到相关队列列表，消费消息后根据队列命名规则将结果塞到对应的响应队列createbranch-response-084Tkjfh中由请求方消费。其中还需要考虑资源的竞争，以及重复消费的问题，需要做好幂等。

```java
//命令发送方
@Test
public void testPub(){
 
Jedis jedis = jedisPool.getResource();
jedis.publish("channel", "业务id+命令发送方接听channel"); 
System.out.println("==========已经发布消息===================");
}

// 服务订阅处理消息
@Test
public void testSub() {
final RedisSubPubListener listener = new RedisSubPubListener();
Jedis jedis = jedisPool.getResource();
jedis.subscribe(listener, "channel"); 
}

//服务处理完成后将消息返回给指定的订阅者
@Test
public void testSub() {
Jedis jedis = jedisPool.getResource();
jedis.publish("截取命令发送方接听的channel", "业务idl"); 
System.out.println("==========已经发布消息===================");
}

//思路，命令发送方发送消息的时候将业务id和命令发送方接听的channel 发送给服务处理者，服务处理者处理完成后将数据发送命令发送方发过来的指定channel中
```

在使用消息队列时，可以指定消息的目的地（或称为目标），使消息能够被发送到指定的接收者。具体的实现方式因消息队列技术的不同而有所差异。下面以 RabbitMQ 为例，介绍如何指定消息的目标。 RabbitMQ 是一个开源的消息队列系统，支持多种消息协议，包括 AMQP（Advanced Message Queuing Protocol）和 STOMP（Streaming Text Oriented Messaging Protocol）等。在 RabbitMQ 中，可以使用 Exchange 和 Queue 来实现消息的发布和订阅。 当一个消息被发布到 Exchange 中时，Exchange 会根据预定义的路由规则将消息路由到一个或多个 Queue 中。在实际应用中，可以为每个 Queue 绑定一个或多个消费者，用于处理 Queue 中的消息。当消费者完成对消息的处理后，可以将处理结果发送到指定的 Exchange 中。为了将处理结果返回给发布命令的实例，可以在 Exchange 中定义一个名为“result”的 Queue，并将该 Queue 绑定到一个 Direct Exchange 中。在发送处理结果时，可以将结果发送到“result”队列中，并指定消息的 Routing Key 为发布命令时所用的 Correlation Id，这样就可以确保处理结果会被发送到正确的实例中。 在 RabbitMQ 中，可以使用 Spring AMQP 或者原生的 Java API 来实现消息的发送和接收。需要注意的是，当使用消息队列时，需要考虑消息的序列化和反序列化、消息的可靠性和幂等性等问题，以确保消息的正确性和一致性。


## 参考文章

[浅谈spring servlet异步编程 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/612546966)