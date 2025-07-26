---
title: 'Springboot部分源码相关记录'
author: ['kyle']
date: '2025-07-26T16:40:13+08:00'
draft: true
tags:
- Java
- Spring
- SpringBoot

keywords:
- Java
- Spring
- SpringBoot
---

## 方法或构造器参数名字获取：
DefaultParameterNameDiscoverer
> 其中默认用到了StandardReflectionParameterNameDiscoverer （利用反射中的参数名获取）、和LocalVariableTableParameterNameDiscoverer（利用asm解析class文件获取）

## ClassPathBeanDefinitionScanner
* doScan
> 通过findCandidateComponents扫描到beanDefinition，然后判断register中是否已存在定义，不存在的话会将扫描到的注册到register中
* scanCandidateComponents
> 实际扫描的逻辑


## 其他注入Bean的方式
### 通过实现BeanDefinitionRegistryPostProcessor注入BeanDefinition也能注册bean
```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor;
import org.springframework.beans.factory.support.GenericBeanDefinition;
 
public class OO implements BeanDefinitionRegistryPostProcessor {
 
	@Override
	public void postProcessBeanFactory(
			ConfigurableListableBeanFactory beanFactory) throws BeansException {		
	}
 
	@Override
	public void postProcessBeanDefinitionRegistry(
			BeanDefinitionRegistry registry) throws BeansException {
		
		GenericBeanDefinition helloBean = new GenericBeanDefinition();
		helloBean.setBeanClass(OOO.class);
		// 新增Bean定义
		registry.registerBeanDefinition("hello", helloBean);
	}
}
```

### ApplicationContext
```java

public class SpringObjectFactory {
    
    private static ApplicationContext applicationContext;
    
    public void register(String beanName,Class<?> beanClass){
    	BeanDefinitionRegistry beanRegistry = (BeanDefinitionRegistry)applicationContext.getAutowireCapableBeanFactory();
    	try {
    		beanRegistry.getBeanDefinition(beanName);
    		return;
    	} catch (Exception e) {
    		GenericBeanDefinition definition =  new GenericBeanDefinition();
    		definition.setBeanClass(beanClass);
            definition.setScope(BeanDefinition.SCOPE_SINGLETON);
            beanRegistry.registerBeanDefinition(beanName, definition);
    	}
    }
    
    @Autowired
    public void setApplicationContext(ApplicationContext appCtx) {
        applicationContext = appCtx;
        Assert.isTrue(applicationContext.getAutowireCapableBeanFactory() instanceof BeanDefinitionRegistry, "autowireCapableBeanFactory should be BeanDefinitionRegistry");
    }
    
}
```

### 实现 BeanFactoryPostProcessor
```java

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.GenericBeanDefinition;
 
public class DemoBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        BeanDefinitionRegistry beanRegistry = (BeanDefinitionRegistry)beanFactory;
        //自定义逻辑，注入beandefinition
        beanRegistry.registerBeanDefinition("",new GenericBeanDefinition());
    }

}
```


## @Autowire实现原理：AutowiredAnnotationBeanPostProcessor

## RootBeanDefinition、GenericBeanDefinition、ChildBeanDefinition
`RootBeanDefinition`、`GenericBeanDefinition` 和 `ChildBeanDefinition` 是 Spring 框架中的三个不同的 `BeanDefinition` 子类，用于定义 Spring Bean 的配置信息。让我为你详细解释一下它们之间的区别：

1. **RootBeanDefinition**:
   - `RootBeanDefinition` 是最常用的实现类，对应一般性的 `<bean>` 元素标签。
   - 存储 `BeanDefinitionHolder` 对象，该对象用来存储 Bean 的名称与别名的对应关系。
   - 提供了访问 `BeanDefinition` 的方式，例如 `AnnotatedElement`，开发者可以通过它来操作注解。
   - 允许缓存，是否允许缓存的属性是 `allowCaching`。
   - 不能作为其他 `BeanDefinition` 的子类，但可以作为其他 `BeanDefinition` 的父类。
   - 通常在配置文件中定义没有父 `<bean>` 的情况下使用。

