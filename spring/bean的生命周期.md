这边讲的bean的生命周期

一.（容器的实例化）(1-4不算bean的实例化)【】

1.实例化BeanFactoryPostProcessor实现类

2.执行BeanFactoryPostProcessor的postProcessBeanFactory方法



```
@FunctionalInterface
public interface BeanFactoryPostProcessor {

	/**
	 * Modify the application context's internal bean factory after its standard
	 * initialization. All bean definitions will have been loaded, but no beans
	 * will have been instantiated yet. This allows for overriding or adding
	 * properties even to eager-initializing beans.
	 * @param beanFactory the bean factory used by the application context
	 * @throws org.springframework.beans.BeansException in case of errors
	 */
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

}

```

3.实例化BeanPostProcessor实现类

```
public interface BeanPostProcessor {

	/**
	 * Apply this {@code BeanPostProcessor} to the given new bean instance 				 *<i>before</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * <p>The default implementation returns the given {@code bean} as-is.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 * if {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 */
	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	/**
	 * Apply this {@code BeanPostProcessor} to the given new bean instance <i>after</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * <p>In case of a FactoryBean, this callback will be invoked for both the FactoryBean
	 * instance and the objects created by the FactoryBean (as of Spring 2.0). The
	 * post-processor can decide whether to apply to either the FactoryBean or created
	 * objects or both through corresponding {@code bean instanceof FactoryBean} checks.
	 * <p>This callback will also be invoked after a short-circuiting triggered by a
	 * {@link InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation} method,
	 * in contrast to all other {@code BeanPostProcessor} callbacks.
	 * <p>The default implementation returns the given {@code bean} as-is.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 * if {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 * @see org.springframework.beans.factory.FactoryBean
	 */
	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

}

```

4.实例化InstantiationAwareBeanPostProcessorAdapter实现类

![InstantiationAwareBeanPostProcessorAdapter](D:\制作地点\InstantiationAwareBeanPostProcessorAdapter.png)

```
public abstract class InstantiationAwareBeanPostProcessorAdapter implements SmartInstantiationAwareBeanPostProcessor {
	@Override
	@Nullable
	public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		return null;
	}

	@Override
	public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
		return true;
	}
	@Override
	public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)
			throws BeansException {

		return null;
	}
	@Deprecated
	@Override
	public PropertyValues postProcessPropertyValues(
			PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {

		return pvs;
	}

	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

}

```

二.执行Bean的构造器（bean的实例化）

5.执行InstantiationAwareBeanPostProcessorAdapter的postProcessBeforeInstantiation方法（postProcessBeforeInstantiation调用时机为bean实例化(**Instantiation**)之前 如果返回了bean实例, 则会替代原来正常通过target bean生成的bean的流程）

6.执行Bean的构造器

7.执行InstantiationAwareBeanPostProcessorAdapter的postProcessAfterInstantiation方法

8.为Bean注入属性

调用BeanNameAware的setBeanName方法

```
public interface BeanNameAware extends Aware {
	void setBeanName(String name);

}

```

调用BeanFactoryAware的setBeanFactory方法


```
public interface BeanFactoryAware extends Aware {

	/**
	 * Callback that supplies the owning factory to a bean instance.
	 * <p>Invoked after the population of normal bean properties
	 * but before an initialization callback such as
	 * {@link InitializingBean#afterPropertiesSet()} or a custom init-method.
	 * @param beanFactory owning BeanFactory (never {@code null}).
	 * The bean can immediately call methods on the factory.
	 * @throws BeansException in case of initialization errors
	 * @see BeanInitializationException
	 */
	void setBeanFactory(BeanFactory beanFactory) throws BeansException;

}
```

回调到

执行BeanPostProcessor的postProcessBeforeInitialization方法

bean调用InitializingBean的afterPropertiesSet方法

```
package org.springframework.beans.factory;
public interface InitializingBean {

	/**
	 * Invoked by the containing {@code BeanFactory} after it has set all bean properties
	 * and satisfied {@link BeanFactoryAware}, {@code ApplicationContextAware} etc.
	 * <p>This method allows the bean instance to perform validation of its overall
	 * configuration and final initialization when all bean properties have been set.
	 * @throws Exception in the event of misconfiguration (such as failure to set an
	 * essential property) or if initialization fails for any other reason
	 */
	void afterPropertiesSet() throws Exception;

}

```

同时调用<bean>的 init-method属性指定的初始化方法

执行BeanPostProcessor的postProcessAfterInitialization方法

*bean初始化完成

*容器初始化完成，

下面是bean的销毁容器

调用<bean>的destory-method()属性指定的初始化方法

然后调用DiposibleBean的destory方法

```
package org.springframework.beans.factory;
public interface DisposableBean {
	void destroy() throws Exception;
}

```









·