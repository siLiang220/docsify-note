
## 1. 用户线程与守护线程

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
## 2. Future
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

## 3.CompletableFuture
`CompletableFuture` 提供一种观察者模式类似的机制，可以让任务执行完成之后通知监听的一方
``CompletableFuture``实现了`CompletionStage`接口和`Future`接口，前者是对后者的一个扩展，增加了异步回调、流式处理、多个Future组合处理的能力，使Java在处理多任务的协同工作时更加顺畅便利。

### 创建异步任务

#### runAsync/SupplyAsync
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1671191944490.png)


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

- `thenRun(Runnable action)` 它是同第一个任务同一个线程，所以当第一个任务结束后，他才开始运行
- `thenRunAsync(Runnable action,Executor executor)` 如果第一个任务传入的是自定义线程池，则第一个任务就是以自定义线程池运行，第二个任务使用的是`ForkJoin`线程池
- `thenRunAsync(Runnable action,Executor executor)` 执行时使用的是自定义的B线程池

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

-   如果该CompletableFuture实例的result不为null，则返回一个基于该result新的CompletableFuture实例；
-   如果该CompletableFuture实例为null，然后就执行这个新任务

```java
public class ThenComposeTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        CompletableFuture<String> f = CompletableFuture.completedFuture("第一个任务");
        //第二个异步任务
        ExecutorService executor = Executors.newSingleThreadExecutor();
        CompletableFuture<String> future = CompletableFuture
                .supplyAsync(() -> "第二个任务", executor)
                .thenComposeAsync(data -> {
                    System.out.println(data); return f; //使用第一个任务作为返回
                }, executor);
        System.out.println(future.join());
        executor.shutdown();

    }
}
//输出
第二个任务
第一个任务
```

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

## 主线程捕获其他线程产生的异常
### completeExceptionally

```java
public static String getTestResult()
{
	int i = 10/0;
	return "test";
}

public static void main(String[] args) {
	CompletableFuture<String> completableFuture  = new CompletableFuture();
	new Thread(()->{
		try {
			completableFuture.complete(getTestResult());
		} catch (Exception e) {
			System.out.println("get exception in side");
			completableFuture.completeExceptionally(e);
		}
	}).start();
	completableFuture.exceptionally(e->"we hava a exception"+e.getMessage())
			.thenAccept(u-> System.out.println(u));
}
```
### CompletableFuture使用注意点
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

### 实践

- 多线程处理List 数据
```java
@Test
    public void Test31(){
        // 1. 创建线程池
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        List<Integer> list = Arrays.asList(1, 2, 3);
        // 2. 提交任务，并调用join()阻塞等待所有任务执行完成
        CompletableFuture
                .allOf(
                        list.stream().map(key ->
                                CompletableFuture.runAsync(() -> {
                                    // 模拟处理业务
                                    try {
                                        Thread.sleep(1000L);
                                    } catch (InterruptedException e) {
                                    }
                                    System.out.println("多线程处理" + key);
                                }, executorService))
                                .toArray(CompletableFuture[]::new))
                .join();
        executorService.shutdown();
    }
```

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20221216193925.jpg)

- 多线程处理指定范围内数据
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20221216194134.jpg)

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20221216194143.jpg)

### 参考
- [异步编程利器：CompletableFuture详解 ｜Java 开发实战 - 掘金 (juejin.cn)](https://juejin.cn/post/6970558076642394142#heading-25)
- [尚硅谷2022版JUC并发编程（对标阿里P6-P7）_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1ar4y1x727/?spm_id_from=333.999.0.0&vd_source=a93579eb32613bed04d8b58488ca962a)
- [CompletableFuture原理与实践-美团外卖商家端API的异步化 (qq.com)](https://mp.weixin.qq.com/s/GQGidprakfticYnbVYVYGQ)
- [Java8 CompletableFuture 用法全解](https://blog.csdn.net/qq_31865983/article/details/106137777)
- [ 多线程的利器：CompletableFuture 你还可以这样使用多线程_Java编程Code的博客-CSDN博客](https://blog.csdn.net/qq_39664892/article/details/128297003#:~:text=1%20%E9%81%8D%E5%8E%86list%E9%9B%86%E5%90%88%EF%BC%8C%E6%8F%90%E4%BA%A4CompletableFuture%E4%BB%BB%E5%8A%A1%EF%BC%8C%E6%8A%8A%E7%BB%93%E6%9E%9C%E8%BD%AC%E6%8D%A2%E6%88%90%E6%95%B0%E7%BB%84%202%20%E5%86%8D%E6%8A%8A%E6%95%B0%E7%BB%84%E6%94%BE%E5%88%B0CompletableFuture%E7%9A%84allOf,%28%29%E6%96%B9%E6%B3%95%E9%87%8C%E9%9D%A2%203%20%E6%9C%80%E5%90%8E%E8%B0%83%E7%94%A8join%20%28%29%E6%96%B9%E6%B3%95%E9%98%BB%E5%A1%9E%E7%AD%89%E5%BE%85%E6%89%80%E6%9C%89%E4%BB%BB%E5%8A%A1%E6%89%A7%E8%A1%8C%E5%AE%8C%E6%88%90)

## 4. 线程锁
### 悲观锁
概念：认为自己在使用数据的时候一定有别的线程来修改数据,因此在获取数据的时候会先加锁,确保数据不会被别的线程修改。

`synchronized`关键字和`Lock` 的实现都是悲观锁 适合写操作的场景，用于对象，方法，代码块提供安全操作，属于独占式的悲观锁和可重入式锁
#### synchronized 关键字使用方式

**synchronized关键字解决的是多个线程之间访问资源的同步性，synchronized关键字可以保证被它修饰的方法或者代码
块在任意时刻只能有一个线程执行**。

1. 对于普通同步示例方法，锁的是当前实例对象，所有的普通同步方法用的都是同一把锁一>实例对象本身
2. 对静态同步方法锁的是当前类的Class.对象，`.class` 唯一的一个模板。进入同步代码前要获得 当前 `class` 的锁。因为静态成员不属于任何一个实例对象，是类成员（ `static` 表明这是该类的一个静态资源，不管 new 了多少个对象，只有一份）。所以，如果一个线程 A 调用一个实例对象的非静态 `synchronized` 方法，而线程 B 需要调用这个实例对象所属类的静态 `synchronized` 方法，是允许的，不会发生互斥现象，因为访问静态 `synchronized` 方法占用的锁是当前类的锁，而访问非静态 `synchronized` 方法占用的锁是当前实例对象锁。
3. 对于同步方法块，锁的是 `synchronized` 括号内的对象/类
>synchronized(this|object) 表示进入同步代码库前要获得给定对象的锁。  
>synchronized(类.class) 表示进入同步代码前要获得 当前 class 的锁  

总结：  
- `synchronized` 关键字加到 static 静态方法和 synchronized(class) 代码块上都是是给 Class 类上锁。  
- `synchronized` 关键字加到实例方法上是给对象实例上锁.
- 实例对象与.Class所的是两个不同的对象，不会有发生竞态条件
- 尽量不要使用 synchronized(String a) 因为 JVM 中，字符串常量池具有缓存功能！  
- 构造方法可以使用 `synchronized` 关键字修饰么？先说结论：**构造方法不能使用 `synchronized` 关键字修饰**。构造方法本身就属于线程安全的，不存在同步的构造方法一说。

#### Synchronized关键字的底层原理

##### 1. Synchronized同步代码块
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20210314181051606.png)

如果在方法中直接抛出异常就只有一个 `monitorexit`
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20210314181153474.png)

##### 2.Synchronized 普通同步方法
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/2021031418130816.png)

##### 3.synchronized静态同步方法
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20210314181650733.png)

