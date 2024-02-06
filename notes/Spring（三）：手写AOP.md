# Spring（三）：手写AOP

手写 IoC 和 DI 后已经实现的类图结构

![Spring（三）：手写AOP_1.png](./pics/Spring（三）：手写AOP_1.png)

## AOP分析
### AOP是什么
AOP[Aspect Oriented Programming] 面向切面编程，在不改变类的代码的情况下，对类方法进行功能的增强。

### 我们要做什么
我们需要在前面手写IoC、手写DI的基础上给用户提供AOP功能，让他们可以通过AOP技术实现对类方法功能增强。

![Spring（三）：手写AOP_2.png](./pics/Spring（三）：手写AOP_2.png)

### 我们的需求是什么
提供AOP功能，然后呢...没有了。关键还是得从上面的定义来理解
