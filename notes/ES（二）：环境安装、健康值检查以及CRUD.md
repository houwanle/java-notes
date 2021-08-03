## ES（二）：环境安装、健康值检查以及CRUD

### 1. 安装环境

#### 1.1 安装ES
- 安装六字箴言
  - JDK -> 依赖
  - 下载 -> elastic.co
  - 启动 -> ./rlasticsearch -d
  - 验证 -> http://localhost:9200/
- 开发模式和生产模式
  - 开发模式：默认配置（未配置发现设置），用于学习阶段；
  - 生产模式：会触发ES的引导检查，学习阶段不建议修改集群相关的配置；

#### 1.2 安装Kibana
> 从版本6.0.0开始，kibana仅支持64位操作系统。

- 下载：http://elastic.co
- 启动：开箱即用
  - Linux：./kibana
  - Windows:.\kibana.bat
- 验证：localhost:5601

#### 1.3 安装head插件（选装）
- 介绍
  - 提供可视化的操作页面，对 Elasticsearch 搜索引擎进行各种设置和数据检索功能，可以很直观的查看集群的健康状况，索引分配情况，还可以管理索引和集群以及提供方便快捷的搜索功能等等。
- 下载：https://github.com/mobz/elasticsearch-head
- 安装：依赖于 node 和 grunt 管理工具
- 启动：npm run start
- 验证：http://localhost:9100/

### 2. 集群健康值
- 健康值检查
  - _cat/health
  - _cluster/health

    字段|释义
    ---|---
    cluster_name|集群名字
    status|集群健康状态
    timed_out|是否超时
    number_of_nodes|节点的总数量
    number_of_data_nodes|数据节点的数量
    active_primary_shards|活跃的主分片
    active_shards|活跃的分片数
    relocating_shards|迁移中的分片的数量
    initializing_shards|初始化的分片的数量
    unassigned_shards|未分配的分片的数量
    delayed_unassigned_shards|
    number_of_pending_tasks|
    number_of_in_flight_fetch|
    task_max_waiting_in_queue_millis|
    active_shards_percent_as_number|活跃的分片比例

    ![ES（二）：ES容错机制](./pics/ES（二）：ES容错机制.png)

- 健康值状态
  - Green：所有 Primary 和 Replica 均为 active，集群健康；
  - Yellow：至少一个 Replica 不可用，但是所有 Primary 均为 active，数据仍然是可以保证完整性的；
  - Red：至少有一个 Primary 为不可用状态，数据不完整，集群不可用；

### 3. 基于XX系统的 CRUD
- 创建索引：PUT /product?pretty
- 查询索引：GET _cat/indices?v
- 删除索引：DELETE /product?pretty
- 插入数据：

  ```
  PUT /index/_doc/id
  {
      Json数据
  }
  ```

- 更新数据
  - 全量更新
  - 指定字段更新
- 删除数据

  ```
  DELETE /index/type/id/
  ```

### 4. ES 分布式文档系统

#### 4.1 ES 如何实现高可用（生产环境均为一台机器一个节点）
- ES 在分配单个索引的分片时会将每个分片尽可能分配到更多的节点上。但是，实际情况取决于集群拥有的分片和索引的数量以及它们的大小，不一定总是均匀地分布。
- ES 不允许 Primary 和它的 Replica 放在同一个节点中，并且同一个节点不接受完全相同的两个 Replica。
- 同一个节点允许多个索引的分片同时存在。

#### 4.2 容错机制
- 容错：
  - 傻X的代码你能看懂，牛X的代码你能看懂；
  - 只能看懂自己的代码，容错性低；
  - PS：各种情况（支持的情况越多，容错性越好）下，都能保证work正常运行；
  - 换到 ES 上就是在局部出错异常的情况下，保证服务正常运行并且有自行恢复能力；
- ES-node
  - Master：主节点，每个集群都有且只有一个；
    - 尽量避免Master节点 node.data ＝ true；
  - voting：投票节点
    - Node.voting_only = true（仅投票节点，即使配置了 data.master = true，也不会参选，但是仍然可以作为数据节点）
  - coordinating：协调节点
    - 每一个节点都隐式的是一个协调节点，如果同时设置了 data.master = false 和 data.data = false，那么此节点将成为仅协调节点；
  - Master-eligible node（候选节点）
  - Data node（数据节点）
  - Ingest node
  - Machine learning node（机器学习节点）
- 两个配置：node.master 和 node.data
  - node.master = true  node.data = true：这是 ES 节点默认配置，既作为候选节点又作为数据节点，这样的节点一旦被选举为 Master 节点应该只承担较为轻量级的任务，比如创建删除索引，分片均衡等。
  - node.master = true node.data = false：只作为候选节点，不作为数据节点，可参选 Master 节点，当选后成为真正的 Master 节点；
  - node.master = false  node.data = false：既不当选节点，也不作为数据节点，那就是仅协调节点，负责负载均衡；
  - node.master = false  node.data = true：不作为候选节点，但是作为数据节点，这样的节点主要负责数据存储和查询服务。
- 图解容错机制
  - 第一步：Master 选举（假如宕机节点是master）
    - 脑裂：可能产生多个 Master 节点；
    - 解决：discovery.zen.minimum_master_nodes=N/2+1
  - 第二步：Replica容错，新的（或者原有）Master节点会将丢失的Primary对应的某个副本提升为Primary；
