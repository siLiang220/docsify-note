## 用户线程与守护线程

```java
package com.java.bilibili.base;  
  
import java.util.concurrent.TimeUnit;  
  
public class DaemonDemo {  
  
    public static void main(String[] args) throws InterruptedException { //一切程序的入口  
        Thread thread = new Thread(()->{  
            System.out.println(Thread.currentThread().getName() + "\t" +  
                    ((Thread.currentThread().isDaemon())?"守护线程":"用户线程"));  
            while (true){  
  
            }  
        },"t1");  
  
        thread.setDaemon(true); //用户线程结束了守护线程也跟着结束 setDaemon(true);要在start 方法之前  
        thread.start();  
  
        TimeUnit.SECONDS.sleep(3);  
  
        System.out.println(Thread.currentThread().getName()+"\t"+"主线程");  
    }  
}
```
## Future
应用场景 主线程让一个子线程取执行任务，子线程可能比较耗时，启动子线程开始执行任务后，主线程过了一会儿取获取子任务的执行结果或变更任务的状态
- 多线程 
- 异步 Runnable
- 有返回结果 Callable

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1654086918710.png)


```java
package com.java.bilibili.base;  
  
import java.util.concurrent.Callable;  
import java.util.concurrent.ExecutionException;  
import java.util.concurrent.FutureTask;  
  
public class CompletableFutureDemo {  
    public static void main(String[] args) throws ExecutionException, InterruptedException {  
  
        FutureTask<String> futureTask = new FutureTask<>(new MyThread2());  
        Thread thread = new Thread(futureTask,"t1");  
        thread.start();
        System.out.println(futureTask.get());  
    }  
}  
  
class MyThread2 implements Callable<String>{  
  
    @Override  
    public String call() throws Exception {  
        System.out.println("hello callable");  
        return "hello callable";  
    }  
}
```
 
 ### Future线程复用
 
```java
package com.java.bilibili.base;  
  
import java.util.concurrent.*;  
  
public class FutureThreadPoolDemo {  
  
    public static void main(String[] args) throws ExecutionException, InterruptedException {  
        FutureTask futureTask = new FutureTask(()->{  
            TimeUnit.MILLISECONDS.sleep(500);  
            return "task over";  
        });  
        Thread thread = new Thread(futureTask, "t1");  
        thread.start();  
          
        //线程复用  
        FutureTask futureTask2 = new FutureTask(()->{  
            TimeUnit.MILLISECONDS.sleep(500);  
            return "task over";  
        });  
        ExecutorService executorService = Executors.newFixedThreadPool(3);  
        executorService.submit(futureTask2);  
        System.out.println(futureTask.get());  
        System.out.println(futureTask2.get());  
        try {  
            TimeUnit.MILLISECONDS.sleep(500);  
        } catch (InterruptedException e) {  
            throw new RuntimeException(e);  
        }  
        executorService.shutdown();  
        System.out.println("程序执行结束");  
    }  
}
```

### future get 会导致线程阻塞  isDone 轮询获取结果耗费CPU
```java
package com.java.bilibili.base;  
  
import java.util.concurrent.ExecutionException;  
import java.util.concurrent.FutureTask;  
import java.util.concurrent.TimeUnit;  
import java.util.concurrent.TimeoutException;  
  
public class FutureAPIDemo {  
  
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {  
        FutureTask<String> futureTask = new FutureTask<>(() ->{  
            System.out.println(Thread.currentThread().getName() +"------- come in");  
            TimeUnit.SECONDS.sleep(5);  
            return "task over";  
        });  
        Thread t1 = new Thread(futureTask,"t1");  
        t1.start();  
  
//        System.out.println(futureTask.get()); //get 容易导致线程阻塞，  
//        System.out.println(futureTask.get(3,TimeUnit.SECONDS)); //过时不候自动离开,不建议使用  
  
        while (true){  
            if (futureTask.isDone()){  
                System.out.println(futureTask.get());  
                break;  
            }else {  
                TimeUnit.MILLISECONDS.sleep(500);  
            }  
        }  
        System.out.println(Thread.currentThread().getName() +"执行其他任务");  
    }  
}
```

