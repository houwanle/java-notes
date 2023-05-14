# Redis（四）：消息订阅、pipeline、事务、modules、布隆过滤器和缓存LRU
- 微信群中新入群的成员既能看到新的消息又能看到历史消息，怎么实现？
  - 历史性（历史消息）：
    - 3天之内的数据：sorted set 留一个区间或窗口
  - 实时性（新的消息）：
    - 发布订阅：
      - redis缓存解决数据的读请求快

## 消息订阅

- publish



## 管道（pipeline）


## 事务

### 事务常用命令
- DISCARD:当执行 DISCARD 命令时，事务会被放弃，事务队列会被清空，并且客户端会从事务状态中退出；
- EXEC:在服务端，按照顺序执行命令（执行事务）
  - 多个客户端发指令给服务端时，谁的exec指令先到，先执行谁的指令；
- MULTI:开启事务
- UNWATCH:
- WATCH:实现CAS
  - 开启事务之前加一个监控某个k，后续若k被更改了，k1对应的exec是不执行的（k对应的事务不执行的）


**为什么 Redis 不支持回滚（roll back）**
- 优点：
  - Redis 命令只会因为错误的语法而失败（并且这些问题不能在入队时发现），或是命令用在了错误类型的键上面：这就是说，从实用性的角度来说，失败的命令是由编程错误造成的，而这些错误应该在开发得过程中被发现，而不应该出现在生产环境中。
  - 因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。
> 有种观点认为 Redis处理事务的做法会产生 bug，然而需要注意的是，在通常情况下，回滚并不能解决编程错误带来的问题。举个例子，如果你本来想通过 INCR 命令将将键的值加1，却不小心加上了2，又或者对错误类型的键执行了 INCR，回滚是没有办法处理这些情况的。

## modules

## 布隆过滤器(RedisBloom)
> 用小的空间去解决大的数据量的匹配问题（解决缓存穿透的问题）

**安装**
- 访问redis.io
- modules
- 访问redisbloom的github：https://github.com/RedisBloom/RedisBloom
- linux中 wget *.zip
- yum install unzip
- unzip *.zip
- make
- cp bloom.so /opt/lele/redis5/
- redis-server --loadmodule /opt/mashibing/redis5/redisbloom.so
- 新的客户端 redis-cli
- bf.add ooxx abc
- bf.exits abc
- bf.exits sdfsdf



**原理**
  - 1.数据库中有什么？
  - 2.数据库中有的数据就在bitmap中标记出来
  - 3.请求的有可能被误标记；
  - 4.但是，一定概率会大量减少放行（穿透）
  - 5.成本低

**实现boom的三种形式**
- 1.client实现bloom算法，自己承载bitmap，redis的server端不用做什么；
- 2.client实现bloom算法，redis的server端承载bitmap；
- 3.client不做什么，redis的server端实现bloom算法，承载bitmap；
  - 符合微服务、server mesh的理念；

> bloom过滤器、升级版bloom（counting bloom）、布谷鸟过滤器(cf.add)，了解

**注意：**
1. 穿透了，不存在，client需要增加redis中的key，value标记；
2. 数据库增加了元素，需要完成元素对bloom的添加；


## 缓存LRU

### redis作为数据库和缓存的区别
- 缓存数据“不重要”；
- 缓存不是全量数据；
- 缓存应该随着访问而变化；（热点数据）

**redis作为缓存**
- redis中的数据怎么能随着业务变化而变化：只保留热数据，因为内存大小有限的，也就是瓶颈；


**key的有效期**
- 产生原因
  - 业务逻辑导致key的有效期
  - 业务运转导致：内存是有限的，随着访问的变化，应该淘汰掉冷数据


**redis的内存回收策略**
> 当maxmemory限制达到的时候，redis会使用的行为由redis的maxmemory-policy配置指令进行配置。

策略：
- noeviction:直接返回错误，不淘汰任何已经存在的redis键（大部分的写入指令，但del和几个例外）；
- allkey-lru:所有的键使用LRU算法进行淘汰；
- volatile-lru:有过期时间的使用LRU算法进行淘汰；
- allkeys-random:随机删除redis键
- volatile-random:随机删除有过期时间的redis键
- volatile-ttl:删除快过期的redis键
- volatile-lfu:根据lfu算法从有过期时间的键删除；
- allkeys-lfu:根据lfu算法从所有键删除；


- maxmemory
- maxmemory-policy noeviction

