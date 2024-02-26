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

通过上面的分析，我们要设计实现AOP功能，其实就是要设计实现上面分析的相关概念对应的组件。

![Spring（三）：手写AOP_9.png](./pics/Spring（三）：手写AOP_9.png)


### Advice

#### 面向接口编程
Advice：通知，是由用户提供的，我们来使用，主要是用户提供就突出了`多变性`。针对这块我们应该怎么设计？这里有两个问题：
1. 我们如何能够识别用户提供的东西呢？用户在我们写好框架后使用我们的框架；
2. 如何让我们的代码隔绝用户提供的多变性呢？

针对这种情况我们定义一套标准的接口，用户实现接口类提供他们不同的逻辑。是否可行？

这里有个重要的设计原则大家要注意：如何应对变化，通过面向接口编程来搞定！！！

我们先定义一个空的接口，可以先思考下我们为什么定义一个空的接口呢？

![Spring（三）：手写AOP_10.png](./pics/Spring（三）：手写AOP_10.png)

#### Advice的特点分析

Advice的特点：可选时机，可选择在方法执行前、后、异常时进行功能的增强。

![Spring（三）：手写AOP_11.png](./pics/Spring（三）：手写AOP_11.png)


结合上面的情况我们可以分析出Advice通知的几种情况
- 前置增强-Before
- 后置增强-AfterReturn
- 环绕增强-Around
- 最终通知-After
- 异常通知-Throwing

有这么多的情况我们应该要怎么来实现呢？我们可以定义标准的接口方法，让用户来实现它，提供各种具体的增强内容。那么这四种增强相关的方法定义是怎样的呢？我们一一来分析下。

#### 各种通知分析

##### 前置增强
定义：在方法执行前进行增强

**它可能需要的参数？**  
目的是对方法进行增强，应该需要的是方法相关的信息，我们使用它的时候能给予它的就是当前要执行方法的相关信息了。

**运行时方法有哪些信息**
1. 方法本身 Method；
2. 方法所属的对象 Object；
3. 方法的参数 Object[];

**前置增强的返回值是什么？**  
在方法执行前进行增强，不需要返回值！

```java
public interface MethodBeforeAdvice extends Advice {

	/**
	 * 实现该方法进行前置增强
	 * 
	 * @param method
	 *            被增强的方法
	 * @param args
	 *            方法的参数
	 * @param target
	 *            被增强的目标对象
	 * @throws Throwable
	 */
	void before(Method method, Object[] args, Object target) throws Throwable;
}
```

##### 最终通知
定义：在方法执行后进行增强

**它可能需要的参数？**
- 方法本身 Method
- 方法所属的对象 Object
- 方法的参数 Object[]
- 方法的返回值 Object 可能没有

**它的返回值是什么？**  
这个就需要看是否允许在After中更改返回的结果，如果规定只可用、不可修改返回值就不需要返回值。

```java
public interface AfterAdvice extends Advice {
	/**
	 * 实现该方法，提供后置增强
	 * 
	 * @param returnValue
	 *            返回值
	 * @param method
	 *            被增强的方法
	 * @param args
	 *            方法的参数
	 * @param target
	 *            方法的所属对象
	 * @throws Throwable
	 */
	void after(Object returnValue, Method method, Object[] args, Object target) throws Throwable;
}
```

##### 后置通知
> 在方法执行后进行增强

**它可能需要的参数？**
- 方法本身 Method
- 方法所属的对象 Object
- 方法的参数 Object[]
- 方法的返回值 Object

**它的返回值是什么？**  
这个就需要看是否允许在After中更改返回的结果，如果规定只可用、不可修改返回值就不需要返回值。

```java
public interface AfterReturningAdvice extends Advice {
	/**
	 * 实现该方法，提供AfterRetun增强
	 * 
	 * @param returnValue
	 *            返回值
	 * @param method
	 *            被增强的方法
	 * @param args
	 *            方法的参数
	 * @param target
	 *            方法的所属对象
	 * @throws Throwable
	 */
	void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable;
}
```

##### 环绕通知
> 包裹方法进行增强

**它可能需要的参数？**
- 方法本身 Method
- 方法所属的对象 Object
- 方法的参数 Object[]

**它的返回值是什么？**  
方法被它包裹，即方法将由它来执行，它需要返回方法的返回值。

