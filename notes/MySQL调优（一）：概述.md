## MySQL调优（一）：概述

  ![MySQL调优（一）：概述_1](./pics/MySQL调优（一）：概述_1.png)

### 性能监控
#### 使用show profile查询剖析工具，可以指定具体的type

- 此工具默认是禁用的，可以通过服务器变量在会话级别动态的修改；

  ```
  set profiling=1;
  ```

- 当设置完成之后，在服务器上执行的所有语句，都会测量其耗费的时间和其他一些查询执行状态变更相关的数据。

  ```sql
  select * from emp;
  ```

- 在mysql的命令行模式下只能显示两位小数的时间，可以使用如下命令查看具体的执行时间；

  ```sql
  show profiles;
  ```

- 执行如下命令可以查看详细的每个步骤的时间；

  ```sql
  show profile for query 1;
  ```

**type**
- all：显示所有性能信息 `show profile all for query n`
- block io：显示块io操作的次数 `show  profile block io for query n`
- context switches：显示上下文切换次数，被动和主动 `show profile context switches for query n`
- cpu：显示用户cpu时间、系统cpu时间 `show profile cpu for query n`
- IPC：显示发送和接受的消息数量 `show profile ipc for query n`
- Memory：暂未实现
- page faults：显示页错误数量 `show profile page faults for query n`
- source：显示源码中的函数名称与位置 `show profile source for query n`
- swaps：显示swap的次数 `show profile swaps for query n`

#### 使用performance schema来更加容易的监控mysql
> MySQL调优（二）：performance_schema详解.md


#### 使用show processlist查看连接的线程个数，来观察是否有大量线程处于不正常的状态或者其他不正常的特征
**属性说明**
- id表示session id
- user表示操作的用户
- host表示操作的主机
- db表示操作的数据库
- command表示当前状态
  - sleep：线程正在等待客户端发送新的请求
  - query：线程正在执行查询或正在将结果发送给客户端
  - locked：在mysql的服务层，该线程正在等待表锁
  - analyzing and statistics：线程正在收集存储引擎的统计信息，并生成查询的执行计划
  - Copying to tmp table：线程正在执行查询，并且将其结果集都复制到一个临时表中
  - sorting result：线程正在对结果集进行排序
  - sending data：线程可能在多个状态之间传送数据，或者在生成结果集或者向客户端返回数据
- info表示详细的sql语句
- time表示相应命令执行时间
- state表示命令执行状态


**数据库连接池**


### schema与数据类型优化
#### 数据类型的优化
##### 1. 更小的通常更好
- 应该尽量使用可以正确存储数据的最小数据类型，更小的数据类型通常更快，因为它们占用更少的磁盘、内存和CPU缓存，并且处理时需要的CPU周期更少，但是要确保没有低估需要存储的值的范围，如果无法确认哪个数据类型，就选择你认为不会超过范围的最小类型
- 案例：设计两张表，设计不同的数据类型，查看表的容量

##### 2. 简单就好
- 简单数据类型的操作通常需要更少的CPU周期，例如，
  - 整型比字符操作代价更低，因为字符集和校对规则是字符比较比整型比较更复杂，
  - 使用mysql自建类型而不是字符串来存储日期和时间
  - 用整型存储IP地址
- 案例：创建两张相同的表，改变日期的数据类型，查看SQL语句执行的速度

##### 3. 尽量避免null
- 如果查询中包含可为NULL的列，对mysql来说很难优化，因为可为null的列使得索引、索引统计和值比较都更加复杂，坦白来说，通常情况下null的列改为not null带来的性能提升比较小，所有没有必要将所有的表的schema进行修改，但是应该尽量避免设计成可为null的列;

##### 4. 实际细则
**4.1 整数类型**
- 可以使用的几种整数类型：TINYINT，SMALLINT，MEDIUMINT，INT，BIGINT分别使用8，16，24，32，64位存储空间。
- 尽量使用满足需求的最小数据类型；

**4.2 字符和字符串类型**

```
char长度固定，即每条数据占用等长字节空间；最大长度是255个字符，适合用在身份证号、手机号等定长字符串；

varchar可变程度，可以设置最大长度；最大空间是65535个字节，适合用在长度可变的属性；

text不设置长度，当不知道属性的最大长度时，适合用text;按照查询速度：char>varchar>text；
```

