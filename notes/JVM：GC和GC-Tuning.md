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
**新生代：老年代 = 1:2**
1. 部分垃圾回收器使用的模型
    - 除Epslion、ZGC、Shenandoah之外的GC都是使用逻辑分代模型；
    - G1是逻辑分代，物理不分代；
    - 除此之外不仅逻辑分代，而且物理分代；
2. 新生代 + 老年代 + 永久代（jdk1.7）Perm Generation/ 元数据区(jdk1.8) Metaspace
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
- 动态年龄（不重要）
  - Eden + s0 -> s1 ： 超过s1的50%，把年龄最大的对象放入老年代；
- 分配担保（不重要）：YGC期间，survivor区空间不够了，空间担保直接进入老年代

> 对象的分配过程图

  ![对象分配过程图](./pics/对象分配过程图.png)


#### 5.常见的垃圾回收器
> JDk 诞生 Serial追随，提高效率，诞生了PS，为了配合CMS，诞生了PN，CMS是1.4版本后期引入，CMS是里程碑式的GC，它开启了并发回收的过程，但是CMS毛病较多，因此目前没有任何一个JDK默认GC是CMS。

> 并发垃圾回收是因为无法忍受STW


> 图中连在一起的，都可以组合

  ![垃圾回收器](./pics/GC和GC-Tuning_8.png)

> 常见的垃圾回收器的组合
- Serial 和 Serial Old
- Parallel Scavenge 和 Parallel Old
- ParNew 和 CMS


> 常见的十种垃圾回收器

- Serial（单线程）：干活的时候，所有的工作线程都停止了；
  - STW（stop-the-world，停顿时间），然后清理垃圾
  - safe point：线程停止，不是立马停止，需要找到安全点才能停止；
  - 年轻代 串行回收
  - 现在用的很少

  ![Serial](./pics/GC和GC-Tuning_9.png)

- Parallel Scavenge（多线程）
  - STW（stop-the-world，停顿时间），然后清理垃圾
  - 年轻代 并行回收

  ![Parallel Scavenge](./pics/GC和GC-Tuning_10.png)

- Parallel New
  - STW（stop-the-world，停顿时间），然后清理垃圾
  - 年轻代；
  - 配合CMS的并行回收

  ![ParNew](./pics/GC和GC-Tuning_11.png)

- SerialOld 老年代
- ParallelOld 老年代
- ConcurrentMarkSweep（CMS使用的是 三色标记+Incremental Update算法）：老年代 并发的， 垃圾回收和应用程序同时运行，降低STW的时间(200ms)
  - 初始标记（多线程）：STW，只标记GC root上的不可回收的，垃圾不多，短时间能完成；
  - 并发标记：一边标记，一边清理，最浪费时间；
  - 重新标记（多线程）：STW，并发标记中产生的新垃圾需要重新标记，垃圾不多，短时间完成；
  - 并发清理：会产生新的垃圾（浮动垃圾需等下一次CMS来清掉）

  ![JVM：CMS](./pics/JVM：CMS.png)

  - CMS的问题
    - Memory Fragmentation（内存碎片化）：新的对象不能往老年代装的时候，CMS会变成SerialOld，单线程清理；
       - -XX:+UseCMSCompactAtFullCollection
       - -XX:CMSFullGCsBeforeCompaction 默认为0，指的是经过多少次FGC才进行压缩
    - Floating Garbage（浮动垃圾）：
      - Concurrent Mode Failure：老年代满了，同时浮动垃圾没有清理完，这时会用SerialOld
      - 解决方案：降低触发CMS的阈值
    - PromotionFailed
      - 解决方案：保持老年代有足够的空间
        - -XX:CMSInitiatingOccupancyFraction 92%：92%的时候会产生FGC，可以降低这个值，让CMS保持老年代足够的空间