## CompletableFuture
`CompletableFuture` 提供一种观察者模式类似的机制，可以让任务执行完成之后通知监听的一方
``CompletableFuture``实现了`CompletionStage`接口和`Future`接口，前者是对后者的一个扩展，增加了异步回调、流式处理、多个Future组合处理的能力，使Java在处理多任务的协同工作时更加顺畅便利。

### 创建异步任务

#### runAsync/SupplyAsync
 这两方法各有一个重载版本，可以指定执行异步任务的Executor实现，如果不指定，默认使用`ForkJoinPool.commonPool()`，如果机器是单核的，则默认使用`ThreadPerTaskExecutor`，该类是一个内部类，每次执行execute都会创建一个新线程，
 - `runAsync`表示创建无返回值的异步任务，相当于ExecutorService submit(Runnable task)方法
 -  `supplyAsync`执行`CompletableFuture任务`，支持返回值
 
```java
package com.java.bilibili.base;  
  
import java.util.concurrent.*;  
  
public class CompletableFutureBuildDemo {  
  
    public static void main(String[] args) throws ExecutionException, InterruptedException {  
        ExecutorService threadPool = Executors.newFixedThreadPool(3);//runAsync 无返回值  
        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {  
            System.out.println(Thread.currentThread().getName());  
            try {  
                TimeUnit.SECONDS.sleep(1);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        },threadPool);  
        System.out.println(completableFuture.get()); //输出null  
        threadPool.shutdown();  
  
        //supplyAsync 有返回值  
        ExecutorService threadPool1 = Executors.newFixedThreadPool(3);  
        CompletableFuture<Object> completableFuture1 = CompletableFuture.supplyAsync(() -> {  
            System.out.println(Thread.currentThread().getName());  
            try {  
                TimeUnit.SECONDS.sleep(1);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
            return "hello supplyAsync";  
        },threadPool1);  
        System.out.println(completableFuture1.get());  
        threadPool.shutdown();  
    }  
}
```

实际应用
```java
package com.java.bilibili.base;  
  
import lombok.AllArgsConstructor;  
import lombok.Data;  
import lombok.Getter;  
import lombok.NoArgsConstructor;  
import lombok.experimental.Accessors;  
  
import java.util.Arrays;  
import java.util.List;  
import java.util.concurrent.CompletableFuture;  
import java.util.concurrent.ThreadLocalRandom;  
import java.util.concurrent.TimeUnit;  
import java.util.stream.Collectors;  
  
public class CompletableFutureMallDemo {  
  
    static  List<NetMall> list = Arrays.asList(new NetMall("jd"),new NetMall("dangdang"),new NetMall("taobao"));  
    public static List<String> getPrice(List<NetMall> list, String productName){  
        return list.stream()  
                .map(netMall ->  
                        String.format(productName + "in %s price is %.2f",  
                                netMall.getName(),  
                                netMall.calcPrice(productName)))  
                .collect(Collectors.toList());  
    }  
  
    public static List<String> getPriceCompletableFuture(List<NetMall> list, String productName){  
        return list.stream()  
                .map(netMall -> CompletableFuture.supplyAsync( () ->  
                    String.format(productName + " in %s price is %.2f",  
                    netMall.getName(),  
                    netMall.calcPrice(productName))  
        )).collect(Collectors.toList())  
                .stream()  
                .map(s -> s.join())  
                .collect(Collectors.toList());  
    }  
  
    public static void main(String[] args) {  
  
        List<String> futures = getPriceCompletableFuture(list, "mysql");  
        for (String str: futures){  
            System.out.println(str);  
        }  
  
    }  
}  
  
  
@AllArgsConstructor  
@NoArgsConstructor  
@Data  
@Accessors(chain = true)  
class NetMall{  
  
    @Getter  
    private String name;  
  
  public  double calcPrice(String productName)  
  {  
      try {  
          TimeUnit.SECONDS.sleep(3);  
      } catch (InterruptedException e) {  
          throw new RuntimeException(e);  
      }  
      return ThreadLocalRandom.current().nextDouble()*2 +productName.charAt(0);  
  }  
  
}
```

