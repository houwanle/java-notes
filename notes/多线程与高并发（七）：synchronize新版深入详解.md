## 多线程与高并发（七）：synchronize新版深入详解

### 1. 用户态和内核态
Intel指令分为 ring0~ring3 级；

- 内核态：执行在内核空间，可以访问所有的指令（ring0）；
- 用户态：只能访问用户可以访问的指令（ring3）；

JDK早期，synchronized 叫做重量级锁， 因为申请锁资源必须通过kernel, 系统调用。
- 重量级锁：JVM加锁，需要从用户态到内核态的一个调用（0x80），即从操作系统中申请锁；


```assembly
;hello.asm
;write(int fd, const void *buffer, size_t nbytes)

section data
    msg db "Hello", 0xA
    len equ $ - msg

section .text
global _start
_start:

    mov edx, len
    mov ecx, msg
    mov ebx, 1 ;文件描述符1 std_out
    mov eax, 4 ;write函数系统调用号 4
    int 0x80

    mov ebx, 0
    mov eax, 1 ;exit函数系统调用号
    int 0x80

```

### 2. CAS
- Compare And Swap（Compare And Exchange）/自旋/自旋锁/无锁（无重量锁）
- 因为经常配合循环操作，直到完成为止，所以泛指一类操作；
- cas(v,a,b)，变量v，期待值a，修改b；
- ABA问题，你的女朋友在离开你的这段时间经历了别人，自旋就是你空转等待，一直等到它接纳你为止；
  - 解决办法：（版本号AtomicStampedReference），基础类型简单值不需要版本号；

![多线程与高并发：CAS原理示意图](./pics/多线程与高并发：CAS原理示意图.png)


### 3. Unsafe

- AtomicInteger:

  ```java
  public final int incrementAndGet() {
      for (;;) {
          int current = get();
          int next = current + 1;
          if (compareAndSet(current, next))
              return next;
      }
  }

  public final boolean compareAndSet(int expect, int update) {
      return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
  }
  ```

- Unsafe:

  ```java
  public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
  ```

- 运用：

  ```java
  package com.mashibing.jol;

  import sun.misc.Unsafe;

  import java.lang.reflect.Field;

  public class T02_TestUnsafe {

      int i = 0;
      private static T02_TestUnsafe t = new T02_TestUnsafe();

      public static void main(String[] args) throws Exception {
          //Unsafe unsafe = Unsafe.getUnsafe();

          Field unsafeField = Unsafe.class.getDeclaredFields()[0];
          unsafeField.setAccessible(true);
          Unsafe unsafe = (Unsafe) unsafeField.get(null);

          Field f = T02_TestUnsafe.class.getDeclaredField("i");
          long offset = unsafe.objectFieldOffset(f);
          System.out.println(offset);

          boolean success = unsafe.compareAndSwapInt(t, offset, 0, 1);
          System.out.println(success);
          System.out.println(t.i);
          //unsafe.compareAndSwapInt()
      }
  }
  ```

- jdk8u: unsafe.cpp:
- cmpxchg = compare and exchange

  ```c++
  UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
    UnsafeWrapper("Unsafe_CompareAndSwapInt");
    oop p = JNIHandles::resolve(obj);
    jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
    return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
  UNSAFE_END
  ```

- jdk8u: atomic_linux_x86.inline.hpp **93行**
- is_MP = Multi Processor

  ```c++
  inline jint     Atomic::cmpxchg    (jint     exchange_value, volatile jint*     dest, jint     compare_value) {
    int mp = os::is_MP();
    __asm__ volatile (LOCK_IF_MP(%4) "cmpxchgl %1,(%3)"
                      : "=a" (exchange_value)
                      : "r" (exchange_value), "a" (compare_value), "r" (dest), "r" (mp)
                      : "cc", "memory");
    return exchange_value;
  }
  ```

- jdk8u: os.hpp is_MP()

  ```c++
  static inline bool is_MP() {
    // During bootstrap if _processor_count is not yet initialized
    // we claim to be MP as that is safest. If any platform has a
    // stub generator that might be triggered in this phase and for
    // which being declared MP when in fact not, is a problem - then
    // the bootstrap routine for the stub generator needs to check
    // the processor count directly and leave the bootstrap routine
    // in place until called after initialization has ocurred.
    return (_processor_count != 1) || AssumeMP;
  }
  ```

