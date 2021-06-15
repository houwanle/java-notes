## 多线程与高并发：synchronize新版深入详解

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