```java
public interface MethodInterceptor extends Advice {
	/**
	 * 对方法进行环绕（前置、后置）增强、异常处理增强，方法实现中需调用目标方法。
	 * 
	 * @param method
	 *            被增强的方法
	 * @param args
	 *            方法的参数
	 * @param target
	 *            方法所属对象
	 * @return Object 返回值
	 * @throws Throwable
	 */
	Object invoke(Method method, Object[] args, Object target) throws Throwable;
}
```

##### 异常通知
> 对方法执行时的异常，进行增强处理。

**它可能需要什么参数?**
- 一定需要Exception
- 可能需要方法本身 Method
- 可能需要方法所属的对象 Object
- 可能需要方法的参数 Object[]

**它的返回值是什么?**  
这个就需要看是否允许在After中更改返回的结果，如果规定只可用、不可修改返回值就不需要返回值。

```java
public interface ThrowsAdvice extends Advice {

    void afterThrowing(Method method, Object[] args, Object target, Exception ex) throws Throwable;
}
```

#### Advice设计
结合上面的分析，我们就可以得出Advice的体系图了

![Spring（三）：手写AOP_12.png](./pics/Spring（三）：手写AOP_12.png)


### Pointcut

#### Pointcut的特点

- 用户性：由用户指定；
- 变化性：用户可灵活指定；
- 多点性：用户可以选择在多个点上进行增强

#### Pointcut分析

为用户提供一个东西，让他们可以灵活地指定多个方法点，而且我们还能看懂！


思考：切入点是由用户来指定在哪些方法点上进行增强，那么这个哪些方法点如何来表示能满足上面的需求呢？

分析：
1. 指定哪些方法，是不是一个描述信息？
2. 如何来指定一个方法？
3. 如果有重载的情况怎么办？
4. 123要求的起始就是一个完整的方法签名

```java
com.boge.spring.aop.Girl.dbj(Boy,Time)

com.boge.spring.aop.Girl.dbj(Boy,Girl,Time)
```

我们还得进一步分析：如何做到多点性和灵活性，在一个描述中指定一类类的某些方法？

- 某个包下的某个类的某个方法
- 某个包下的所有类中的所有方法
- 某个包下的所有类中的do开头的方法
- 某个包下的以service结尾的类中的do开头的方法
- ......

也就是我们需要有这样一个表达式能够灵活的描述上面的这些信息。

这个表达式表达的内容有：

![Spring（三）：手写AOP_13.png](./pics/Spring（三）：手写AOP_13.png)

而且每个部分的要求是怎么样的呢？
- 包名：有父子特点，要能模糊匹配；
- 类名：要能模糊匹配；
- 方法名：要能模糊匹配；
- 参数类型：参数可以有多个

那么我们设计的这个表达式将被我们用来决定是否需要对某个类的某个方法进行增强，这个决定过程应该是怎么样的？

![Spring（三）：手写AOP_14.png](./pics/Spring（三）：手写AOP_14.png
)

针对需求，我们的选择是：

![Spring（三）：手写AOP_15.png](./pics/Spring（三）：手写AOP_15.png)

AspectJ官网：http://www.eclipse.org/aspectj

![Spring（三）：手写AOP_16.png](./pics/Spring（三）：手写AOP_16.png)

切入点表达式要匹配的对象就是目标方法的方法名。所以，execute表达式中明显就是方法的签名。注意，表达式中加[ ]的部分表示可省略部分，各部分间用空格分开。在其中可以使用以下符号

![Spring（三）：手写AOP_17.png](./pics/Spring（三）：手写AOP_17.png)

举例：

> execution(public * *(. .))
>
> 指定切入点为：任意公共方法。

> execution(* set *(. .))
>
> 指定切入点为：任何一个以“set”开始的方法。

> execution(* com.xyz.service.*.*(. .))
>
> 指定切入点为：定义在service包里的任意类的任意方法。

> execution(* com.xyz.service. .*.*(. .))
>
> 指定切入点为：定义在service包或者子包里的任意类的任意方法。“..”出现在类名中时，
> 后面必须跟“*”，表示包、子包下的所有类。

> execution(* *.service.*.*(. .))
>
> 指定只有一级包下的serivce子包下所有类(接口)中的所有方法为切入点

> execution(* *. .service.*.*(. .))
>
> 指定所有包下的serivce子包下所有类(接口)中的所有方法为切入点


