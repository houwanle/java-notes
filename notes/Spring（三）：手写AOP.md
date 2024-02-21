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

![Spring（三）：手写AOP_3.png](./pics/Spring（三）：手写AOP_3.png)

![Spring（三）：手写AOP_4.png](./pics/Spring（三）：手写AOP_4.png)

## AOP概念讲解
上面在分析AOP需求的时候，我们介绍了相关的概念，Advice、Pointcuts和weaving等，首先我们来看看在AOP中，我们会接触到的相关概念都有哪些。

![Spring（三）：手写AOP_5.png](./pics/Spring（三）：手写AOP_5.png)

更加形象的描述

![Spring（三）：手写AOP_6.png](./pics/Spring（三）：手写AOP_6.png)

然后对于上面的相关概念，我们就要考虑哪些是用户需要提供的，哪些是框架要写好的？

![Spring（三）：手写AOP_7.png](./pics/Spring（三）：手写AOP_7.png)

思考：Advice、Pointcuts和Weaving各自的特点

![Spring（三）：手写AOP_8.png](./pics/Spring（三）：手写AOP_8.png)

## 切面实现

通过上面的分析，我们要设计实现AOP功能，其实就是要设计实现上面分析的相关概念对应的组件