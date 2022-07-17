# Java高并发（三十四）：谷歌提供的一些好用的并发工具类
> 关于并发方面的，juc已帮我们提供了很多好用的工具，而谷歌在此基础上做了扩展，使并发编程更容易，这些工具放在guava.jar包中。

## guava maven配置

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>27.0-jre</version>
</dependency>
```

## guava中常用几个类

- MoreExecutors：提供了一些静态方法，是对juc中的Executors类的一个扩展。
- Futures：也提供了很多静态方法，是对juc中Future的一个扩展。

## 案例1：异步执行任务完毕之后回调

```java
import com.google.common.util.concurrent.ListenableFuture;
import com.google.common.util.concurrent.ListeningExecutorService;
import com.google.common.util.concurrent.MoreExecutors;
import lombok.extern.slf4j.Slf4j;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

@Slf4j
public class Demo1 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //创建一个线程池
        ExecutorService delegate = Executors.newFixedThreadPool(5);
        try {
            ListeningExecutorService executorService = MoreExecutors.listeningDecorator(delegate);
            //异步执行一个任务
            ListenableFuture<Integer> submit = executorService.submit(() -> {
                log.info("{}", System.currentTimeMillis());
                //休眠2秒，默认耗时
                TimeUnit.SECONDS.sleep(2);
                log.info("{}", System.currentTimeMillis());
                return 10;
            });
            //当任务执行完毕之后回调对应的方法
            submit.addListener(() -> {
                log.info("任务执行完毕了，我被回调了");
            }, MoreExecutors.directExecutor());
            log.info("{}", submit.get());
        } finally {
            delegate.shutdown();
        }
    }
}
```

说明：

ListeningExecutorService接口继承于juc中的ExecutorService接口，对ExecutorService做了一些扩展，看其名字中带有Listening，说明这个接口自带监听的功能，可以监听异步执行任务的结果。通过MoreExecutors.listeningDecorator创建一个ListeningExecutorService对象，需传递一个ExecutorService参数，传递的ExecutorService负责异步执行任务。

ListeningExecutorService的submit方法用来异步执行一个任务，返回ListenableFuture，ListenableFuture接口继承于juc中的Future接口，对Future做了扩展，使其带有监听的功能。调用submit.addListener可以在执行的任务上添加监听器，当任务执行完毕之后会回调这个监听器中的方法。

ListenableFuture的get方法会阻塞当前线程直到任务执行完毕。

上面的还有一种写法，如下：

```java
import com.google.common.util.concurrent.*;
import lombok.extern.slf4j.Slf4j;
import org.checkerframework.checker.nullness.qual.Nullable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

@Slf4j
public class Demo2 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService delegate = Executors.newFixedThreadPool(5);
        try {
            ListeningExecutorService executorService = MoreExecutors.listeningDecorator(delegate);
            ListenableFuture<Integer> submit = executorService.submit(() -> {
                log.info("{}", System.currentTimeMillis());
                TimeUnit.SECONDS.sleep(4);
                //int i = 10 / 0;
                log.info("{}", System.currentTimeMillis());
                return 10;
            });
            Futures.addCallback(submit, new FutureCallback<Integer>() {
                @Override
                public void onSuccess(@Nullable Integer result) {
                    log.info("执行成功:{}", result);
                }
                @Override
                public void onFailure(Throwable t) {
                    try {
                        TimeUnit.MILLISECONDS.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    log.error("执行任务发生异常:" + t.getMessage(), t);
                }
            }, MoreExecutors.directExecutor());
            log.info("{}", submit.get());
        } finally {
            delegate.shutdown();
        }
    }
}
```

上面通过调用Futures的静态方法addCallback在异步执行的任务中添加回调，回调的对象是一个FutureCallback，此对象有2个方法，任务执行成功调用onSuccess，执行失败调用onFailure。

失败的情况可以将代码中int i = 10 / 0;注释去掉，执行一下可以看看效果。

## 示例2：获取一批异步任务的执行结果

```java
import com.google.common.util.concurrent.Futures;
import com.google.common.util.concurrent.ListenableFuture;
import com.google.common.util.concurrent.ListeningExecutorService;
import com.google.common.util.concurrent.MoreExecutors;
import lombok.extern.slf4j.Slf4j;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;
import java.util.stream.Collectors;

@Slf4j
public class Demo3 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        log.info("star");
        ExecutorService delegate = Executors.newFixedThreadPool(5);
        try {
            ListeningExecutorService executorService = MoreExecutors.listeningDecorator(delegate);
            List<ListenableFuture<Integer>> futureList = new ArrayList<>();
            for (int i = 5; i >= 0; i--) {
                int j = i;
                futureList.add(executorService.submit(() -> {
                    TimeUnit.SECONDS.sleep(j);
                    return j;
                }));
            }
            //获取一批任务的执行结果
            List<Integer> resultList = Futures.allAsList(futureList).get();
            //输出
            resultList.forEach(item -> {
                log.info("{}", item);
            });
        } finally {
            delegate.shutdown();
        }
    }
}
```

结果中按顺序输出了6个异步任务的结果，此处用到了Futures.allAsList方法，看一下此方法的声明：

```java
public static <V> ListenableFuture<List<V>> allAsList(
      Iterable<? extends ListenableFuture<? extends V>> futures)
```

传递一批ListenableFuture，返回一个ListenableFuture<List<V>>，内部将一批结果转换为了一个ListenableFuture对象。

## 示例3：一批任务异步执行完毕之后回调

异步执行一批任务，最后计算其和

```java
import com.google.common.util.concurrent.*;
import lombok.extern.slf4j.Slf4j;
import org.checkerframework.checker.nullness.qual.Nullable;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

@Slf4j
public class Demo4 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        log.info("star");
        ExecutorService delegate = Executors.newFixedThreadPool(5);
        try {
            ListeningExecutorService executorService = MoreExecutors.listeningDecorator(delegate);
            List<ListenableFuture<Integer>> futureList = new ArrayList<>();
            for (int i = 5; i >= 0; i--) {
                int j = i;
                futureList.add(executorService.submit(() -> {
                    TimeUnit.SECONDS.sleep(j);
                    return j;
                }));
            }
            ListenableFuture<List<Integer>> listListenableFuture = Futures.allAsList(futureList);
            Futures.addCallback(listListenableFuture, new FutureCallback<List<Integer>>() {
                @Override
                public void onSuccess(@Nullable List<Integer> result) {
                    log.info("result中所有结果之和：" + result.stream().reduce(Integer::sum).get());
                }
                @Override
                public void onFailure(Throwable t) {
                    log.error("执行任务发生异常:" + t.getMessage(), t);
                }
            }, MoreExecutors.directExecutor());
        } finally {
            delegate.shutdown();
        }
    }
}
```

代码中异步执行了一批任务，所有任务完成之后，回调了上面的onSuccess方法，内部对所有的结果进行sum操作。

通过guava提供的一些工具类，方便异步执行任务并进行回调.