#### Pointcut设计

- 思考1：首先考虑切入点应该具有的属性---->切入点表达式
- 思考2：切入点应对外提供什么行为
- 思考3：切入点被我们设计用来做什么？

对类和方法进行匹配，切入点应该提供匹配类，匹配方法的行为。

- 思考4：如果在我们设计的框架中要能灵活的扩展切点，我们应该如何设计？

这又是一个要支持可多变的问题，像通知一样，我们定义一套标准接口，定义好基本行为，面向接口编程，屏蔽掉具体的实现。不管哪些方案，都实现匹配类，匹配方法的接口。

![Spring（三）：手写AOP_18.png](./pics/Spring（三）：手写AOP_18.png)

案例代码

```java
public interface Pointcut {

	boolean matchsClass(Class<?> targetClass);

	boolean matchsMethod(Method method, Class<?> targetClass);
}
```

然后来看看AspectJ的实现

![Spring（三）：手写AOP_19.png](./pics/Spring（三）：手写AOP_19.png)


案例代码

```java
public class AspectJExpressionPointcut implements Pointcut {

	private static PointcutParser pp = PointcutParser
			.getPointcutParserSupportingAllPrimitivesAndUsingContextClassloaderForResolution();

	private String expression;

	private PointcutExpression pe;

	public AspectJExpressionPointcut(String expression) {
		super();
		this.expression = expression;
		pe = pp.parsePointcutExpression(expression);
	}

	@Override
	public boolean matchsClass(Class<?> targetClass) {
		return pe.couldMatchJoinPointsInType(targetClass);
	}

	@Override
	public boolean matchsMethod(Method method, Class<?> targetClass) {
		ShadowMatch sm = pe.matchesMethodExecution(method);
		return sm.alwaysMatches();
	}

	public String getExpression() {
		return expression;
	}

}
```

### 切面Aspect

搞定了两个难点后，我们来看看用户该如何使用我们提供的东西

![Spring（三）：手写AOP_20.png](./pics/Spring（三）：手写AOP_20.png)

为此我们需要创建对应的接口来管理。


### Advisor

为用户提供更简单的外观，Advisor(通知者)组合Advice和Pointcut。

![Spring（三）：手写AOP_21.png](./pics/Spring（三）：手写AOP_21.png)

当然扩展形式比较多：

![Spring（三）：手写AOP_22.png](./pics/Spring（三）：手写AOP_22.png)

或者：

![Spring（三）：手写AOP_23.png](./pics/Spring（三）：手写AOP_23.png)


## 织入实现

### 织入的分析

织入要完成的是什么？织入其实就是要把用户提供的增强功能加到指定的方法上。

![Spring（三）：手写AOP_24.png](./pics/Spring（三）：手写AOP_24.png)

**思考1：在什么时候织入？**  
> 创建Bean实例的时候，在Bean初始化后，再对其进行增强。

**思考2：如何确定bean要增强？**
> 对bean类及方法挨个匹配用户配置的切面，如果有切面匹配就是要增强。

**思考3：如何实现织入？**
> 代理方式

### 织入的设计
为了更好的去设计织入的实现，先整理下AOP的使用流程。

![Spring（三）：手写AOP_25.png](./pics/Spring（三）：手写AOP_25.png)

这里我们要考虑匹配、织入逻辑写到哪里？是写在BeanFactory中吗?

这时我们要考虑如果我们直接在BeanFactory中来处理，后续如果还有其他的需求是不是也要在BeanFactory中处理呢？这样操作有什么不好的地方呢？

- BeanFactory代码爆炸，不专情
- 不易扩展


我们先回顾下Bean的生产过程

![Spring（三）：手写AOP_26.png](./pics/Spring（三）：手写AOP_26.png)

在这个过程中，将来会有更多处理逻辑加入到Bean生产过程的不同阶段。我们现在最好是设计出能让我们后面不用再改BeanFactory的代码就能灵活的扩展。

这时我们可以考虑用观察者模式，通过在各个节点加入扩展点，加入注册机制。

![Spring（三）：手写AOP_27.png](./pics/Spring（三）：手写AOP_27.png)

那么在这块我们就应用观察者模式来加入一个Bean的后置处理器 BeanPostProcessor

![Spring（三）：手写AOP_28.png](./pics/Spring（三）：手写AOP_28.png)

