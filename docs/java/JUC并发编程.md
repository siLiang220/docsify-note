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
        thread.start();;  
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
 ###### future线程复用
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
###### future get 会导致线程阻塞  isDone 轮询获取结果耗费CPU
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
**CompletableFuture** 提供一种观察者模式类似的机制，可以让任务执行完成之后通知监听的一方
**CompletableFuture**实现了CompletionStage接口和Future接口，前者是对后者的一个扩展，增加了异步回调、流式处理、多个Future组合处理的能力，使Java在处理多任务的协同工作时更加顺畅便利。
###### runAsync/SupplyAsync
 这两方法各有一个重载版本，可以指定执行异步任务的Executor实现，如果不指定，默认使用ForkJoinPool.commonPool()，如果机器是单核的，则默认使用ThreadPerTaskExecutor，该类是一个内部类，每次执行execute都会创建一个新线程
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
###### 主动计算
- **getNow(T valueIfAbsent)** 没有计算完成的情况下valueIfAbsent
- **complete(T value)**  是否打断get 方法立即返回value
### 异步回调
###### thenApply/thenApplyAsync 
 `thenApply` 表示某个任务执行完成后执行的动作，即回调方法，会将该任务的执行结果即方法返回值作为入参传递到回调方法中
计算结果存在依赖关系，线程串行化，**当前步骤出现异常，不继续走下一步**
###### handle 
`handle`有异常也可以继续往下一步走，根据异常参数可以进一步处理
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
        //cf执行完成后会将执行结果和执行过程中抛出的异常传入回调方法，如果是正常执行的则传入的异常为null  
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
###### thenAccept / thenRun
 `thenAccept` 同 `thenApply` 接收上一个任务的返回值作为参数，但是无返回值；thenRun 的方法没有入参，也没有返回值
 ###### exceptionally
 `exceptionally`方法指定某个任务执行异常时执行的回调方法，会将抛出异常作为参数传递到回调方法中，如果该任务正常执行则会exceptionally方法返回的CompletionStage的result就是该任务正常执行的结果
 ### 组合处理
 ######  applyToEither / acceptEither / runAfterEither
  这三个方法都是将两个CompletableFuture组合起来，只要其中一个执行完了就会执行某个任务，其区别在于applyToEither会将已经执行完成的任务的执行结果作为方法入参，并有返回值；acceptEither同样将已经执行完成的任务的执行结果作为方法入参，但是没有返回值；runAfterEither没有方法入参，也没有返回值。注意两个任务中只要有一个执行异常，则将该异常信息作为指定任务的执行结果
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
```

  ##### thenCombine / thenAcceptBoth / runAfterBoth
   这三个方法都是将两个CompletableFuture组合起来，只有这两个都正常执行完了才会执行某个任务，区别在于，thenCombine会将两个任务的执行结果作为方法入参传递到指定方法中，且该方法有返回值；thenAcceptBoth同样将两个任务的执行结果作为方法入参，但是无返回值；runAfterBoth没有入参，也没有返回值。注意两个任务中只要有一个执行异常，则将该异常信息作为指定任务的执行结果
######  thenCompose
`thenCompose`方法会在某个任务执行完成后，将该任务的执行结果作为方法入参然后执行指定的方法，该方法会返回一个新的CompletableFuture实例
##### allOf / anyOf
`allOf`是等待所有任务完成，构造后CompletableFuture完成，`anyOf`是只要有一个任务完成，构造后CompletableFuture就完成 只有有一个任务执行异常，则返回的CompletableFuture执行get方法时会抛出异常，如果都是正常执行，则get返回null。

## 线程锁
#### 悲观锁
`synchronized`关键字和`Lock` 的实现都是悲观锁 适合写操作的场景，用于对象，方法，代码块提供安全操作，属于独占式的悲观锁和可重入式锁
- synchronized作用范围
1. 对于普通同步方法，锁的是当前实例对象，所有的普通同步方法用的都是同一把锁一>实例对象本身
2. 对静态同步方法锁的是当前类的Class.对象，.class唯一的一个模板
3. 对于同步方法块，锁的是synchronized括号内的对象

#### 乐观锁
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