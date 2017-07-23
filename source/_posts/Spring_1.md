---
title: Spring IoC
tags: Spring
categories: Spring
---

### Spring IoC 容器：
Spring 容器是 Spring 框架的核心。容器将创建对象，把它们连接在一起，配置它们，并管理他们的整个生命周期从创建到销毁。Spring 容器使用依赖注入（DI）来管理组成一个应用程序的组件。这些对象被称为 Spring Beans.

通过阅读配置元数据提供的指令，容器知道对哪些对象进行实例化，配置和组装。配置元数据可以通过 XML，Java 注释或 Java 代码来表示。下图是 Spring 如何工作的高级视图。 Spring IoC 容器利用 Java 的 POJO 类和配置元数据来生成完全配置和可执行的系统或应用程序。
![](/images/spring_1.png)

#### Spring BeanFactory 容器
它是最简单的容器，给 DI 提供了基本的支持，它用 org.springframework.beans.factory.BeanFactory 接口来定义。BeanFactory 或者相关的接口，如 BeanFactoryAware，InitializingBean，DisposableBean，在 Spring 中仍然存在具有大量的与 Spring 整合的第三方框架的反向兼容性的目的。

#### Spring ApplicationContext 容器
该容器添加了更多的企业特定的功能，例如从一个属性文件中解析文本信息的能力，发布应用程序事件给感兴趣的事件监听器的能力。该容器是由org.springframework.context.ApplicationContext 接口定义。



### Spring Bean:
被称作 bean 的对象是构成应用程序的支柱也是由 Spring IoC 容器管理的。bean 是一个被实例化，组装，并通过 Spring IoC 容器所管理的对象。这些 bean 是由用容器提供的配置元数据创建的.
<bean id="..." class="..." init-method="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

- class	这个属性是强制性的，并且指定用来创建 bean 的 bean 类。
- name	这个属性指定唯一的 bean 标识符。在基于 XML 的配置元数据中，你可以使用 ID 和/或 name 属性来指定 bean 标识符。
- scope	这个属性指定由特定的 bean 定义创建的对象的作用域，它将会在 bean 作用域的章节中进行讨论。
- constructor-arg	它是用来注入依赖关系的，并会在接下来的章节中进行讨论。
- properties	它是用来注入依赖关系的，并会在接下来的章节中进行讨论。
- autowiring mode	它是用来注入依赖关系的，并会在接下来的章节中进行讨论。
- lazy-initialization mode	延迟初始化的 bean 告诉 IoC 容器在它第一次被请求时，而不是在启动时去创建一个 bean 实例。
- initialization 方法	在 bean 的所有必需的属性被容器设置之后，调用回调方法。它将会在 bean 的生命周期章节中进行讨论。
- destruction 方法	当包含该 bean 的容器被销毁时，使用回调方法。它将会在 bean 的生命周期章节中进行讨论。

#### Bean 作用域：

作用域	描述
singleton	该作用域将 bean 的定义的限制在每一个 Spring IoC 容器中的一个单一实例(默认)。
prototype	该作用域将单一 bean 的定义限制在任意数量的对象实例。
request	该作用域将 bean 的定义限制为 HTTP 请求。只在 web-aware Spring ApplicationContext 的上下文中有效。
session	该作用域将 bean 的定义限制为 HTTP 会话。 只在web-aware Spring ApplicationContext的上下文中有效。
global-session	该作用域将 bean 的定义限制为全局 HTTP 会话。只在 web-aware Spring ApplicationContext 的上下文中有效。

#### singleton 作用域：

如果作用域设置为 singleton，那么 Spring IoC 容器刚好创建一个由该 bean 定义的对象的实例。该单一实例将存储在这种单例 bean 的高速缓存中，以及针对该 bean 的所有后续的请求和引用都返回缓存对象。

#### prototype 作用域

如果作用域设置为 prototype，那么每次特定的 bean 发出请求时 Spring IoC 容器就创建对象的新的 Bean 实例。一般说来，满状态的 bean 使用 prototype 作用域和没有状态的 bean 使用 singleton 作用域。

