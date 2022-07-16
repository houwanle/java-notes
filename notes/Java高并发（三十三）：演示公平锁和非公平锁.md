# Java高并发（三十三）：演示公平锁和非公平锁
> 主要用juc中的ReentrantLock来说一下公平锁和非公平锁的东西。

## 什么是公平锁、非公平锁？

公平锁和非公平锁体现在别人释放锁的一瞬间，如果前面已经有排队的，新来的是否可以插队，如果可以插队表示是非公平的，如果不可用插队，只能排在最后面，是公平的方式。

## 示例

测试公平锁和非公平锁的时候，可以这么来：在主线程中先启动一个t1线程，在t1里面获取锁，获取锁之后休眠一会，然后在主线中启动10个father线程去排队获取锁，然后在t1中释放锁代码的前面一步再启动一个线程，在这个线程内部再创建10个son线程，去获取锁，看看后面这10个son线程会不会排到上面10个father线程前面去，如果会表示插队了，说明是非公平的，如果不会，表示排队执行的，说明是公平的方式，示例代码如下：

```java
import lombok.extern.slf4j.Slf4j;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

@Slf4j
public class Demo8 {
    public static void main(String[] args) throws InterruptedException {
        //非公平锁
        test1(false);
        TimeUnit.SECONDS.sleep(4);
        log.info("------------------------------");
        //公平锁
        test1(true);
    }
    public static void test1(boolean fair) throws InterruptedException {
        ReentrantLock lock = new ReentrantLock(fair);
        Thread t1 = new Thread(() -> {
            lock.lock();
            try {
                log.info("start");
                TimeUnit.SECONDS.sleep(3);
                new Thread(() -> {
                    m1(lock, "son");
                }).start();
                log.info("end");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        });
        t1.setName("t1");
        t1.start();
        TimeUnit.SECONDS.sleep(1);
        m1(lock, "father");
    }
    public static void m1(ReentrantLock lock, String threadPre) {
        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(() -> {
                lock.lock();
                try {
                    log.info("获取到锁!");
                } finally {
                    lock.unlock();
                }
            });
            thread.setName(threadPre + "-" + i);
            thread.start();
        }
    }
}
```

> 运行代码可以创建一个springboot项目，需要安装lombook插件

上面代码中以son开头的线程在father线程之后启动的，分析一下结果：
1. test1(false);执行的是非公平锁的过程，看一下son的输出排到father前面去了，说明插队了，说明采用的是非公平锁的方式。
2. test1(true);执行的是公平锁的过程，看一下输出，son都是在father后面输出的，说明排队执行的，说明采用的是公平锁的方式。