- varchar根据实际内容长度保存数据
  - 使用最小的符合需求的长度；
  - varchar(n) n小于等于255使用额外一个字节保存长度，n>255使用额外两个字节保存长度；
  - varchar(5)与varchar(255)保存同样的内容，硬盘存储空间相同，但内存空间占用不同，是指定的大小；
  - varchar在mysql5.6之前变更长度，或者从255一下变更到255以上时，都会导致锁表；
  - 应用场景
    - 存储长度波动较大的数据，如：文章，有的会很短有的会很长；
    - 字符串很少更新的场景，每次更新后都会重算并使用额外存储空间保存长度；
    - 适合保存多字节字符，如：汉字，特殊字符等；
- char固定长度的字符串
  - 最大长度：255
  - 会自动删除末尾的空格
  - 检索效率、写效率 会比varchar高，以空间换时间
  - 应用场景
    - 存储长度波动不大的数据，如：md5摘要；
    - 存储短字符串、经常更新的字符串；

**4.3 BLOB和TEXT类型**
- MySQL 把每个 BLOB 和 TEXT 值当作一个独立的对象处理。两者都是为了存储很大数据而设计的字符串类型，分别采用二进制和字符方式存储。

**4.4 datetime和timestamp**
- datetime
  - 占用8个字节；
  - 与时区无关，数据库底层时区配置，对datetime无效；
  - 可保存到毫秒；
  - 可保存时间范围大；支持的时间范围是“1000-00-00 00:00:00” 到 “9999-12-31 23:59:59”。
  - 不要使用字符串存储日期类型，占用空间大，损失日期类型函数的便捷性；
- timestamp
  - 占用4个字节；
  - 时间范围：1970-01-01到2038-01-19；
  - 精确到秒；
  - 采用整形存储；
  - 依赖数据库设置的时区；
  - 自动更新timestamp列的值；
- date
  - 占用的字节数比使用字符串、datetime、int存储要少，使用date类型只需要3个字节；
  - 使用date类型还可以利用日期时间函数进行日期之间的计算；
  - date类型用于保存1000-01-01到9999-12-31之间的日期；

**4.5 使用枚举代替字符串类型**
- 有时可以使用枚举类代替常用的字符串类型，mysql存储枚举类型会非常紧凑，会根据列表值的数据压缩到一个或两个字节中，mysql在内部会将每个值在列表中的位置保存为整数，并且在表的.frm文件中保存“数字-字符串”映射关系的查找表；

  ```sql
  create table enum_test(e enum('fish','apple','dog') not null);
  insert into enum_test(e) values('fish'),('dog'),('apple');
  select e+0 from enum_test;
  ```

**4.6 特殊类型数据**
- 人们经常使用varchar(15)来存储ip地址，然而，它的本质是32位无符号整数不是字符串，可以使用INET_ATON()和INET_NTOA函数在这两种表示方法之间转换；
- 案例：

  ```sql
  select inet_aton('1.1.1.1')
  select inet_ntoa(16843009)
  ```
#### 合理使用范式和反范式
> 三范式为了解决数据冗余问题

**三范式**
- 1NF：表的列的具有原子性,不可再分解；
- 2NF：表中的记录是唯一的, 就满足2NF, 通常我们设计一个主键来实现；
- 3NF：表中不要有冗余数据, 就是说，表的信息，如果能够被推导出来，就不应该单独的设计一个字段来存放；

##### 1. 范式
**优点**
- 范式化的更新通常比反范式要快；
- 当数据较好的范式化后，很少或者没有重复的数据；
- 范式化的数据比较小，可以放在内存中，操作比较快；

**缺点**
- 通常需要进行关联

##### 2. 反范式
**优点**
- 所有的数据都在同一张表中，可以避免关联；
- 可以设计有效的索引；

**缺点**
- 表格内的冗余较多，删除数据时候会造成表有些有用的信息丢失；

##### 3. 注意
**在企业中很好能做到严格意义上的范式或者反范式，一般需要混合使用**
- 在一个网站实例中，这个网站，允许用户发送消息，并且一些用户是付费用户。现在想查看付费用户最近的10条信息。  在user表和message表中都存储用户类型(account_type)而不用完全的反范式化。这避免了完全反范式化的插入和删除问题，因为即使没有消息的时候也绝不会丢失用户的信息。这样也不会把user_message表搞得太大，有利于高效地获取数据；
- 另一个从父表冗余一些数据到子表的理由是排序的需要；
- 缓存衍生值也是有用的。如果需要显示每个用户发了多少消息（类似论坛的），可以每次执行一个昂贵的自查询来计算并显示它；也可以在user表中建一个num_messages列，每当用户发新消息时更新这个值；