- jdk8u: atomic_linux_x86.inline.hpp

  ```c++
  #define LOCK_IF_MP(mp) "cmp $0, " #mp "; je 1f; lock; 1: "
  ```

  - MP：multi processor多个处理器

- 最终实现：
  - cmpxchg = cas修改变量值

  ```assembly
  #加锁是为保证操作的原子性（CAS在比较并写的时候不是原子操作）
  lock cmpxchg 指令
  ```

- 硬件：

  - lock指令在执行后面指令的时候锁定一个北桥信号

  - （不采用锁总线的方式）

### 4. markword
#### 对象在内存中的布局（JVM hotspot实现）
- 8字节markword
- 4字节class pointer（默认开启压缩，压缩之后是4字节）
- 成员变量（int占4字节）

**要求对象8字节对齐：对象的大小（字节数）必须是8的整数倍，若不够则凑成8的整数倍**

### 5. 工具：JOL = Java Object Layout

```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/org.openjdk.jol/jol-core -->
    <dependency>
        <groupId>org.openjdk.jol</groupId>
        <artifactId>jol-core</artifactId>
        <version>0.9</version>
    </dependency>
</dependencies>
```

```java
import org.openjdk.jol.info.ClassLayout;

/**
 * 加入锁处于偏向状态，这时来了竞争者，那么它的状态是什么？
 */

public class HelloJOL {
  public static void main(String[] args) throws Exception {
    //Thread.sleep(5000);

    Object o = new Object();
    System.out.println(ClassLayout.parseInstance(o).toPrintable());

    synchronized(o) {
      System.out.println(ClassLayout.parseInstance(o).toPrintable());
    }
  }
}
```

- 给对象上锁，在hotspot中实现是修改对象的markword；

jdk8u: markOop.hpp

```java
// Bit-format of an object header (most significant first, big endian layout below):
//
//  32 bits:
//  --------
//             hash:25 ------------>| age:4    biased_lock:1 lock:2 (normal object)
//             JavaThread*:23 epoch:2 age:4    biased_lock:1 lock:2 (biased object)
//             size:32 ------------------------------------------>| (CMS free block)
//             PromotedObject*:29 ---------->| promo_bits:3 ----->| (CMS promoted object)
//
//  64 bits:
//  --------
//  unused:25 hash:31 -->| unused:1   age:4    biased_lock:1 lock:2 (normal object)
//  JavaThread*:54 epoch:2 unused:1   age:4    biased_lock:1 lock:2 (biased object)
//  PromotedObject*:61 --------------------->| promo_bits:3 ----->| (CMS promoted object)
//  size:64 ----------------------------------------------------->| (CMS free block)
//
//  unused:25 hash:31 -->| cms_free:1 age:4    biased_lock:1 lock:2 (COOPs && normal object)
//  JavaThread*:54 epoch:2 cms_free:1 age:4    biased_lock:1 lock:2 (COOPs && biased object)
//  narrowOop:32 unused:24 cms_free:1 unused:4 promo_bits:3 ----->| (COOPs && CMS promoted object)
//  unused:21 size:35 -->| cms_free:1 unused:7 ------------------>| (COOPs && CMS free block)
```

### 5. synchronized的横切面详解
1. synchronized原理
2. 升级过程
3. 汇编实现
4. vs reentrantLock的区别

#### 5.1 java源码层级
synchronized(o)

#### 5.2 字节码层级
monitorenter moniterexit

#### 5.3 JVM层级（Hotspot）

```java
package com.mashibing.insidesync;

import org.openjdk.jol.info.ClassLayout;

public class T01_Sync1 {


    public static void main(String[] args) {
        Object o = new Object();

        System.out.println(ClassLayout.parseInstance(o).toPrintable());
    }
}
```

```java
com.mashibing.insidesync.T01_Sync1$Lock object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4   (object header)  05 00 00 00 (00000101 00000000 00000000 00000000) (5)
      4     4   (object header)  00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4   (object header)  49 ce 00 20 (01001001 11001110 00000000 00100000) (536923721)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
```

