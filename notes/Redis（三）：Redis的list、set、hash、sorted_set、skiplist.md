# Redis（三）：Redis的list、set、hash、sorted_set、skiplist

## list
> 可有重复出现的元素、有序的（存入的顺序，不会排序）

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

- linsert:在指定的value值前或后插入一个值（list若存在多个指定的value，只能在一个前面或者后面插入）

  ```bash
  #在value为6的后面插入一个a
  linsert k3 after 6 a

  lrange k3 0 -1
  #结果
  d
  6
  a
  5
  c
  4
  3
  b
  2
  a
  1

  # 在value为3的前面插入一个a
  linsert k3 before 3 a

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
  ```

- llen:统计list中的元素个数
- blpop
- ltrim:两端删除

  ```bash
  lpush k4 a f d e r c g df s

  #由于索引是两端，所以一个元素都不会删
  ltrim k4 0 -1

  #删除前两个和最后一个元素
  ltrim k4 2 -2

  ```


**list可以实现**
- 做栈：同向命令（lpush/lpop）
- 做队列：反向命令（lpush/rpop)
- 做数组：
- 阻塞、单播队列（blpop/）：FIFO

## hash


```bash
set sean::name 'zzl'
get sean::name
#结果
zzl

set sean::age 18
get sean::age
#结果
18

keys sean*
#结果
sean::name
sean::age
```

**hash常用操作**
- hset

  ```bash
  hset sean name zzl
  ```
- hmset

  ```bash
  hmset sean age 18 address bj
  hget sean name
  #结果
  zzl

  hmget sean name age
  #结果
  zzl
  18

  hkeys sean
  #结果
  name
  age
  address

  hvals sean
  #结果
  zzl
  18
  bj

  hgetall sean
  #结果
  name
  zzl
  age
  18
  address
  bj

  hincrbyfloat sean age 0.5
  #结果
  18.5

  hincrbyfloat sean age -1
  #结果
  17.5
  ```

**应用场景**
- 对field数值计算：点赞、收藏、详情页

## Set
- 去重的，不维护顺序（无序去重）
- 集合操作（用的非常多）
- 随机事件（srandmember key count）：抽奖（用户数量不确定、中奖是否重复、解决家庭争斗）
  - count若为正数：取出一个去重的结果集（不能超过已有集合）
  - count若为负数：取出一个带重复的结果集，一定满足你要的数量
  - count若为0：不返回

    ```bash
    sadd k1 tom ooxx xxoo xoxo oxox xoox oxxo

    ##########一个人最多只能中一件礼物########
    srandmember k1 3
    #返回随机3个去重的结果
    tom
    xoox
    oxox

    srandmember k1 3
    #返回随机3个去重的结果
    ooxx
    tom
    oxox

    ##########一个人最多只能中一件礼物########

    srandmember k1 -3
    #
    oxox
    ooxx
    xxoo

    srandmember k1 -3
    #会出现重复的结果
    oxox
    ooxx
    oxox

    ##############抽奖的人少于礼物数或取名字###############
    srandmember k1 -20

    #######年会抽奖######
    spop k1


    ```

**Set常用操作**
- sadd:添加元素
- smembers:去重
- srem:移除
- sinter:交集
- sinterstore:会将交集结果存到服务端的目标key中
- sunion:并集，生成的结果会去重
- sunionstore:
- sdiff:差集，左差还是右差取决于谁放在前面
- spop:取出一个元素（不再放回）
- zincrby:根据元素增加其分值
- zunionsstore:并集

  ```bash
  sadd k1 tom sean peter ooxx tom xxoo

  smembers k1
  #结果
  sean
  tom
  ooxx
  peter
  xxoo

  srem k1 ooxx xxoo
  smembers k1
  #结果
  tom
  peter
  sean


  sadd k2 1 2 3 4 5
  sadd k3 4 5 6 7 8

  sinter k2 k3
  #结果
  4
  5

  sinterstore dest k2 k3
  smembers dest
  #结果
  4
  5

  sunion k2 k3
  #结果
  1
  2
  3
  4
  5
  6
  7
  8

  sdiff k2 k3
  #结果
  1
  2
  3

  sdiff k3 k2
  #结果
  6
  7
  8
  ```

## sorted set
- 去重，排序
- 物理内存，左小右大，不随命令发生变化（zrange/zrevrange）
- 集合操作（并集、交集），权重和聚合
- 排序是怎么实现的，增删改查的速度：skip list(跳跃表)

**sorted set常用操作**
- zadd:增加元素
- zrange:按照索引取出元素
- zrangebyscore:按照分值取出元素
- zrevrange:
- zscore:通过元素取出其分值
- zrank:通过元素取出其排名



```bash
zadd k1 8 apple 2 banana 3 orange

zrange k1 0 -1
#结果
banana
orange
apple

zrange k1 0 -1 withscores
#结果
banana
2
orange
3
apple
8

zrangebyscore k1 3 8
#结果
orange
apply

zrange k1 0 -1
#结果
banana
orange

#取出分值由高到底的前两个元素
zrevrange k1 0 1
#结果
apple
orange

zscore k1 apple
#结果
8

zrank k1 apple
#结果
2

#给banana的分值增加2.5
zincrby k1 2.5 banana
#结果
4.5


zadd k1 80tom 60 sean 70 baby
zadd k2 60 tom 100 sean 40 yiming

zunionstore unkey 2 k1 k2
zrange unkey 0 -1 withsocres
#结果
yiming
40
baby
70
tom
140
sean 
160

#加权重
zunionstore unkey1 2 k1 k2 weights 1 0.5
zrange unkey1 0 -1 withscores
#结果
yiming
20
baby
70
sean
110
tom
110

#求最大值
zunionstore unkey1 2 k1 k2 aggregate max
zrange unkey1 0 -1 withscores
#结果
yiming
40
baby
70
tom
80
sean
100
```