**案例**
- 范式设计

  ![MySQL调优（一）：范式设计](./pics/MySQL调优（一）：范式设计.png)

- 反范式设计

  ![MySQL调优（二）：反范式设计](./pics/MySQL调优（一）：反范式设计.png)

#### 主键的选择
##### 1. 代理主键
- 与业务无关的，无意义的数字序列（id）；

##### 2. 自然主键
- 事物属性中的自然唯一标识，与业务属性相关（人的身份证号）；

##### 3. 推荐使用代理主键
- 它们不与业务耦合，因此更容易维护；
- 一个大多数表，最好是全部表，通用的键策略能够减少需要编写的源码数量，减少系统的总体拥有成本；

#### 字符集的选择
> utf8 字符存储中文时，有的中文是2个字节，有的是3个字节存储，例如：“中”就是三个字节存储；所以一般用utf8mb4；

1. 纯拉丁字符能表示的内容，没必要选择 latin1 之外的其他字符编码，因为这会节省大量的存储空间。
2. 如果我们可以确定不需要存放多种语言，就没必要非得使用UTF8或者其他UNICODE字符类型，这回造成大量的存储空间浪费。
3. MySQL的数据类型可以精确到字段，所以当我们需要大型数据库中存放多字节数据的时候，可以通过对不同表不同字段使用不同的数据类型来较大程度减小数据存储量，进而降低 IO 操作次数并提高缓存命中率。

#### 存储引擎的选择
- 存储引擎的对比

  ![MySQL调优（一）：存储引擎的对比.png](./pics/MySQL调优（一）：存储引擎的对比.png)

#### 适当的数据冗余
1. 被频繁引用且只能通过 Join 2张(或者更多)大表的方式才能得到的独立小字段。
2. 这样的场景由于每次Join仅仅只是为了取得某个小字段的值，Join到的记录又大，会造成大量不必要的 IO，完全可以通过空间换取时间的方式来优化。不过，冗余的同时需要确保数据的一致性不会遭到破坏，确保更新的同时冗余字段也被更新。

#### 适当的拆分
- 当我们的表中存在类似于 TEXT 或者是很大的 VARCHAR类型的大字段的时候，如果我们大部分访问这张表的时候都不需要这个字段，我们就该义无反顾的将其拆分到另外的独立表中，以减少常用数据所占用的存储空间。这样做的一个明显好处就是每个数据块中可以存储的数据条数可以大大增加，既减少物理 IO 次数，也能大大提高内存中的缓存命中率。

### 执行计划
> MySQL调优（三）：MySQL执行计划.md

### 通过索引进行优化
#### 1. 索引基本知识

##### 1.1 索引的优点
- 大大减少了服务器需要扫描的数据量；
- 帮助服务器避免排序和临时表；
- 将随机io变成顺序io；

##### 1.2 索引的用处
- 快速查找匹配WHERE子句的行；
- 从consideration中消除行,如果可以在多个索引之间进行选择，mysql通常会使用找到最少行的索引；
- 如果表具有多列索引，则优化器可以使用索引的任何最左前缀来查找行；
- 当有表连接的时候，从其他表检索行数据；
- 查找特定索引列的min或max值；
- 如果排序或分组时在可用索引的最左前缀上完成的，则对表进行排序和分组；
- 在某些情况下，可以优化查询以检索值而无需查询数据行；

##### 1.3 索引的分类
- 主键索引：值不能为空
- 唯一索引：值可以为空，效率与主键索引一样；
- 普通索引
- 全文索引：一般是char、varchar上用
- 组合索引

##### 1.4 面试技术名词
- 回表
  - 普通列的索引（B+） -> 主键（B+） -> 取出整行记录
- 覆盖索引
  - 能用覆盖索引，尽量使用覆盖索引；
- 最左匹配：必须要先有最左边，才能查右边
- 索引下推
  - 优点：在非主键索引上的优化，可以有效减少回表的次数，大大提升了查询的效率。
  - 关闭索引下推可以使用如下命令：

  ```bash
  set optimizer_switch='index_condition_pushdown=off';
  ```


索引页分裂？
索引页合并？
##### 1.5 索引采用的数据结构
- 哈希表（Memory）
- B+树（MyISAM、InnoDB）