### 主动计算
- **getNow(T valueIfAbsent)** 没有计算完成的情况下返回默认的结果，如果计算完成返回正常的计算结果
```java
package com.java.bilibili.base;  
  
import java.util.concurrent.CompletableFuture;  
import java.util.concurrent.TimeUnit;  
  
public class CompletableGetNowDemo {  
  
    public static void main(String[] args) {  
  
        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {  
  
            try {  
                TimeUnit.SECONDS.sleep(1);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
            return "abc";  
        });  
        System.out.println(completableFuture.getNow("xxx"));  
    }  
}
//输出
xxx
```

- **complete(T value)**  是否打断get() join()方法立即将complete 设置的value 作为get join 方法的结果返回W
```java
package com.java.bilibili.base;  
  
import java.util.concurrent.CompletableFuture;  
import java.util.concurrent.TimeUnit;  
  
public class CompletableCompleteDemo {  
  
    public static void main(String[] args) {  
  
        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {  
  
            try {  
                TimeUnit.SECONDS.sleep(1);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
            return "abc";  
        });  
        System.out.println(completableFuture.complete("xxxx")+ "\t" + completableFuture.join());  
    }  
}

//输出 
true	xxxx
```

### 异步回调

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1668228795868.png)

#### thenRun/thenRunAsync
CompletableFuture的thenRun方法，通俗点讲就是，**做完第一个任务后，再做第二个任务**。某个任务执行完成后，执行回调方法；但是前后两个任务**没有参数传递，第二个任务也没有返回值**
```java
public class FutureThenRunTest {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        CompletableFuture<String> orgFuture = CompletableFuture.supplyAsync(
                ()->{
                    System.out.println("先执行第一个CompletableFuture方法任务");
                    return "捡田螺的小男孩";
                }
        );

        CompletableFuture thenRunFuture = orgFuture.thenRun(() -> {
            System.out.println("接着执行第二个任务");
        });

        System.out.println(thenRunFuture.get());
    }
}
//输出
先执行第一个CompletableFuture方法任务
接着执行第二个任务
null
```

#### thenApply/thenApplyAsync 
 `thenApply` 表示某个任务执行完成后执行的动作，即回调方法，会将该任务的执行结果即方法返回值作为入参传递到回调方法中如果使用thenApplyAsync，那么执行的线程是从ForkJoinPool.commonPool()中获取不同的线程进行执行，如果使用thenApply，如果supplyAsync方法执行速度特别快，那么thenApply任务就是主线程进行执行，如果执行特别慢的话就是和supplyAsync执行线程一样。
计算结果存在依赖关系，线程串行化，**当前步骤出现异常，不继续走下一步**
```java
public class FutureThenApplyTest {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        CompletableFuture<String> orgFuture = CompletableFuture.supplyAsync(
                ()->{
                    System.out.println("原始CompletableFuture方法任务");
                    return "捡田螺的小男孩";
                }
        );

        CompletableFuture<String> thenApplyFuture = orgFuture.thenApply((a) -> {
            if ("捡田螺的小男孩".equals(a)) {
                return "关注了";
            }

            return "先考虑考虑";
        });

        System.out.println(thenApplyFuture.get());
    }
}
//输出
原始CompletableFuture方法任务
关注了

```