- G1(10ms；jdk1.7才有，使用的是 三色标记+SATB算法)
  - https://www.oracle.com/technical-resources/articles/java/g1gc.html
  - G1是一种服务端应用使用的垃圾收集器，目标是用在多核、大内存的机器上，他在大多数情况下可以实现指定的GC暂停时间，同时还能保持较高的吞吐量；

  ![JVM：G1原理性模型](./pics/JVM：G1原理性模型.png)

  - 每个分区都可能是年轻代也可能是老年代，但是在同一时刻只能属于某个代。
  - 年轻代、幸存区、老年代这些概念还存在，称为逻辑上的概念，这样方便复用之前分代框架的逻辑。在物理上不需要连续，则带来了额外的好处——有的分区内垃圾对象特别多，有的分区内垃圾特别少，G1会优先回收垃圾对象特别多的分区，这样可以花费较少的时间来回收这些分区的垃圾，这也就是G1名字的由来，即首先手机垃圾最多的分区。
  - 新生代其实并不是适用于这种算法的，依然是在新生代满了的时候，对整个新生代进行回收——整个新生代中的对象，要么被回收、要么晋升，至于新生代也采取分区机制的原因，则是因为这样跟老年代的策略统一，方便调整代的大小。
  - G1还是一种带压缩的收集器，在回收老年代的分区时，是将存活的对象从一个分区拷贝到另一个可用分区，这个拷贝的过程就实现了局部的压缩。每个分区的大小从1M到32M不等，但都是2的幂次方。
  - 特点：
    - 并发收集（和CMS区别不大）
    - 压缩空闲空间不会延长GC的暂停时间；
    - 更易预测的GC暂停时间；
    - 适用不需要实现很高的吞吐量的场景；
    - 新老年代比例：5% ~ 60%
      - 一般不用手工指定，也不要手工指定，因为这是G1预测停顿时间的基准；
    - humongous object：超过单个region的50%；也有跨越多个region的
  - 基本概念：
    - Card Table（GC中）
      - 由于做YGC时，需要扫描整个old区，效率非常低，所以JVM设计了CardTable，如果一个old区CardTable中有对象指向Y区，就将它设为Dirty，下次扫描时，只需要扫描Dirty Card；
      - 在结构上，Card Table用BitMap来实现；
    - CSet = Collection Set（G1；CSet中存放的是需要回收的CardTable）
      - 一组可被回收的分区的集合；
      - 在CSet中存活的数据会在GC过程中被移动到另一个可用分区，CSet中的分区可以来自Eden空间、survivor空间、或者老年代。CSet会占用不到整个堆空间的1%大小；
    - RSet = RememberedSet
      - 记录了其他Region中的对象到本Region的引用
      - RSet的价值在于使得垃圾收集器不需要扫描整个堆，找到谁引用了当前分区中的对象，只需要扫描RSet即可。
      - Rset 与 赋值的效率
        - 由于RSet的存在，那么每次给对象赋引用的时候，就得做一些额外的操作：指的是在RSet中做一些额外的记录（在GC中被称为写屏障）；这里的 写屏障 不等于 内存屏障；

        ![JVM：RSet模型](./pics/JVM：RSet模型.png)

  - GC何时触发
    - YGC -> 到达45%(MixedGC默认阈值) -> MixedGC是G1的正常回收过程
    - YGC：Eden区空间不足；多线程并行执行；
    - FGC：Old区空间不足；System.gc()；G1的FGC用的是serial
  - 如果产生FGc，你应该做什么？
    - 扩内存
    - 提高CPU性能（回收的快，业务逻辑产生对象的速度固定，垃圾回收越快，内存空间越大）
    - 降低MixedGC（相当于CMS）触发的阈值，让MixedGC提早发生（默认是45%）
      - XX:InitiatingHeapOccupacyPercent
        - 默认值45%
        - 当对象超过这个值时，启动MIxedGC
      - MixedGC的过程
        - 初始标记 STW
        - 并发标记
        - 最终标记 STW（重新标记）
        - 筛选回收 STW（并行）：筛选最需要回收的

- ZGC (1ms，使用的是 ColoredPointers + 写屏障 算法) PK C++
- Shenandoah
- Eplison（使用的是 ColoredPointers + 读屏障 算法）

jdk1.8 默认的垃圾回收：PS + ParallelOld

> Parallel New Vs Parallel Scavenge
- Parallel New 响应时间优先，配合CMS；
- Parallel Scavenge 吞吐量优先；
  - 高吞吐量可以高效的利用CPU时间，尽快完成程序的运算任务，（意味着暂停时间可能长一些）主要适合那些在后台计算而不需要交互的任务。


> 垃圾收集器与内存大小的关系
- Serial --- 几十兆
- PS --- 上百兆、几个G
- CMS --- 20G
- G1 --- 上百G
- ZGC --- 4T~16T(jdk13文档提到支持16T)

> 常见垃圾回收器组合参数设定（1.8）
- -XX:+UseSerialGC = Serial New(DefNew) + Serial Old
  - 小型程序。默认情况下不会是这种选项，HotSpot会根据计算及配置和JDK版本自动选择收集器
- -XX:+UseParNewGC = ParNew + Serial Old
  - 这个组合已经很少用（在某些版本中已经废弃）
  - https://stackoverflow.com/questions/34962257/why-remove-support-for-parnewserialold-anddefnewcms-in-the-future
- -XX:+UseConc(urrent)MarkSweepGC = ParNew + CMS + Serial Old
- -XX:+UseParallelGC = Parallel Scavenge + Parallel Old (1.8默认) 【PS + SerialOld】
- -XX:+UseParallelOldGC = Parallel Scavenge + Parallel Old
- -XX:+UseG1GC = G1
- Linux中没找到默认GC的查看方法，而windows中会打印UseParallelGC
  - java -XX:+PrintCommandLineFlags -version
  - 通过GC的日志来分辨
- Linux下1.8版本默认的垃圾回收器到底是什么？
  - 1.8.0_181 默认（看不出来）Copy MarkCompact
  - 1.8.0_222 默认 PS + PO

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

java -version

java -X

实验程序：

```java
import java.util.List;
import java.util.LinkedList;

public class HelloGC {
  public static void main(String[] args) {
    System.out.println("HelloGC!");
    List list = new LinkedList();
    for(;;) {
      byte[] b = new byte[1024*1024];
      list.add(b);
    }
  }
}
```