##### 1.6 索引匹配方式

  ```sql
  create table staffs(

      id int primary key auto_increment,

      name varchar(24) not null default '' comment '姓名',

      age int not null default 0 comment '年龄',

      pos varchar(20) not null default '' comment '职位',

      add_time timestamp not null default current_timestamp comment '入职时间'

    ) charset utf8 comment '员工记录表';

  -----------
  alter table staffs add index idx_nap(name, age, pos);
  ```

- 全值匹配：全值匹配指的是和索引中的所有列进行匹配

  ```sql
  explain select * from staffs where name = 'July' and age = '23' and pos = 'dev';
  ```

- 匹配最左前缀：只匹配前面的几列

  ```sql
  explain select * from staffs where name = 'July' and age = '23';

  explain select * from staffs where name = 'July';
  ```

- 匹配列前缀：可以匹配某一列的值的开头部分；

  ```sql
  explain select * from staffs where name like 'J%';

  explain select * from staffs where name like '%y';
  ```

- 匹配范围值：可以查找某一个范围的数据；

  ```sql
  explain select * from staffs where name > 'Mary';
  ```

- 精确匹配某一列并范围匹配另外一列：可以查询第一列的全部和第二列的部分（必须按照索引的顺序）；

  ```sql
  explain select * from staffs where name = 'July' and age > 25;
  ```

- 只访问索引的查询：查询的时候只需要访问索引，不需要访问数据行，本质上就是覆盖索引（extra的为 using index，则用到了索引）；

  ```sql
  explain select name,age,pos from staffs where name = 'July' and age = 25 and pos = 'dev';
  ```

#### 2. 哈希索引
- 基于哈希表的实现，只有精确匹配索引所有列的查询才有效；
- 在mysql中，只有memory的存储引擎显式支持哈希索引；
- 哈希索引自身只需存储对应的hash值，所以索引的结构十分紧凑，这让哈希索引查找的速度非常快；
- 哈希索引的限制
  1. 哈希索引只包含哈希值和行指针，而不存储字段值，索引不能使用索引中的值来避免读取行；
  2. 哈希索引数据并不是按照索引值顺序存储的，所以无法进行排序；
  3. 哈希索引不支持部分列匹配查找，哈希索引是使用索引列的全部内容来计算哈希值；
  4. 哈希索引支持等值比较查询，也不支持任何范围查询；
  5. 访问哈希索引的数据非常快，除非有很多哈希冲突，当出现哈希冲突的时候，存储引擎必须遍历链表中的所有行指针，逐行进行比较，直到找到所有符合条件的行；
  6. 哈希冲突比较多的话，维护的代价也会很高；(如何避免哈希冲突？)
- 案例
  > 当需要存储大量的URL，并且根据URL进行搜索查找，如果使用B+树，存储的内容就会很大
  select id from url where url=""
  也可以利用将url使用CRC32做哈希，可以使用以下查询方式：
  select id fom url where url="" and url_crc=CRC32("")
  此查询性能较高原因是使用体积很小的索引来完成查找

#### 3. 组合索引
- 当包含多个列作为索引，需要注意的是正确的顺序依赖于该索引的查询，同时需要考虑如何更好的满足排序和分组的需要；
- 案例，建立组合索引a,b,c
  - 不同SQL语句使用索引情况

    ![MySQL调优（一）：不同SQL语句使用索引情况](./pics/MySQL调优（一）：不同SQL语句使用索引情况.png)

    - like中%作为前缀，索引失效；
    - 范围查找后面的匹配条件所使用的索引，索引失效；

#### 4. 聚簇索引与非聚簇索引
##### 4.1 聚簇索引
不是单独的索引类型，而是一种数据存储方式，指的是数据行跟相邻的键值紧凑的存储在一起；

- 优点：
   1. 可以把相关数据保存在一起；
   2. 数据访问更快，因为索引和数据保存在同一个树中；
   3. 使用覆盖索引扫描的查询可以直接使用页节点中的主键值；
- 缺点：
   1. 聚簇数据最大限度地提高了IO密集型应用的性能，如果数据全部在内存，那么聚簇索引就没有什么优势；
   2. 插入速度严重依赖于插入顺序，按照主键的顺序插入是最快的方式；
   3. 更新聚簇索引列的代价很高，因为会强制将每个被更新的行移动到新的位置；
   4. 基于聚簇索引的表在插入新行，或者主键被更新导致需要移动行的时候，可能面临页分裂的问题；
   5. 聚簇索引可能导致全表扫描变慢，尤其是行比较稀疏，或者由于页分裂导致数据存储不连续的时候；