#### thenAccept / thenRun
`thenAccept` 同 `thenApply` 接收上一个任务的返回值作为参数，但是无返回值；thenRun 的方法没有入参，也没有返回值
```java
public class FutureThenAcceptTest {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        CompletableFuture<String> orgFuture = CompletableFuture.supplyAsync(
                ()->{
                    System.out.println("原始CompletableFuture方法任务");
                    return "捡田螺的小男孩";
                }
        );

        CompletableFuture thenAcceptFuture = orgFuture.thenAccept((a) -> {
            if ("捡田螺的小男孩".equals(a)) {
                System.out.println("关注了");
            }

            System.out.println("先考虑考虑");
        });

        System.out.println(thenAcceptFuture.get());
    }
}
```

#### whenComplete
CompletableFuture的whenComplete方法表示，某个任务执行完成后，执行的回调方法，**无返回值**；并且whenComplete方法返回的CompletableFuture的**result是上个任务的结果**。
```java
public class FutureWhenTest {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        CompletableFuture<String> orgFuture = CompletableFuture.supplyAsync(
                ()->{
                    System.out.println("当前线程名称：" + Thread.currentThread().getName());
                    try {
                        Thread.sleep(2000L);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    return "捡田螺的小男孩";
                }
        );

        CompletableFuture<String> rstFuture = orgFuture.whenComplete((a, throwable) -> {
            System.out.println("当前线程名称：" + Thread.currentThread().getName());
            System.out.println("上个任务执行完啦，还把" + a + "传过来");
            if ("捡田螺的小男孩".equals(a)) {
                System.out.println("666");
            }
            System.out.println("233333");
        });

        System.out.println(rstFuture.get());
    }
}
//输出
当前线程名称：ForkJoinPool.commonPool-worker-1
当前线程名称：ForkJoinPool.commonPool-worker-1
上个任务执行完啦，还把捡田螺的小男孩传过来
666
233333
捡田螺的小男孩

```

#### handle 
`handle`有异常也可以继续往下一步走，根据异常参数可以进一步处理，**某个任务执行完成后，执行回调方法，并且是有返回值的**，并且handle方法返回的CompletableFuture的result是**回调方法**执行的结果。
```java
package com.java.bilibili.base;  
  
import java.util.concurrent.CompletableFuture;  
import java.util.concurrent.ExecutionException;  
import java.util.concurrent.ThreadLocalRandom;  
  
public class CompletableFutureAPI2Demo {  
    public static void main(String[] args) throws ExecutionException, InterruptedException {  
  
        CompletableFuture<Integer> cf = CompletableFuture.supplyAsync(() -> {  
            System.out.println(Thread.currentThread().getName());  
            try {  
                Thread.sleep(2000);  
            } catch (InterruptedException e) {  
            }  
            int i = ThreadLocalRandom.current().nextInt(10);  
            System.out.println(i);  
            if (i>5) {  
             i = i/0;  
            } else {  
                System.out.println(Thread.currentThread().getName());  
                return i;  
            }  
            return i;  
        });  
        //cf执行完成后会将执行结果和执行过程中抛出的异常传入回调方法，如果是正常执行的则传入的异常为null,参数a是上一步正常的执行结果，参数b是上一步执行异常
        CompletableFuture<String> cf2 = cf.handle((a, b) -> {  
            System.out.println(Thread.currentThread().getName());  
            try {  
                Thread.sleep(2000);  
            } catch (InterruptedException e) {  
            }  
            if (b != null) {  
                b.printStackTrace();  
            } else {  
                System.out.println("run success,result->" + a);  
            }  
            System.out.println(Thread.currentThread() .getName());  
            if (b != null) {  
                return "run error";  
            } else {  
                return "run success";  
            }  
        });  
        //等待子任务执行完成  
        //get的结果是cf2的返回值，跟cf没关系了  
        System.out.println( cf2.get());  
    }  
}
```


