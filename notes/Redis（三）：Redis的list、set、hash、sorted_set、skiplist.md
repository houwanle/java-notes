# Redis（三）：Redis的list、set、hash、sorted_set、skiplist

## list

**list常用操作**
- lpush:从左边往里放

  ```bash
  # 其value的存放顺序为：f e d c b a
  lpush k1 a b c d e f
  ```
- rpush:从右边往里放

  ```bash
  # 其value的存放顺序为：a b c d e f
  rpush k2 a b c d e f
  ```

- lpop:从左边往外弹

  ```bash
  lpop k1
  #结果：f
  ```

- rpop:从右边往外弹
- lrange:按指定范围打印对应key的value

  ```bash
  lrange k1 0 -1
  #结果
  f
  e
  d
  c
  b
  a
  ```

- lindex:根据索引下标取出对应的value

  ```bash
  # 取出k1中下标为2对应的value
  lindex k1 2
  #结果 d

  # 取出k1中最后一个值
  lindex k1 -1
  # 结果 a
  ```

- lset:根据索引下标更新对应的value

  ```bash
  # 更新k1中索引下标为3的value
  lset k1 3 xxxxx
  ```

- lrem:按索引下标移除list中指定value（移除索引下标数量个）

  ```bash
  # 新建 k3 list
  lpush k3 1 a 2 b 3 a 4 c 5 a 6 d

  # 打印k3全部value
  lrange k3 0 -1
  #结果
  d
  6
  a
  5
  c
  4
  a
  3
  b
  2
  a
  1

  # 从索引下标为2的位置开始移除k3中2个value为a的元素
  lrem k3 2 a

  # 再次打印k3全部value
  lrange k3 0 -1
  #结果
  d
  6
  5
  c
  4
  3
  b
  2
  a
  1
  ```


**list可以实现**
- 做栈：同向命令（lpush/lpop）
- 做队列：反向命令（lpush/rpop)
- 做数组：