- 区分概念：内存泄漏memory leak，内存溢出out of memory
  - 内存泄漏：分配内存空间后，其他的的占不了，被废了的对象占用，不回收；
  - 内存溢出：不断产生对象，内存不够用；
  - 内存泄漏不会产生内存溢出；
- java -XX:+PrintCommandLineFlags HelloGC
  - HelloGC程序的默认参数；
  - -XX:InitialHeapSize、-XX:MaxHeapSize、-XX:+PrintCommandLineFlags、-XX:UseCompressedClassPointers、-XX:+UseCompressedOops
- java -Xmn10M -Xms40M -Xmx60M -XX:+PrintCommandLineFlags -XX:+PrintGC  HelloGC
  - -Xmn：新生代大小
  - -Xms、-Xmx 一般会设置成一样的，避免堆产生弹性的压缩（浪费系统的计算资源）
  - -XX:+PrintGC：打印GC的回收信息
  - PrintGCDetails：打印GC详细信息
  - PrintGCTimeStamp：打印GC相应的时间
  - PrintGCCauses：打印GC出现的原因
- java -XX:+UseConcMarkSweepGC -XX:+PrintCommandLineFlags HelloGC
  - 查看CMS的GC相关参数
- java -XX:+PrintFlagsInitial 默认参数值
- java -XX:+PrintFlagsFinal 最终参数值
- java -XX:+PrintFlagsFinal | grep xxx 找到对应的参数
- java -XX:+PrintFlagsFinal -version |grep GC

> PS GC日志详解
- 每种垃圾回收器的日志格式是不同的！
- PS日志格式

  ![JVM：PS_GC日志详解](./pics/JVM：PS_GC日志详解.png)

- heap dump部分
  - eden space 5632K, 94% used [0x00000000ff980000,0x0000000ffeb3e28,0x00000000ff00000]  后面的内存地址指的是：起始地址、使用空间结束地址、整体空间地址

  ![JVM：GCHeapDump](./pics/JVM：GCHeapDump.png)

  - total = eden + 1个survivor

> CMS日志分析
- 执行命令：
```bash
java -Xms20M -Xmx20M -XX:+PrintGCDetailsXX:UseConcMarkSweepGC com.lele.jvm.gc.T15_FullGC_Problem01
```
```
[GC (Allocation Failure) [ParNew: 6144K->640K(6144K), 0.0265885 secs] 6585K->2770K(19840K), 0.0268035 secs] [Times: user=0.02 sys=0.00, real=0.02 secs]
```
- ParNew：年轻代收集器
- 6144->640：收集前后的对比
- （6144）：整个年轻代容量
- 6585 -> 2770：整个堆的情况
- （19840）：整个堆大小

```
[GC (CMS Initial Mark) [1 CMS-initial-mark: 8511K(13696K)] 9866K(19840K), 0.0040321 secs] [Times: user=0.01 sys=0.00, real=0.00 secs]
	//8511 (13696) : 老年代使用（最大）
	//9866 (19840) : 整个堆使用（最大）
[CMS-concurrent-mark-start]
[CMS-concurrent-mark: 0.018/0.018 secs] [Times: user=0.01 sys=0.00, real=0.02 secs]
	//这里的时间意义不大，因为是并发执行
[CMS-concurrent-preclean-start]
[CMS-concurrent-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
	//标记Card为Dirty，也称为Card Marking
[GC (CMS Final Remark) [YG occupancy: 1597 K (6144 K)][Rescan (parallel) , 0.0008396 secs][weak refs processing, 0.0000138 secs][class unloading, 0.0005404 secs][scrub symbol table, 0.0006169 secs][scrub string table, 0.0004903 secs][1 CMS-remark: 8511K(13696K)] 10108K(19840K), 0.0039567 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
	//STW阶段，YG occupancy:年轻代占用及容量
	//[Rescan (parallel)：STW下的存活对象标记
	//weak refs processing: 弱引用处理
	//class unloading: 卸载用不到的class
	//scrub symbol(string) table:
		//cleaning up symbol and string tables which hold class-level metadata and
		//internalized string respectively
	//CMS-remark: 8511K(13696K): 阶段过后的老年代占用及容量
	//10108K(19840K): 阶段过后的堆占用及容量

[CMS-concurrent-sweep-start]
[CMS-concurrent-sweep: 0.005/0.005 secs] [Times: user=0.00 sys=0.00, real=0.01 secs]
	//标记已经完成，进行并发清理
[CMS-concurrent-reset-start]
[CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
	//重置内部结构，为下次GC做准备
```

> G1
https://www.oracle.com/technical-resources/articles/java/g1gc.html

> G1日志详解

```bash
java -Xms20M -Xmx20M -XX:+PrintGCDetailsXX:UseG1GC com.lele.jvm.gc.T15_FullGC_Problem01
```

- 年轻代与其他区回收已经混在一起了；
- 有initial-mark表示MixedGC已经开始了；

