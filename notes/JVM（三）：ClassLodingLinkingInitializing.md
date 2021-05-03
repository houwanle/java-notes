## Class Loding Linking Initializing

![JVM（三）：Class加载过程](./pics/JVM（三）：Class加载过程.png)

- 加载过程
  - Loading：将Class文件加载到内存中
    - 双亲委派，主要出于安全来考虑
      - 父加载器不是“类加载器的加载器”，也不是“类加载器的父类加载器”
      - 双亲委派是一个孩子向父亲方向，然后父亲向孩子方向的双亲委派过程。
  - Linking
    - Verification（校验）：是否符合标准
    - Preparation：将Class文件赋默认值
    - Resolution（解析）：
  - Initializing：将静态变量赋初始值

- 类加载器

  ![JVM（三）：类加载器](./pics/JVM（三）：类加载器.png)

- 类加载器中的类加载过程

  ![JVM（三）：类加载器中的类加载过程](./pics/JVM（三）：类加载器中的类加载过程.png)


- 类加载器范围
  - （来自Launcher源码）
    - sun.boot.class.path
      - Bootstrap ClassLoader加载路径
    - java.ext.dirs
      - ExtensionClassLoader加载路径
    - java.class.path
      - AppClassLoader加载路径

  - 测试代码
  ```java
  package com.lele.jvm.classloader;

  /**
   * @author: lele
   * @date: 2021/5/3 20:30
   * @description:
   */
  public class T003_ClassLoaderScope {
      public static void main(String[] args) {
          String pathBoot = System.getProperty("sun.boot.class.path");
          System.out.println(pathBoot.replaceAll(";",System.lineSeparator()));

          System.out.println("------------------------");
          String pathExt = System.getProperty("java.ext.dirs");
          System.out.println(pathExt.replaceAll(";",System.lineSeparator()));

          System.out.println("------------------------");
          String pathApp = System.getProperty("java.class.path");
          System.out.println(pathApp.replaceAll(";",System.lineSeparator()));

      }
  }
  ```

- 自定义类加载器
  - 继承ClassLoader
  - 重写模板方法findClass
    - 调用defineClass
  - 自定义加载器加载自加密的class
    - 防止反编译
    - 防止篡改

- Classloader源码解析

  ![JVM（三）：Classloader源码解析](./pics/JVM（三）：Classloader源码解析.png)

  - findClass
  - 如果是AppClassLoader首先会执行URLClassLoader的findClass方法
  - ClassLoader源码中用到了 模板方法 设计模式
