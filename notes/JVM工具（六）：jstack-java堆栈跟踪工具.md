## JVM工具（六）：jstack-java堆栈跟踪工具

### jstack简介

jstack（stack trace for java）是java虚拟机自带的一种堆栈跟踪工具。jstack用于打印出给定的java进程ID或core file或远程调试服务的Java堆栈信息，如果是在64位机器上，需要指定选项”-J-d64”，Windows的jstack使用方式只支持以下的这种方式：```jstack [-l] pid```

主要分为两个功能：
- 针对活着的进程做本地的或远程的线程dump
- 针对core文件做线程dump

jstack用于生成java虚拟机当前时刻的线程快照。

线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。

线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。

如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。

另外，jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息, 如果现在运行的java程序呈现hung的状态，jstack是非常有用的。

> So，jstack命令主要用来查看Java线程的调用堆栈的，可以用来分析线程问题（如死锁）。

### 线程状态

想要通过jstack命令来分析线程的情况的话，首先要知道线程都有哪些状态，下面这些状态是我们使用jstack命令查看线程堆栈信息时可能会看到的线程的几种状态：

- NEW：未启动的。不会出现在Dump中。
- RUNNABLE：在虚拟机内执行的。运行中状态，可能里面还能看到locked字样，表明它获得了某把锁。
- BLOCKED：受阻塞并等待监视器锁。被某个锁(synchronizers)給block住了。
- WATING：无限期等待另一个线程执行特定操作。等待某个condition或monitor发生，一般停留在park(), wait(),sleep(),join() 等语句里。
- TIMED_WATING：有时限的等待另一个线程的特定操作。和WAITING的区别是wait() 等语句加上了时间限制 wait(timeout)。
- TERMINATED：已退出的。

关于线程状态，具体也可以查看：java.lang.Thread.State类。