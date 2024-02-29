# Spring（八）：DI的过程

接下来我们分析下Spring源码中Bean初始化过程中的DI过程，也就是属性的依赖注入。

## 构造参数依赖

### 如何确定构造方法

在Spring中生成Bean实例的时候默认是调用对应的无参构造方法来处理。

```java
@Component
public class BeanK {

    private BeanE beanE;
    private BeanF beanF;

  
    public BeanK(BeanE beanE) {
        this.beanE = beanE;
    }

    public BeanK(BeanE beanE, BeanF beanF) {
        this.beanE = beanE;
        this.beanF = beanF;
    }
}
```

声明了两个构造方法，但是没有提供无参的构造方法。这时从容器中获取会报错。

![Spring（八）：DI的过程_1.png](./pics/Spring（八）：DI的过程_1.png)

这时我们需要在显示使用的构造方法中添加 @Autowired 注解即可。

![Spring（八）：DI的过程_2.png](./pics/Spring（八）：DI的过程_2.png)

源码层面的核心

```java
	protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
		// Make sure bean class is actually resolved at this point.
		// 确认需要创建的bean实例的类可以实例化
		Class<?> beanClass = resolveBeanClass(mbd, beanName);

		// 确保class不为空，并且访问权限是public
		if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
			throw new BeanCreationException(mbd.getResourceDescription(), beanName,
					"Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
		}

		// 判断当前beanDefinition中是否包含实例供应器，此处相当于一个回调方法，利用回调方法来创建bean
		Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
		if (instanceSupplier != null) {
			return obtainFromSupplier(instanceSupplier, beanName);
		}

		// 如果工厂方法不为空则使用工厂方法初始化策略
		if (mbd.getFactoryMethodName() != null) {
			return instantiateUsingFactoryMethod(beanName, mbd, args);
		}

		// 一个类可能有多个构造器，所以Spring得根据参数个数、类型确定需要调用的构造器
		// 在使用构造器创建实例后，Spring会将解析过后确定下来的构造器或工厂方法保存在缓存中，避免再次创建相同bean时再次解析

		// Shortcut when re-creating the same bean...
		// 标记下，防止重复创建同一个bean
		boolean resolved = false;
		// 是否需要自动装配
		boolean autowireNecessary = false;
		// 如果没有参数
		if (args == null) {
			synchronized (mbd.constructorArgumentLock) {
				// 因为一个类可能由多个构造函数，所以需要根据配置文件中配置的参数或传入的参数来确定最终调用的构造函数。
				// 因为判断过程会比较，所以spring会将解析、确定好的构造函数缓存到BeanDefinition中的resolvedConstructorOrFactoryMethod字段中。
				// 在下次创建相同时直接从RootBeanDefinition中的属性resolvedConstructorOrFactoryMethod缓存的值获取，避免再次解析
				if (mbd.resolvedConstructorOrFactoryMethod != null) {
					resolved = true;
					autowireNecessary = mbd.constructorArgumentsResolved;
				}
			}
		}
		// 有构造参数的或者工厂方法
		if (resolved) {
			// 构造器有参数
			if (autowireNecessary) {
				// 构造函数自动注入
				return autowireConstructor(beanName, mbd, null, null);
			}
			else {
				// 使用默认构造函数构造
				return instantiateBean(beanName, mbd);
			}
		}

		// Candidate constructors for autowiring?
		// 从bean后置处理器中为自动装配寻找构造方法, 有且仅有一个有参构造或者有且仅有@Autowired注解构造
		Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
		// 以下情况符合其一即可进入
		// 1、存在可选构造方法
		// 2、自动装配模型为构造函数自动装配
		// 3、给BeanDefinition中设置了构造参数值
		// 4、有参与构造函数参数列表的参数
		if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
				mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
			return autowireConstructor(beanName, mbd, ctors, args);
		}

		// Preferred constructors for default construction?
		// 找出最合适的默认构造方法
		ctors = mbd.getPreferredConstructors();
		if (ctors != null) {
			// 构造函数自动注入
			return autowireConstructor(beanName, mbd, ctors, null);
		}

		// No special handling: simply use no-arg constructor.
		// 使用默认无参构造函数创建对象，如果没有无参构造且存在多个有参构造且没有@AutoWired注解构造，会报错
		return instantiateBean(beanName, mbd);
	}
```

### 