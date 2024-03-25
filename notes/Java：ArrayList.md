# Java：ArrayList

## ArrayList的扩容机制
ArrayList 的扩容机制是在添加元素时判断当前数组大小是否已经满了，如果已经满了，则创建一个新的更大的数组，并将原来的元素全部复制到新的数组中。具体的扩容规则如下：
- 当添加元素后，size 大小已经等于或超过了数组的容量 capacity 时，就会触发扩容操作；
- 扩容的大小默认情况下为原数组大小的一半。比如原数组大小为 10，那么扩容后的数组大小为 15；
- 如果扩容后的大小还是不够，那么就以添加元素的数量作为扩容大小，即新数组的大小为 oldCapacity + (oldCapacity >> 1) + 1；

需要注意的是，ArrayList 中的数组无法动态地调整大小，因此每次扩容都需要创建新的数组和复制元素，这可能会带来一些性能损失。为了避免频繁扩容，我们可以在使用 ArrayList 时尽量预估元素数量，初始化时指定一个合适的初始容量。

## ArrayList 线程不安全及解决方案
> 多线程环境下，ArrayList是线程不安全的，故提供以下四种保证ArrayList线程安全的方法.
1. 使用 Collections.synchronizedList() 方法：Java 提供了 Collections 类中的 synchronizedList() 方法，可以将一个普通的 ArrayList 转换成线程安全的 List。

    ```java
    List<E> myArrayList = Collections.synchronizedList(new ArrayList<E>());
    ```

2. 使用 java.util.concurrent.CopyOnWriteArrayList 类：CopyOnWriteArrayList 是 Java 并发包（java.util.concurrent）中提供的一个线程安全的列表实现。它在读取操作上没有锁，并且支持在迭代期间进行修改操作，而不会抛出 ConcurrentModificationException 异常。示例如下：

    ```java
    CopyOnWriteArrayList<E> myArrayList = new CopyOnWriteArrayList<E>();
    ```

3. 写一个myArrayList继承自Arraylist，然后重写或按需求编写自己的方法，这些方法要写成synchronized，在这些synchronized的方法中调用ArrayList的方法。

    ```java
    import java.util.ArrayList;
    
    public class MyArrayList<E> extends ArrayList<E> {
    
        @Override
        public synchronized boolean add(E e) {
            // 在 synchronized 方法中调用 ArrayList 的 add 方法
            return super.add(e);
        }
    
        @Override
        public synchronized void add(int index, E element) {
            // 在 synchronized 方法中调用 ArrayList 的 add 方法
            super.add(index, element);
        }
    
        @Override
        public synchronized E remove(int index) {
            // 在 synchronized 方法中调用 ArrayList 的 remove 方法
            return super.remove(index);
        }
    
        @Override
        public synchronized boolean remove(Object o) {
            // 在 synchronized 方法中调用 ArrayList 的 remove 方法
            return super.remove(o);
        }
    
        // 可以按需求继续重写其他需要同步的方法
    
        // 注意：需要根据具体的需求选择要同步的方法，不一定需要将所有方法都同步
    
    }
    ```

4. 使用显式的锁：可以使用 java.util.concurrent.locks 包中提供的显式锁（如 ReentrantLock）来手动实现对 ArrayList 的同步。这需要在访问 ArrayList 的地方显式地获取和释放锁，从而确保在同一时刻只有一个线程可以访问 ArrayList。以下是使用显式锁 (ReentrantLock) 实现对 ArrayList 进行同步的示例代码：

    ```java
    import java.util.ArrayList;
    import java.util.concurrent.locks.ReentrantLock;
    
    public class MyArrayList<E> {
    
        private ArrayList<E> arrayList = new ArrayList<>();
        private ReentrantLock lock = new ReentrantLock();
    
        public void add(E e) {
            lock.lock();
            try {
                // 在锁内部调用 ArrayList 的 add 方法
                arrayList.add(e);
            } finally {
                lock.unlock();
            }
        }
    
        public void add(int index, E element) {
            lock.lock();
            try {
                // 在锁内部调用 ArrayList 的 add 方法
                arrayList.add(index, element);
            } finally {
                lock.unlock();
            }
        }
    
        public E remove(int index) {
            lock.lock();
            try {
                // 在锁内部调用 ArrayList 的 remove 方法
                return arrayList.remove(index);
            } finally {
                lock.unlock();
            }
        }
    
        public boolean remove(Object o) {
            lock.lock();
            try {
                // 在锁内部调用 ArrayList 的 remove 方法
                return arrayList.remove(o);
            } finally {
                lock.unlock();
            }
        }
    
        // 可以按需求继续实现其他需要同步的方法
    
    }
    ```