#### synchronized 锁Monitor
1. Monitor其实是一种同步机制,它的义务是保证(在同一时间)只有一个线程可以访问被保护的数据和代码
2. JVM中同步时基于进入和退出的监视器对象(Monitor,管程),每个对象实例都有一个Monitor对象。
3. Monitor对象和JVM对象一起销毁,底层由C来实现
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/0852f9b97be844409eb8c0e0770b468b.png)
根据《Java虚拟机规范》的要求，在执行 `monitorenter` 指令时，首先要去尝试获取对象的锁。如果 这个对象没被锁定，或者当前线程已经持有了那个对象的锁，就把锁的计数器的值增加一，而在执行 `monitorexit `指令时会将锁计数器的值减一。一旦计数器的值为零，锁随即就被释放了。如果获取对象 锁失败，那当前线程就应当被阻塞等待，直到请求锁定的对象被持有它的线程释放为止。
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669560506775.png)


### 乐观锁
 概念:乐观锁认为自己在使用数据时不会有别的线程修改数据,所以不会添加锁,只是在更新数据的时候去判断之前有没有别的线程更新了这个数据。如果这个数据没有被更新,当前线程将自己修改的数据成功写入。如果数据已经被其他线程更新,则根据不同的实现方式执行不同的操作

适合在读操作场景
- 版本号机制version
- 采用CAS算法，就java原子类中的递增操作就通过CAS自旋来实现的

### 公平锁|非公平锁
公平指的式线程竞争锁的机制式公平的
**`synchronized` 和 `ReentrantLock` 默认是非公平锁**

#### 公平锁
多个线程按照申请锁的顺序去获得锁，线程会直接进入队列去排队，永远都是队列的第一位才能得到锁。

优点：所有的线程都能得到资源，不会饿死在队列中。
缺点：吞吐量会下降很多，队列里面除了第一个线程，其他的线程都会阻塞，cpu唤醒阻塞线程的开销会很大。
```java
ReentrantLock lock = new ReentrantLock(true);
```

#### 非公平锁
多个线程去获取锁的时候，会直接去尝试获取，获取不到，再去进入等待队列，如果能获取到，就直接获取到锁。
优点：可以减少CPU唤醒线程的开销，整体的吞吐效率会高点，CPU也不必取唤醒所有线程，会减少唤起线程的数量。
缺点：这样可能导致队列中间的线程一直获取不到锁或者长时间获取不到锁，导致饿死。
```java
ReentrantLock lock = new ReentrantLock(false);
```

>[!tip]
> ReentrantLock 中的lock和unlock方法一定要一 一匹配,如果少了或多了,都会坑到别的线程

示例:
```java
package com.bilibili.juc.locks;

import java.util.concurrent.locks.ReentrantLock;

class Ticket {
    private int number = 50;

    private Lock lock = new ReentrantLock(true); //默认用的是非公平锁，分配的平均一点，=--》公平一点
    public void sale() {
        lock.lock();
        try {
            if(number > 0) {
                System.out.println(Thread.currentThread().getName()+"\t 卖出第: "+(number--)+"\t 还剩下: "+number);
            }
        }finally {
            lock.unlock();
        }
    }
    /*Object objectLock = new Object();

    public void sale(){
        synchronized (objectLock)
        {
            if(number > 0)
            {
                System.out.println(Thread.currentThread().getName()+"\t 卖出第: "+(number--)+"\t 还剩下: "+number);
            }
        }
    }*/
}
public class SaleTicketDemo {
    public static void main(String[] args) {
        Ticket ticket = new Ticket();
        new Thread(() -> { for (int i = 1; i <=55; i++) ticket.sale(); },"a").start();
        new Thread(() -> { for (int i = 1; i <=55; i++) ticket.sale(); },"b").start();
        new Thread(() -> { for (int i = 1; i <=55; i++) ticket.sale(); },"c").start();
        new Thread(() -> { for (int i = 1; i <=55; i++) ticket.sale(); },"d").start();
        new Thread(() -> { for (int i = 1; i <=55; i++) ticket.sale(); },"e").start();
    }
}
```

#### 为什么会有公平锁、非公平锁设计？

恢复挂起的线程到真正的锁获取还有时间差，所以非公平锁可以更充分的利用CPU的时间片，尽量减少CPU的空闲时间，使用多线程很重要的考量点是线程开销

总结就是公平和效率


### 可重入锁
**可重入锁**： 也叫递归锁，是指在同一个线程中，在外层函数中获取到了该锁之后，再进入内层方法会自动获取锁（前提锁对象是同一把锁），不会因为之前获取到锁还没有释放而阻塞，`ReentrantLock` 和 `Synchronized` 都是可重入锁。

**例如：** 如果有一个  修饰的递归方法，程序进入第二次进入的时候被自己阻塞，出现作茧自缚。 

**优点：** 自己可以自动获取到自己的锁，一定程度可以避免死锁

##### 隐式锁
`synchronized` 关键字使用的锁，`synchronized` 修饰的方法或代码快内部调用本类的其他`synchronized` 修饰的方法或代码块，是永远得到锁的，不会发生死锁
```java
package com.bilibili.juc.locks;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;


public class ReEntryLockDemo
{
    public synchronized void m1()
    {
        //指的是可重复可递归调用的锁，在外层使用锁之后，在内层仍然可以使用，并且不发生死锁，这样的锁就叫做可重入锁。
        System.out.println(Thread.currentThread().getName()+"\t ----come in");
        m2();
		// 如果可以打印出 end m1说明没有发生死锁，结果是可以打印的
        System.out.println(Thread.currentThread().getName()+"\t ----end m1");
    }
    public synchronized void m2()
    {
        System.out.println(Thread.currentThread().getName()+"\t ----come in");
        m3();
    }
    public synchronized void m3()
    {
        System.out.println(Thread.currentThread().getName()+"\t ----come in");
    }

    static Lock lock = new ReentrantLock();

    public static void main(String[] args)
    {
        ReEntryLockDemo reEntryLockDemo = new ReEntryLockDemo();

        new Thread(() -> {
            reEntryLockDemo.m1();
        },"t1").start();
    }
}
```