```
[GC pause (G1 Evacuation Pause) (young) (initial-mark), 0.0015790 secs]
//young -> 年轻代 Evacuation-> 复制存活对象
//initial-mark 混合回收的阶段，这里是YGC混合老年代回收
   [Parallel Time: 1.5 ms, GC Workers: 1] //一个GC线程
      [GC Worker Start (ms):  92635.7]
      [Ext Root Scanning (ms):  1.1]
      [Update RS (ms):  0.0]
         [Processed Buffers:  1]
      [Scan RS (ms):  0.0]
      [Code Root Scanning (ms):  0.0]
      [Object Copy (ms):  0.1]
      [Termination (ms):  0.0]
         [Termination Attempts:  1]
      [GC Worker Other (ms):  0.0]
      [GC Worker Total (ms):  1.2]
      [GC Worker End (ms):  92636.9]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.0 ms]
   [Other: 0.1 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.0 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.0 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 0.0B(1024.0K)->0.0B(1024.0K) Survivors: 0.0B->0.0B Heap: 18.8M(20.0M)->18.8M(20.0M)]
 [Times: user=0.00 sys=0.00, real=0.00 secs]
//以下是混合回收其他阶段
[GC concurrent-root-region-scan-start]
[GC concurrent-root-region-scan-end, 0.0000078 secs]
[GC concurrent-mark-start]
//无法evacuation，进行FGC
[Full GC (Allocation Failure)  18M->18M(20M), 0.0719656 secs]
   [Eden: 0.0B(1024.0K)->0.0B(1024.0K) Survivors: 0.0B->0.0B Heap: 18.8M(20.0M)->18.8M(20.0M)], [Metaspace: 38
76K->3876K(1056768K)] [Times: user=0.07 sys=0.00, real=0.07 secs]
```

#### 7. 调优前的基础概念
- 吞吐量：用户代码时间 /（用户代码执行时间 + 垃圾回收时间）
- 响应时间：STW越短，响应时间越好

所谓调优，首先确定，追求啥？吞吐量优先，还是响应时间优先？还是在满足一定的响应时间的情况下，要求达到多大的吞吐量...

1. 吞吐量优先（CPU计算所用的大部分时间用在了用户线程上；PS+PO）：科学计算、数据挖掘；
2. 响应时间优先（jdk9默认垃圾回收器 G1）：网站 GUI API

#### 8. 什么是调优
- 根据需求进行JVM规划和预调优
- 优化运行JVM运行环境（慢，卡顿）
- 解决JVM运行过程中出现的各种问题(OOM)

##### 8.1 调优，从规划开始
- 调优，从业务场景开始，没有业务场景的调优都是耍流氓
- 无监控（压力测试，能看到结果），不调优
- 步骤：
  - 熟悉业务场景（没有最好的垃圾回收器，只有最合适的垃圾回收器）
    - 响应时间、停顿时间 [CMS G1 ZGC] （需要给用户作响应）
    - 吞吐量 = 用户时间 /( 用户时间 + GC时间) [PS]
  - 选择回收器组合
  - 计算内存需求（经验值 1.5G 16G）
  - 选定CPU（越高越好）
  - 设定年代大小、升级年龄
  - 设定日志参数
    - -Xloggc:/opt/xxx/logs/xxx-xxx-gc-%t.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=20M -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCCause
    - 或者每天产生一个日志文件
  - 观察日志情况
- 案例1：垂直电商，最高每日百万订单，处理订单系统需要什么样的服务器配置？
  ```
  这个问题比较业余，因为很多不同的服务器配置都能支撑(1.5G 16G)
  1小时360000集中时间段， 100个订单/秒，（找一小时内的高峰期，1000订单/秒）
  经验值，
  非要计算：一个订单产生需要多少内存？512K * 1000 500M内存
  专业一点儿问法：要求响应时间100ms
  压测！
  ```

- 案例2：12306遭遇春节大规模抢票应该如何支撑？
  ```
  12306应该是中国并发量最大的秒杀网站：
  号称并发量100W最高
  CDN -> LVS -> NGINX -> 业务系统 -> 每台机器1W并发（10K问题） 100台机器
  普通电商订单 -> 下单 ->订单系统（IO）减库存 ->等待用户付款
  12306的一种可能的模型： 下单 -> 减库存 和 订单(redis kafka) 同时异步进行 ->等付款
  减库存最后还会把压力压到一台服务器
  可以做分布式本地库存 + 单独服务器做库存均衡
  大流量的处理方法：分而治之
  ```

- 怎么得到一个事务会消耗多少内存？
  ```
  弄台机器，看能承受多少TPS？是不是达到目标？扩容或调优，让它达到
  用压测来确定
  ```

##### 8.2 优化环境
1. 有一个50万PV的资料类网站（从磁盘提取文档到内存）原服务器32位，1.5G
的堆，用户反馈网站比较缓慢，因此公司决定升级，新的服务器为64位，16G
的堆内存，结果用户反馈卡顿十分严重，反而比以前效率更低了.
    - 为什么原网站慢?
  很多用户浏览数据，很多数据load到内存，内存不足，频繁GC，STW长，响应时间变慢;
    - 为什么会更卡顿？
  内存越大，FGC时间越长
    - 咋办？：PS +PO -> PN + CMS  或者 G1


