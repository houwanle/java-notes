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

#### 主键的选择

#### 字符集的选择

#### 存储引擎的选择

#### 适当的数据冗余

#### 适当的拆分

### 执行计划
