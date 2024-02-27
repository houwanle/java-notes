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

上面分析了 `ApplicationContext`接口的结构。然后我们来看看 `BeanFactory`在ApplicationContext中具体的实现是怎么样的

![Spring（五）：ApplicationContext_4.png](./pics/Spring（五）：ApplicationContext_4.png)

![Spring（五）：ApplicationContext_5.png](./pics/Spring（五）：ApplicationContext_5.png)

可以看到具体的实现是 DefaultListableBeanFactory，然后我们来看看它的体系结构。

![Spring（五）：ApplicationContext_6.png](./pics/Spring（五）：ApplicationContext_6.png)

BeanFactory的继承体系

![Spring（五）：ApplicationContext_7.png](./pics/Spring（五）：ApplicationContext_7.png)



## BeanDefinition

然后我们来了解下ApplicationContext是如何来加载Bean定义的。具体代码我们需要分为XML配置文件和基于注解的两种方式来看。

### 基于XML方式

我们先定义对应的application.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

	<bean id="beanE" class="com.study.spring.sample.config.BeanE" />

	<bean id="beanF" class="com.study.spring.sample.config.BeanF" ></bean>

	<context:annotation-config/>

	<context:component-scan base-package="com.study.spring.sample.config" ></context:component-scan>

</beans>
```

然后我们的测试类代码

```java
public class XMLConfigMain {

	public static void main(String[] args) {
		ApplicationContext context = new GenericXmlApplicationContext(
				"classpath:com/study/spring/sample/config/application.xml");
		BeanF bf = context.getBean(BeanF.class);
		bf.do1();
	}
}
```

处理的过程：解析XML--> BeanDefinition --> BeanDefinitionRegistry --> BeanFactory

![Spring（五）：ApplicationContext_8.png](./pics/Spring（五）：ApplicationContext_8.png)


### 基于注解方式

然后来看看基于注解方式的使用情况。首先是我们的配置类。

```java
@Configuration
@ComponentScan("com.study.spring.sample.config")
public class JavaBasedMain {

	@Bean
	public BeanH getBeanH() {
		return new BeanH();
	}

	public static void main(String[] args) {
		ApplicationContext context = new AnnotationConfigApplicationContext(JavaBasedMain.class);

		BeanH bh = context.getBean(BeanH.class);
		bh.doH();
	}
}
```

然后是我们的测试类

```java
public class AnnotationMain {

	public static void main(String[] args) {
		ApplicationContext context = new AnnotationConfigApplicationContext("com.study.spring.sample.config");

		BeanG bg = context.getBean(BeanG.class);
		bg.dog();
	}
}
```

注解使用有两种方法：
1. 配置扫描路径；
2. 配置@Configuration的注解类

#### this构造方法
在this的构造方法中会完成相关的配置处理

```java
	public AnnotationConfigApplicationContext() {
		this.reader = new AnnotatedBeanDefinitionReader(this);
		this.scanner = new ClassPathBeanDefinitionScanner(this);
	}
```

首先是AnnotatedBeanDefinitionReader(this)方法。会完成核心的ConfigurationClassPostProcessor的注入。ConfigurationClassPostProcessor 会完成@Configuration相关的注解的解析

![Spring（五）：ApplicationContext_9.png](./pics/Spring（五）：ApplicationContext_9.png)

this.scanner其实就是创建了一个对应的扫描器

![Spring（五）：ApplicationContext_10.png](./pics/Spring（五）：ApplicationContext_10.png)

#### 扫描实现

扫描就需要进入到scan方法中。

![Spring（五）：ApplicationContext_11.png](./pics/Spring（五）：ApplicationContext_11.png)

![Spring（五）：ApplicationContext_12.png](./pics/Spring（五）：ApplicationContext_12.png)

![Spring（五）：ApplicationContext_13.png](./pics/Spring（五）：ApplicationContext_13.png)

完成相关的注册

![Spring（五）：ApplicationContext_14.png](./pics/Spring（五）：ApplicationContext_14.png)


#### @Configuration

@Configuration的解析其实是在refresh方法中来实现的。

![Spring（五）：ApplicationContext_15.png](./pics/Spring（五）：ApplicationContext_15.png)


### 小结

通过上面的分析其实我们已经对Bean定义的扫描，解析和注册过程有了一定的了解。归纳为：
1. reader解析XML，完成xml方法配置的bean定义
2. scanner扫描指定包下的类，找出带有@Component注解的类，注册成Bean定义
3. 通过ConfigurationClassPostProcessor对带有@Configuration注解的类进行处理，解析它上面的注解，以及类中带有@Bean 注解，加入这些的Bean的定义。