2. 系统CPU经常100%，如何调优？(面试高频)
  - CPU100%那么一定有线程在占用系统资源
    - 找出哪个进程cpu高（top）
    - 该进程中的哪个线程cpu高（top -Hp）
    - 导出该线程的堆栈 (jstack)
    - 查找哪个方法（栈帧）消耗时间 (jstack)
    - 工作线程占比高 | 垃圾回收线程占比高


3. 系统内存飙高，如何查找问题？（面试高频）
  - 堆栈比较多 -> 导出堆内存 (jmap)
  - 分析 (jhat jvisualvm mat jprofiler ... )


4. 如何监控JVM
  - jstat jvisualvm jprofiler arthas top...


##### 8.3 解决JVM中运行中的问题
###### 一个案例理解常用工具

1. 测试代码

  ```java
  package com.mashibing.jvm.gc;

  import java.math.BigDecimal;
  import java.util.ArrayList;
  import java.util.Date;
  import java.util.List;
  import java.util.concurrent.ScheduledThreadPoolExecutor;
  import java.util.concurrent.ThreadPoolExecutor;
  import java.util.concurrent.TimeUnit;

  /**
   * 从数据库中读取信用数据，套用模型，并把结果进行记录和传输
   */

  public class T15_FullGC_Problem01 {

      private static class CardInfo {
          BigDecimal price = new BigDecimal(0.0);
          String name = "张三";
          int age = 5;
          Date birthdate = new Date();

          public void m() {}
      }

      private static ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(50,
              new ThreadPoolExecutor.DiscardOldestPolicy());

      public static void main(String[] args) throws Exception {
          executor.setMaximumPoolSize(50);

          for (;;){
              modelFit();
              Thread.sleep(100);
          }
      }

      private static void modelFit(){
          List<CardInfo> taskList = getAllCardInfo();
          taskList.forEach(info -> {
              // do something
              executor.scheduleWithFixedDelay(() -> {
                  //do sth with info
                  info.m();

              }, 2, 3, TimeUnit.SECONDS);
          });
      }

      private static List<CardInfo> getAllCardInfo(){
          List<CardInfo> taskList = new ArrayList<>();

          for (int i = 0; i < 100; i++) {
              CardInfo ci = new CardInfo();
              taskList.add(ci);
          }

          return taskList;
      }
  }
  ```

2. java -Xms200M -Xmx200M -XX:+PrintGC com.mashibing.jvm.gc.T15_FullGC_Problem01
3. 一般是运维团队首先受到报警信息（CPU Memory）
4. top命令观察到问题：内存不断增长 CPU占用率居高不下
5. top -Hp 观察进程中的线程，哪个线程CPU和内存占比高（命令：top -Hp 进程号）
6. jps定位具体java进程
jstack 定位线程状况（jstack 运行的线程号），重点关注：WAITING BLOCKED
eg.
waiting on <0x0000000088ca3310> (a java.lang.Object)
假如有一个进程中100个线程，很多线程都在waiting on  ，一定要找到是哪个线程持有这把锁
怎么找？搜索jstack dump的信息，找 ，看哪个线程持有这把锁RUNNABLE
作业：1：写一个死锁程序，用jstack观察 2 ：写一个程序，一个线程持有锁不释放，其他线程等待
7. 为什么阿里规范里规定，线程的名称（尤其是线程池）都要写有意义的名称
怎么样自定义线程池里的线程名称？（自定义ThreadFactory）
8. jinfo pid 列出进程详细信息
9. jstat -gc 动态观察gc情况 / 阅读GC日志发现频繁GC / arthas观察 / jconsole/jvisualVM/ Jprofiler（最好用）
jstat -gc 4655 500 : 每个500个毫秒打印GC的情况
如果面试官问你是怎么定位OOM问题的？如果你回答用图形界面（错误）
  - 已经上线的系统不用图形界面用什么？（cmdline arthas）
  - 图形界面到底用在什么地方？测试！测试的时候进行监控！（压测观察）
10. jmap - histo 4655 | head -20，查找4555进程的前20行，有多少对象产生；
11. jmap -dump:format=b,file=xxx pid ：

  ```
  线上系统，内存特别大，jmap执行期间会对进程产生很大影响，甚至卡顿（电商不适合）
  1：设定了参数HeapDump，OOM的时候会自动产生堆转储文件
  2：很多服务器备份（高可用），停掉这台服务器对其他服务器不影响
  3：在线定位(一般小点儿公司用不到)
  ```

12. java -Xms20M -Xmx20M -XX:+UseParallelGC -XX:+HeapDumpOnOutOfMemoryError com.mashibing.jvm.gc.T15_FullGC_Problem01
13. 使用MAT / jhat /jvisualvm 进行dump文件分析
https://www.cnblogs.com/baihuitestsoftware/articles/6406271.html
jhat -J-mx512M xxx.dump
http://192.168.17.11:7000
拉到最后：找到对应链接
可以使用OQL查找特定问题对象
14. 找到代码的问题

###### jconsole远程连接
1. 程序启动加入参数：

  ```bash
  java -Djava.rmi.server.hostname=192.168.17.11 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=11111 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false XXX
  ```

