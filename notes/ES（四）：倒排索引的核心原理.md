## ES（四）：倒排索引的核心原理

![ES（四）：倒排索引的核心原理](./pics/ES（四）：倒排索引的核心原理.png)

![ES（四）：倒排索引的数据结构](./pics/ES（四）：倒排索引的数据结构.png)

- 倒排表（posting list）
- 词项字典（term dictionary）
- 词项索引（term index）

### 1. 倒排索引的核心算法

#### 1.1 倒排表的压缩算法
- FOR：Frame Of Reference

  ![ES（四）：FOR压缩算法](./pics/ES（四）：FOR压缩算法.png)

- RBM：RoaringBitmap

#### 1.2 词项索引的检索原理
- FST：Finit state Transducers