#### Bean 生命周期：
理解 Spring bean 的生命周期很容易。当一个 bean 被实例化时，它可能需要执行一些初始化使它转换成可用状态。同样，当 bean 不再需要，并且从容器中移除时，可能需要做一些清除工作。

#### 初始化回调
org.springframework.beans.factory.InitializingBean 接口指定一个单一的方法：

void afterPropertiesSet() throws Exception;

#### 销毁回调
org.springframework.beans.factory.DisposableBean 接口指定一个单一的方法：

void destroy() throws Exception;

#### 默认的初始化和销毁方法

如果你有太多具有相同名称的初始化或者销毁方法的 Bean，那么你不需要在每一个 bean 上声明初始化方法和销毁方法。框架使用 元素中的 default-init-method 和 default-destroy-method 属性提供了灵活地配置这种情况.

#### Spring——Bean 后置处理器
BeanPostProcessor 接口定义回调方法，你可以实现该方法来提供自己的实例化逻辑，依赖解析逻辑等。你也可以在 Spring 容器通过插入一个或多个 BeanPostProcessor 的实现来完成实例化，配置和初始化一个bean之后实现一些自定义逻辑回调方法。

ApplicationContext 会自动检测由 BeanPostProcessor 接口的实现定义的 bean，注册这些 bean 为后置处理器，然后通过在容器中创建 bean，在适当的时候调用它。
```java
package com.tutorialspoint;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.beans.BeansException;
public class InitHelloWorld implements BeanPostProcessor {
   public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
      System.out.println("BeforeInitialization : " + beanName);
      return bean;  // you can return any other object as well
   }
   public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
      System.out.println("AfterInitialization : " + beanName);
      return bean;  // you can return any other object as well
   }
}
```
你需要注册一个在 AbstractApplicationContext 类中声明的关闭 hook 的 registerShutdownHook() 方法。它将确保正常关闭，并且调用相关的 destroy 方法。


### Autowiring + @Autowired + default-autowire

为避免 Spring 注入的人为工作很麻烦， 可以设置 bean Autowiring， 搜寻 appContext 进行 自动注入。

#### Autowiring:
可选值	功能说明
no	默认不使用autowiring。 必须显示的使用”“标签明确地指定bean。
byName	根据属性名自动装配。此选项将检查容器并根据名字查找与属性完全一致的bean，并将其与属性自动装配。
byType	如果容器中存在一个与指定属性类型相同的bean，那么将与该属性自动装配。如果存在多个该类型的bean，那么将会抛出异常，并指出不能使用byType方式进行自动装配。若没有找到相匹配的bean，则什么事都不发生，属性也不会被设置。如果你不希望这样，那么可以通过设置 dependency-check=”objects”让Spring抛出异常。
constructor	与byType的方式类似，不同之处在于它应用于构造器参数。如果在容器中没有找到与构造器参数类型一致的bean，那么将会抛出异常。
autodetect	通过bean类的自省机制（introspection）来决定是使用constructor还是byType方式进行自动装配。如果发现默认的构造器，那么将使用byType方式。

@Autowired注解注入　
　　@Autowired往往用在类中注解注入，在配置xml需要配置才能使用@Autowired标识
    <context:annotation-config/>
@Autowired
@Qualifier("teacher”)
private Teacher teacher;
public void setTeacher(Teacher teacher)
{this.teacher = teacher; }
beans标签设置default-autowire参数

　　在spring的配置文件中可以参照如下设置default-autowire参数

    default-autowire="byName"

#### Spring 注解 配置
    
    1	@Required
    @Required 注解应用于 bean 属性的 setter 方法。

    2	@Autowired
    @Autowired 注解可以应用到 bean 属性的 setter 方法，非 setter 方法，构造函数和属性。

    3	@Qualifier
    通过指定确切的将被连线的 bean，@Autowired 和 @Qualifier 注解可以用来删除混乱。

    4	JSR-250 Annotations
    Spring 支持 JSR-250 的基础的注解，其中包括了 @Resource，@PostConstruct 和 @PreDestroy 注解。