##### 显示锁
Lock  锁，`ReentrantLock` 是可重入锁

```java
package com.bilibili.juc.locks;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;


public class ReEntryLockDemo
{
    public synchronized void m1()
  

    static Lock lock = new ReentrantLock();

    public static void main(String[] args)
    {
        new Thread(() -> {
            lock.lock();
            try
            {
                System.out.println(Thread.currentThread().getName()+"\t ----come in外层调用");
                lock.lock();
                try
                {
                    System.out.println(Thread.currentThread().getName()+"\t ----come in内层调用");
                }finally {
                    lock.unlock();
                }

            }finally {
	            lock.unlock()
                // 由于加锁次数和释放次数不一样，第二个线程始终无法获取到锁，导致一直在等待。
                //lock.unlock();// 正常情况，加锁几次就要解锁几次
            }
        },"t1").start();

		//测试缺失lock.unlock()导致t2无法获得锁，t2无法执行
        new Thread(() -> {
            lock.lock();
            try
            {
                System.out.println(Thread.currentThread().getName()+"\t ----come in外层调用");
            }finally {
                lock.unlock();
            }
        },"t2").start();
    }
}

//输出
t1    ----come in外层调用
t1    ----come in内层调用

```

## 5. 死锁及排查
### 死锁
1. 什么是死锁
死锁是指两个或两个以上的线程执行过程中，因争夺资源而造成的一种**相互等待的现象**，若无外力干涉他们将无法推进下去，如果资源充足，死锁出现的可能性很低，否则就会因争夺有限的资源而陷入死锁。
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669557524660.png)

2. 死锁的产生原因
	- 系统资源不足
	- 进程运行推进的顺序不合适
	- 资源分配不当

3. 代码示例

```java
public class DeadLockDemo{

    static Object lockA = new Object();
    static Object lockB = new Object();
    public static void main(String[] args){
        Thread a = new Thread(() -> {
            synchronized (lockA) {
                System.out.println(Thread.currentThread().getName() + "\t" + " 自己持有A锁，期待获得B锁");

                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                synchronized (lockB) {
                    System.out.println(Thread.currentThread().getName() + "\t 获得B锁成功");
                }
            }
        }, "A");
        a.start();

        new Thread(() -> {
            synchronized (lockB){
            
                System.out.println(Thread.currentThread().getName()+"\t"+" 自己持有B锁，期待获得A锁");

                try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

                synchronized (lockA){
                
                    System.out.println(Thread.currentThread().getName()+"\t 获得A锁成功");
                }
            }
        },"B").start();
    }
}

//输出
A    自己持有A锁期待获得B锁
B    自己持有B锁，希望获得A锁
```
线程 A 通过 `synchronized (lockA)` 获得 `LockA` 的监视器锁，然后通过`Thread.sleep(1000);`让线程 A 休眠 1s 为的是让线程 B 得到执行然后获取到 `LockB` 的监视器锁。线程 A 和线程 B 休眠结束了都开始企图请求获取对方的资源，然后这两个线程就会陷入互相等待的状态，这也就产生了死锁。

### 死锁的四个必要条件：

1.  互斥条件：该资源任意一个时刻只由一个线程占用。
2.  请求与保持条件：一个线程因请求资源而阻塞时，对已获得的资源保持不放。
3.  不剥夺条件：线程已获得的资源在未使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。
4.  循环等待条件： 若干线程之间形成一种头尾相接的循环等待资源关系。

---

著作权归所有 原文链接：https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html

### 死锁排查
- 使用jstack

```shell
>jps
10048 Launcher
6276 DeadLockDemo
6332 Jps
9356
>jstack 6276 (最后面有一个发现了一个死锁)
2021-07-28 16:05:36
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.111-b14 mixed mode):

"DestroyJavaVM" #16 prio=5 os_prio=0 tid=0x0000000003592800 nid=0x830 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"b" #15 prio=5 os_prio=0 tid=0x00000000253d5000 nid=0x1ba8 waiting for monitor entry [0x0000000025c8e000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.xiaozhi.juc.DeadLockDemo.lambda$main$1(DeadLockDemo.java:31)
        - waiting to lock <0x0000000741404050> (a java.lang.Object)
        - locked <0x0000000741404060> (a java.lang.Object)
        at com.xiaozhi.juc.DeadLockDemo$$Lambda$2/2101440631.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:745)

"a" #14 prio=5 os_prio=0 tid=0x00000000253d3800 nid=0xad8 waiting for monitor entry [0x0000000025b8e000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.xiaozhi.juc.DeadLockDemo.lambda$main$0(DeadLockDemo.java:20)
        - waiting to lock <0x0000000741404060> (a java.lang.Object)
        - locked <0x0000000741404050> (a java.lang.Object)
        at com.xiaozhi.juc.DeadLockDemo$$Lambda$1/1537358694.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:745)

"Service Thread" #13 daemon prio=9 os_prio=0 tid=0x000000002357b800 nid=0x1630 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread3" #12 daemon prio=9 os_prio=2 tid=0x00000000234f6000 nid=0x1fd4 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread2" #11 daemon prio=9 os_prio=2 tid=0x00000000234f3000 nid=0x5c0 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" #10 daemon prio=9 os_prio=2 tid=0x00000000234ed800 nid=0x1afc waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #9 daemon prio=9 os_prio=2 tid=0x00000000234eb800 nid=0x2ae0 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"JDWP Command Reader" #8 daemon prio=10 os_prio=0 tid=0x0000000023464800 nid=0xc50 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"JDWP Event Helper Thread" #7 daemon prio=10 os_prio=0 tid=0x000000002345f800 nid=0x1b0c runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"JDWP Transport Listener: dt_socket" #6 daemon prio=10 os_prio=0 tid=0x0000000023451000 nid=0x2028 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Attach Listener" #5 daemon prio=5 os_prio=2 tid=0x000000002343f800 nid=0x1ea0 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 tid=0x00000000233eb800 nid=0x10dc runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=1 tid=0x00000000233d3000 nid=0xafc in Object.wait() [0x000000002472f000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x0000000741008e98> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:143)
        - locked <0x0000000741008e98> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:164)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)

"Reference Handler" #2 daemon prio=10 os_prio=2 tid=0x0000000021d0d000 nid=0x28ec in Object.wait() [0x000000002462f000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x0000000741006b40> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:502)
        at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
        - locked <0x0000000741006b40> (a java.lang.ref.Reference$Lock)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

JNI global references: 2504


Found one Java-level deadlock:
=============================
"b":
  waiting to lock monitor 0x0000000021d10b58 (object 0x0000000741404050, a java.lang.Object),
  which is held by "a"
"a":
  waiting to lock monitor 0x0000000021d13498 (object 0x0000000741404060, a java.lang.Object),
  which is held by "b"

Java stack information for the threads listed above:
===================================================
"b":
        at com.xiaozhi.juc.DeadLockDemo.lambda$main$1(DeadLockDemo.java:31)
        - waiting to lock <0x0000000741404050> (a java.lang.Object)
        - locked <0x0000000741404060> (a java.lang.Object)
        at com.xiaozhi.juc.DeadLockDemo$$Lambda$2/2101440631.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:745)
"a":
        at com.xiaozhi.juc.DeadLockDemo.lambda$main$0(DeadLockDemo.java:20)
        - waiting to lock <0x0000000741404060> (a java.lang.Object)
        - locked <0x0000000741404050> (a java.lang.Object)
        at com.xiaozhi.juc.DeadLockDemo$$Lambda$1/1537358694.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:745)

Found 1 deadlock.

```

