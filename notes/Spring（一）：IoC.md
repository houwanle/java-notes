# Spring（一）：IoC

## IoC分析
### Spring的核心
在Spring中非常核心的内容是IOC和AOP。

### IoC是什么？
IoC：Inversion of Control 控制反转，简单理解就是：依赖对象的获得被反转了。

![Spring（一）：IoC_1](./pics/Spring（一）：IoC_1.png)

### IoC有什么好处

**IoC带来的好处**
- 代码更加简洁，不需要去 new 要使用的对象了；
- 面向接口编程，使用者与具体类解耦，易扩展、替换实现者；
- 可以方便进行AOP编程；

![Spring（一）：IoC_2.png](./pics/Spring（一）：IoC_2.png)

### IoC容器做了什么工作
IoC容器的工作：负责创建、管理类实例，向使用者提供实例。

![Spring（一）：IoC_3.png](./pics/Spring（一）：IoC_3.png)

### IoC容器是否是工厂模式的实例
IoC容器是工厂模式的实例，IoC容器负责来创建类实例对象，需要从IoC容器中get获取。IoC容器也称为Bean工厂。

![Spring（一）：IoC_4.png](./pics/Spring（一）：IoC_4.png)

那么我们一直说的Bean是什么呢？bean：组件，也就是类对象。


## IoC实现
IoC的核心就是Bean工厂，那么Bean工厂应该如何设计实现它呢？

### Bean工厂的作用
首先Bean工厂的作用，上面分析了就是创建、管理Bean，并且需要对外提供Bean的实例。

![Spring（一）：IoC_5.png](./pics/Spring（一）：IoC_5.png)

### Bean工厂的初步设计
基于Bean工厂的基本作用，我们可以分析Bean工厂应该具备的相关行为。

首先Bean工厂应该要对外提供获取bean实例的方法，所以需要定义一个 getBean() 方法，同时工厂需要知道生产的 bean 的类型，所以 getBean() 方法需要接受对应的参数，同时返回类型这块也可能有多个类型，我们就用 Object 来表示，这样 Bean 工厂的定义就出来了。

![Spring（一）：IoC_6.png](./pics/Spring（一）：IoC_6.png)

上面定义了 Bean 工厂对外提供 bean 实例的方法，但是 Bean 工厂如何知道要创建上面对象，怎么创建该对象呢？

