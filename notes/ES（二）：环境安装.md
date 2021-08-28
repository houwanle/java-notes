## ES（二）：环境安装

### 1. 安装 Java 环境

#### 1.1 安装 JDK
- 版本选择：最好是 Java8、Java11 或者 Java14
- JDK 兼容性：https://www.elastic.co/cn/supportmatrix#matrix_jvm
- 操作系统的兼容性：https://www.elastic.co/cn/support/matrix
- 自身兼容性：https://www.elastic.co/cn/support/matrix#matrix_compatibility

### 2. 安装 Elasticsearch

- 下载地址：http://elastic.co
- Elasticsearch 目录结构

  目录名称 | 备注
  ---|---
  bin | 可执行脚本文件，包括启动 Elasticsearch 服务、插件管理、函数命令等。
  config | 配置文件目录， 如 Elasticsearch 配置、角色配置、jvm 配置等。
  lib | Elasticsearch 所依赖的 Java 库。
  data | 默认数据存放目录，包含节点、分片、索引、文档的所有数据，生产环境要求必须修改。
  logs | 默认的日志文件存储路径，生产环境务必修改。
  modules | 包含所有的 Elasticsearch 模块，如 Cluster、Discovery、Indices等。
  plugins | 已经安装的插件的目录
  jdk/jdk.app | 7.0 以后才有，自带的 Java环境。

- 启动单节点服务

  . | Windows | Linux | MacOS
  ---|---|---|---
  命令行 | cd elasticsearch\bin  .\elasticsearch -d | cd elasticsearch/bin  ./elasticsearch -d | cd elasticsearch/bin ./elasticsearch -d
  图形界面 | 在 bin 目录下双击 elasticsearch.bat |——| 在 bin 目录下双击 elasticsearch
  Shell | start elasticsearch\bin\elasticsearch.bat |——| open elasticsearch/bin/elasticsearch

- 验证服务启动成功：http://localhost:9200
- 在本机单个项目启动多节点

  操作系统 | 命令
  ---|---
  Linux/MacOS | ```./elasticsearch -E path.data=data1 -E path.logs=log1 -E node.name=node1 -E cluster.name=test1``` <br> ```./elasticsearch -E path.data=data2 -E path.logs=log2 -E node.name=node2 -E cluster.name=test2```
  Windows | ```.\elasticsearch.bat -E path.data=data1 -E path.logs=log1 -E node.name=node1 -E cluster.name=test1``` <br> ```.\elasticsearch.bat -E path.data=data2 -E path.logs=log2 -E node.name=node2 -E cluster.name=test2```

- 在本机多个项目启动多个单节点

  操作系统 | 脚本
  ---|---
  MacOS | ```open /node1/bin/elasticsearch``` <br> ```open /node2/bin/elasticsearch``` <br> ```open /node3/bin/elasticsearch```
  Windows | ```start D:\node1\bin\elasticsearch.bat``` <br>  ```start D:\node2\bin\elasticsearch.bat``` <br> ```start D:\node3\bin\elasticsearch.bat```

### 3. 安装 Kibana

- 下载地址：https://www.elastic.co/cn/downloads/kibana
- 启动服务：（从版本6.0.0开始，Kibana仅支持64位操作系统。）

  —— | Windows | Linux | MacOS
  ---|---|---|---
  命令行 | ```cd kibana\bin``` <br> ```.\kibana.bat``` | ```cd kibana\bin``` <br> ```./kibana``` | ```cd kibana\bin``` <br> ```./kibana```
  图形界面 | 在 bin 目录下双击 kibana.bat | —— | 在bin目录下双击kibana
  Shell | ```start kibana\bin\kibana.bat``` | —— | ```open kibana/bin/kibana```

- 验证服务启动成功：http://localhost:5601
- 配置 elasticsearch 服务的地址

  ![ES（二）：配置elasticsearch服务的地址](./pics/ES（二）：配置elasticsearch服务的地址.jpg)

- 命令行关闭 kibana
  - 关闭窗口
  - ```ps -ef | grep 5601``` 或者 ```ps -ef | grep kibana``` 或者 ```lsof -i :5601```
  - ```kill -9 pid```

**关于 “Kibana Server is not ready yet” 问题的原因及解决办法**
- Kibana 和 Elasticsearch 的版本不兼容
  > 解决方法：保持版本一致
- Elasticsearch 的服务地址和 Kibana 中配置的 elasticsearch.hosts 不同
  > 解决方法：修改 kibana.yml 中的 elasticsearch.hosts 配置
- Elasticsearch 中禁止跨域访问
  > 解决方法：在elasticsearch.yml中配置允许跨域
- 服务器中开启了防火墙
  > 解决方法：关闭防火墙或者服务器的安全策略
- Elasticsearch所在磁盘剩余空间不足90%
  > 解决方法：清理磁盘空间，配置监控和报警

### 4. 安装 Elasticsearch-Head 插件

#### 4.1 安装依赖

**（1）下载node**
- 下载地址：https://nodejs.org/en/download
- 检查是否安装成功：Win+R CMD输入```node -v```命令检查，如果输出了版本号，则node安装成功；

**（2）安装grunt**
- CMD中执行```npm install -g grunt-cli```命令等待安装完成；
- 输入：```grunt -version``` 命令检查是否安装成功；

#### 4.2 下载Head插件
- 下载地址：https://github.com/mobz/elasticsearch-head
- 解压，打开 elasticsearch-head-master 文件夹，修改 Gruntfile.js 文件，添加 hostname:'*'；

  ```json
  connect: {
    server: {
      options: {
        hostname: '*',
        port: '9100',
        base: '.',
        keepalive: true
      }
    }
  }
  ```

- 输入 ```cd elasticsearch-head-master``` ```npm install```
- ```npm run start``` 启动服务
- 验证：http://localhost:9100/
- 若无法发现ES节点，尝试在ES配置文件中设置允许跨域

  ```json
  http.cors.enabled: true
  http.cors.allow-origin: "*"
  ```

  从Chrome网上应用商店安装 Elasticsearch Head

### 5. 集群的健康检查
- 健康状态
  - Green：所有 Primary 和 Replica 均为 active，集群健康；
  - Yellow：至少一个 Replica 不可用，但是所有 Primary 均为 active，数据仍然是可以保证完整性的；
  - Red：至少一个 Primary 为不可用状态，数据不完整，集群不可用；
- 健康值检查
  - _cat/health
  - _cluster/health