- 使用jconsole (命令行输入jconsole,选择进程，点击线程，再选择检测死锁按钮)
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669559500001.png)



## 6. 线程中断

### 线程中断机制
什么是中断机制
- 一个线程不应该由其他线程来强制中断或停止，而是应该由线程自己自行停止，自己来决定自己的命运。所以，Thread.stop,Thread.suspend,Thread.resume都已经被废弃了。

- 在Java中没有办法立即停止一条线程，然而停止线程却显得尤为重要，如取消一个耗时操作。因此，Java提供了一种用于停止线程的协商机制一一中断，也即**中断标识协商机制**。

- 中断只是一种协作协商机制，Java没有给中断增加任何语法，中断的过程完全需要程序员自己实现。

- 若要中断一个线程，你需要手动调用该线程的interrupt方法，该方法也仅仅是将线程对象的中断标识设成true;
接着你需要自己写代码不断地检测当前线程的标识位，如果为true,表示别的线程请求这条线程中断，此时究竞该做什么需要你自己写代码实现。

- 每个线程对象中都有一个中断标识位，用于表示线程是否被中断；该标识位为true表示中断，为false表示未中断；通过调用线程对象的interrupt方法将该线程的标识位设为true;可以在别的线程中调用，也可以在自己的线程中调用。

#### 中断API

##### interrupt()实例方法
- **如果线程处于正常状态**，仅仅设置线程的中断状态为true,发起一个协商而不会立即停止线程，可当前线程中断，也可其他线程中断

- 如果该线程处于被阻塞状态wait(),wait(long),或wait(long,int)的方法0bject类，或join(),join(long),join(long,int),sleep(long),或sleep(Long,int),在别的线程中调用当前线程的 `interrupt` 方法那么当前线程的**中断状态将被清除**，并且抛出`InterruptedException`。简单来说就是当**阻塞**方法收到**中断**请求的时候就会抛出 `InterruptedException` 异常，并清除中断状态 .
 >[!tip]
 > 要切记在catch 需要再做`interrupt` 防止线程死循环

- **中断不活动的线程不受影响**



![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20210314182351852.png)

```java
public class InterruptDemo3
{
    public static void main(String[] args)
    {
        Thread t1 = new Thread(() -> {
            while (true)
            {
                if(Thread.currentThread().isInterrupted())
                {
                    System.out.println(Thread.currentThread().getName()+"\t " +
                            "中断标志位："+Thread.currentThread().isInterrupted()+" 程序停止");
                    break;
                }

                try {
                    Thread.sleep(200);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt(); //sleep阻塞状态下被t2线程调用interuppt,会抛出InterruptedException 并清除线程的中断状态，所以需要再中断一次.如果不处理isInterrupted = false会导致死循环
                    e.printStackTrace();
                }

                System.out.println("-----hello InterruptDemo3");
            }
        }, "t1");
        t1.start();

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> t1.interrupt(),"t2").start();
    }
}
```

##### interrupted()静态方法
判断线程是否中断并清除当前的中断状态
1. 返回当前线程的中断状态，测试当前线程是否已被中断
2. 将当前线程的中断状态清零并重新设置为false，清除线程的中断状态
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669604554068.png)


##### isInterrupted()实例方法
- 判断当前线程是否被中断（通过检查中断标志位）
- 已经销毁的线程不受影响,返回false

#### 如何中断运行中的线程

##### volatile变量实现
```java   
public class InterruptDemo
{
    static volatile boolean isStop = false;
 
    public static void main(String[] args)
    {
        Thread t1 = new Thread(() -> {
            while (true)
            {
                if(isStop)
                {
                    System.out.println(Thread.currentThread().getName()+"\t isInterrupted()被修改为true，程序停止");
                    break;
                }
                System.out.println("t1 -----hello volatile ");
            }
        }, "t1");
        t1.start();

        //暂停毫秒
        try { TimeUnit.MILLISECONDS.sleep(20); } catch (InterruptedException e) { e.printStackTrace(); }

        //当t2 线程将isStop 设为true 时，t1线程停止
        new Thread(() -> {
         isStop = true
        },"t2").start();
    }
}    
```

##### AtomicBoolean实现
```java
public class InterruptDemo
{
    static AtomicBoolean atomicBoolean = new AtomicBoolean(false);

    public static void main(String[] args)
    {
        
        new Thread(() -> {
            while (true)
            {
                if(atomicBoolean.get())
                {
                    System.out.println(Thread.currentThread().getName()+"\t atomicBoolean被修改为true，程序停止");
                    break;
                }
                System.out.println("t1 -----hello atomicBoolean");
            }
        },"t1").start();

        //暂停毫秒
        try { TimeUnit.MILLISECONDS.sleep(20); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            atomicBoolean.set(true);
        },"t2").start();
    }
}
```

##### interrupt实现
```java
public class InterruptDemo
{
    static volatile boolean isStop = false;
    static AtomicBoolean atomicBoolean = new AtomicBoolean(false);

    public static void main(String[] args)
    {
        Thread t1 = new Thread(() -> {
            while (true)
            {
                if(Thread.currentThread().isInterrupted())
                {
                    System.out.println(Thread.currentThread().getName()+"\t isInterrupted()被修改为true，程序停止");
                    break;
                }
                System.out.println("t1 -----hello interrupt api");
            }
        }, "t1");
        t1.start();

        System.out.println("-----t1的默认中断标志位："+t1.isInterrupted());

        //暂停毫秒
        try { TimeUnit.MILLISECONDS.sleep(20); } catch (InterruptedException e) { e.printStackTrace(); }

        //t2向t1发出协商，将t1的中断标志位设为true希望t1停下来
        new Thread(() -> {
            t1.interrupt();
        },"t2").start();
        //也可以t1自己中断标志位
        //t1.interrupt();
    }
}  
```

