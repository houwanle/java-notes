# JVM：GC和GC-Tuning

### GC的基础知识

#### 1.什么是垃圾
> C语言申请内存：malloc free
>
> C++： new delete
>
> Java: new ？
>
> 自动内存回收，编程上简单，系统不容易出错，手动释放内存，容易出两种类型的问题：
>
> 1. 忘记回收
> 2. 多次回收

没有任何引用指向的一个对象或者多个对象（循环引用）

#### 2.如何定位垃圾
- 引用计数（Reference Count）
  > 不能解决循环引用的问题

  ![引用计数1](./pics/GC和GC-Tuning_1.png)

  对象的引用数变为0了，那么该对象就成为垃圾了。

  ![引用计数2](./pics/GC和GC-Tuning_2.png)

- 根可达算法（Root Searching）
  - 从根对象开始搜索
  - 根对象：从程序运行起来后，马上需要的的对象；
  - 根对象包含：
    - JVM stack
    - native method stack
    - run-time constant pool
    - static references in method area
    - Clazz

  ![根可达算法](./pics/GC和GC-Tuning_3.png)

#### 3.常见的垃圾回收算法
- 标记清除（Mark-Sweep）
  - 首先找到有用的对象，然后将没有用的标记出来，然后清除；
  - 算法相对简单，存活对象比较多的情况下效率较高；
  - 两遍扫描（第一次找有用的，第二次找没有用的），效率偏低；
  - 位不连续，容易产生碎片；
- 拷贝算法（Copying）----适合Eden区YGC
  - 将内存一分为二，将存活对象拷贝到未使用区域，然后清除；
  - 适用于存活对象较少的情况；
  - 只扫描一次，效率提高；
  - 没有碎片，但浪费空间；
  - 移动复制对象，需要调整对象引用；
- 标记压缩（Mark-Compact）----适合FGC
  - 将所有整理、清理的过程都压缩到头上去（有用的对象往头上挪）
  - 不会产生内存减半；
  - 不会产生碎片，方便对象分配；
  - 扫描两次，需要移动对象，效率偏低；

- 标记清除

  ![标记清除](./pics/GC和GC-Tuning_4.png)

- 拷贝算法

  ![拷贝算法](./pics/GC和GC-Tuning_5.png)

- 标记压缩

  ![标记压缩](./pics/GC和GC-Tuning_6.png)

#### 4.JVM内存分代模型（用于分代垃圾回收算法）
1. 部分垃圾回收器使用的模型
    - 除Epslion、ZGC、Shenandoah之外的GC都是使用逻辑分代模型；
    - G1是逻辑分代，物理不分代；
    - 除此之外不仅逻辑分代，而且物理分代；
2. 新生代 + 老年代 + 永久代（jdk1.7）/ 元数据区(jdk1.8) Metaspace
   1. 永久代/元数据：装Class对象
   2. 永久代必须指定大小限制；元数据可以设置，也可以不设置，无上限（受限于物理内存）
   3. 字符串常量存放： 1.7 - 永久代；1.8 - 堆
   4. MethodArea逻辑概念 - 永久代、元数据
3. 新生代 = Eden（伊甸） + 2个suvivor区

    ![对象创建过程](./pics/GC和GC-Tuning_7.png)

   1. YGC（young GC）回收之后，大多数的对象会被回收，活着的进入s0
   2. 再次YGC，活着的对象eden + s0 -> s1
   3. 再次YGC，eden + s1 -> s0
   4. 年龄足够 -> 老年代 （15 CMS 6）
   5. s区装不下 -> 老年代
4. 老年代
   1. 顽固分子
   2. 老年代满了，会触发FGC（Full GC）
5. GC Tuning (Generation)
   1. 尽量减少FGC（FGC会停止用户所有线程，效率比较慢，会出现卡顿现象）
   2. MinorGC = YGC：年轻代空间耗尽时触发；
   3. MajorGC = FGC：在老年代无法继续分配空间时触发，新生代老年代同时进行回收；


> 对象分配详解
- 栈上分配
  - 线程私有小对象
  - 无逃逸：就在某一段代码使用
  - 支持标量替换：普通的变量来代替整个对象
  - 无需调整（调优）
- 线程本地分配TLAB（Thread Local Allocation Buffer）
  - 占用eden，默认1%（每个线程独有，效率提高）
  - 多线程的时候不用竞争eden就可以申请空间，提高效率
  - 小对象
  - 无需调整（调优）
- 老年代
  - 大对象
- eden

> 对象何时进入老年代
- 超过 XX:MaxTenuringThreshold 指定次数（YGC）
  - Parallel Scavenge 15
  - CMS 6
  - G1 15
- 动态年龄
  - Eden + s0 -> s1 ： 超过s1的50%，把年龄最大的对象放入老年代；

> 对象的分配过程图

  ![对象分配过程图](./pics/对象分配过程图.png)



#### 5.常见的垃圾回收器

  ![垃圾回收器](./pics/GC和GC-Tuning_8.png)

1. Serial 年轻代 串行回收

  ![Serial](./pics/GC和GC-Tuning_9.png)

2. PS 年轻代 并行回收

  ![Parallel Scavenge](./pics/GC和GC-Tuning_10.png)

3. ParNew 年轻代 配合CMS的并行回收

  ![ParNew](./pics/GC和GC-Tuning_11.png)

4. SerialOld 老年代
5. ParallelOld 老年代
6. ConcurrentMarkSweep 老年代 并发的， 垃圾回收和应用程序同时运行，降低STW的时间(200ms)
7. G1(10ms)
8. ZGC (1ms) PK C++
9. Shenandoah
10. Eplison

jdk1.8 默认的垃圾回收：PS + ParallelOld

#### 6.JVM调优第一步，了解生产环境下的垃圾回收器组合

* JVM的命令行参数参考：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html

* JVM参数分类

  > 标准： - 开头，所有的HotSpot都支持
  >
  > 非标准：-X 开头，特定版本HotSpot支持特定命令
  >
  > 不稳定：-XX 开头，下个版本可能取消

  -XX:+PrintCommandLineFlags 查看当前设置了哪些参数

  -XX:+PrintFlagsFinal 最终参数值

  -XX:+PrintFlagsInitial 默认参数值

### 参考资料

1. [https://blogs.oracle.com/
    ](https://blogs.oracle.com/jonthecollector/our-collectors)[jonthecollector](https://blogs.oracle.com/jonthecollector/our-collectors)[/our-collectors](https://blogs.oracle.com/jonthecollector/our-collectors)
2. https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html
3. http://java.sun.com/javase/technologies/hotspot/vmoptions.jsp
