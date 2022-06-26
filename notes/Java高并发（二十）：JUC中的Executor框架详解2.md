# Java高并发（二十）：JUC中的Executor框架详解2

## 需要解决的问题

举个例子说明

买新房了，然后在网上下单买冰箱、洗衣机，电器商家不同，所以送货耗时不一样，然后等他们送货，快递只愿送到楼下，然后我们自己将其搬到楼上的家中。

用程序来模拟上面的实现。示例代码如下：

```java
import java.util.concurrent.*;

public class Demo12 {
    static class GoodsModel {
        //商品名称
        String name;
        //购物开始时间
        long startime;
        //送到的时间
        long endtime;
        public GoodsModel(String name, long startime, long endtime) {
            this.name = name;
            this.startime = startime;
            this.endtime = endtime;
        }
        @Override
        public String toString() {
            return name + "，下单时间[" + this.startime + "," + endtime + "]，耗时:" + (this.endtime - this.startime);
        }
    }
    /**
     * 将商品搬上楼
     *
     * @param goodsModel
     * @throws InterruptedException
     */
    static void moveUp(GoodsModel goodsModel) throws InterruptedException {
        //休眠5秒，模拟搬上楼耗时
        TimeUnit.SECONDS.sleep(5);
        System.out.println("将商品搬上楼，商品信息:" + goodsModel);
    }
    /**
     * 模拟下单
     *
     * @param name     商品名称
     * @param costTime 耗时
     * @return
     */
    static Callable<GoodsModel> buyGoods(String name, long costTime) {
        return () -> {
            long startTime = System.currentTimeMillis();
            System.out.println(startTime + "购买" + name + "下单!");
            //模拟送货耗时
            try {
                TimeUnit.SECONDS.sleep(costTime);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            long endTime = System.currentTimeMillis();
            System.out.println(startTime + name + "送到了!");
            return new GoodsModel(name, startTime, endTime);
        };
    }
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        long st = System.currentTimeMillis();
        System.out.println(st + "开始购物!");
        //创建一个线程池，用来异步下单
        ExecutorService executor = Executors.newFixedThreadPool(5);
        //异步下单购买冰箱
        Future<GoodsModel> bxFuture = executor.submit(buyGoods("冰箱", 5));
        //异步下单购买洗衣机
        Future<GoodsModel> xyjFuture = executor.submit(buyGoods("洗衣机", 2));
        //关闭线程池
        executor.shutdown();
        //等待冰箱送到
        GoodsModel bxGoodModel = bxFuture.get();
        //将冰箱搬上楼
        moveUp(bxGoodModel);
        //等待洗衣机送到
        GoodsModel xyjGooldModel = xyjFuture.get();
        //将洗衣机搬上楼
        moveUp(xyjGooldModel);
        long et = System.currentTimeMillis();
        System.out.println(et + "货物已送到家里咯，哈哈哈！");
        System.out.println("总耗时:" + (et - st));
    }
}
```

输出：

```java
1564653121515开始购物!
1564653121588购买冰箱下单!
1564653121588购买洗衣机下单!
1564653121588洗衣机送到了!
1564653121588冰箱送到了!
将商品搬上楼，商品信息:冰箱，下单时间[1564653121588,1564653126590]，耗时:5002
将商品搬上楼，商品信息:洗衣机，下单时间[1564653121588,1564653123590]，耗时:2002
1564653136591货物已送到家里咯，哈哈哈！
总耗时:15076
```

从输出中我们可以看出几个时间：
1. 购买冰箱耗时5秒
2. 购买洗衣机耗时2秒
3. 将冰箱送上楼耗时5秒
4. 将洗衣机送上楼耗时5秒
5. 共计耗时15秒

购买洗衣机、冰箱都是异步执行的，我们先把冰箱送上楼了，然后再把冰箱送上楼了。上面大家应该发现了一个问题，洗衣机先到的，洗衣机到了，我们并没有去把洗衣机送上楼，而是在等待冰箱到货（bxFuture.get();），然后将冰箱送上楼，中间导致浪费了3秒，现实中应该是这样的，先到的先送上楼，修改一下代码，洗衣机先到的，先送洗衣机上楼，代码如下：

