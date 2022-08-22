# DeferredResult异步处理+长轮询
Servlet API3.0版本开始，引入异步处理机制。具体做法是Servlet容器的主线程池的工作线程接受到一个HTTP请求后，把他交给一个专门的任务处理子线程来处理该请求，工作线程又被释放回 主线程池，用于接收其他的HTTP请求，任务处理结果可通过在字线程中返回，这样既能实现线程隔离，又能同步返回处理结果给客户端。
- 异步servlet 示例
```java
@WebServlet(URLPatterns = "/async",asyncSupported = true)
public class AsyncServlet extends HttpServlet {

    ExecutorService executorService =Executors.newSingleThreadExecutor();

    @Override
     protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1. 开启异步,获取异步上下文
        final AsyncContext ctx = req.startAsync();
        //2. 提交线程池异步执行
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
                //4. 执行完成后完成回调
                ctx.complete();
            }
        });
    }
}
```
- spring mvc 线程池+DeferredResult 实现文件上传 异步处理和同步返回
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