2. 如果遭遇 Local host name unknown：XXX的错误，修改/etc/hosts文件，把XXX加入进去
  ```
  192.168.17.11 basic localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
  ```

3. 关闭linux防火墙（实战中应该打开对应端口）

  ```bash
  service iptables stop
  chkconfig iptables off #永久关闭
  ```

4. windows上打开 jconsole远程连接 192.168.17.11:11111


###### jvisualvm远程连接

https://www.cnblogs.com/liugh/p/7620336.html （简单做法）

###### jprofiler (收费)

##### arthas 在线排查工具
- 为什么需要在线排查？
  - 在生产上我们经常会碰到一些不好排查的问题，例如线程安全问题，用最简单的threaddump或者heapdump不好查到问题原因。为了排查这些问题，有时我们会临时加一些日志，比如在一些关键的函数里打印出入参，然后重新打包发布，如果打了日志还是没找到问题，继续加日志，重新打包发布。对于上线流程复杂而且审核比较严的公司，从改代码到上线需要层层的流转，会大大影响问题排查的进度。
- jvm观察jvm信息
- thread定位线程问题
- dashboard 观察系统情况
- heapdump + jhat分析
- jad反编译
  - 动态代理生成类的问题定位
  - 第三方的类（观察代码）
  - 版本问题（确定自己最新提交的版本是不是被使用）
- redefine 热替换
  - 目前有些限制条件：只能改方法实现（方法已经运行完成），不能改方法名，不能改属性
- sc（search class）
- watch（watch method）
- 没有包含的功能：jmap

#### 案例汇总
OOM产生的原因多种多样，有些程序未必产生OOM，不断FGC（CPU飙高，但内存回收特别少）
- 硬件升级系统反而卡顿的问题（见上）
- 线程池不当运用产生OOM问题（见上）
- smile jira 问题
  - 实际系统不断重启
  - 解决问题 加内存 +  G1
  - 真正的问题在哪儿？不知道
- tomcat http-header-size 参数设置过大问题
- lambda表达式导致方法区溢出问题（MethodArea）
  - LambdaGC.java -XX:MaxMetaspaceSize=9M -XX:PrintGCDetails

  ```
  "C:\Program Files\Java\jdk1.8.0_181\bin\java.exe" -XX:MaxMetaspaceSize=9M -XX:+PrintGCDetails "-javaagent:C:\Program Files\JetBrains\IntelliJ IDEA Community Edition 2019.1\lib\idea_rt.jar=49316:C:\Program Files\JetBrains\IntelliJ IDEA Community Edition 2019.1\bin" -Dfile.encoding=UTF-8 -classpath "C:\Program Files\Java\jdk1.8.0_181\jre\lib\charsets.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\deploy.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\access-bridge-64.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\cldrdata.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\dnsns.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\jaccess.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\jfxrt.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\localedata.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\nashorn.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\sunec.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\sunjce_provider.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\sunmscapi.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\sunpkcs11.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\ext\zipfs.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\javaws.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\jce.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\jfr.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\jfxswt.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\jsse.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\management-agent.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\plugin.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\resources.jar;C:\Program Files\Java\jdk1.8.0_181\jre\lib\rt.jar;C:\work\ijprojects\JVM\out\production\JVM;C:\work\ijprojects\ObjectSize\out\artifacts\ObjectSize_jar\ObjectSize.jar" com.mashibing.jvm.gc.LambdaGC
  [GC (Metadata GC Threshold) [PSYoungGen: 11341K->1880K(38400K)] 11341K->1888K(125952K), 0.0022190 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
  [Full GC (Metadata GC Threshold) [PSYoungGen: 1880K->0K(38400K)] [ParOldGen: 8K->1777K(35328K)] 1888K->1777K(73728K), [Metaspace: 8164K->8164K(1056768K)], 0.0100681 secs] [Times: user=0.02 sys=0.00, real=0.01 secs]
  [GC (Last ditch collection) [PSYoungGen: 0K->0K(38400K)] 1777K->1777K(73728K), 0.0005698 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
  [Full GC (Last ditch collection) [PSYoungGen: 0K->0K(38400K)] [ParOldGen: 1777K->1629K(67584K)] 1777K->1629K(105984K), [Metaspace: 8164K->8156K(1056768K)], 0.0124299 secs] [Times: user=0.06 sys=0.00, real=0.01 secs]
  java.lang.reflect.InvocationTargetException
  	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
  	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
  	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  	at java.lang.reflect.Method.invoke(Method.java:498)
  	at sun.instrument.InstrumentationImpl.loadClassAndStartAgent(InstrumentationImpl.java:388)
  	at sun.instrument.InstrumentationImpl.loadClassAndCallAgentmain(InstrumentationImpl.java:411)
  Caused by: java.lang.OutOfMemoryError: Compressed class space
  	at sun.misc.Unsafe.defineClass(Native Method)
  	at sun.reflect.ClassDefiner.defineClass(ClassDefiner.java:63)
  	at sun.reflect.MethodAccessorGenerator$1.run(MethodAccessorGenerator.java:399)
  	at sun.reflect.MethodAccessorGenerator$1.run(MethodAccessorGenerator.java:394)
  	at java.security.AccessController.doPrivileged(Native Method)
  	at sun.reflect.MethodAccessorGenerator.generate(MethodAccessorGenerator.java:393)
  	at sun.reflect.MethodAccessorGenerator.generateSerializationConstructor(MethodAccessorGenerator.java:112)
  	at sun.reflect.ReflectionFactory.generateConstructor(ReflectionFactory.java:398)
  	at sun.reflect.ReflectionFactory.newConstructorForSerialization(ReflectionFactory.java:360)
  	at java.io.ObjectStreamClass.getSerializableConstructor(ObjectStreamClass.java:1574)
  	at java.io.ObjectStreamClass.access$1500(ObjectStreamClass.java:79)
  	at java.io.ObjectStreamClass$3.run(ObjectStreamClass.java:519)
  	at java.io.ObjectStreamClass$3.run(ObjectStreamClass.java:494)
  	at java.security.AccessController.doPrivileged(Native Method)
  	at java.io.ObjectStreamClass.<init>(ObjectStreamClass.java:494)
  	at java.io.ObjectStreamClass.lookup(ObjectStreamClass.java:391)
  	at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1134)
  	at java.io.ObjectOutputStream.defaultWriteFields(ObjectOutputStream.java:1548)
  	at java.io.ObjectOutputStream.writeSerialData(ObjectOutputStream.java:1509)
  	at java.io.ObjectOutputStream.writeOrdinaryObject(ObjectOutputStream.java:1432)
  	at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1178)
  	at java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:348)
  	at javax.management.remote.rmi.RMIConnectorServer.encodeJRMPStub(RMIConnectorServer.java:727)
  	at javax.management.remote.rmi.RMIConnectorServer.encodeStub(RMIConnectorServer.java:719)
  	at javax.management.remote.rmi.RMIConnectorServer.encodeStubInAddress(RMIConnectorServer.java:690)
  	at javax.management.remote.rmi.RMIConnectorServer.start(RMIConnectorServer.java:439)
  	at sun.management.jmxremote.ConnectorBootstrap.startLocalConnectorServer(ConnectorBootstrap.java:550)
  	at sun.management.Agent.startLocalManagementAgent(Agent.java:137)

  ```

