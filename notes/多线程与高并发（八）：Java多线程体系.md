## 多线程与高并发（八）：Java多线程体系

### 1.线程的创建

#### 1.1 继承Thread
1. 定义Thread类的子类，并重写该类的run方法，该方法的方法体就代表了线程要完成的任务。因此把run()方法称为执行体；
2. 创建Thread子类的实例，即创建了线程对象；
3. 调用线程对象的start()方法来启动线程；

  ```java
  public class DemoThread extends Thread {
    public void run() {
      System.out.println("hello");
    }

    public static void main(String[] args) {
      new DemoThread().start();
    }
  }
  ```

#### 1.2 实现Runnable
1. 定义Runnable接口的实现类，并重写该接口的run()方法，该run()方法的方法体同样是该线程的线程执行体；
2. 创建Runnable实现类的实例，并以此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象；
3. 调用线程对象的start()方法来启动该线程；

```java
public class Demo implements Runnable {
  public void run() {
    System.out.println("hello");
  }
  public static void main(String[] args) {
    Demo de = new Demo();
    new Thread(de, "新线程").start();
  }
}
```

#### 1.3 实现Callable与Future
1. 创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值；

    ```java
    public interface Callable {
      V call() throws Exception;
    }
    ```

2. 创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call()方法的返回值。（FutureTask是一个包装器，它通过接受Callable来创建，它同时实现了Future和Runnable接口）；
3. 使用FutureTask对象作为Thread对象的target创建并启动新线程；
4. 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值；

  ```java
  public class Demo implements Callable<Integer> {
    public static void main(String[] args) {
      Demo demo = new Demo();
      FutureTask<Integer> ft = new FutureTask<>(demo);
      new Thread(ft, "有返回值的线程").start();
      ft.get();
    }

    @Override
    public Integer call() throws Exception {
      return 1;
    }
  }
  ```

### 常用线程池体系结构
1. Executor：线程池顶级接口；
2. ExecutorService：线程池次级接口，对Executor做了一些扩展，增加了一些功能；
3. ScheduleExecutorService：对ExecutorService做了一些扩展，增加了一些定时任务相关的功能；
4. AbstractExecutorService：抽象类，运用模板方法设计模式实现了一部分方法；
5. ThreadPoolExecutor：普通线程池类，包含最基本的一些线程池操作相关的方法实现；
6. ScheduledThreadPoolExecutor：定时任务线程池类，用于实现定时任务相关功能；
7. ForkJoinPool：新型线程池类，java7中新增的线程池类，基于工作窃取理论实现，运用与大任务拆小任务，任务无限多的场景；
8. Executors：线程池工具类，定义了一些快速实现线程池的方法；
