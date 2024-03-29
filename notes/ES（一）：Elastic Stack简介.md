## ES（一）：Elastic Stack 简介

### 1. Elastic Stack
- Elasticsearch：基于Json的分布式搜索混合分析引擎；
- Logstash：动态数据收集管道，生态丰富；
- Kibana：提供数据的可视化界面；
- Beats：轻量级的数据采集器；

#### 1.1 Elasticsearch（Elastic Stack 的核心）
- 搜索、聚合分析、大数据存储
- 分布式、高性能、高可用、可伸缩、易维护；
- 支持文本搜索、结构化数据、非结构化数据、地理位置搜索等；

#### 1.2 Logstash（管道）
- 采集
  > 数据往往以各种各样的形式，或分散或集中地存在于很多系统中。Logstash 支持各种输入选择，可以同时从众多常用来源捕捉事件。能够以连续的流式传输方式，轻松地从您的日志、指标、Web应用、数据存储以及各种 AWS 服务采集数据。
- 过滤
  > Logstash 能够动态地转换和解析数据，不受格式或复杂度的影响：
    - 利用 Grok 从非结构化数据中派生出结构
    - 从 IP 地址破译出地理坐标；
    - 将 PII 数据匿名化，完全排除敏感字段
    - 简化整体处理，不受数据源、格式或架构的影响；
- 输出
  > Elasticsearch 是官方首选输出的方式，但并非唯一选择。Logstash 提供多种输出选择，目前官方支持 200 多个插件；

#### 1.3 Kibana（Elastic Stack 的窗户）
- 可视化
  - 图表：柱状图、线状图、饼图、旭日图等；
  - 位置分析：位置搜索、形状搜索、地图检测等；
  - 时序分析：可以方便的从各种不同时间维度查看；
- 管理和监控
  - 机器学习：非监督型异常和隐患检测；
  - 安全监控：堆栈检测、异常报警、策略可配置柱状图、线状图、饼图、旭日图等；

#### 1.4 Beats 轻量级数据采集器
- 开源：Beats 是一个免费且开放的平台，集合了多种单一用途数据采集器。它们从成百上千或成千上万台机器和系统向 Logstash 或 Elasticsearch 发送数据；
- 轻量级：Beats 使用 GO 语言开发，对服务器资源占用极低。Beats 可以采集符合 Elastic Common Schema（ECS）要求的数据，可以将数据转发至 Logstash进行转换和解析；
- 即插即用：Filebeat 和 Metricbeat 中包含的一些模块能够简化从关键数据源（例如云平台、容器和系统，以及网络技术）采集、解析和可视化信息的过程。只需要运行一行命令，即可开始探索；
- 可扩展：由于 Beats 开源的特性，如果现有 Beats 不能满足开发需要，我们可以自行构建，并且完善 Beats 社区；