#### exceptionally
`exceptionally`方法指定某个任务执行异常时执行的回调方法，会将抛出异常作为参数传递到回调方法中，如果该任务正常执行则会exceptionally方法返回的CompletionStage的result就是该任务正常执行的结果
```java
public class FutureExceptionTest {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        CompletableFuture<String> orgFuture = CompletableFuture.supplyAsync(
                ()->{
                    System.out.println("当前线程名称：" + Thread.currentThread().getName());
                    throw new RuntimeException();
                }
        );

        CompletableFuture<String> exceptionFuture = orgFuture.exceptionally((e) -> {
            e.printStackTrace();
            return "你的程序异常啦";
        });

        System.out.println(exceptionFuture.get());
    }
}
//输出
当前线程名称：ForkJoinPool.commonPool-worker-1
java.util.concurrent.CompletionException: java.lang.RuntimeException
	at java.util.concurrent.CompletableFuture.encodeThrowable(CompletableFuture.java:273)
	at java.util.concurrent.CompletableFuture.completeThrowable(CompletableFuture.java:280)
	at java.util.concurrent.CompletableFuture$AsyncSupply.run(CompletableFuture.java:1592)
	at java.util.concurrent.CompletableFuture$AsyncSupply.exec(CompletableFuture.java:1582)
	at java.util.concurrent.ForkJoinTask.doExec(ForkJoinTask.java:289)
	at java.util.concurrent.ForkJoinPool$WorkQueue.runTask(ForkJoinPool.java:1056)
	at java.util.concurrent.ForkJoinPool.runWorker(ForkJoinPool.java:1692)
	at java.util.concurrent.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:157)
Caused by: java.lang.RuntimeException
	at cn.eovie.future.FutureWhenTest.lambda$main$0(FutureWhenTest.java:13)
	at java.util.concurrent.CompletableFuture$AsyncSupply.run(CompletableFuture.java:1590)
	... 5 more
你的程序异常啦
```

### 组合处理

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1668229310563.png)

#### AND组合

##### thenCombine / thenAcceptBoth / runAfterBoth

这三个方法都是将两个CompletableFuture组合起来，只有这两个都正常执行完了才会执行某个任务，区别在于，
- `thenCombine`会将两个任务的执行结果作为方法入参传递到指定方法中，且该方法有返回值；
- `thenAcceptBoth`同样将两个任务的执行结果作为方法入参，但是无返回值；
- `runAfterBoth`没有入参，也没有返回值。注意两个任务中只要有一个执行异常，则将该异常信息作为指定任务的执行结果
```java
public class ThenCombineTest {

    public static void main(String[] args) throws InterruptedException, ExecutionException, TimeoutException {

        CompletableFuture<String> first = CompletableFuture.completedFuture("第一个异步任务");
        ExecutorService executor = Executors.newFixedThreadPool(10);
        CompletableFuture<String> future = CompletableFuture
                //第二个异步任务
                .supplyAsync(() -> "第二个异步任务", executor)
                // (w, s) -> System.out.println(s) 是第三个任务
                .thenCombineAsync(first, (s, w) -> {
                    System.out.println(w);
                    System.out.println(s);
                    return "两个异步任务的组合";
                }, executor);
        System.out.println(future.join());
        executor.shutdown();

    }
}
//输出
第一个异步任务
第二个异步任务
两个异步任务的组合
```

#### OR 组合

##### applyToEither / acceptEither / runAfterEither
 这三个方法都是将两个`CompletableFuture`组合起来，只要其中一个执行完了就会执行某个任务，其区别在于

 - `applyToEither`会将已经执行完成的任务的执行结果作为方法入参，并有返回值；
 - `acceptEither`同样将已经执行完成的任务的执行结果作为方法入参，但是没有返回值；
 - `runAfterEither`没有方法入参，也没有返回值。注意两个任务中只要有一个执行异常，则将该异常信息作为指定任务的执行结果

