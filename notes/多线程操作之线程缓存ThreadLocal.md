# 多线程操作之线程缓存ThreadLocal

## java.lang.ThreadLocal的作用
ThreadLocal可以保证当前拿到的变量属于当前访问的线程。也就是每个线程自己的独立小空间。实现了线程之间的数据隔离。

## 例子

```java
public class ThreadLocalTest {

  private static ThreadLocal<String> tl = new ThreadLocal<String>();
    
    public static void main(String[] args) {
      tl.set("哈哈");
        
      new Thread(new Runnable() {

        @Override
        public void run() {
          System.out.println(Thread.currentThread().getName() + ":" + tl.get());
        }
      }).start();
        
        System.out.println(Thread.currentThread().getName() + ":" + tl.get());
    }
}
```

运行结果：

```java
main:哈哈
Thread-0:null
```

解释：

虽然主线程 main 往 ThreadLocal 里面设置了值并且其也可以拿到值，但是线程 Thread-0 却没有拿到值，可以看出线程之间的数据隔离。

## 源码分析

### set 方法

```java
public void set(T value) {
        //获取当前线程
        Thread t = Thread.currentThread();
        //获取当前线程对应的本地Map映射
        ThreadLocalMap map = getMap(t);
        //如果map不为空
        if (map != null)
            //将值设置到本地map中
            map.set(this, value);
        else//如果map为空
            //创建线程本地map，将value值进行初始化 
            createMap(t, value);
}

void createMap(Thread t, T firstValue) {
        //初始化线程本地Map
        t.threadLocals = new ThreadLocalMap(this, firstValue);
}

ThreadLocalMap(ThreadLocal firstKey, Object firstValue) {
        table = new Entry[INITIAL_CAPACITY];
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        table[i] = new Entry(firstKey, firstValue);
        size = 1;
        setThreshold(INITIAL_CAPACITY);
}
```

解释：

获取当前线程对应的 map，如果 map 不为空，就将值设置到 map 中。如果 map 为空就将创建 Map 结构，然后将值放入到链表中。

注意：可以发现 ThreadLocalMap 的构造器采用 HashMap 的套路，初始化节点数组，然后使用键 key 进行按位与运算计算数组下标，设置最大临界值为初始化容量。

### get 方法

```java
public T get() {
        // 获取当前线程
        Thread t = Thread.currentThread();
        // 获取当前线程的本地map映射
        ThreadLocalMap map = getMap(t);
        // 如果map不为空
        if (map != null) {
            // 获取映射map的链表
            ThreadLocalMap.Entry e = map.getEntry(this);
            // 如果节点不为空，返回节点的值
            if (e != null)
                return (T)e.value;
        }
        //返回初始化的值
        return setInitialValue();
}

private T setInitialValue() {
        // 获取初始化时传入的值
        T value = initialValue();
        // 获取当前线程
        Thread t = Thread.currentThread();
        // 获取当前线程的本地map映射
        ThreadLocalMap map = getMap(t);
        // 如果映射map不为空，设置当前值
        if (map != null)
            map.set(this, value);
        else // 如果映射map为空，那么创建map映射，然后初始化值到链表中
            createMap(t, value);
        return value;
}
```

解释：
1. 首先获取当前线程的本地 map 映射；
2. 如果 map 不为空，那么就走 hashmap 的套路，通过键按位与运算计算下标，然后通过下标获取值；
3. 否则会再判断下 map 是否为空，如果为空就创建下 ThreadLocalMap ，但是最后的效果就是拿到初始化传入的值，初始化传入的值就是 null，除非自己重写 initialValue 方法。


ThreadLocal 的应用场景：

每个线程都有一个自己的对象，并且可以很多处用到，放到 ThreadLocal 中便于存取。