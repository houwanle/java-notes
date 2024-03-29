## JVM（二）：class文件格式
- 二进制字节流
- 数据类型：u1 u2 u4 u8 和 _info（表类型）
  - _info 的来源是hotspot源码中的写法
- 查看16进制格式的ClassFile
  - sublime/notepad/
  - IDEA插件-BinEd
- 有很多可以观察ByteCode的方法
  - javap：显示class文件信息（java自带）
  - JBE:可以直接修改
  - JClassLib:IDEA插件之一
- classfile构成
  ```java
  classFile {
    u4 magic;
    u2 minor_version;
    u2 major_version;
    u2 constant_pool_count;
    cp_info constant_pool[constant_pool_count - 1];
    u2
  }
  ```

- jdk1.8类文件格式

  ![JVM（二）：jdk1.8类文件格式](./pics/JVM（二）：jdk1.8类文件格式.png)

- 类文件的十六进制编码

  ![JVM（二）：类文件的十六进制编码](./pics/JVM（二）：类文件的十六进制编码.png)

  - magic：
  - minor version：小版本号
  - major version：大版本号
  - constant_pool_count：常量池中存在多少个常量（2个字节，65535）
  - access flags：修饰符（public、private）
