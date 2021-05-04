## Class Loding Linking Initializing

![JVM（三）：Class加载过程](./pics/JVM（三）：Class加载过程.png)

- 类加载过程
  - Loading：将Class文件加载到内存中
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
    - 双亲委派，主要出于安全来考虑
      - 父加载器不是“类加载器的加载器”，也不是“类加载器的父类加载器”
      - 双亲委派是一个孩子向父亲方向，然后父亲向孩子方向的双亲委派过程。
    - lazyloading（lazyInitializing）必须初始化的五种情况：
      - new getstatic putstatic invokestatic指令，访问final变量除外
      - java.lang.reflect对类进行反射调用时
      - 初始化子类的时候，父类首先初始化
      - 虚拟机启动时，被执行的主类必须初始化
      - 动态语言支持java.lang.invoke.MethodHandle解析的结果为REF_getstatic REF_putstatic REF_invokestatic的方法句柄时，该类必须初始化；
    - ClassLoader的源码
      - findInCache -> parent.loadClass -> findClass()

      ![JVM（三）：Classloader源码解析](./pics/JVM（三）：Classloader源码解析.png)

      - 如果是AppClassLoader首先会执行URLClassLoader的findClass方法
      - ClassLoader源码中用到了 模板方法 设计模式
    - 自定义类加载器
      - extends ClassLoader（继承ClassLoader）
      - overwrite findClass() -> defineClass(byte[] -> Class clazz)（重写模板方法findClass，调用defineClass）
      - 加密（自定义加载器加载自加密的class，防止反编译，防止篡改）
    - 混合执行 编译执行 解释执行
      - 解释器（bytecode intepreter）
      - JIT（Just In-Time compiler）
      - 混合模式
        - 混合使用解释器+热点代码编译
        - 起始阶段采用解释执行
        - 热点代码检测
          - 多次被调用的方法（方法计数器：检测方法执行效率）
          - 多次被调用的循环（循环计数器：检测循环执行效率）
          - 进行编译
      - -Xmixed 默认为混合模式，开始解释执行，启动速度较快对热点代码实行检测和编译
      - -Xint 使用纯解释模式，启动很快，执行较慢
      - -Xcomp 使用纯编译模式，执行很快，启动很慢
      - -XX:CompileThreshold=10000 检测热点代码
  - Linking
    - Verification（校验）：是否符合标准
    - Preparation：将Class文件赋默认值
    - Resolution（解析）：
  - Initializing：将静态变量赋初始值