##### 4.2 非聚簇索引
数据文件跟索引文件分开存放

#### 5. 覆盖索引
> 是一种现象，而不是分类；
##### 5.1 基本介绍
1. 如果一个索引包含所有需要查询的字段的值，我们称之为覆盖索引；
2. 不是所有类型的索引都可以称为覆盖索引，覆盖索引必须要存储索引列的值；
3. 不同的存储实现覆盖索引的方式不同，不是所有的引擎都支持覆盖索引，memory不支持覆盖索引；

##### 5.2 优势
1. 索引条目通常远小于数据行大小，如果只需要读取索引，那么mysql就会极大的减少数据访问量；
2. 因为索引是按照列值顺序存储的，所以对于IO密集型的范围查询会比随机从磁盘读取每一行数据的IO要少的多；
3. 一些存储引擎如MYISAM在内存中只缓存索引，数据则依赖于操作系统来缓存，因此要访问数据需要一次系统调用，这可能会导致严重的性能问题；
4. 由于INNODB的聚簇索引，覆盖索引对INNODB表特别有用；

##### 5.3 案例演示

1. 当发起一个被索引覆盖的查询时，在explain的extra列可以看到using index的信息，此时就使用了覆盖索引；

    ```sql
    mysql> explain select store_id,film_id from inventory\G
    *************************** 1. row ***************************
               id: 1
      select_type: SIMPLE
            table: inventory
       partitions: NULL
             type: index
    possible_keys: NULL
              key: idx_store_id_film_id
          key_len: 3
              ref: NULL
             rows: 4581
         filtered: 100.00
            Extra: Using index
    1 row in set, 1 warning (0.01 sec)
    ```

2. 在大多数存储引擎中，覆盖索引只能覆盖那些只访问索引中部分列的查询。不过，可以进一步的进行优化，可以使用innodb的二级索引来覆盖查询。

    > 例如：actor使用innodb存储引擎，并在last_name字段又二级索引，虽然该索引的列不包括主键actor_id，但也能够用于对actor_id做覆盖查询。

    ```sql
    mysql> explain select actor_id,last_name from actor where last_name='HOPPER'\G
    *************************** 1. row ***************************
               id: 1
      select_type: SIMPLE
            table: actor
       partitions: NULL
             type: ref
    possible_keys: idx_actor_last_name
              key: idx_actor_last_name
          key_len: 137
              ref: const
             rows: 2
         filtered: 100.00
            Extra: Using index
    1 row in set, 1 warning (0.00 sec)
    ```

#### 6. 优化小细节

1. 当使用索引列进行查询的时候尽量不要使用表达式，把计算放到业务层而不是数据库层；

    ```sql
    select actor_id from actor where actor_id=4;

    select actor_id from actor where actor_id+1=5;
    ```

2. 尽量使用主键查询，而不是其他索引，因此主键查询不会触发回表查询；
3. 使用前缀索引；
    - 实例说明：
      - 有时候需要索引很长的字符串，这会让索引变的大且慢，通常情况下可以使用某个列开始的部分字符串，这样大大的节约索引空间，从而提高索引效率，但这会降低索引的选择性，索引的选择性是指不重复的索引值和数据表记录总数的比值，范围从1/#T到1之间。索引的选择性越高则查询效率越高，因为选择性更高的索引可以让mysql在查找的时候过滤掉更多的行。
      - 一般情况下某个列前缀的选择性也是足够高的，足以满足查询的性能，但是对应BLOB,TEXT,VARCHAR类型的列，必须要使用前缀索引，因为mysql不允许索引这些列的完整长度，使用该方法的诀窍在于要选择足够长的前缀以保证较高的选择性，通过又不能太长。

      ```sql
      --创建数据表
      create table citydemo(city varchar(50) not null);
      insert into citydemo(city) select city from city;

      --重复执行5次下面的sql语句
      insert into citydemo(city) select city from citydemo;

      --更新城市表的名称
      update citydemo set city=(select city from city order by rand() limit 1);

      --查找最常见的城市列表，发现每个值都出现45-65次，
      select count(*) as cnt,city from citydemo group by city order by cnt desc limit 10;

      --查找最频繁出现的城市前缀，先从3个前缀字母开始，发现比原来出现的次数更多，可以分别截取多个字符查看城市出现的次数
      select count(*) as cnt,left(city,3) as pref from citydemo group by pref order by cnt desc limit 10;
      select count(*) as cnt,left(city,7) as pref from citydemo group by pref order by cnt desc limit 10;
      --此时前缀的选择性接近于完整列的选择性

      --还可以通过另外一种方式来计算完整列的选择性，可以看到当前缀长度到达7之后，再增加前缀长度，选择性提升的幅度已经很小了
      select count(distinct left(city,3))/count(*) as sel3,
      count(distinct left(city,4))/count(*) as sel4,
      count(distinct left(city,5))/count(*) as sel5,
      count(distinct left(city,6))/count(*) as sel6,
      count(distinct left(city,7))/count(*) as sel7,
      count(distinct left(city,8))/count(*) as sel8
      from citydemo;

      --计算完成之后可以创建前缀索引
      alter table citydemo add key(city(7));

      --注意：前缀索引是一种能使索引更小更快的有效方法，但是也包含缺点：mysql无法使用前缀索引做order by 和 group by。
      ```

      基数？Hyperloglog？