```java
package com.java.bilibili.base;  
  
import java.util.concurrent.CompletableFuture;  
import java.util.concurrent.ExecutionException;  
  
public class CompletableFutureAPIDemo {  
    public static void main(String[] args) throws ExecutionException, InterruptedException {  
        // 创建异步执行任务: 都使用的ForkJoinPool  
        CompletableFuture<Double> cf = CompletableFuture.supplyAsync(()->{  
            try {  
                Thread.sleep(2000);  
                System.out.println(Thread.currentThread().getName());  
            } catch (InterruptedException e) {  
            }  
            return 1.2;  
        });  
        CompletableFuture<Double> cf2 = CompletableFuture.supplyAsync(()->{  
            try {  
                Thread.sleep(1500);  
                int i = 1/0;  //抛出异常
                System.out.println(Thread.currentThread().getName());  
            } catch (InterruptedException e) {  
            }  
            return 3.2;  
        }).exceptionally(throwable -> {  
            System.out.println(throwable.getCause());  
            return 2.0;  
        });  
        //cf和cf2的异步任务都执行完成后，会将其执行结果作为方法入参传递给cf3,且有返回值  
        CompletableFuture<Double> cf3=cf.applyToEither(cf2,(result)->{  
            System.out.println("job3 param result->"+result);  
            try {  
                Thread.sleep(2000);  
            } catch (InterruptedException e) {  
            }  
            System.out.println(Thread.currentThread().getName());  
            return result;  
        });  
  
        System.out.println(cf3.get());  
    }  
}
//输出
java.lang.ArithmeticException: / by zero程序异常了
job3 param result->2.0
ForkJoinPool.commonPool-worker-1
ForkJoinPool.commonPool-worker-2
2.0
```



#### thenCompose

`thenCompose`方法会在某个任务执行完成后，将该任务的执行结果作为方法入参然后执行指定的方法，该方法会返回一个新的CompletableFuture实例

#### allOf
所有任务都执行完成后，才执行 allOf返回的CompletableFuture。如果任意一个任务异常，allOf的CompletableFuture，执行get方法，会抛出异常
```java
public class allOfFutureTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        CompletableFuture<Void> a = CompletableFuture.runAsync(()->{
            System.out.println("我执行完了");
        });
        CompletableFuture<Void> b = CompletableFuture.runAsync(() -> {
            System.out.println("我也执行完了");
        });
        CompletableFuture<Void> allOfFuture = CompletableFuture.allOf(a, b).whenComplete((m,k)->{
            System.out.println("finish");
        });
    }
}
//输出
我执行完了
我也执行完了
finish

```

#### anyOf
任意一个任务执行完，就执行anyOf返回的CompletableFuture。如果执行的任务异常，anyOf的CompletableFuture，执行get方法，会抛出异常
```java
public class AnyOfFutureTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        CompletableFuture<Void> a = CompletableFuture.runAsync(()->{
            try {
                Thread.sleep(3000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("我执行完了");
        });
        CompletableFuture<Void> b = CompletableFuture.runAsync(() -> {
            System.out.println("我也执行完了");
        });
        CompletableFuture<Object> anyOfFuture = CompletableFuture.anyOf(a, b).whenComplete((m,k)->{
            System.out.println("finish");
//            return "捡田螺的小男孩";
        });
        //join()和get()方法都是阻塞调用它们的线程（通常为主线程）来获取CompletableFuture异步之后的返回值。
        //get() 方法会抛出经检查的异常，可被捕获，自定义处理或者直接抛出。而 join() 会抛出未经检查的异常。
        anyOfFuture.join();
    }
}
//输出
我也执行完了
finish
```

### CompletableFuture使用有哪些注意点
#### 1. Future需要获取返回值，才能获取异常信息

```java
ExecutorService executorService = new ThreadPoolExecutor(5, 10, 5L,
    TimeUnit.SECONDS, new ArrayBlockingQueue<>(10));
CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> {
      int a = 0;
      int b = 666;
      int c = b / a;
      return true;
   },executorService).thenAccept(System.out::println);
   
 //如果不加 get()方法这一行，看不到异常信息
 //future.get();
复制代码
```