### 线程等待和唤醒

#### 线程唤醒的方法

##### 1. 使用Object 的wait()方法让线程等待，使用Object 的 notify()方法唤醒
```java
    private static void syncWaitNotify()
    {
        Object objectLock = new Object();

        new Thread(() -> {
            //try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); } //测试先执行notify 后执行wait
            synchronized (objectLock){
                System.out.println(Thread.currentThread().getName()+"\t ----come in");
                try {
                    objectLock.wait(); //wait 会释放锁 所以t2线程可以执行synchroized
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+"\t ----被唤醒");
            }
        },"t1").start();

        //暂停几秒钟线程
        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            synchronized (objectLock){
                objectLock.notify();
                System.out.println(Thread.currentThread().getName()+"\t ----发出通知");
            }
        },"t2").start();
    }
//输出
t1   ----come in
t2   ----发出通知
t1   ----被唤醒
如果去掉 synchronized (objectLock) 代码块 会报管程状态异常，wait 和notify必须使用synchronized 包裹
如果notify方法先执行，wait方法后会导致程序无法唤醒
```

##### 2.使用JUC包中Condition的await()方法让线程等待，使用signal()方法唤醒
```java
private static void lockAwaitSignal()
{
	Lock lock = new ReentrantLock();
	Condition condition = lock.newCondition();

	new Thread(() -> {
		//测试先执行signal 后执行await 也是会导致程序线程阻塞
		//try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
		lock.lock();
		try
		{
			System.out.println(Thread.currentThread().getName()+"\t ----come in");
			condition.await(); //让线程进入等待队列并释放锁
			System.out.println(Thread.currentThread().getName()+"\t ----被唤醒");
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	},"t1").start();

	//暂停几秒钟线程
	try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

	new Thread(() -> {
		lock.lock();
		try
		{
			condition.signal();
			System.out.println(Thread.currentThread().getName()+"\t ----发出通知");
		}finally {
			lock.unlock();
		}
	},"t2").start();
}
//输出
t1   ----come in
t2   ----发出通知
t1   ----被唤醒
```

await 和signal 也必须在加锁的情况下使用

##### 3.`LockSupport` 类可以阻塞当前线程以及唤醒指定被阻塞的线程
`LockSupport` 类使用了一种名为Permit（许可）的概念来做到阻塞和唤醒线程的功能，每个线程都有关联一个许可（permit）

###### park 阻塞线程或阻塞传入的线程
- 如果有许可证，则直接消耗这个许可证，然后正常退出
- 如果无许可证，就必须等待许可证
-
###### unpark(Thread thread) 唤醒处于阻塞状态的指定线程
会增加一个许可证，但许可证最多只能有1个，累加无效

```java
public static void main(String[] args)
    {
        Thread t1 = new Thread(() -> {
		    //测试t2提前唤醒，程序可以正常执行，upspark会为其提供凭证
            //try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
            System.out.println(Thread.currentThread().getName() + "\t ----come in"+System.currentTimeMillis());
            LockSupport.park();
            System.out.println(Thread.currentThread().getName() + "\t ----被唤醒"+System.currentTimeMillis());
        }, "t1");
        t1.start();

        //暂停几秒钟线程
	    try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

        new Thread(() -> {
            LockSupport.unpark(t1);
            System.out.println(Thread.currentThread().getName()+"\t ----发出通知");
        },"t2").start();

    }
```


**总结**

- 通过对象Object 和`Condition` 使用的限制条件
	 - 线程先要获得并持有锁，必须在锁块（`synchroinzed`或`lock`）中
	- 必须要先等待后唤醒，线程才能够被唤醒 
- `LockSupport`唤醒
	- 无论先唤醒还是先等待程序都可以执行，但先执行了`upspark`就会导致`park`方法形同虚设无效 没有`park`拦截了
	- 每个线程只有一个关联许可证，不会累计最多只有一个

## 7. java内存模型 JMM

CPU的运行并不是直接操作内存（cpu和物理内存的速度不一致）而是先把内存里边的数据先写到缓存，而内存的读和写操作就会造成不一致的问题
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669636168023.png)


java内存模型是抽象的概念描述的是一组约定或规范，是**屏蔽各种硬件和操作系统的内存访问差异**，以实现让java程序在各种平台下都能达到一致的内存访问效果 

**原则：** *JMM 的关键技术是围绕多线程的原子性、可见性和有序性*展开的

### JMM 三大特性

#### 可见性
- 当一个线程修改了某一个共享变量，其他线程能够立即知道变更，JMM规定了所有的变量存储在**主内存**中
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669639345066.png)


#### 原子性
-   指一个操作是不可中断的,即多线程坏境下,操作不能被其他线程干扰

#### 有序性
- 计算机在执行程序时,为了提高性能,编译器和处理器常常会做指令重排，只要程序最终的执行结果与它顺序化执行的结果相同
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669641937019.png)

- 单线程坏境里面确保程序最终执行结果和代码顺序执行的结果一致
- 处理器在进行重新排序是必须要考虑指令之间的数据依赖性
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669642135274.png)

- 多线程坏境中线程交替执行,由于编译器优化重排的存在,两个线程使用的变量能否保持一致是无法确认的,结果无法预测（在特殊场景下要禁止指令重排，保证程序执行的有序性）

### 多线程对变量的读写过程
由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存（有些地方称为栈空间），工作内存是每个线程的私有数据区域，而Java内存模型中规定所有变量都存储在主内存，主内存是共享内存区域，所有线程都可以访问，但*线程对变量的操作(读取赋值等)必须在工作内存中进行，首先要将变量从主内存拷贝到的线程自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存*，不能直接操作主内存中的变量，各个线程中的工作内存中存储着主内存中的**变量副本拷贝**，因此不同的线程间无法访问对方的工作内存，线程间的通信（传值）必须通过主内存来完成，其简要访问过程如下图
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669687881077.png)


### 多线程先行发生原则happens-before

#### 先行发生原则说明
- 如果Java内存模型中有序性仅靠volatile和synchronized来完成,那么有很多操作都将会变得非常啰嗦,但是我们在编写Java并发代码的时候并没有察觉到这一点