4. 使用索引扫描来排序；
    - mysql有两种方式可以生成有序的结果；通过排序操作或者按索引顺序扫描，如果explain出来的type列的值为index，则说明mysql使用了索引扫描来做排序；
    - 扫描索引本身是很快的，因为只要从一条索引记录移动到紧接着的下一条记录。但如果索引不能覆盖查询所需的全部列，那么就不得不每扫描一条索引记录就得回表查询一次对应的行，这基本都是随机IO，因此按索引顺序读取数据的速度通常要比顺序地全表扫描慢。
    - mysql可以使用同一个索引即满足排序，又用于查找行，如果可能的话，设计索引时应该尽可能地同时满足这两种任务。
    - 只有当索引的列顺序和order by字句的顺序完全一致，并且所有的排序方式都一样时，mysql才能够使用索引来对结果进行排序，如果查询需要关联多张表，则只有当order by子句引用的字段全部为第一张表时，才能使用索引做排序。order by子句和查找型查询的限制是一样的，需要满足索引的最左前缀的要求，否则，mysql都需要执行顺序操作，而无法利用索引排序；

    ```sql
    --sakila 数据库中rental表在rental_date,inventory_id,customer_id上有rental_date的索引
    --使用rental_date索引为下面的查询做排序
    explain select rental_id,staff_id from rental where rental_date='2005-05-25' order by inventory_id,customer_id\G
    ......
    --order by 子句不满足索引的最左前缀要求，也可以用于查询排序，这是因为索引的第一列被指定为一个常数


    --该查询为索引的第一列提供了常量条件，而使用第二列进行排序，将两个列组合在一起，就形成了索引的最左前缀
    explain select rental_id,staff_id, from rental where rental_date='2005-05-25' order by inventory_id desc\G
    ```
5. union all,in,or都能够使用索引，但是推荐使用in；

    ```sql
    explain select * from actor where actor_id = 1 union all select * from actor where actor_id = 2;

    explain select * from actor where actor_id in (1,2);

    explain select * from actor where actor_id = 1 or actor_id =2;
    ```
6. 范围列可以用到索引；
    - 范围条件是：<、>
    - 范围列可以用到索引，但是范围列后面的列无法用到索引，索引最多用于一个范围列
7. 强制类型转换会全表扫描；

    ```sql
    create table user(id int,name varchar(10),phone varchar(11));

    alter table user add index idx_1(phone);
    ```

    ```sql
    -- 不会触发索引
    explain select * from user where phone=13800001234;

    -- 触发索引
    explain select * from user where phone='13800001234';
    ```

8. 更新十分频繁，数据区分度不高的字段上不宜建立索引；
    - 更新会变更B+树，更新频繁的字段建议索引会大大降低数据库性能；
    - 类似于性别这类区分不大的属性，建立索引是没有意义的，不能有效的过滤数据；
    - 一般区分度在80%以上的时候就可以建立索引，区分度可以使用 count(distinct(列名))/count(*) 来计算；
9. 创建索引的列，不允许为null，可能会得到不符合预期的结果；
10. 当需要进行表连接的时候，最好不要超过三张表，因为需要join的字段，数据类型必须一致；
11. 能使用limit的时候尽量使用limit；
12. 单表索引建议控制在5个以内；
13. 单索引字段数不允许超过5个（组合索引）；
14. 创建索引的时候应该避免以下错误概念；
    - 索引越多越好；
    - 过早优化，在不了解系统的情况下进行优化；

#### 7. 索引监控


#### 8. 简单案例



### 查询优化