- 直接内存溢出问题（少见）
  - 《深入理解java虚拟机》P59，使用Unsafe分配直接内存，或者使用NIO的问题
- 栈溢出问题（递归调用、循环调用）
  - Xss设定太小
- 比较一下这两段程序的异同，分析哪一个是更优的写法：

```java
// 这种更优
Object o = null;
for (int i = 0; i < 100; i++) {
  o = new Object();
}


for (int i = 0; i < 100; i++) {
  Object o = new Object();
}
```

- 重写finalize引发频繁GC
  - 小米云，HBase同步系统，系统通过nginx访问超时报警，最后排查，C++程序员重写finalize引发频繁GC问题
    - 为什么C++程序员会重写finalize？（new delete）
    - finalize耗时较长（200ms）
- 如果有一个系统，内存一直消耗不超过10%，但是观察GC日志，发现FGC总是频繁产生，会是什么引起的？
  - System.gc();(显示调用gc，这个较low)
- Distuptor有个可以设置链的长度，如果过大，然后对象大，消费完不主动释放，会溢出。
- 用jvm都会溢出，mycat用崩过，1.6.5某个临时版本解析sql子查询算法有问题，9个exists的联合sql就导致生成几百万的对象。
- new 大量线程，会产生native thread OOM，（low）应该用线程池；
  - 解决：减少堆空间（太low了），预留更多内存产生 native thread；
  - JVM内存占物理内存比例50%~80%

#### GC常用参数
- -Xmn -Xms -Xmx -Xss：年轻代 最小堆 最大堆 栈空间
- -XX:+UseTLAB：使用TLAB，默认打开（不建议随便改）
- -XX:+PrintTLAB：打印TLAB的使用情况（不建议随便改）
- -XX:TLABSize：设置TLAB大小（不建议随便改）
- -XX:+DisableExplictGC：线上系统建议开启此参数，让System.gc()不管用 ，FGC
- -XX:+PrintGC
- -XX:+PrintGCDetails
- -XX:+PrintHeapAtGC：打印堆栈情况
- -XX:+PrintGCTimeStamps
- -XX:+PrintGCApplicationConcurrentTime (优先级低)：打印应用程序时间
- -XX:+PrintGCApplicationStoppedTime （优先级低）：打印暂停时长
- -XX:+PrintReferenceGC （重要性低）：记录回收了多少种不同引用类型的引用
- -verbose:class：类加载详细过程
- -XX:+PrintVMOptions
- -XX:+PrintFlagsFinal  -XX:+PrintFlagsInitial：必须会用
- -Xloggc:opt/log/gc.log
- -XX:MaxTenuringThreshold：升代年龄，最大值15
- 锁自旋次数 -XX:PreBlockSpin 热点代码检测参数-XX:CompileThreshold 逃逸分析 标量替换 ...
这些不建议设置

