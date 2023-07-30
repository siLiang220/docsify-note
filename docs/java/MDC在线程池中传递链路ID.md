
## 一、场景需求

当客户端需要向设备发送固件升级指令时，服务端会按照固定的块大小将固件包分批次发送给设备。
如果设备在规定的时间内未响应，则需要重复向设备发送未响应的固件包。
在这个过程中，每次下发指令客户端、服务端和设备之间对应的是同一个Trace ID
## 二、实现方式：
在 固件升级过程中，数据被分块发送。每个请求线程都会创建一个调度线程来执行轮询重试操作。重试的判断是通过 Trace ID 判断设备是否响应，当收到设备的响应时，当前块数会递增。如果未收到响应，则会重新向设备发送当前块的数据。如果重试次数超过最大重试次数，或者当前块与缓存中的块不一致，将清除线程池不再重试。
## 三、存在的问题：
在调度线程池中的子线程无法获取到主线程MDC的traceId

## 四、对使用的线程池进行改造

修改之前使用调度线程池
```java
private static final int MAX_RETRY_COUNT = 3;
@Override
public void doProcess(DeviceEntity device, ProductEntity product, OtaUpgradeCmdV2 otaUpgradeCmdV2) {
	//todo 查询文件换切分的文件块大小，块大小应该是每个批次都是一样的不一定每次发指令查询设备
	String traceId = MDC.get("traceId");
	OtaEntity ota = otaService.getByProductId(product.getGuid())
			.orElseThrow(() -> new ApiException("ota.error", "api.response.code.ota.not.exist", httpStatusHelper.getResourceNotFound()));
	OtaVersionEntity otaVersion = otaVersionService.getLastVersion(ota.getGuid())
			.orElseThrow(() -> new ApiException("ota.version.error", "api.response.code.ota.version.not.exist", httpStatusHelper.getResourceNotFound()));
	byte[] fileBytes;
	try {
		fileBytes = ossUtils.getSingleFileToBytes(otaVersion.getFilePath());
	} catch (IOException e) {
		log.error("查询oss固件文件失败：{}", otaVersion.getFilePath());
		throw new RuntimeException("查询固件服务器文件失败");
	}
	Integer splitNumber = CacheUtil.SPLIT_NUMBER_MAP.get(traceId, value -> otaUpgradeCmdV2.getSplitNumber());
	List<byte[]> chunks = OtaPackageUtil.splitByteArray(fileBytes, splitNumber);
	// 判断trace是否为0，0则从第一个包开始发
	if (CacheUtil.OTA_CURRENT_CHUNK_NUM_MAP.getIfPresent(traceId) == null) {
		log.info("ota更新升级traceId缓存为null,traceId: {}", traceId);
	}
	AtomicInteger chunkNum = CacheUtil.OTA_CURRENT_CHUNK_NUM_MAP.get(traceId, value -> new AtomicInteger(-1));
	chunkNum.getAndIncrement();
	Map<String, Object> threadLocalMap = ThreadLocalUtil.get();
	// 定义超时时间(单位：秒)
	int timeout = 10;
	ScheduledExecutorService executorService = Executors.newScheduledThreadPool(1);
	//启动定时任务，等待一段时间重试
	final int[] retryCount = {1};
	executorService.scheduleAtFixedRate(() -> {
		int currentChunkNum = chunkNum.get();
		if (CacheUtil.OTA_CURRENT_CHUNK_NUM_MAP.getIfPresent(traceId).get() == currentChunkNum) {
				// 发送包数据
				if (chunkNum.get() < chunks.size()) {
					//region 这些代码是复制以前的代码
					byte[] chunk = chunks.get(currentChunkNum);
					String offset = HexConvertUtils.intToHexStringByLen(chunkNum.get() * splitNumber, 8);
					byte[] offsetBytes = HexConvertUtils.hexStrToByteArray(offset);
					byte[] byteMerger = HexConvertUtils.byteMerger(offsetBytes, chunk);
					String hexStr = HexConvertUtils.byteArrayToHexStr(byteMerger);
					log.info("发送包数据:traceId = {}  hexStr = {}", traceId, hexStr); // 添加日志
					//endregion
					// 要发送的内容有问题
					cmdMessageSender.buildSendMessage(device, product, CmdEnum.UPGRADE_PACKAGE_TRANSFER, hexStr);
					// 输出日志
					log.info("traceId = {} OTA升级包序号 {} 未收到响应，正在重新发送，重试次数: {}", traceId, currentChunkNum, retryCount[0]);
				} else {
					// 发送最后一个包，最后一个包是ota文件的大小
					int fileSize = fileBytes.length;
					String frameId = (String) threadLocalMap.get("frameId");
					CacheUtil.OTA_LAST_PACKAGE_FRAME_ID.put("frameId", frameId);
					String hexStr = HexConvertUtils.intToHexStringByLen(fileSize, 8);
					cmdMessageSender.buildSendMessage(device, product, CmdEnum.UPGRADE_LAST_PACKAGE_TRANSFER, hexStr);
					log.info("traceId = {} OTA升级包最后一包 {} 未收到响应，正在重新发送，重试次数: {}", traceId, currentChunkNum, retryCount[0]);
				}
				retryCount[0]++;
			if (retryCount[0] > MAX_RETRY_COUNT || CacheUtil.OTA_CURRENT_CHUNK_NUM_MAP.getIfPresent(traceId).get() != currentChunkNum) {
				// 达到最大重试次数，进行相应处理
				log.error("traceId = {}, OTA升级包序号 {} 未收到响应，已达到最大重试次数", traceId, currentChunkNum);
				// 取消定时任务
				executorService.shutdown();
			}
		}
	}, 0L, timeout, TimeUnit.SECONDS);
}
```