```java
import java.util.concurrent.*;

public class Demo13 {
    static class GoodsModel {
        //商品名称
        String name;
        //购物开始时间
        long startime;
        //送到的时间
        long endtime;
        public GoodsModel(String name, long startime, long endtime) {
            this.name = name;
            this.startime = startime;
            this.endtime = endtime;
        }
        @Override
        public String toString() {
            return name + "，下单时间[" + this.startime + "," + endtime + "]，耗时:" + (this.endtime - this.startime);
        }
    }
    /**
     * 将商品搬上楼
     *
     * @param goodsModel
     * @throws InterruptedException
     */
    static void moveUp(GoodsModel goodsModel) throws InterruptedException {
        //休眠5秒，模拟搬上楼耗时
        TimeUnit.SECONDS.sleep(5);
        System.out.println("将商品搬上楼，商品信息:" + goodsModel);
    }
    /**
     * 模拟下单
     *
     * @param name     商品名称
     * @param costTime 耗时
     * @return
     */
    static Callable<GoodsModel> buyGoods(String name, long costTime) {
        return () -> {
            long startTime = System.currentTimeMillis();
            System.out.println(startTime + "购买" + name + "下单!");
            //模拟送货耗时
            try {
                TimeUnit.SECONDS.sleep(costTime);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            long endTime = System.currentTimeMillis();
            System.out.println(endTime + name + "送到了!");
            return new GoodsModel(name, startTime, endTime);
        };
    }
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        long st = System.currentTimeMillis();
        System.out.println(st + "开始购物!");
        //创建一个线程池，用来异步下单
        ExecutorService executor = Executors.newFixedThreadPool(5);
        //异步下单购买冰箱
        Future<GoodsModel> bxFuture = executor.submit(buyGoods("冰箱", 5));
        //异步下单购买洗衣机
        Future<GoodsModel> xyjFuture = executor.submit(buyGoods("洗衣机", 2));
        //关闭线程池
        executor.shutdown();
        //等待洗衣机送到
        GoodsModel xyjGooldModel = xyjFuture.get();
        //将洗衣机搬上楼
        moveUp(xyjGooldModel);
        //等待冰箱送到
        GoodsModel bxGoodModel = bxFuture.get();
        //将冰箱搬上楼
        moveUp(bxGoodModel);
        long et = System.currentTimeMillis();
        System.out.println(et + "货物已送到家里咯，哈哈哈！");
        System.out.println("总耗时:" + (et - st));
    }
}
```

输出：

```java
1564653153393开始购物!
1564653153466购买洗衣机下单!
1564653153466购买冰箱下单!
1564653155467洗衣机送到了!
1564653158467冰箱送到了!
将商品搬上楼，商品信息:洗衣机，下单时间[1564653153466,1564653155467]，耗时:2001
将商品搬上楼，商品信息:冰箱，下单时间[1564653153466,1564653158467]，耗时:5001
1564653165469货物已送到家里咯，哈哈哈！
总耗时:12076
```

耗时12秒，比第一种少了3秒。

问题来了，上面是我们通过调整代码达到了最优效果，实际上，购买冰箱和洗衣机具体哪个耗时时间长我们是不知道的，怎么办呢，有没有什么解决办法？

## CompletionService接口

CompletionService相当于一个执行任务的服务，通过submit丢任务给这个服务，服务内部去执行任务，可以通过服务提供的一些方法获取服务中已经完成的任务。

接口内的几个方法：

方法 | 说明
---|---
Future<V> submit(Callable<V> task); | 用于向服务中提交有返回结果的任务，并返回Future对象
Future<V> submit(Runnable task, V result); | 用户向服务中提交有返回值的任务去执行，并返回Future对象
Future<V> take() throws InterruptedException; | 从服务中返回并移除一个已经完成的任务，如果获取不到，会一致阻塞到有返回值为止。此方法会响应线程中断。
Future<V> poll(); | 从服务中返回并移除一个已经完成的任务，如果内部没有已经完成的任务，则返回空，此方法会立即响应。
Future<V> poll(long timeout, TimeUnit unit) throws InterruptedException; | 尝试在指定的时间内从服务中返回并移除一个已经完成的任务，等待的时间超时还是没有获取到已完成的任务，则返回空。此方法会响应线程中断

通过submit向内部提交任意多个任务，通过take方法可以获取已经执行完成的任务，如果获取不到将等待。

## ExecutorCompletionService类

ExecutorCompletionService类是CompletionService接口的具体实现。