```java
com.mashibing.insidesync.T02_Sync2$Lock object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4   (object header)  05 90 2e 1e (00000101 10010000 00101110 00011110) (506368005)
      4     4   (object header)  1b 02 00 00 (00011011 00000010 00000000 00000000) (539)
      8     4   (object header)  49 ce 00 20 (01001001 11001110 00000000 00100000) (536923721)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes tota
```

- InterpreterRuntime:: monitorenter方法

```c++
IRT_ENTRY_NO_ASYNC(void, InterpreterRuntime::monitorenter(JavaThread* thread, BasicObjectLock* elem))
#ifdef ASSERT
  thread->last_frame().interpreter_frame_verify_monitor(elem);
#endif
  if (PrintBiasedLockingStatistics) {
    Atomic::inc(BiasedLocking::slow_path_entry_count_addr());
  }
  Handle h_obj(thread, elem->obj());
  assert(Universe::heap()->is_in_reserved_or_null(h_obj()),
         "must be NULL or an object");
  if (UseBiasedLocking) {
    // Retry fast entry if bias is revoked to avoid unnecessary inflation
    ObjectSynchronizer::fast_enter(h_obj, elem->lock(), true, CHECK);
  } else {
    ObjectSynchronizer::slow_enter(h_obj, elem->lock(), CHECK);
  }
  assert(Universe::heap()->is_in_reserved_or_null(elem->obj()),
         "must be NULL or an object");
#ifdef ASSERT
  thread->last_frame().interpreter_frame_verify_monitor(elem);
#endif
IRT_END
```

- synchronizer.cpp
- revoke_and_rebias

```c++
void ObjectSynchronizer::fast_enter(Handle obj, BasicLock* lock, bool attempt_rebias, TRAPS) {
 if (UseBiasedLocking) {
    if (!SafepointSynchronize::is_at_safepoint()) {
      BiasedLocking::Condition cond = BiasedLocking::revoke_and_rebias(obj, attempt_rebias, THREAD);
      if (cond == BiasedLocking::BIAS_REVOKED_AND_REBIASED) {
        return;
      }
    } else {
      assert(!attempt_rebias, "can not rebias toward VM thread");
      BiasedLocking::revoke_at_safepoint(obj);
    }
    assert(!obj->mark()->has_bias_pattern(), "biases should be revoked by now");
 }

 slow_enter (obj, lock, THREAD) ;
}
```

```c++
void ObjectSynchronizer::slow_enter(Handle obj, BasicLock* lock, TRAPS) {
  markOop mark = obj->mark();
  assert(!mark->has_bias_pattern(), "should not see bias pattern here");

  if (mark->is_neutral()) {
    // Anticipate successful CAS -- the ST of the displaced mark must
    // be visible <= the ST performed by the CAS.
    lock->set_displaced_header(mark);
    if (mark == (markOop) Atomic::cmpxchg_ptr(lock, obj()->mark_addr(), mark)) {
      TEVENT (slow_enter: release stacklock) ;
      return ;
    }
    // Fall through to inflate() ...
  } else
  if (mark->has_locker() && THREAD->is_lock_owned((address)mark->locker())) {
    assert(lock != mark->locker(), "must not re-lock the same lock");
    assert(lock != (BasicLock*)obj->mark(), "don't relock with same BasicLock");
    lock->set_displaced_header(NULL);
    return;
  }

#if 0
  // The following optimization isn't particularly useful.
  if (mark->has_monitor() && mark->monitor()->is_entered(THREAD)) {
    lock->set_displaced_header (NULL) ;
    return ;
  }
#endif

  // The object header will never be displaced to this lock,
  // so it does not matter what the value is, except that it
  // must be non-zero to avoid looking like a re-entrant lock,
  // and must not look locked either.
  lock->set_displaced_header(markOopDesc::unused_mark());
  ObjectSynchronizer::inflate(THREAD, obj())->enter(THREAD);
}
```

- inflate方法：膨胀为重量级锁


### 7. 锁升级过程
#### 7.1 JDK8 markword实现表

![多线程与高并发：markword-64](./pics/多线程与高并发：markword-64.png)

- 偏向锁：上偏向锁，指的就是，把markword的线程ID改为自己线程ID的过程；上偏向锁没有竞争
  - 偏向锁不可重偏向 批量偏向 批量撤销；

