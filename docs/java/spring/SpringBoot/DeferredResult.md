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

## 参考文章

[浅谈spring servlet异步编程 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/612546966)