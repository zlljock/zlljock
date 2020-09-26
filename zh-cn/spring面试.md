常见的配置方式有三种：基于XML的配置、基于注解的配置、基于Java的配置。

## I. IOC

**，即“控制反转”，不是什么技术，而是一种设计思想**

**Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制**

传统Java SE程序设计，我们直接在对象内部通过new进行创建对象，是程序主动去创建依赖对象；而IoC是有专门一个容器来创建这些对象，即由Ioc容器来控制对 象的创建

IOC让对象的创建不用去new了，可以由spring自动生产，使用java的反射机制，根据配置文件在运行时动态的去创建对象以及管理对象，并调用对象的方法的。

）Spring的IOC有三种注入方式 ：构造器注入、setter方法注入、根据注解注入。

降低耦合，

## II. 请解释Spring Bean的生命周期？

1. 实例化 Instantiation
2. 属性赋值 Populate
3. 初始化 Initialization
4. 销毁 Destruction

 Spring上下文中的Bean生命周期也类似，如下：

#### （1）实例化Bean：

对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。对于ApplicationContext容器，当容器启动结束后，通过获取BeanDefinition对象中的信息，实例化所有的bean。

#### （2）设置对象属性（依赖注入）：

实例化后的对象被封装在BeanWrapper对象中，紧接着，Spring根据BeanDefinition中的信息 以及 通过BeanWrapper提供的设置属性的接口完成依赖注入。

#### （3）处理Aware接口：

接着，Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给Bean：

①如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName(String beanId)方法，此处传递的就是Spring配置文件中Bean的id值；

②如果这个Bean已经实现了BeanFactoryAware接口，会调用它实现的setBeanFactory()方法，传递的是Spring工厂自身。

③如果这个Bean已经实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文；

#### （4）BeanPostProcessor：

如果想对Bean进行一些自定义的处理，那么可以让Bean实现了BeanPostProcessor接口，那将会调用postProcessBeforeInitialization(Object obj, String s)方法。

#### （5）InitializingBean 与 init-method：

如果Bean在Spring配置文件中配置了 init-method 属性，则会自动调用其配置的初始化方法。

#### （6）如果这个Bean实现了BeanPostProcessor接口，

将会调用postProcessAfterInitialization(Object obj, String s)方法；由于这个方法是在Bean初始化结束时调用的，所以可以被应用于内存或缓存技术；

> 以上几个步骤完成后，Bean就已经被正确创建了，之后就可以使用这个Bean了。

#### （7）DisposableBean：

当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用其实现的destroy()方法；

#### （8）destroy-method：

最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。

## III. Spring AOP 

#### 1. 面向切面编程

弥补面向对象编程(Object Oriented Programming)的不足

一个横切关注点是一个可以影响到整个应用的关注点，而且应该被尽量地集中到代码的一个地方，常用于日志记录，性能统计，安全控制，事务处理，异常处理等等。

可以减少代码的量

#### 2. 通过以下两种方式来使用

注解和xml配置

#### 3. Advice主要有以下5种类型

1. **前置通知(Before Advice)**: 在连接点之前执行的Advice，不过除非它抛出异常，否则没有能力中断执行流。使用 `@Before` 注解使用这个Advice。
2. **返回之后通知(After Retuning Advice)**: 在连接点正常结束之后执行的Advice。例如，如果一个方法没有抛出异常正常返回。通过 `@AfterReturning` 关注使用它。
3. **抛出（异常）后执行通知(After Throwing Advice)**: 如果一个方法通过抛出异常来退出的话，这个Advice就会被执行。通用 `@AfterThrowing` 注解来使用。
4. **后置通知(After Advice)**: 无论连接点是通过什么方式退出的(正常返回或者抛出异常)都会执行在结束后执行这些Advice。通过 `@After` 注解使用。
5. **围绕通知(Around Advice)**: 围绕连接点执行的Advice，就你一个方法调用。这是最强大的Advice。通过 `@Around` 注解使用。

#### 4. 代理

Spring AOP默认为 AOP 代理使用标准的 JDK 动态代理。这使得任何接口（或者接口的集合）可以被代理。Spring AOP 也可以使用 CGLIB 代理。这对代理类而不是接口是必须的。

#### 5. 什么是织入(weaving)？

织入(Weaving)，[AOP](https://baike.baidu.com/item/AOP)术语。把切面（aspect）连接到其它的应用程序类型或者对象上，并创建一个被通知（advised）的对象，这样的行为叫做织入。

## IV. 事务

事务最重要的两个特性，是事务的传播级别和数据隔离级别。传播级别定义的是事务的控制范围，事务隔离级别定义的是事务在数据库读写方面的控制范围。