- 自旋锁：两个线程来竞争这把锁，首先撤销原来的偏向锁或者没有偏向锁，直接来竞争 -> 两个线程谁能把自己的 lock Record 指针贴上去，谁就获得这把自旋锁，另外一个就自旋等待。

- 重量级锁：需要向操作系统去申请，markword中记录的是Object Monitor（JVM空间写的C++对象，需要访问操作系统）

**用户空间锁 VS 重量级锁**
- 偏向锁、自旋锁 都是在用户空间完成；
- 重量级锁是需要向内核申请；


![多线程与高并发：锁升级初步](./pics/多线程与高并发：锁升级初步.png)

  - new -> 普通对象 -> 加synchronize -> 偏向锁 -> 轻度竞争 -> 轻量级锁（自旋锁或无锁） -> 竞争加剧 -> 重量级锁
  - synchronized优化的过程和markword息息相关；

**用markword中最低的三位代表锁的状态，其中1位是偏向锁位，另两位是普通锁位**
1. Object o = new Object()
    - 锁 = 0 01 无锁态
    - 注意：如果偏向锁打开，默认是匿名偏向状态；
2. o.hashCode()
    - 001 + hashcode

    ```java
    00000001 10101101 00110100 00110110
    01011001 00000000 00000000 00000000
    ```

    - little endian big endian
    - 00000000 00000000 00000000 01011001 00110110 00110100 10101101 00000000
3. 默认synchronized(o)
    - 00 -> 轻量级锁
    - 默认情况 偏向锁有个时延，默认是4秒；
    - why? 因为JVM虚拟机自己有一些默认启动的线程，里面有好多sync代码，这些sync代码启动时就知道肯定会有竞争，如果使用偏向锁，就会造成偏向锁不断的进行锁撤销和锁升级的操作，效率较低。
    - `-XX:BiasedLockingStartupDelay=0`
4. 如果设定上述参数
    - new Object () - > 101 偏向锁 ->线程ID为0 -> Anonymous BiasedLock
    - 打开偏向锁，new出来的对象，默认就是一个可偏向匿名对象101
5. 如果有线程上锁
    - 上偏向锁，指的就是，把markword的线程ID改为自己线程ID的过程
    - 偏向锁不可重偏向 批量偏向 批量撤销
6. 如果有线程竞争
    - 撤销偏向锁，升级轻量级锁;
    - 线程在自己的线程栈生成LockRecord ，用CAS操作将markword设置为指向自己这个线程的LR的指针，设置成功者得到锁;
7. 如果竞争加剧
    - 竞争加剧：有线程超过10次自旋， -XX:PreBlockSpin， 或者自旋线程数超过CPU核数的一半， 1.6之后，加入自适应自旋 Adapative Self Spinning ， JVM自己控制
    - 升级重量级锁：-> 向操作系统申请资源，linux mutex , CPU从3级-0级系统调用，线程挂起，进入等待队列，等待操作系统的调度，然后再映射回用户空间
    - (以上实验环境是JDK11，打开就是偏向锁，而JDK8默认对象头是无锁)
    - 偏向锁默认是打开的，但是有一个时延，如果要观察到偏向锁，应该设定参数

**如果计算过对象的hashCode，则对象无法进入偏向状态！**

**轻量级锁重量级锁的hashCode存在与什么地方？**
- 线程栈中，轻量级锁的LR中，或是代表重量级锁的ObjectMonitor的成员中