在调用buildSendMessage时无法获取到traceId
```java
public void buildSendMessage(DeviceEntity deviceEntity, ProductEntity product, CmdEnum cmdEnum, String dataHex) {
        String traceId = MDC.get("traceId");
        }
```

创建自定义的MDC线程池工厂，并重写Runable方法
```java
public class MdcThreadPoolFactory {
    public static ThreadPoolExecutor createThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime,
                                                              TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        return new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue) {
            @Override
            public void execute(Runnable task) {
                super.execute(MdcTaskDecorator.decorate(task));
            }

            // Override other methods if needed
        };
    }

    public static ScheduledExecutorService createScheduledExecutorService(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize) {
            @Override
            public ScheduledFuture<?> schedule(Runnable task, long delay, TimeUnit unit) {
                return super.schedule(MdcTaskDecorator.decorate(task), delay, unit);
            }

            @Override
            public ScheduledFuture<?> scheduleAtFixedRate(Runnable task, long initialDelay, long period, TimeUnit unit) {
                return super.scheduleAtFixedRate(MdcTaskDecorator.decorate(task), initialDelay, period, unit);
            }

            @Override
            public ScheduledFuture<?> scheduleWithFixedDelay(Runnable task, long initialDelay, long delay, TimeUnit unit) {
                return super.scheduleWithFixedDelay(MdcTaskDecorator.decorate(task), initialDelay, delay, unit);
            }

            // Override other methods if needed
        };
    }

    private static class MdcTaskDecorator implements Runnable {
        private final Runnable task;
        private final Map<String, String> contextMap;

        private MdcTaskDecorator(Runnable task, Map<String, String> contextMap) {
            this.task = task;
            this.contextMap = contextMap;
        }

        @Override
        public void run() {
            if (contextMap == null) {
                MDC.clear();
            } else {
                MDC.setContextMap(contextMap);
            }
            try {
                task.run();
            } finally {
                MDC.clear();
            }
        }

        public static Runnable decorate(Runnable task) {
            Map<String, String> contextMap = MDC.getCopyOfContextMap();
            return new MdcTaskDecorator(task, contextMap);
        }
    }
```
修改doProcess方法，使用自定义的线程池工厂创建调度线程池：
```java
ScheduledExecutorService executorService = MdcThreadPoolFactory.createScheduledExecutorService(1);
```
这样就可以在调度线程中获取到主线程的MDC上下文，确保可以获取到Trace ID