2. **GenericBeanDefinition**:
   - `GenericBeanDefinition` 是自 Spring 2.5 版本以后新加入的 bean 配置属性定义类，是一站式服务类。
   - 用于注册用户可见的 `BeanDefinition`，允许动态定义父依赖项，而不是一个以“硬编码”定义 `BeanDefinition` 的角色。
   - 可以在运行时将 `GenericBeanDefinition` 转换为 `RootBeanDefinition`。
   - 通常用于具有父/子关系的 `BeanDefinition` 预定义。

3. **ChildBeanDefinition**:
   - `ChildBeanDefinition` 表示子类，不可以单独存在，必须依赖一个父 `BeanDefinition`。
   - 用于定义具有父/子关系的 `BeanDefinition`。
   - 一个 `RootBeanDefinition` 定义表明它是一个可合并的 `BeanDefinition`，即在 Spring BeanFactory 运行期间，可以返回一个特定的 Bean。

总结：
- 如果你需要编程定义 `BeanDefinition`，推荐使用 `GenericBeanDefinition`。
- 如果在配置文件中定义父子关系的 `<bean>`，父 `<bean>` 使用 `RootBeanDefinition`，而子 `<bean>` 使用 `ChildBeanDefinition`。


### BeanDifinition方法描述
1. 首先一开始定义了两个变量用来描述 Bean 是不是单例的，后面的 setScope/getScope 方法可以用来修改/获取 scope 属性。
2. ROLE_xxx 用来描述一个 Bean 的角色，ROLE_APPLICATION 表示这个 Bean 是用户自己定义的 Bean；ROLE_SUPPORT 表示这个 Bean 是某些复杂配置的支撑部分；ROLE_INFRASTRUCTURE 表示这是一个 Spring 内部的 Bean，通过 setRole/getRole 可以修改。
3. setParentName/getParentName 用来配置 parent 的名称，这块可能有的小伙伴使用较少，这个对应着 XML 中的 <bean parent=""> 配置。
4. setBeanClassName/getBeanClassName 这个就是配置 Bean 的 Class 全路径，对应 XML 中的 <bean class=""> 配置。
5. setLazyInit/isLazyInit 配置/获取 Bean 是否懒加载，这个对应了 XML 中的 <bean lazy-init=""> 配置。
6. setDependsOn/getDependsOn 配置/获取 Bean 的依赖对象，这个对应了 XML 中的 <bean depends-on=""> 配置。
7. setAutowireCandidate/isAutowireCandidate 配置/获取 Bean 是否是自动装配，对应了 XML 中的 <bean autowire-candidate=""> 配置。
8. setPrimary/isPrimary 配置/获取当前 Bean 是否为首选的 Bean，对应了 XML 中的 <bean primary=""> 配置。
9. setFactoryBeanName/getFactoryBeanName 配置/获取 FactoryBean 的名字，对应了 XML 中的 <bean factory-bean=""> 配置，factory-bean 松哥在之前的入门视频中讲过，小伙伴们可以参考这里:https://www.bilibili.com/video/BV1Wv41167TU。
10. setFactoryMethodName/getFactoryMethodName 和上一条成对出现的，对应了 XML 中的 <bean factory-method=""> 配置，不再赘述。
11. getConstructorArgumentValues 返回该 Bean 构造方法的参数值。
12. hasConstructorArgumentValues 判断上一条是否是空对象。
13. getPropertyValues 这个是获取普通属性的集合。
14. hasPropertyValues 判断上一条是否为空对象。
15. setInitMethodName/setDestroyMethodName 配置 Bean 的初始化方法、销毁方法。
16. setDescription/getDescription 配置/返回 Bean 的描述。
17. isSingleton Bean 是否为单例。
18. isPrototype Bean 是否为原型。
19. isAbstract Bean 是否抽象。
20. getResourceDescription 返回定义 Bean 的资源描述。
21. getOriginatingBeanDefinition 如果当前 BeanDefinition 是一个代理对象，那么该方法可以用来返回原始的 BeanDefinition 。


## ProxyFactory

创建代理，