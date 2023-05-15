# Redis（五）：持久化RDB、fork、copyonwrite、AOF、RDB与AOF混合使用

**管道**
- 衔接，前一个命令的输出作为后一个命令的输入；
- 管道会触发创建【子进程】
  - 使用linux的时候：会出现父子进程的概念，父进程的数据，子进程是否可以看得到？
    - 常规思想，进程间是有数据隔离的。
    - 进阶思想，父进程其实可以让子进程看到数据。
    - linux中，export的环境变量，子进程的修改不会破坏父进程，父进程的修改也不会破坏子进程。
    - 创建子进程的速度应该是什么程度（如果父进程是redis，内存数据比如10G)
      - 速度
      - 内存空间够不够
    - fork()
      - 速度快
      - 空间小
      - 实现了copy on write:写时复制
        - 创建子进程并不发生复制，创建进程变快了
        - 根据经验，不可能父子进程把所有数据都改一边
        - 操作的是指针

  ```bash
  echo $$
  #结果
  114017

  echo $$ | more
  #结果
  114017

  echo $BAHSPID
  #结果
  114017

  # $取一个普通变量的名字时，其优先级低于管道
  echo $BASHPID | more
  #结果
  114105
  ```

**存储层**
- 快照/副本
- 日志

## RDB
- 时点性
- 非阻塞：redis继续对外提供服务
  - fork: 系统调用
  - copy on write: 内核机制
  - 8点创建子进程，8点之后的数据修改不会影响子进程（父子进程对数据的修改，对方看不到）
  - rdb的方式落地成文件

**实现或触发RDB的方式**
- 手动触发（命令行触发，前台阻塞）：save，很明确，比如关机维护
- 后端非阻塞方式触发：bgsave（fork，创建子进程）
- 配置文件中给出 bgsave 的规则：用的是save标识，触发的是bgsave

  ```bash
  save 900 1
  save 300 10
  save 60 10000

  dbfilename dump.rdb
  dir /var/lib/redis/6379
  ```

- 弊端：
  - 不支持拉链，只有一个dumb.rdb
  - 丢失数据相对多一些，时点与时点之间的窗口数据容易丢失（8点得到一个rdb，9点刚要落一个rdb，挂机了）
  - 优点：类似Java中国的序列化，恢复的速度相对快。

## AOF
- redis的写操作记录到文件中
  - 丢失数据少
  - redis中RDB和AOF可以同时开启
    - 如果开启了AOF只会用AOF恢复
      - 4.0版本后，AOF中包含RDB全量，增加记录新的写操作
  - 弊端：体量无限变大，恢复慢
  - 优点：日志如果能保住，还是可以用的
    - 设计一个方案让日志、AOF足够小
      - 4.0版本以前：重写，删除抵消的命令，合并重复的命令，最终也是一个纯指令的日志文件
      - 4.0版本以后：重写，将老的数据RDB到AOF文件中，将增量的数据以指令的方式Append到AOF，故AOF是一个混合体，利用了RDB的快，利用了日志的全量
    - hdfs，fsimage+edits.log(让日志只记录增量、合并的过程)
- 原点：redis是内存数据库
  - 写操作会触发IO
  
    ```bash
    appendonly yes
    appendfilename "appendonly.aof"

    auto-aof-rewrite-percentage 100
    auto-aof-rewrite-min-size 64mb

    #数据最可靠
    appendfsync alaways

    #每秒 每秒种调一次flush，会丢数据
    appendfsync everysec
    appendfsync no
    ```

> redis运行了10年,且开启了AOF，10年头，redis挂了
1. AOF多大：很大，10T，恢复，会不会溢出
2. 恢复要多久，恢复用5年