**关于epoch: (不重要)**
- **批量重偏向与批量撤销**渊源：从偏向锁的加锁解锁过程中可看出，当只有一个线程反复进入同步块时，偏向锁带来的性能开销基本可以忽略，但是当有其他线程尝试获得锁时，就需要等到safe point时，再将偏向锁撤销为无锁状态或升级为轻量级，会消耗一定的性能，所以在多线程竞争频繁的情况下，偏向锁不仅不能提高性能，还会导致性能下降。于是，就有了批量重偏向与批量撤销的机制。
- **原理**以class为单位，为每个class维护**解决场景**批量重偏向（bulk rebias）机制是为了解决：一个线程创建了大量对象并执行了初始的同步操作，后来另一个线程也来将这些对象作为锁对象进行操作，这样会导致大量的偏向锁撤销操作。批量撤销（bulk revoke）机制是为了解决：在明显多线程竞争剧烈的场景下使用偏向锁是不合适的。
- 一个偏向锁撤销计数器，每一次该class的对象发生偏向撤销操作时，该计数器+1，当这个值达到重偏向阈值（默认20）时，JVM就认为该class的偏向锁有问题，因此会进行批量重偏向。每个class对象会有一个对应的epoch字段，每个处于偏向锁状态对象的Mark Word中也有该字段，其初始值为创建该对象时class中的epoch的值。每次发生批量重偏向时，就将该值+1，同时遍历JVM中所有线程的栈，找到该class所有正处于加锁状态的偏向锁，将其epoch字段改为新值。下次获得锁时，发现当前对象的epoch值和class的epoch不相等，那就算当前已经偏向了其他线程，也不会执行撤销操作，而是直接通过CAS操作将其Mark Word的Thread Id 改成当前线程Id。当达到重偏向阈值后，假设该class计数器继续增长，当其达到批量撤销的阈值后（默认40），JVM就认为该class的使用场景存在多线程竞争，会标记该class为不可偏向，之后，对于该class的锁，直接走轻量级锁的逻辑。

加锁，指的是锁定对象

**锁升级的过程**
- JDK较早的版本：OS的资源 互斥量 用户态 -> 内核态的转换 重量级 效率比较低
- 现代版本进行了优化：无锁 - 偏向锁 -轻量级锁（自旋锁）-重量级锁


偏向锁 - markword 上记录当前线程指针，下次同一个线程加锁的时候，不需要争用，只需要判断线程指针是否同一个，所以，偏向锁，偏向加锁的第一个线程 。hashCode备份在线程栈上 线程销毁，锁降级为无锁

有争用 - 锁升级为轻量级锁 - 每个线程有自己的LockRecord在自己的线程栈上，用CAS去争用markword的LR的指针，指针指向哪个线程的LR，哪个线程就拥有锁；

自旋超过10次，升级为重量级锁 - 如果太多线程自旋 CPU消耗过大，不如升级为重量级锁，进入等待队列（不消耗CPU）-XX:PreBlockSpin

自旋锁在 JDK1.4.2 中引入，使用 -XX:+UseSpinning 来开启。JDK 6 中变为默认开启，并且引入了自适应的自旋锁（适应性自旋锁）。

自适应自旋锁意味着自旋的时间（次数）不再固定，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也是很有可能再次成功，进而它将允许自旋等待持续相对更长的时间。如果对于某个锁，自旋很少成功获得过，那在以后尝试获取这个锁时将可能省略掉自旋过程，直接阻塞线程，避免浪费处理器资源。

偏向锁由于有锁撤销的过程revoke，会消耗系统资源，所以，在锁争用特别激烈的时候，用偏向锁未必效率高。还不如直接使用轻量级锁。

**自旋锁什么时候升级为重量级锁？**
- 竞争加剧：有线程超过10次自旋（jdk1.6之前可以通过调整 `-XX:PreBlockSpin`参数控制；jdk1.6之后加入自适应自旋 `Adapative Self Spining`，JVM自己控制），或者自旋线程数超过CPU核数的一半，升级为重量级锁；
- 升级重量级锁：向操作系统申请资源，linux mutex，CPU从3级-0级系统调用，线程挂起，进入等待队列，等待操作系统的调度，然后再映射回用户空间；

**为什么有自旋锁还需要重量级锁？**
- 自旋是消耗CPU资源的，如果锁的时间长，或者自旋线程多，CPU会被大量消耗；
- 重量级锁有等待队列，所有拿不到锁的进入等待队列（waitSet），不需要消耗CPU资源；

**偏向锁是否一定比自旋锁效率高？**
- 不一定，在明确知道会有多线程竞争的情况下，偏向锁肯定会涉及锁撤销（需要消耗资源），这时候直接使用自旋锁；
- JVM启动过程，会有很多线程竞争（明确），所以默认情况启动时不打开偏向锁，过一段儿时间再打开；


#### 7.2 锁重入
- synchronize是可重入锁
  - 重入次数必须记录，因为要解锁几次必须对应；
    - 偏向锁的重入次数记录在线程栈中：每重入一次，加一个 Lock Record
    - 重量级锁的重入次数记录在Object Monitor的一个字段上