-  我们没有时时、处处、次次,添加volatile和synchronized来完成程序,这是因为Java语言中JMM原则下,有一个"先行发生"(Happens-Before)的原则限制和规则

- 在JMM中,如果一个操作执行的结果需要对另一个操作可见性或者代码重排序,那么这两个操作之间必须存在happens-before关系
- 案例说明 (y=0 还是y=5)
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1669700040239.png)



#### heppens-before 总原则
 - 如果一个操作happens-before另一个操作,那么第一个操作的执行结果对第二个操作可见,而且第一个操作的执行顺序排在第二个操作之前(可见性,有序性)
- 两个操作之间存在happens-before关系,并不意外着一定要按照happens-before原则制定的顺序来执行。如果重排序之后的执行结果与按照happens-before关系来执行的结果一致,那么这种重排序并不非法(可以指令重排)

(值日:周一张三周二李四,假如有事情调换班可以的1+2+3=3+2+1)

#### happens-before之8条
①. 次序规则
一个线程内,按照代码顺序,写在前面的操作先行发生于写在后面的操作(强调的是一个线程)
前一个操作的结果可以被后续的操作获取。将白点就是前面一个操作把变量X赋值为1,那后面一个操作肯定能知道X已经变成了1

②. 锁定规则
(一个unlock操作先行发生于后面((这里的"后面"是指时间上的先后))对同一个锁的lock操作(上一个线程unlock了,下一个线程才能获取到锁,进行lock))

③. volatile变量规则
(对一个volatile变量的写操作先行发生于后面对这个变量的读操作,前面的写对后面的读是可见的,这里的"后面"同样是指时间是的先后)

④. 传递规则
(如果操作A先行发生于操作B,而操作B又先行发生于操作C,则可以得出A先行发生于操作C)

⑤. 线程启动规则(Thread Start Rule)
(Thread对象的start( )方法先行发生于线程的每一个动作)

⑥. 线程中断规则(Thread Interruption Rule)

对线程interrupt( )方法的调用先发生于被中断线程的代码检测到中断事件的发生；可以通过Thread.interrupted( )检测到是否发生中断；也就是说先调用interrupt()方法设置中断标志位，才能检测中断是否发送

⑦. 线程终止规则(Thread Termination Rule)
(线程中的所有操作都先行发生于对此线程的终止检测)

⑧. 对象终结规则(Finalizer Rule)
(对象没有完成初始化之前,是不能调用finalized( )方法的 )

## 8. Volatile
特点：可见性、有序性（通过禁止重排序）

### Volatile 内存语义

1. 当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值立即刷新回主内存中。
2. 当读一个volatile变量时，JMM会把该线程对应的本地内存设置为无效，直接从主内存中读取共享变量
3. 所以volatile的写内存语义是直接刷新到主内存中，读的内存语义是直接从主内存中读取

### 内存屏障


①. 内存屏障（也称内存栅栏,内存栅障,屏障指令等,是一类同步屏障指令,是CPU或编译器在对内存随机访问的操作中的一个同步点,使得此点之前的所有读写操作都执行后才可以开始执行此点之后的操作）,避免代码重排序。内存屏障其实就是一种JVM指令,Java内存模型的重排规则会要求Java编译器在生成JVM指令时插入特定的内存屏障指令,通过这些内存屏障指令,volatile实现了Java内存模型中的可见性和有序性,但**volatile无法保证原子性**

②. 内存屏障之前的所有写操作都要回写到主内存
内存屏障之后的所有读操作都能获得内存屏障之前的所有写操作的最新结果(实现了可见性)

③. 一句话:对一个volatile域的写, happens-before于任意后续对这个volatile域的读,也叫写后读

#### 内存屏障的分类

##### 粗分类
##### 读屏障
在读指令之前插入读屏障，让工作内存或CPU高速缓存当中的缓存数据失效，重新回到主内存中获取最新数据

##### 写屏障
在写指令之后插入写屏障，强制把写缓冲区的数 据刷回到主内存中

## ThreadLocal

### ThreadLocal 是什么

-  ThreadLocal本地线程变量,线程自带的变量副本(实现了每一个线程副本都有一个专属的本地变量,主要解决的就是让每一个线程绑定自己的值,自己用自己的,不跟别人争抢。通过使用get()和set()方法,获取默认值或将其值更改为当前线程所存的副本的值从而避免了线程安全的问题)
- synchronized或者lock,有个管理员,好比,现在大家签到,多个同学(线程),但是只有一只笔,只能同一个时间,只有一个线程(同学)签到,加锁(同步机制是以时间换空间,执行时间不一样,类似于排队)
- ThreadLocal,人人有份,每个同学手上都有一支笔,自己用自己的,不用再加锁来维持秩序(同步机制是以空间换时间,为每一个线程都提供了一份变量的副本,从而实现同时访问,互不干扰同时访问,肯定效率高啊)

### ThreadLocal使用场景

1.  每个线程都需要有属于自己的实例数据（线程隔离）；
2.  框架跨层数据的传递；
3.  需要参数全局传递的复杂调用链路的场景；
4.  数据库连接的管理，在AOP的各种嵌套调用中保证事务的一致性；

### API介绍
1. protected T initialValue():initialValue():返回此线程局部变量的当前线程的"初始值"
(对于initialValue()较为老旧,jdk1.8又加入了withInitial()方法)

2. `static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier)`:创建线程局部变量

3. T get():返回当前线程的此线程局部变量的副本中的值