说一下其内部原理，ExecutorCompletionService创建的时候会传入一个线程池，调用submit方法传入需要执行的任务，任务由内部的线程池来处理；ExecutorCompletionService内部有个阻塞队列，任意一个任务完成之后，会将任务的执行结果（Future类型）放入阻塞队列中，然后其他线程可以调用它take、poll方法从这个阻塞队列中获取一个已经完成的任务，获取任务返回结果的顺序和任务执行完成的先后顺序一致，所以最先完成的任务会先返回。

看一下构造方法：

```java
public ExecutorCompletionService(Executor executor) {
        if (executor == null)
            throw new NullPointerException();
        this.executor = executor;
        this.aes = (executor instanceof AbstractExecutorService) ?
            (AbstractExecutorService) executor : null;
        this.completionQueue = new LinkedBlockingQueue<Future<V>>();
    }
```

构造方法需要传入一个Executor对象，这个对象表示任务执行器，所有传入的任务会被这个执行器执行。

completionQueue是用来存储任务结果的阻塞队列，默认用采用的是LinkedBlockingQueue，也支持开发自己设置。通过submit传入需要执行的任务，任务执行完成之后，会放入completionQueue中，有兴趣的可以看一下原码，还是很好理解的。

## 使用ExecutorCompletionService解决文章开头的问题

代码如下：

```java
import java.util.concurrent.*;

public class Demo14 {
    static class GoodsModel {
        //商品名称
        String name;
        //购物开始时间
        long startime;
        //送到的时间
        long endtime;
        public GoodsModel(String name, long startime, long endtime) {
            this.name = name;
            this.startime = startime;
            this.endtime = endtime;
        }
        @Override
        public String toString() {
            return name + "，下单时间[" + this.startime + "," + endtime + "]，耗时:" + (this.endtime - this.startime);
        }
    }
    /**
     * 将商品搬上楼
     *
     * @param goodsModel
     * @throws InterruptedException
     */
    static void moveUp(GoodsModel goodsModel) throws InterruptedException {
        //休眠5秒，模拟搬上楼耗时
        TimeUnit.SECONDS.sleep(5);
        System.out.println("将商品搬上楼，商品信息:" + goodsModel);
    }
    /**
     * 模拟下单
     *
     * @param name     商品名称
     * @param costTime 耗时
     * @return
     */
    static Callable<GoodsModel> buyGoods(String name, long costTime) {
        return () -> {
            long startTime = System.currentTimeMillis();
            System.out.println(startTime + "购买" + name + "下单!");
            //模拟送货耗时
            try {
                TimeUnit.SECONDS.sleep(costTime);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            long endTime = System.currentTimeMillis();
            System.out.println(endTime + name + "送到了!");
            return new GoodsModel(name, startTime, endTime);
        };
    }
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        long st = System.currentTimeMillis();
        System.out.println(st + "开始购物!");
        ExecutorService executor = Executors.newFixedThreadPool(5);
        //创建ExecutorCompletionService对象
        ExecutorCompletionService<GoodsModel> executorCompletionService = new ExecutorCompletionService<>(executor);
        //异步下单购买冰箱
        executorCompletionService.submit(buyGoods("冰箱", 5));
        //异步下单购买洗衣机
        executorCompletionService.submit(buyGoods("洗衣机", 2));
        executor.shutdown();
        //购买商品的数量
        int goodsCount = 2;
        for (int i = 0; i < goodsCount; i++) {
            //可以获取到最先到的商品
            GoodsModel goodsModel = executorCompletionService.take().get();
            //将最先到的商品送上楼
            moveUp(goodsModel);
        }
        long et = System.currentTimeMillis();
        System.out.println(et + "货物已送到家里咯，哈哈哈！");
        System.out.println("总耗时:" + (et - st));
    }
}
```

输出：

```java
1564653208284开始购物!
1564653208349购买冰箱下单!
1564653208349购买洗衣机下单!
1564653210349洗衣机送到了!
1564653213350冰箱送到了!
将商品搬上楼，商品信息:洗衣机，下单时间[1564653208349,1564653210349]，耗时:2000
将商品搬上楼，商品信息:冰箱，下单时间[1564653208349,1564653213350]，耗时:5001
1564653220350货物已送到家里咯，哈哈哈！
总耗时:12066
```