#### Parallel常用参数
- -XX:SurvivorRatio：Eden区与Survivor区的比例设置（默认8:1:1）
- -XX:PreTenureSizeThreshold：设置大对象的大小
- -XX:MaxTenuringThreshold
- -XX:+ParallelGCThreads：并行收集器的线程数，同样适用于CMS，一般设为和CPU核数相同
- -XX:+UseAdaptiveSizePolicy：自动选择各区大小比例

#### CMS常用参数
- -XX:+UseConcMarkSweepGC
- -XX:ParallelCMSThreads：CMS线程数量（默认为核数的一半）
- -XX:CMSInitiatingOccupancyFraction：使用多少比例的老年代后开始CMS收集，默认是68%(近似值)，如果频繁发生SerialOld卡顿，应该调小，（频繁CMS回收）
- -XX:+UseCMSCompactAtFullCollection：在FGC时进行压缩
- -XX:CMSFullGCsBeforeCompaction：多少次FGC之后进行压缩
- -XX:+CMSClassUnloadingEnabled
- -XX:CMSInitiatingPermOccupancyFraction：达到什么比例时进行Perm回收
- GCTimeRatio：设置GC时间占用程序运行时间的百分比
- -XX:MaxGCPauseMillis：停顿时间，是一个建议时间，GC会尝试用各种手段达到这个时间，比如减小年轻代

#### G1常用参数
- -XX:+UseG1GC
- -XX:MaxGCPauseMillis：建议值，G1会尝试调整Young区的块数来达到这个值
- -XX:GCPauseIntervalMillis：？GC的间隔时间
- -XX:+G1HeapRegionSize：分区大小，建议逐渐增大该值，1 2 4 8 16 32。随着size增加，垃圾的存活时间更长，GC间隔更长，但每次GC的时间也会更长；ZGC做了改进（动态区块大小）
- G1NewSizePercent：新生代最小比例，默认为5%
- G1MaxNewSizePercent：新生代最大比例，默认为60%
- GCTimeRatio：GC时间建议比例，G1会根据这个值调整堆空间
- ConcGCThreads：线程数量
- InitiatingHeapOccupancyPercent：启动G1的堆空间占用比例

#### 作业
- -XX:MaxTenuringThreshold控制的是什么？
A: 对象升入老年代的年龄
B: 老年代触发FGC时的内存垃圾比例
- 生产环境中，倾向于将最大堆内存和最小堆内存设置为：（为什么？）
A: 相同 B：不同
JDK1.8默认的垃圾回收器是：
A: ParNew + CMS
B: G1
C: PS + ParallelOld
D: 以上都不是
- 什么是响应时间优先？
- 什么是吞吐量优先？
- ParNew和PS的区别是什么？
- ParNew和ParallelOld的区别是什么？（年代不同，算法不同）
- 长时间计算的场景应该选择：A：停顿时间 B: 吞吐量
- 大规模电商网站应该选择：A：停顿时间 B: 吞吐量
- HotSpot的垃圾收集器最常用有哪些？
- 常见的HotSpot垃圾收集器组合有哪些？
- JDK1.7 1.8 1.9的默认垃圾回收器是什么？如何查看？
- 所谓调优，到底是在调什么？
- 如果采用PS + ParrallelOld组合，怎么做才能让系统基本不产生FGC
- 如果采用ParNew + CMS组合，怎样做才能够让系统基本不产生FGC
  - 加大JVM内存
  - 加大Young的比例
  - 提高Y-O的年龄
  - 提高S区比例
  - 避免代码内存泄漏
- G1是否分代？G1垃圾回收器会产生FGC吗？
- 如果G1产生FGC，你应该做什么？
  - 扩内存
  - 提高CPU性能（回收的快，业务逻辑产生对象的速度固定，垃圾回收越快，内存空间越大）
  - 降低MixedGC触发的阈值，让MixedGC提早发生（默认是45%）
- 问：生产环境中能够随随便便的dump吗？
小堆影响不大，大堆会有服务暂停或卡顿（加live可以缓解），dump前会有FGC
- 问：常见的OOM问题有哪些？
栈 堆 MethodArea 直接内存



### 参考资料

1. [https://blogs.oracle.com/
    ](https://blogs.oracle.com/jonthecollector/our-collectors)[jonthecollector](https://blogs.oracle.com/jonthecollector/our-collectors)[/our-collectors](https://blogs.oracle.com/jonthecollector/our-collectors)
2. https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html
3. http://java.sun.com/javase/technologies/hotspot/vmoptions.jsp
4. JVM调优参考文档：https://docs.oracle.com/en/java/javase/13/gctuning/introduction-garbage-collection-tuning.html#GUID-8A443184-7E07-4B71-9777-4F12947C8184
5. https://www.cnblogs.com/nxlhero/p/11660854.html 在线排查工具
6. https://www.jianshu.com/p/507f7e0cc3a3 arthas常用命令
7. Arthas手册：
    - 启动arthas java -jar arthas-boot.jar
    - 绑定java进程
    - dashboard命令观察系统整体情况
    - help 查看帮助
    - help xx 查看具体命令帮助
8. jmap命令参考： https://www.jianshu.com/p/507f7e0cc3a3
    - jmap -heap pid
    - jmap -histo pid
    - jmap -clstats pid