```java
/**
 * Returns the value in the current thread's copy of this
 * thread-local variable.  If the variable has no value for the
 * current thread, it is first initialized to the value returned
 * by an invocation of the {@link #initialValue} method.
 *
 * @return the current thread's value of this thread-local
 */
public T get() {
	// 1. 获取当前线程
	Thread t = Thread.currentThread();
	// 2. 获取当前线程内部的ThreadLocalMap变量t.threadLocals;
	ThreadLocalMap map = getMap(t);
	// 3. 判断map是否为null
	if (map != null) {
		// 4. 使用当前threadLocal变量获取entry
		ThreadLocalMap.Entry e = map.getEntry(this);
		// 5. 判断entry是否为null
		if (e != null) {
	// 6.返回Entry.value
			@SuppressWarnings("unchecked")
			T result = (T) e.value;
			return result;
		}
	}
	// 7. 如果map/entry为null设置初始值
	return setInitialValue();
}

/**
 * Variant of set() to establish initialValue. Used instead
 * of set() in case user has overridden the set() method.
 *
 * @return the initial value
 */
private T setInitialValue() {
	// 1. 初始化value，如果重写就用重写后的value，默认null
	T value = initialValue();
	// 2. 获取当前线程
	Thread t = Thread.currentThread();
	// 3. 获取当前线程内部的ThreadLocalMap变量
	ThreadLocalMap map = getMap(t);
	if (map != null)
		// 4. 不为null就set, key: threadLocal, value: value
		map.set(this, value);
	else
		// 5. map若为null则创建ThreadLocalMap对象
		createMap(t, value);
	return value;
}

/**
 * Create the map associated with a ThreadLocal. Overridden in
 * InheritableThreadLocal.
 *
 * @param t          the current thread
 * @param firstValue value for the initial entry of the map
 */
void createMap(Thread t, T firstValue) {
	t.threadLocals = new ThreadLocalMap(this, firstValue);
}

/**
 * Construct a new map initially containing (firstKey, firstValue).
 * ThreadLocalMaps are constructed lazily, so we only create
 * one when we have at least one entry to put in it.
 */
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
	// 1. 初始化entry数组，size: 16
	table = new Entry[INITIAL_CAPACITY];
	// 2. 计算value的index
	int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
	// 3. 在对应index位置赋值
	table[i] = new Entry(firstKey, firstValue);
	// 4. entry size
	size = 1;
	// 5. 设置threshold: threshold = len * 2 / 3;
	setThreshold(INITIAL_CAPACITY);
}

/**
 * Set the resize threshold to maintain at worst a 2/3 load factor.
 */
private void setThreshold(int len) {
	threshold = len * 2 / 3;
}
```

4. void set(T value):将当前线程的此线程局部变量的副本设置为指定的值

```java
/**
 * Sets the current thread's copy of this thread-local variable
 * to the specified value.  Most subclasses will have no need to
 * override this method, relying solely on the {@link #initialValue}
 * method to set the values of thread-locals.
 *
 * @param value the value to be stored in the current thread's copy of
 *              this thread-local.
 */
public void set(T value) {
	// 1. 获取当前线程
	Thread t = Thread.currentThread();
	// 2. 获取当前线程内部的ThreadLocalMap变量
	ThreadLocalMap map = getMap(t);
	if (map != null)
		// 3. 设置value
		map.set(this, value);
	else
		// 4. 若map为null则创建ThreadLocalMap
		createMap(t, value);
}
```

5. void remove():删除此线程局部变量的当前线程的值
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20230212161037.png)


### ThreadLocal可能带来的问题

#### 1.  ThreadLocalMap中的Entry为什么要设计为弱引用类型？
若使用强引用类型，则threadlocal的引用链为：Thread -> ThreadLocal.ThreadLocalMap -> Entry[] -> Entry -> key（threadLocal对象）和value；在这种场景下，只要这个线程还在运行（如线程池场景），若不调用remove方法，则该对象及关联的所有强引用对象都不会被垃圾回收器回收。

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20230212160850.png)

#### 2.  使用static和不使用static修饰threadlocal变量有和区别？
若使用static关键字进行修饰，则一个线程仅对应一个线程变量；否则，threadlocal语义变为perThread-perInstance，容易引发内存泄漏，如下述示例：
```java
public class ThreadLocalTest {
    public static class ThreadLocalDemo {
        private ThreadLocal<String> threadLocalHolder = new ThreadLocal();

        public void setValue(String value) {
            threadLocalHolder.set(value);
        }

        public String getValue() {
            return threadLocalHolder.get();
        }
    }

    public static void main(String[] args) {
        int count = 3;
        List<ThreadLocalDemo> list = new LinkedList<>();
        for (int i = 0; i < count; i++) {
            ThreadLocalDemo demo = new ThreadLocalDemo();
            demo.setValue("demo-" + i);
            list.add(demo);
        }
        System.out.println();
    }
}
```

在上述main方法第22行debug，可见线程的threadLocals变量中有3个threadlocal实例。在工程实践中，使用threadlocal时通常期望一个线程只有一个threadlocal实例，因此，若不使用static修饰，期望的语义发生了变化，同时易引起内存泄漏。

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1675257747094.png)

###  InheritableThreadLocal


>ThreadLocal的不可继承性指的是子线程无法继承获取到父线程中的值。而`InheritableThreadLocal`则通过继承ThreadLocal类实现了可继承性

```java
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class InheritableThreadLocalDemo {
    /**
     * ThreadLocal继承性测试
     */
    @Test
    public void test1() {
        System.out.println("\n--------------------------- Test 1 ---------------------------");
        ThreadLocal<String> userInfo = new ThreadLocal<>();
        userInfo.set("Aaron");
        new Thread( ()-> System.out.println("<子线程> userInfo: " + userInfo.get()) )
            .start();
        System.out.println("<父线程> userInfo: " + userInfo.get());
    }
    /**
     * InheritableThreadLocal继承性测试
     */
    @Test
    public void test2() {
        System.out.println("\n--------------------------- Test 2 ---------------------------");
        InheritableThreadLocal<String> userInfo = new InheritableThreadLocal<>();
        userInfo.set("Aaron");
        new Thread( ()-> System.out.println("<子线程> userInfo: " + userInfo.get()) )
            .start();
        System.out.println("<父线程> userInfo: " + userInfo.get());
    }
}
```
测试结果

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/uTools_1676188746755.png)

>[!tip]
>**使用线程池时，父子线程传递慎用，因为初始化时机为线程创建时**

![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20230212160455.png)