从输出中可以看出和我们希望的结果一致，代码中下单顺序是：冰箱、洗衣机，冰箱送货耗时5秒，洗衣机送货耗时2秒，洗衣机先到的，然后被送上楼了，冰箱后到被送上楼，总共耗时12秒，和期望的方案一样。

## 示例：执行一批任务，然后消费执行结果

代码如下：

```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.concurrent.*;
import java.util.function.Consumer;

public class Demo15 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        List<Callable<Integer>> list = new ArrayList<>();
        int taskCount = 5;
        for (int i = taskCount; i > 0; i--) {
            int j = i * 2;
            list.add(() -> {
                TimeUnit.SECONDS.sleep(j);
                return j;
            });
        }
        solve(executorService, list, a -> {
            System.out.println(System.currentTimeMillis() + ":" + a);
        });
        executorService.shutdown();
    }
    public static <T> void solve(Executor e, Collection<Callable<T>> solvers, Consumer<T> use) throws InterruptedException, ExecutionException {
        CompletionService<T> ecs = new ExecutorCompletionService<T>(e);
        for (Callable<T> s : solvers) {
            ecs.submit(s);
        }
        int n = solvers.size();
        for (int i = 0; i < n; ++i) {
            T r = ecs.take().get();
            if (r != null) {
                use.accept(r);
            }
        }
    }
}
```

输出：

```java
1564667625648:2
1564667627652:4
1564667629649:6
1564667631652:8
1564667633651:10
```

代码中传入了一批任务进行处理，最终将所有处理完成的按任务完成的先后顺序传递给Consumer进行消费了。

## 示例：异步执行一批任务，有一个完成立即返回，其他取消

### 方式1

使用ExecutorCompletionService实现，ExecutorCompletionService提供了获取一批任务中最先完成的任务结果的能力。

```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.concurrent.*;
import java.util.function.Consumer;

public class Demo16 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        long startime = System.currentTimeMillis();
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        List<Callable<Integer>> list = new ArrayList<>();
        int taskCount = 5;
        for (int i = taskCount; i > 0; i--) {
            int j = i * 2;
            String taskName = "任务"+i;
            list.add(() -> {
                TimeUnit.SECONDS.sleep(j);
                System.out.println(taskName+"执行完毕!");
                return j;
            });
        }
        Integer integer = invokeAny(executorService, list);
        System.out.println("耗时:" + (System.currentTimeMillis() - startime) + ",执行结果:" + integer);
        executorService.shutdown();
    }
    public static <T> T invokeAny(Executor e, Collection<Callable<T>> solvers) throws InterruptedException, ExecutionException {
        CompletionService<T> ecs = new ExecutorCompletionService<T>(e);
        List<Future<T>> futureList = new ArrayList<>();
        for (Callable<T> s : solvers) {
            futureList.add(ecs.submit(s));
        }
        int n = solvers.size();
        try {
            for (int i = 0; i < n; ++i) {
                T r = ecs.take().get();
                if (r != null) {
                    return r;
                }
            }
        } finally {
            for (Future<T> future : futureList) {
                future.cancel(true);
            }
        }
        return null;
    }
}
```

程序输出下面结果然后停止了：

```java
任务1执行完毕!
耗时:2072,执行结果:2
```

代码中执行了5个任务，使用CompletionService执行任务，调用take方法获取最先执行完成的任务，然后返回。在finally中对所有任务发送取消操作（future.cancel(true);），从输出中可以看出只有任务1执行成功，其他任务被成功取消了，符合预期结果。

### 方式2

其实ExecutorService已经为我们提供了这样的方法，方法声明如下：

```java
<T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
```

示例代码：

```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.concurrent.*;

public class Demo17 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        long startime = System.currentTimeMillis();
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        List<Callable<Integer>> list = new ArrayList<>();
        int taskCount = 5;
        for (int i = taskCount; i > 0; i--) {
            int j = i * 2;
            String taskName = "任务" + i;
            list.add(() -> {
                TimeUnit.SECONDS.sleep(j);
                System.out.println(taskName + "执行完毕!");
                return j;
            });
        }
        Integer integer = executorService.invokeAny(list);
        System.out.println("耗时:" + (System.currentTimeMillis() - startime) + ",执行结果:" + integer);
        executorService.shutdown();
    }
}
```

输出下面结果之后停止：

```java
任务1执行完毕!
耗时:2061,执行结果:2
```

输出结果和方式1中结果类似。
