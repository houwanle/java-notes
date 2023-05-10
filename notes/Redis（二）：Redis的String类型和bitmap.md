# Redis（二）：Redis的String类型和bitmap

**JVM中，一个线程的成本（线程栈默认1MB）**
- 线程多了调度成本CPU浪费
- 内存成本

## String类型常用命令
> type 可查看value的类型

**字符串常用操作**
- set
- get
- append
- setrange
- getrange
- strlen

**数值常用操作**
- INCR

**bitmap**
- setbit:设置二进制位的值
- bitpos:寻找二进制位在对应字节中第一次出现的位置

  ```bash
  # 查找k1第一个字节中的1的位置
  bitops k1 1 0 0
  ```

- bitcount:统计字节中1出现的次数

  ```bash
  #查k1的value的二进制位第一个字节中1出现的次数
  bitcount k1 0 0
  ```

- bitop:

**正反向索引**
- 正向索引：0,1,2...
- 反向索引：-1,-2,-3...

**bitmap应用场景**
- 1.统计用户登录天数且窗口随机

  ```bash
  setbit sean 1 1
  setbit sean 7 1
  setbit sean 364 1
  strlen sean
  bitcount sean -2 -1
  ```

- 2.618做活动：给登录的用户（共2亿用户）送礼物，大库需要备货多少礼物？
  - 用户：僵尸用户、冷热用户、忠诚用户
  - 活跃用户统计、随机窗口
    - 比如：1号~3号 连续登录需要去重

    ```bash
    setbit 20190101 1 1
    setbit 20190102 1 1
    setbit 20190102 7 1
    bitop or destkey 20190101 20190102
    bitcount destkey 0 -1
    ```

**字符集ascii，其他一般叫做扩展字符集；扩展一般对不再对ascii重编码**