Future需要获取返回值，才能获取到异常信息。如果不加 get()/join()方法，看不到异常信息。小伙伴们使用的时候，注意一下哈,考虑是否加try...catch...或者使用exceptionally方法。

#### 2. CompletableFuture的get()方法是阻塞的。

`CompletableFuture`的get()方法是阻塞的，如果使用它来获取异步调用的返回值，需要添加超时时间~

```java
//反例
 CompletableFuture.get();
//正例
CompletableFuture.get(5, TimeUnit.SECONDS);
复制代码
```

#### 3. 默认线程池的注意点

`CompletableFuture`代码中又使用了默认的线程池，处理的线程个数是电脑CPU核数-1。在**大量请求过来的时候，处理逻辑复杂的话，响应会很慢**。一般建议使用自定义线程池，优化线程池配置参数。

1. 没有传入自定义线程池，都用默认线程池ForkJoinPool
2. 传入自定义线程池
 - 非`Aysny`方法执行第二个任务时，第二个和第一个线程池共用一个线程池
 - `Async`方法执行第二个任务时，第一个任务使用的是自定义线程池，第二个任务使用的是`ForkJoin`线程池
 3. 有可能处理太快，系统优化切换原则，直接使用main线程处理
```java
ExecutorService threadPool1 = new ThreadPoolExecutor(10, 10, 0L, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<>(100));  
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {  
    System.out.println("supplyAsync 执行线程：" + Thread.currentThread().getName());  
    //业务操作  
    return "";  
}, threadPool1);  
//此时，如果future1中的业务操作已经执行完毕并返回，则该thenApply直接由当前main线程执行；否则，将会由执行以上业务操作的threadPool1中的线程执行。  
future1.thenApply(value -> {  
    System.out.println("thenApply 执行线程：" + Thread.currentThread().getName());  
    return value + "1";  
});  
//使用ForkJoinPool中的共用线程池CommonPool  
future1.thenApplyAsync(value -> {  
//do something  
  return value + "1";  
});  
//使用指定线程池  
future1.thenApplyAsync(value -> {  
//do something  
  return value + "1";  
}, threadPool1);
```

#### 4. 自定义线程池时，注意饱和策略

`CompletableFuture`的get()方法是阻塞的，我们一般建议使用`future.get(3, TimeUnit.SECONDS)`。并且一般建议使用自定义线程池。

但是如果线程池拒绝策略是`DiscardPolicy`或者`DiscardOldestPolicy`，当线程池饱和时，会直接丢弃任务，不会抛弃异常。因此建议，CompletableFuture线程池策略**最好使用AbortPolicy**，然后耗时的异步线程，做好**线程池隔离**哈。

#### 5. 异步回调要传线程池
**强制传线程池，且根据实际情况做线程池隔离，不同业务员可分配不同的线程池**。

当不传递线程池时，会使用ForkJoinPool中的公共线程池CommonPool，这里所有调用将共用该线程池，核心线程数=处理器数量-1（单核核心线程数为1），所有异步回调都会共用该CommonPool，核心与非核心业务都竞争同一个池中的线程，很容易成为系统瓶颈。手动传递线程池参数可以更方便的调节参数，*并且可以给不同的业务分配不同的线程池*，以求资源隔离，减少不同业务之间的相互干扰。

#### 6.线程循环引用导致死锁
```java
public Object doGet() {  
  ExecutorService threadPool1 = new ThreadPoolExecutor(10, 10, 0L, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<>(100));  
  CompletableFuture cf1 = CompletableFuture.supplyAsync(() -> {  
  //do sth  
    return CompletableFuture.supplyAsync(() -> {  
        System.out.println("child");  
        return "child";  
      }, threadPool1).join();//子任务  
    }, threadPool1);  
  return cf1.join();  
}
```

