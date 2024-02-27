# Spring（五）：ApplicationContext

> 前面通过手写IoC、DI、AOP和Bean的配置，到最后ApplicationContext的门面处理，对于Spring相关的核心概念应该会比较清楚了，接下来我们就看看在Spring源码中，对于核心组件是如何实现的。

## ApplicationContext

> ApplicationContext到底是什么？字面含义是应用的上下文。这块我们需要看看ApplicationContext的具体的结构。

![Spring（五）：ApplicationContext_1.png](./pics/Spring（五）：ApplicationContext_1.png)

通过ApplicationContext实现的相关接口来分析，ApplicationContext接口在具备BeanFactory的功能的基础上还扩展了 `应用事件发布`、`资源加载`、`环境参数` 和 `国际化` 的能力，然后我们来看看ApplicationContext接口的实现类的情况。

![Spring（五）：ApplicationContext_2.png](./pics/Spring（五）：ApplicationContext_2.png)

在ApplicationContext的实现类中有两个比较重要的分支，`AbstractRefreshableApplicationContext`和 `GenericApplicationContext`。

![Spring（五）：ApplicationContext_3.png](./pics/Spring（五）：ApplicationContext_3.png)


## BeanFactory