针对线程池可使用阿里开源`TransmittableThreadLocal`
- 源码：[GitHub - alibaba/transmittable-thread-local: ](https://github.com/alibaba/transmittable-thread-local)
- 简述：[TransmittableThreadLocal详解 - 简书 (jianshu.com)](https://www.jianshu.com/p/e0774f965aa3)

### 最佳实践

#### ThreadLocal变量值初始化和清理建议成对出现

如果不执行清理操作，则可能会出现：

1. 内存泄漏：由于ThreadLocalMap的中key是弱引用，而Value是强引用。这就导致了一个问题，ThreadLocal在没有外部对象强引用时，发生GC时弱引用Key会被回收，而Value不会回收，从而Entry里面的元素出现<null,value>的情况。如果创建ThreadLocal的线程一直持续运行，那么这个Entry对象中的value就有可能一直得不到回收，这样可能会导致内存泄露，使用完毕后需要remove。
   
2.  脏数据：由于线程复用，在用户1请求时，可能保存了业务数据在ThreadLocal中，若不清理，则用户2的请求进来时，可能会读到用户1的数据。

建议使用try...finally 进行清理。
 
#### ThreadLocal变量建议使用static进行修饰

我们在使用ThreadLocal时，通常期望的语义是perThread，若不使用static进行修饰，则语义变为perThread-perInstance；在线程池场景下，若不用static进行修饰，创建的线程相关实例可能会达到 M * N个（其中M为线程数，N为对应类的实例数），易造成内存泄漏(https://errorprone.info/bugpattern/ThreadLocalUsage)
  
#### 谨慎使用ThreadLocal.withInitial
```java
// 反例，实际上使用了共享对象obj而并未隔离，
private static ThreadLocal<Obj> threadLocal = ThreadLocal.withIntitial(() -> obj)
```

## JUC强大的工具类

### CountDownLatch(闭锁) 做减法

-   `CountDownLatch`主要有两个方法,当一个或多个线程调用await方法时,这些线程会阻塞
    
-   其它线程调用`countDown`方法会将计数器减1(调用`countDown`方法的线程不会阻塞)
    
-   计数器的值变为0时,因`await`方法阻塞的线程会被唤醒,继续执行

```java
//需求:要求6个线程都执行完了,mian线程最后执行
public class CountDownLatchDemo {
    public static void main(String[] args) throws Exception{
        CountDownLatch countDownLatch=new CountDownLatch(6);
        for (int i = 1; i <=6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"\t");
                countDownLatch.countDown();
            },i+"").start();
        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName()+"\t班长关门走人,main线程是班长");
    }
}
```

- 利用枚举减少if else 判断
```java
public enum CountryEnum {

    one(1,"齐"),two(2,"楚"),three(3,"燕"),
    four(4,"赵"),five(5,"魏"),six(6,"韩");

    private Integer retCode;
    private String retMessage;

    private CountryEnum(Integer retCode,String retMessage){
        this.retCode=retCode;
        this.retMessage=retMessage;
    }

    public static CountryEnum getCountryEnum(Integer index){
        CountryEnum[] countryEnums = CountryEnum.values();
        for (CountryEnum countryEnum : countryEnums) {
            if(countryEnum.getRetCode()==index){
                return countryEnum;
            }
        }
        return null;
    }

    public Integer getRetCode() {
        return retCode;
    }

    public String getRetMessage() {
        return retMessage;
    }
}
```

```java
/*
	楚	**国,被灭
	魏	**国,被灭
	赵	**国,被灭
	燕	**国,被灭
	齐	**国,被灭
	韩	**国,被灭
	main	**秦国一统江湖
* */
public class CountDownLatchDemo {
    public static void main(String[] args) throws Exception{
        CountDownLatch countDownLatch=new CountDownLatch(6);
        for (int i = 1; i <=6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"\t"+"**国,被灭");
                countDownLatch.countDown();
            },CountryEnum.getCountryEnum(i).getRetMessage()).start();

        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName()+"\t"+"**秦国一统江湖");
    }
}

```

```java
/*
	楚	**国,被灭
	魏	**国,被灭
	赵	**国,被灭
	燕	**国,被灭
	齐	**国,被灭
	韩	**国,被灭
	main	**秦国一统江湖
* */
public class CountDownLatchDemo {
    public static void main(String[] args) throws Exception{
        CountDownLatch countDownLatch=new CountDownLatch(6);
        for (int i = 1; i <=6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"\t"+"**国,被灭");
                countDownLatch.countDown();
            },CountryEnum.getCountryEnum(i).getRetMessage()).start();

        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName()+"\t"+"**秦国一统江湖");
    }
}
```

### CyclicBarrier 做加法

- `CyclicBarrier`的字面意思是可循环(Cyclic) 使用的屏障(barrier)。它要做的事情是,让一组线程到达一个屏障(也可以叫做同步点)时被阻塞,知道最后一个线程到达屏障时,屏障才会开门,所有被屏障拦截的线程才会继续干活,线程进入屏障通过`CyclicBarrier`的await()方法
```java
   //集齐7颗龙珠就能召唤神龙
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        // public CyclicBarrier(int parties, Runnable barrierAction) {}
        CyclicBarrier cyclicBarrier=new CyclicBarrier(7,()->{
            System.out.println("召唤龙珠");
        });
        for (int i = 1; i <=7; i++) {
            final int temp=i;
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"\t收集到了第"+temp+"颗龙珠");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

### Semaphore 信号量

1. `acquire`(获取)当一个线程调用acquire操作时,它要么通过成功获取信号量(信号量减1),要么一直等下去,直到有线程释放信号量,或超时。

2. release(释放)实际上会将信号量的值加1,然后唤醒等待的线程。

3. 信号量主要用于两个目的,一个是用于多个共享资源的互斥使用,另一个用于并发线程数的控制。

```java
public class SemaphoreDemo {
    public static void main(String[] args) {
        Semaphore semaphore=new Semaphore(3);
        for (int i = 1; i <=6; i++) {
            new Thread(()->{
                try {
                    System.out.println(Thread.currentThread().getName()+"\t抢占了车位");
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"\t离开了车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();
                }
            },String.valueOf(i)).start();
        }
    }
}
```

### 实际使用
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.scheduling.annotation.Async;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.context.request.async.DeferredResult;

import java.util.Map;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ConcurrentHashMap;

@RestController
@RequestMapping("/api")
@EnableAsync
public class AsyncController {

    private final Map<String, DeferredResult<String>> deferredResultMap = new ConcurrentHashMap<>();

    @Autowired
    private AsyncService asyncService;

    @GetMapping("/startTask")
    public DeferredResult<ResponseEntity<String>> startTask() {
        String taskId = "Task1";
        DeferredResult<ResponseEntity<String>> deferredResult = new DeferredResult<>();

        CompletableFuture<String> completableFuture = asyncService.executeAsyncTask(taskId);

        completableFuture.whenComplete((result, ex) -> {
            if (ex != null) {
                deferredResult.setErrorResult(ex.getMessage());
            } else {
                deferredResult.setResult(ResponseEntity.ok("Task completed. Result: " + result));
            }
        });

        deferredResultMap.put(taskId, deferredResult);
        return deferredResult;
    }

    @GetMapping("/checkTask")
    public ResponseEntity<String> checkTask() {
        String taskId = "Task1";
        DeferredResult<String> deferredResult = deferredResultMap.get(taskId);

        if (deferredResult != null) {
            if (deferredResult.hasResult()) {
                return ResponseEntity.ok("Task Result: " + deferredResult.getResult());
            } else {
                return ResponseEntity.ok("Task is still in progress");
            }
        } else {
            return ResponseEntity.ok("Task not found");
        }
    }
}

@Service
class AsyncService {

    @Async
    public CompletableFuture<String> executeAsyncTask(String taskId) {
        return CompletableFuture.supplyAsync(() -> {
            simulateTaskExecution(taskId);
            return "Task completed";
        });
    }

    private void simulateTaskExecution(String taskId) {
        try {
            Thread.sleep(3000); // Simulate task execution time
            System.out.println("Task " + taskId + " executed");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```