如上代码块所示，doGet方法第三行通过supplyAsync向threadPool1请求线程，并且内部子任务又向threadPool1请求线程。threadPool1大小为10，当同一时刻有10个请求到达，则threadPool1被打满，子任务请求线程时进入阻塞队列排队，但是父任务的完成又依赖于子任务，这时由于子任务得不到线程，父任务无法完成。主线程执行cf1.join()进入阻塞状态，并且永远无法恢复。

为了修复该问题，需要将父任务与子任务做线程池隔离，两个任务请求不同的线程池，避免循环依赖导致的阻塞。

#### 7.get与join 方法的区别

**相同点**：
- join()和get()方法都是阻塞调用它们的线程（通常为主线程）来获取`CompletableFuture`异步之后的返回值。
```java
CompletableFuture.get() 和 CompletableFuture.join() 这两个方法是获取异步守护线程的返回值的。
```
**不同点**：
- get() 方法会抛出经检查的异常，可被捕获，自定义处理或者直接抛出，`ExecutionException`, `InterruptedException` 需要用户手动处理。
- join() 会抛出未经检查的异常，会将异常包装成`CompletionException`异常 /`CancellationException`异常，但是本质原因还是代码内存在的真正的异常，。


### 参考
- [异步编程利器：CompletableFuture详解 ｜Java 开发实战 - 掘金 (juejin.cn)](https://juejin.cn/post/6970558076642394142#heading-25)
- [尚硅谷2022版JUC并发编程（对标阿里P6-P7）_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1ar4y1x727/?spm_id_from=333.999.0.0&vd_source=a93579eb32613bed04d8b58488ca962a)
- [CompletableFuture原理与实践-美团外卖商家端API的异步化 (qq.com)](https://mp.weixin.qq.com/s/GQGidprakfticYnbVYVYGQ)
- [Java8 CompletableFuture 用法全解](https://blog.csdn.net/qq_31865983/article/details/106137777)

## 线程锁
### 悲观锁
`synchronized`关键字和`Lock` 的实现都是悲观锁 适合写操作的场景，用于对象，方法，代码块提供安全操作，属于独占式的悲观锁和可重入式锁
- synchronized作用范围
1. 对于普通同步方法，锁的是当前实例对象，所有的普通同步方法用的都是同一把锁一>实例对象本身
2. 对静态同步方法锁的是当前类的Class.对象，.class唯一的一个模板
3. 对于同步方法块，锁的是synchronized括号内的对象


### 乐观锁
适合在读操作场景
- 版本号机制version
- 采用CAS算法，就java原子类中的递增操作就通过CAS自旋来实现的

### 公平锁|非公平锁
公平指的式线程竞争锁的机制式公平的
**公平锁**：多个线程按照申请锁的顺序去获得锁，线程会直接进入队列去排队，永远都是队列的第一位才能得到锁。

优点：所有的线程都能得到资源，不会饿死在队列中。
缺点：吞吐量会下降很多，队列里面除了第一个线程，其他的线程都会阻塞，cpu唤醒阻塞线程的开销会很大。
```java
ReentrantLock lock = new ReentrantLock(true);
```

**非公平锁**：多个线程去获取锁的时候，会直接去尝试获取，获取不到，再去进入等待队列，如果能获取到，就直接获取到锁。
优点：可以减少CPU唤醒线程的开销，整体的吞吐效率会高点，CPU也不必取唤醒所有线程，会减少唤起线程的数量。
缺点：你们可能也发现了，这样可能导致队列中间的线程一直获取不到锁或者长时间获取不到锁，导致饿死。
```java
ReentrantLock lock = new ReentrantLock(false);
```
### 可重入锁
**可重入锁**： 也叫递归锁，是指在同一个线程中，在外层函数中获取到了该锁之后，内层函数仍然可以继续获取到该锁，ReentaantLock 和Synchronzied 都是可重入锁