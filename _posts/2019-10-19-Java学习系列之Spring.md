---
layout: post
title:  "Java学习系列之Spring"
categories: Java
tags:  Java Spring
---

* content
{:toc}

本文系记录对Java中I/O的学习资料，如有异议，欢迎联系我讨论修改。PS:图侵删！如果看不清图请访问:
[传送门]()







## Spring基础

#### Spring是什么
Spring是一个轻量级的开源应用框架，旨在降低应用程序开发的复杂度。Spring具有以下特性：轻量级表现在完整的Spring框架可以在一个大小只有1MB多的JAR文件里发布。Spring是非侵入性，即允许应用系统自由选择和组装Spring框架中的各个功能模块，而不要求应用必需对Spring中的某个类继承或实现，极大地提高一致性。使用IOC容器管理对象的生命周期，以及对象间的依赖关系，降低系统耦合性。基于AOP的面向切面编程，将具有横切性质的业务放到切面中，从而与核心业务逻辑分离并提高组件复用性。Spring可以很好地与其他框架集成，被称为框架的框架。

#### Spring涉及的组件
- Data Access：JDBC；JMS；Transaction
- Web：Web；Servlet；Porlet
- AOP
- Core：Beans；Core；Context；Expression Language

#### Spring特性
- IOC
- DI
- AOP
- 事务
- 低侵入性

#### Autowired的实现原理
Spring-IoC容器具有依赖自动装配功能，不需要对Bean属性的依赖关系做显式的声明，只需要在配置好autowiring属性，IoC容器会自动使用反射查找属性的类型和名称，然后基于属性的类型或者名称来自动匹配容器中管理的Bean，从而自动地完成依赖注入。

AbstractAutoWireCapableBeanFactory对Bean实例进行属性依赖注入。应用第一次通过getBean方法(配置了lazy-init预实例化属性的除外)向IoC容器索取Bean时，容器创建Bean实例对象，并且对Bean实例对象进行属性依赖注入，AbstractAutoWireCapableBeanFactory的populateBean方法就是实现Bean属性依赖注入的功能，其主要源码如下：

#### Autowired中的类型
在使用Autowired时，可以配置在<beans>根标签下，表示对全局<bean>起作用，属性名为default-autowire；也可以配置在<bean>标签下，表示对当前<bean>起作用，属性名为autowire。取值可以分为如下几种：  
- no：默认，即不进行自动装配，每一个对象的注入比如依赖一个<property>标签
- byName：按照beanName进行自动装配，使用setter注入，如果不匹配则报错
- byType：按照bean类型进行自动装配，使用setter注入，当有多个相同类型时会报错，解决方法是将不需要的bean设置autowire-candidate=false或对优先需要进行装配的bean设置为primary=true
- constructor：与byType差不多，不过最终属性通过构造函数进行注入

#### Spring中依赖注入的方式
- 构造函数注入
- setter注入
- 接口注入

#### Spring中自动装配的限制
- 如果使用了构造器注入或者setter注入，那么将覆盖自动装配的依赖关系
- 基本数据类型的值、字符串字面量、类字面量无法使用自动装配来注入
- 优先考虑使用显式的装配来进行更精确的依赖注入而不是使用自动装配

## IOC与DI

#### IOC和DI的概念

IOC指控制反转，是Spring中的一种设计思想以及重要特性。IOC意味着将设计好的类交给容器控制，而不是在对象内部控制。控制，指的是容器控制对象，在传统的开发中，我们通过在对象内部通过new进行对象创建，而在IOC中专门有一个容器用来创建对象。反转指的是获取对象的方式发生了反转，以往对外部资源或对象的获取依赖于程序主动通过new引导，现在则是通过容器实现。容器帮我们查找以及注入依赖对象。IOC一条面向对象的重要法则，可以指导我们设计出松耦合的程序，是的程序的系统结构变得非常灵活。  

DI指依赖注入，使得组件间的依赖关系由容器在运行期决定，容器可以动态地将某个依赖关系注入到组件之中。依赖指的是应用程序依赖于IOC容器注入对象所需的外部资源。注入指的是IOC容器向应用程序中注入某个对象或其他外部资源。依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。

#### Spring的IOC体系
![](http://ww1.sinaimg.cn/large/005L0VzSgy1g449wq7mv5j312i0ne419.jpg)  

![](https://ws1.sinaimg.cn/large/005L0VzSly1g557sm1cb0j31300pwn0i.jpg)

#### IOC的大致初始化流程
![](https://ws1.sinaimg.cn/large/005L0VzSly1g558okuw09j30jp01kjr8.jpg)

#### 依赖注入发生在什么时刻
- 第一次通过getBean方法向IoC容索要Bean时，IoC容器触发依赖注入
- 在Bean定义资源中为<Bean>元素配置了lazy-init属性，即让容器在解析注册Bean定义时进行预实例化，触发依赖注入

#### IOC容器的启动过程
(以ClassPathXmlApplicationContext为例)
- 进入构造函数，并先调用父类的的构造函数
- 根据提供的路径，处理配置文件数组，对应方法为：setConfigLocations(configLocations);
- 执行Refresh方法，该方法可以反复调用，销毁当前ApplicationContext并重新执行一次初始化操作，对应方法为：refresh();
- 进入Refresh方法后，首先进入同步块
- 然后执行准备工作，记录容器启动时间，标记当前状态为已启动并处理文件占位符，对应方法为：prepareRefresh();
- 执行Bean的加载、解析、注册，使得XML变为BeanDefinition，并注册到BeanFactory中。这里只是提取Bean的配置信息而非初始化；创建并初始化BeanFactory。对应方法为——obtainFreshBeanFactory();
- 设置BeanFactory的类加载器，添加BeanPostProcessor接口并手动注册几个特殊的Bean。对应方法为——prepareBeanFactory(beanFactory);  

截止至此，Bean的加载、解析、注册已经完成；BeanFactory的创建以及初始化已经完成，如果不是第一次启动还会涉及BeanFactory的销毁  

- 调用BeanFactoryPostProcessor各个实现类的postProcessBeanFactory(factory)方法在初始化前做一些用户自定义的行为。对应方法为——postProcessBeanFactory(beanFactory);invokeBeanFactoryPostProcessors(beanFactory);
- 注册BeanPostProcessor，对应方法为：registerBeanPostProcessors(beanFactory);
- 做一些额外的初始化：initMessageSource();initApplicationEventMulticaster();onRefresh();registerListeners();
- 对所有可初始化的SingletonBean进行初始化，对应方法——finishBeanFactoryInitialization(beanFactory);
- 广播完成状态，对应方法——finishRefresh();

#### IOC中的预实例化
IoC容器的初始化过程就是对Bean定义资源的定位、载入和注册，此时容器对Bean的依赖注入并没有发生，依赖注入主要是在应用程序第一次向容器索取Bean时，通过getBean方法的调用完成。当Bean定义资源的<Bean>元素中配置了lazy-init属性时，容器将会在初始化的时候对所配置的Bean进行预实例化，Bean的依赖注入在容器初始化的时候就已经完成。这样，当应用程序第一次向容器索取被管理的Bean时，就不用再初始化和对Bean进行依赖注入了，直接从容器中获取已经完成依赖注入的现成Bean，可以提高应用第一次向容器获取Bean的性能。

#### BeanPostProcessor后置处理器的实现
BeanPostProcessor后置处理器是Spring-IoC容器经常使用到的一个特性，这个Bean后置处理器是一个监听器，可以监听容器触发的Bean声明周`期事件。后置处理器向容器注册以后，容器中管理的Bean就具备了接收IoC容器事件回调的能力。
```java
package org.springframework.beans.factory.config;  
import org.springframework.beans.BeansException;  
public interface BeanPostProcessor {  
   //为在Bean的初始化前提供回调入口  
   Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;  
   //为在Bean的初始化之后提供回调入口  
   Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;  
 }
```

#### AOP的概念

AOP指面向切面编程，通过预编译方式和运行期动态代理的一种技术。利用AOP可以对业务逻辑的各个部分进行隔离，从而降低业务逻辑各组件的耦合度，提高程序的可重用性，同时提高了开发的效率。AOP具有以下核心概念：
- 切面：一些Pointcut以及相应的Advice的集合
- 连接点：表示在程序中明确定义的点，典型的包括方法调用，对类成员的访问以及异常处理程序块的执行等等，它自身还可以嵌套其它连接点
- 切点：通过制定某种规则来选定一组连接点，这些连接点或是通过逻辑关系组合起来，或是通过通配、正则表达式等方式集中起来，它定义了相应的增强将要发生的地方
- 增强：定义了在切点里面定义的程序点具体要做的操作
- 目标对象：织入增强的目标对象
- 织入：将切面和其他对象连接起来, 并创建增强的过程

AOP常用于：日志、安全控制、性能统计等场景。

#### AOP中增强的几种方式
- before：在切点前执行，除非before中发生异常，否则切点一定执行
- after：在一个切点正常执行后执行
- around：before+after
- final：无论是正常结束还是异常，都会执行
- after throwing：切点抛出异常后执行

#### AOP的实现原理
在AOP的设计中，每个Bean都会被JDK或cglib代理并有多个方法拦截器。拦截器分为两层，外层由Spring内核控制，内层拦截器由用户设置。当代理方法被调用时，先经过外层拦截器，外层拦截器根据方法提供的信息判断哪些内层拦截器应该被执行，然后会创建并执行拦截器链，最后调用目标对象的方法。内层拦截器的设计采用了职责链的设计。

#### 动态代理在Spring中的应用以及优缺点
Spring中AOP的实现依赖于动态代理的实现。动态代理主要有两种实现，分别是JDK动态代理和cglib代理。采用JDK动态代理，目标类必须实现某个接口，否则不可用；而CGLIB底层是通过使用一个小而块的字节码处理框架ASM来转换字节码并生成新的类，覆盖或添加类中的方法。从上述描述中不难看出，cglib类中方法类型不能设置为final。在执行效率上，早期的JDK动态代理效率远低于cglib，而随着JDK版本的更新，现在JDK动态代理的效率已经和cglib不相伯仲。


## Bean

在Spring中，Bean是组成应用程序的主体及由SpringIoC容器所管理的对象，被称之为bean。简单地讲，Bean就是由IoC容器初始化、装配及管理的对象，除此之外，bean就与应用程序中的其他对象没有什么区别了。而Bean的定义以及Bean相互间的依赖关系将通过配置元数据来描述。

#### Bean的作用域
- singleton
- prototype
- request
- session
- globalsession
五种作用域中，request、session和globalsession三种作用域仅在基于web的应用中使用（不必关心你所采用的是什么web应用框架），只能用在基于web的Spring ApplicationContext环境。一般在<bean>标签中通过scope指定作用域类型，也可以在<beans>下指定默认全局的scope类型。其中Singleton与Prototype类型的区别在于：Prototype在交给用户后，IOC容器就结束了使命，放弃对该bean的生命周期管理，而IOC容器则会对Singleton进行完整的生命周期管理；singleton默认采用非延迟初始化，也可通过设置lazy-init属性改变初始化方式，但是prototype只能采用延迟初始化方式。

#### Bean的生命周期
(获得Bean的步骤)
- Spring对bean进行实例化，默认bean是单例；
- Spring对bean进行依赖注入；
- 如果bean实现了BeanNameAware接口，spring将bean的id传给setBeanName()方法；
- 如果bean实现了BeanFactoryAware接口，spring将调用setBeanFactory方法，将BeanFactory实例传进来；
- 如果bean实现了ApplicationContextAware接口，它的setApplicationContext()方法将被调用，将应用上下文的引用传入到bean中；
- 如果bean实现了BeanPostProcessor接口，它的postProcessBeforeInitialization方法将被调用， postProcessBeforeInitialization方法在bean初始化之前执行；
- 如果bean实现了InitializingBean接口，Spring将调用它的afterPropertiesSet接口方法，用于初始化bean的时候执行，可以针对某个具体的bean进行配置；
- 如果bean使用了init-method属性声明了初始化方法，该方法也会被调用，初始化bean的时候执行，可以针对某个具体的bean进行配置；
- 如果bean实现了BeanPostProcessor接口，它的postProcessAfterInitialization接口方法将被调用，postProcessAfterInitialization方法在bean初始化之后执行；
- 此时bean已经准备就绪，可以被应用程序使用了，他们将一直驻留在应用上下文中，直到该应用上下文被销毁；
- 如果bean实现了DisposableBean接口，spring将调用它的distroy()接口方法；
- 如果bean使用了destroy-method属性声明了销毁方法，则该方法被调用

#### BeanFactory
BeanFactory是一个接口，用于定义工厂的基本职能并对IOC容器的基本行为作了定义。它是负责生产和管理bean的一个工厂。在Spring中，BeanFactory是IOC容器的核心接口，它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。BeanFactory只是个接口，并不是IOC容器的具体实现。Spring提供了许多IOC容器的实现,比如XmlBeanFactory，ClasspathXmlApplicationContext，ApplicationContext等。BeanFactory中大致定义了如下行为：  
- getBean(String name)——根据bean的名字，获取在IOC容器中得到bean实例
- containsBean(String name)——提供对bean的检索，看看是否在IOC容器有这个名字的bean
- isSingleton(String name)——根据bean名字得到bean实例，并同时判断这个bean是不是单例
- getType(String name)——得到bean实例的Class类型

#### FactoryBean
一般情况下，Spring通过反射机制利用<bean>的class属性指定实现类实例化Bean，在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在<bean>中提供大量的配置信息。配置方式的灵活性是受限的，这时采用编码的方式可能会得到一个简单的方案。FactoryBean<T>也是一个接口，首先实现了这个接口的类也是一个Bean，但是这个Bean是一个可以生产其他Bean的特类。通过对接口方法的实现，这个Bean被附加了工厂行为和装饰器行为，而具有了生产能力。FactoryBean<T>接口中的主要方法如下：  
- getObject()——获取对象
- getObjectType()——获取对象类型
- isSingleton——是否是单例
值得注意的是如果要获取FactoryBean本身这个Bean，在根据名字传参时要添加一个前缀&

#### BeanDefinition

BeanDefinition 中保存了Bean信息，比如这个Bean指向的是哪个类、是否是单例的、是否懒加载、Bean依赖了哪些Bean等等。由代码实现的Bean在运行时会转换为BeanDefinition存在于BeanFactory中。

#### Context
- FileSystemXmlApplicationContext：该容器从 XML 文件中加载已被定义的 bean。在这里，你需要提供给构造器 XML 文件的完整路径
- ClassPathXmlApplicationContext：该容器从 XML 文件中加载已被定义的 bean。在这里，你不需要提供 XML 文件的完整路径，只需正确配置 CLASSPATH 环境变量即可，因为，容器会从 CLASSPATH 中搜索 bean 配置文件。
- WebXmlApplicationContext：该容器会在一个 web 应用程序的范围内加载在 XML 文件中已被定义的 bean。

#### Spring中BeanFactory与ApplicationContext的区别
- BeanFactroy采用的是延迟加载形式来注入Bean的，即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化，这样，我们就不能发现一些存在的Spring的配置问题。而ApplicationContext则相反，它是在容器启动时，一次性创建了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误
- BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，但两者之间的区别是：BeanFactory需要手动注册，而ApplicationContext则是自动注册
- ApplicationContext包还提供了以下的功能：资源访问，如URL和文件；事件传播；载入多个（有继承关系）上下文；MessageSource, 提供国际化的消息访问
- 前者不支持依赖注解，后者支持

## MVC

#### MVC的执行流程

![](https://ws1.sinaimg.cn/large/005L0VzSly1g56dc6yde0j30oa0dygmz.jpg)

- 用户请求发送至DispatcherServlet类进行处理。
- DispatcherServlet类遍历所有配置的HandlerMapping类请求查找Handler。
- HandlerMapping类根据request请求的URL等信息查找能够进行处理的Handler，以及相关拦截器interceptor并构造HandlerExecutionChain。
- HandlerMapping类将构造的HandlerExecutionChain类的对象返回给前端控制器DispatcherServlet类。
- 前端控制器拿着上一步的Handler遍历所有配置的HandlerAdapter类请求执行Handler。
- HandlerAdapter类执行相关Handler并获取ModelAndView类的对象。
- HandlerAdapter类将上一步Handler执行结果的ModelAndView 类的对象返回给前端控制器。
- DispatcherServlet类遍历所有配置的ViewResolver类请求进行视图解析。
- ViewResolver类进行视图解析并获取View对象。
- ViewResolver类向前端控制器返回上一步骤的View对象。
- DispatcherServlet类进行视图View的渲染，填充Model。
- DispatcherServlet类向用户返回响应。

#### SpringMVC中的分层
![](http://ww1.sinaimg.cn/large/005L0VzSly1fz0ddni132j30fv0cz74f.jpg)  

Dao层主要做数据持久层的工作，负责与数据库进行联络的一些任务皆封装于此。首先设计Dao层的接口，然后在Spring的配置文件中定义此接口的实现类，随后在模块中调用此接口来进行数据业务的处理，而不用关心此接口的具体实现类是哪个类，这样结构变得非常清晰。  

Service层主要负责业务模块的应用逻辑应用设计。首先设计接口，再设计其实现类，接着在Spring的配置文件中配置关联关系。定义Service层的业务时，具体要调用已经定义的dao层接口，封装Service层业务逻辑有利于通用的业务逻辑的独立性和重复利用性。服务具有如下特征：抽象、独立、稳定。  

Model承载的作用就是数据的抽象，描述了一个数据的定义，Model的实例就是一组组数据。整个系统都可以看成是数据的流动，既然要流动，就一定是有流动的载体。可以理解为将数据库的表结构以Java类的形式表现。

Controller层负责具体的业务模块流程的控制，此层要调用Service层的接口来控制业务流程流转，同样在Spring的配置文件里进行配置。对于具体的业务流程，有不同的控制器。设计过程可以将流程进行抽象归纳，设计出可以重复利用的子单元流程模块。这样不仅使程序结构变得清晰，也大大减少了代码量。

View层是前台页面的展示。  

#### Spring中分层领域模型规约
- DO（Data Object）：与数据库表结构一一对应，通过DAO层向上传输数据源对象。从现实世界中抽象出来的有形或无形的业务实体。
- PO（Persistent Object）：持久化对象，它跟持久层（通常是关系型数据库）的数据结构形成一一对应的映射关系，如果持久层是关系型数据库，那么，数据表中的每个字段（或若干个）就对应PO的一个（或若干个）属性。PO仅仅用于表示数据，没有任何数据操作。通常遵守Java Bean的规范，拥有 getter/setter 方法。
- DTO（Data Transfer Object）：数据传输对象，Service或Manager向外传输的对象。泛指用于展示层与服务层之间的数据传输对象。目的是为了EJB的分布式应用提供粗粒度的数据实体，以减少分布式调用的次数，从而提高分布式调用的性能和降低网络负载。
- BO（Business Object）：业务对象。 由Service层输出的封装业务逻辑的对象。BO 包括了业务逻辑，常常封装了对 DAO、RPC 等的调用，可以进行 PO 与 VO/DTO 之间的转换。BO 通常位于业务层，要区别于直接对外提供服务的服务层：BO 提供了基本业务单元的基本业务操作，在设计上属于被服务层业务流程调用的对象，一个业务流程可能需要调用多个 BO 来完成。
- AO（Application Object）：应用对象。在Web层与Service层之间抽象的复用对象模型，极为贴近展示层，复用度不高。
- VO（View Object）：表示层对象，通常是Web向模板渲染引擎层传输的对象。它的作用是把某个指定页面（或组件）的所有数据封装起来。
- POJO（Plain Ordinary Java Object）：在本手册中，POJO专指只有setter/getter/toString的简单类，包括DO/DTO/BO/VO等。
- Query：数据查询对象，各层接收上层的查询请求。 注意超过2个参数的查询封装，禁止使用Map类来传输。

#### HandlerMapping
HandlerMapping的作用是根据当前请求的找到对应的Handler，并将 Handler（执行程序）与一堆 HandlerInterceptor（拦截器）封装到HandlerExecutionChain对象中。HandlerMapping 是由 DispatcherServlet调用，DispatcherServlet会从容器中取出所有HandlerMapping实例并遍历，让 HandlerMapping实例根据自己实现类的方式去尝试查找Handler。

#### 拦截器
HandlerInterceptor是SpringWebMVC的拦截器，类似于Servlet开发中的过滤器Filter，用于对请求进行拦截和处理。可以应用如下场景：  
- 权限检查，如检测请求是否具有登录权限，如果没有直接返回到登陆页面
- 性能监控，用请求处理前和请求处理后的时间差计算整个请求响应完成所消耗的时间
- 日志记录，可以记录请求信息的日志，以便进行信息监控、信息统计等

## Spring事务
Spring的事务管理不需与任何特定的事务API耦合，并且其提供了两种事务管理方式：编程式事务管理和声明式事务管理。对不同的持久层访问技术，编程式事务提供一致的事务编程风格，通过模板化的操作一致性地管理事务；而声明式事务基于Spring-AOP实现，却并不需要程序开发者成为AOP专家，亦可轻易使用Spring的声明式事务管理。  

#### 编程式事务
Spring事务策略是通过PlatformTransactionManager接口体现的，该接口是Spring事务策略的核心。该接口有如下核心方法：
- getTransaction(TransactionDefinition definition)
- void commit(TransactionStatus status)
- void rollback(TransactionStatus status)
PlatformTransactionManager是一个与任何事务策略分离的接口。PlatformTransactionManager接口有许多不同的实现类，应用程序面向与平台无关的接口编程，而对不同平台的底层支持由PlatformTransactionManager接口的实现类完成，故而应用程序无须与具体的事务API耦合。因此使用PlatformTransactionManager接口，可将代码从具体的事务API中解耦出来。  

而TransactionDefinition接口用于定义一个事务的规则，它包含了事务的一些静态属性，比如：事务传播行为、超时时间等。同时，Spring 还为我们提供了一个默认的实现类：DefaultTransactionDefinition，该类适用于大多数情况。如果该类不能满足需求，可以通过实现TransactionDefinition接口来实现自己的事务定义，TransactionDefinition接口中的核心信息为：  
- int getIsolationLevel();        // 隔离级别
- int getPropagationBehavior();   // 传播行为
- int getTimeout();               // 超时时间
- boolean isReadOnly();           // 是否只读

TransactionStatus接口提供了一个简单的控制事务执行和查询事务状态的方法。该接口的功能如下：  
- boolean isNewTransaction();
- void setRollbackOnly();
- boolean isRollbackOnly();

#### 声明式事务
Spring的声明式事务管理是建立在Spring-AOP机制之上的，其本质是对目标方法前后进行拦截，并在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中作相关的事务规则声明（或通过等价的基于标注的方式），便可以将事务规则应用到业务逻辑中。总的来说，声明式事务得益于Spring-IoC容器和Spring-AOP机制的支持：IoC容器为声明式事务管理提供了基础设施，使得Bean对于Spring框架而言是可管理的；而由于事务管理本身就是一个典型的横切逻辑（正是AOP的用武之地），因此Spring-AOP机制是声明式事务管理的直接实现者。  

除了基于命名空间的事务配置方式，Spring还引入了基于Annotation的方式，具体主要涉及@Transactional标注。@Transactional可以作用于接口、接口方法、类以及类方法上：当作用于类上时，该类的所有public方法将都具有该类型的事务属性；当作用于方法上时，该标注来覆盖类级别的定义。

#### Spring事务传播

- PROPAGATION_REQUIRED：支持当前事务，如果当前没有事务，则新建一个事务执行
- PROPAGATION_SUPPORTS：支持当前事务，如果没有当前事务，则以非事务的方式执行
- PROPAGATION_MANDATORY：支持当前事务，如果当前没有事务，则抛出异常
- PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前已经有事务了，则将当前事务挂起
- PROPAGATION_NOT_SUPPORTED：不支持当前事务，而且总是以非事务方式执行
- PROPAGATION_NEVER：不支持当前事务，如果存在事务，则抛出异常
- PROPAGATION_NESTED：如果当前事务存在，则在嵌套事务中执行，否则行为类似于PROPAGATION_REQUIRED。 EJB中没有类似的功能。

https://www.jianshu.com/p/e1848e2aa7c3

## Spring线程安全

Spring容器本身并没有提供Bean的线程安全策略，即Bean本身不具备线程安全的特性。对于单例Bean，如果该Bean是无状态的Bean，那么不存在多线程问题。对于有状态的Bean，需要使用ThreadLocal解决多线程安全。使用ThreadLocal的好处是使得多线程场景下，多个线程对这个单例Bean的成员变量并不存在资源的竞争，因为ThreadLocal为每个线程保存线程私有的数据。这是一种以空间换时间的方式。当然也可以通过加锁的方法来解决线程安全，这种以时间换空间的场景在高并发场景下显然是不实际的。

#### ThreadLocal和线程同步机制相比有什么优势?
ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。在同步机制中，通过对象的锁机制保证同一时间只有一个线程访问变量。这时该变量是多个线程共享的，使用同步机制要求程序慎密地分析什么时候对变量进行读写，什么时候需要锁定某个对象，什么时候释放对象锁等繁杂的问题，程序设计和编写难度相对较大。而ThreadLocal则从另一个角度来解决多线程的并发访问。ThreadLocal会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进ThreadLocal。

概括起来说，对于多线程资源共享的问题，同步机制采用了“以时间换空间”的方式，而ThreadLocal采用了“以空间换时间”的方式。前者仅提供一份变量，让不同的线程排队访问，而后者为每一个线程都提供了一份变量，因此可以同时访问而互不影响。


## Spring其他

#### Spring中应用到的设计模式
- 简单工厂：Spring中的BeanFactory就是简单工厂模式的体现，根据传入一个唯一的标识来获得Bean对象，但是否是在传入参数后创建还是传入参数前创建这个要根据具体情况来定。
- 工厂方法：Spring中的FactoryBean就是典型的工厂方法模式。
- 单例：Bean的Singleton
- 适配器：Spring中在对于AOP的处理中有Adapter模式的例子
- 包装器：动态地给一个对象添加一些额外的职责。就增加功能来说，Decorator模式相比生成子类更为灵活。Spring中用到的包装器模式在类名上有两种表现：一种是类名中含有Wrapper，另一种是类名中含有Decorator。基本上都是动态地给一个对象添加一些额外的职责。
- 代理：Spring的Proxy模式在aop中有体现，比如JdkDynamicAopProxy和Cglib2AopProxy。
- 观察者：Spring中Observer模式常用的地方是listener的实现。如ApplicationListener。
- 策略：Spring中在实例化对象的时候用到Strategy模式。
- 模板：JdbcTemplate

## Servlet
Servlet，是用Java编写的服务器端程序,其主要功能在于交互式地浏览和修改数据，生成动态Web内容。狭义的Servlet是指Java语言实现的一个接口，广义的Servlet是指任何实现了这个Servlet接口的类。从实现上讲，Servlet可以响应任何类型的请求，但绝大多数情况下Servlet只用来扩展基于HTTP协议的Web服务器。  

#### Servlet的工作流程
当客户端发送HTTP请求后，由Tomcat内核截获。Tomcat内核分析HTTP请求的内容，解析请求的资源并将请求信息封装成一个request对象，此外创建一个response对象。创建一个Servlet对象并调用service方法，例如：  
```java
public void service(request, response) {
    response.getwriter().write("hello");    // write方法将内容写到response缓冲区
}
```
当service方法调用完成后，Tomcat内核从response对象中获取写入的内容，并组装成一个HTTP响应返回给客户端。  
![](https://ws1.sinaimg.cn/large/005L0VzSly1g4elsm48gbj317b0j4tfx.jpg)

#### Servlet与Tomcat的关系
Tomcat是Web应用服务器，是一个Servlet/JSP容器。Tomcat作为Servlet容器，负责处理客户请求，把请求传送给Servlet，并将Servlet的响应传送回给客户。而Servlet是一种运行在支持Java语言的服务器上的组件，用于交互式地浏览和修改数据，生成动态Web内容。

#### Servlet的工作原理
Servlet的接口如下：  
```java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;
    ServletConfig getServletConfig();   // 这个方法会返回由Servlet容器传给init()方法的ServletConfig对象
    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
    String getServletInfo();    // 这个方法会返回Servlet的一段描述，可以返回一段字符串
    void destroy();
    
}
```
Servlet接口定义了Servlet与Servlet容器之间的契约。这个契约是：Servlet容器将Servlet类载入内存，并产生Servlet实例,但是要注意的是，在一个应用程序中，Servlet采用单例模式。用户请求致使Servlet容器调用Servlet的Service()方法，并传入一个ServletRequest对象和一个ServletResponse对象。ServletRequest对象和ServletResponse对象都是由Servlet容器(例如TomCat)封装好的，并不需要程序员去实现，程序员可以直接使用这两个对象。ServletRequest中封装了当前的Http请求，因此，开发人员不必解析和操作原始的Http数据。ServletResponse表示当前用户的Http响应，程序员只需直接操作ServletResponse对象就能把响应轻松的发回给用户。对于每一个应用程序，Servlet容器还会创建一个ServletContext对象。这个对象中封装了上下文（应用程序）的环境详情。每个应用程序只有一个ServletContext。每个Servlet对象也都有一个封装Servlet配置的ServletConfig对象。

#### Servlet的生命周期
在Servlet接口的定义中，init()，service()，destroy()是定义Servlet生命周期的方法。代表了Servlet从“出生”到“工作”再到“死亡”的过程。Servlet容器（例如TomCat）会根据下面的规则来调用这三个方法：  
- 当Servlet第一次被请求时，Servlet容器就会开始调用init方法来初始化一个Servlet对象出来，但是这个方法在后续请求中不会在被Servlet容器调用。调用这个方法时，Servlet容器会传入一个ServletConfig对象进来从而对Servlet对象进行初始化
- 每当请求Servlet时，Servlet容器就会调用service方法，执行主要的业务逻辑
- 当要销毁Servlet时，Servlet容器就会调用destory方法，执行一些后处理逻辑

#### 请求转发和重定向的区别
- 从数据共享来看，请求转发中目标页面和转发到的页面共享request里的数据；redirect不共享任何数据
- 从运用场景来看，请求转发一般用户用户登录，根据角色转发到响应的模块；redirect一般用于用户注销登陆时返回主页面和跳转到其它的网站等
- 从效率上来看，forward高；redirect效率低
- 请求转发是服务器调用不同的资源处理同一请求，始终是同一请求；重定向使得浏览器再次向服务器发送请求，前后是两个不同的请求

#### Servlet中的异步机制
Servlet是单例多线程的机制，因此允许并发访问的线程数目有限。因此Servlet建立了一个线程池，请求必须从线程池中获取了线程才能访问Servlet。若一个请求长时间占有线程，可能导致后面的请求长时间等待，降低了程序的吞吐能力。如果一个线程从Servlet线程池中获取了线程以后，另外开启一个线程处理耗时的任务，及时将主线程归还线程池，就解决这个问题。当异步线程执行完毕后，响应结果。异步机制的作用并非提高请求响应速度，而是增加吞吐量，降低每次访问占用Servlet线程的时间。

#### Servlet中的线程安全问题
Servlet在默认情况下采用单例多线程模式，但是并发环境下多线程访问Servlet的service方法会带来线程安全问题。解决方案如下：  
- doGet方法加synchronized同步
- 静态资源final化
- 全局变量改为局部变量

#### Servlet中为什么重写doGet以及doPost而不是service方法
先看源码：  
```java
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    //判断请求方式
    if(method.equals("GET")) {
    } else if(method.equals("HEAD")) {
        this.doHead(req, resp);
    } else if(method.equals("POST")) {
        this.doPost(req, resp);
    } else if(method.equals("PUT")) {
        this.doPut(req, resp);
    } else if(method.equals("DELETE")) {
        this.doDelete(req, resp);
    } else if(method.equals("OPTIONS")) {
        this.doOptions(req, resp);
    } else if(method.equals("TRACE")) {
        this.doTrace(req, resp);
    } else {
    }
}
public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
    HttpServletRequest request;
    HttpServletResponse response;
    try {
        request = (HttpServletRequest)req;
        response = (HttpServletResponse)res;
    } catch (ClassCastException var6) {
        throw new ServletException("non-HTTP request or response");
    }
    this.service(request, response);
}
```
由源码可知，service方法首先创建request与response对象，然后进一步调用了其他service方法，这个方法仅仅是起到调用转发作用，真正的逻辑实现在doGet、doPost、doPut方法中。所以为了实现业务逻辑，应该重写对应的doGet、doPost方法。如果重写了service方法，那么父类HttpServlet中的service方法就会失效，所收到的任何请求都会由我们自己覆写的service方法来处理。如果同时重写了service和doGet，doPost方法则一定要在执行完自己代码后调用父类service方法，super.service;否则doGet与doPost不会被调用。

否自你的doGet和doPost是不会被执行的

![](https://ws1.sinaimg.cn/large/005L0VzSly1g57ihtcljaj30fk0b074k.jpg)

## 简述Request对象中getAttribute方法与getParameter方法的区别
- 有setAttribute，没有setParameter方法
- getParameter获取到的值只能是字符串，不可以是对象，而getAttribute获取到的值是Object类型的
- 通过form表单或者url来向另一个页面或者servlet传递参数的时候需要用getParameter获取值；getAttribute只能获取setAttribute的值

#### JSP九大内置对象
![](https://ws1.sinaimg.cn/large/005L0VzSly1g4fh1dpocnj30xv0f7gmm.jpg) 

#### 四种会话跟踪技术及其作用域
- page：一个页面
- request：一次请求
- session：一次会话
- application：服务器从启动到停止

## Cookie与Session

#### Cookie
由于HTTP协议是无状态协议，所以服务器单从网络连接上无法判断客户端的身份。为了解决这一问题而使用了Cookie技术。Cookie实际上是一小段的文本信息。客户端请求服务器，如果服务器需要记录该用户状态，就使用response向客户端浏览器颁发一个Cookie。客户端浏览器会把Cookie保存起来。当浏览器再请求该网站时，浏览器把请求的网址连同该Cookie一同提交给服务器。服务器检查该Cookie，以此来辨认用户状态。  

#### Cookie的运行机制
Cookie技术是客户端的解决方案，Cookie就是由服务器发给客户端的特殊信息，而这些信息以文本文件的方式存放在客户端，然后客户端每次向服务器发送请求的时候都会带上这些特殊的信息。具体过程如下：  
- 客户端发送一个http请求到服务器端
- 服务器端发送一个http响应到客户端，其中包含Set-Cookie头部(Cookie信息保存在响应头中)
- 客户端发送一个http请求到服务器端，其中包含Cookie头部
- 服务器端发送一个http响应到客户端

#### Cookie的不可跨域名性
Cookie在客户端是由浏览器来管理的。浏览器能够保证Google只会操作Google的Cookie而不会操作Baidu的Cookie，从而保证用户的隐私安全。浏览器判断一个网站是否能操作另一个网站Cookie的依据是域名。Google与Baidu的域名不一样，因此Google不能操作Baidu的Cookie。

#### Cookie的应用场景
- 记录上次访问时间
- 浏览记录

#### Session
Session是一种记录客户状态的机制，不同于Cookie的是Cookie保存在客户端浏览器中，而Session保存在服务器上。客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录在服务器上。

#### Session的运行机制
Session在服务器端程序运行的过程中创建，当一个用户第一次访问某个网站时会自动创建HttpSession，每个用户可以访问他自己的HttpSession。创建Session的同时，服务器会为该Session生成唯一的session-id，这个session-id在随后的请求中会被用来重新获得已经创建的Session。Session被创建之后，就可以调用Session相关的方法往Session中增加内容了，而这些内容只会保存在服务器中，发到客户端的只有session-id。当客户端再次发送请求的时候，会将这个session-id带上，服务器接受到请求之后就会依据session-id找到相应的Session，从而再次使用Session。  

#### Session的生命周期
Session在用户第一次访问服务器的时候自动创建。需要注意只有访问JSP、Servlet等程序时才会创建Session，只访问HTML、IMAGE等静态资源并不会创建Session。如果尚未生成Session，也可以使用request.getSession(true)强制生成Session。Session生成后，只要用户继续访问，服务器就会更新Session的最后访问时间，并维护该Session。用户每访问服务器一次，无论是否读写Session，服务器都认为该用户的Session"活跃(active)"了一次。而当Session越来越多时，为了防止内存溢出，会使用超时机制将长期不活跃的Session删除，使其失效。

#### Session如何传递其id值
- 保存session id的方式可以采用cookie，这样在交互过程中浏览器可以自动的按照规则把这个标识发送给服务器
- 由于cookie可以被人为的禁止，必须有其它的机制以便在cookie被禁止时仍然能够把session id传递回服务器，经常采用的一种技术叫做URL重写
- 另一种技术叫做表单隐藏字段。就是服务器会自动修改表单，添加一个隐藏字段，以便在表单提交时能够把session id传递回服务器

#### Session与Cookie的区别
- 工作位置：服务端；客户端
- 持久化：存在于Session集群也可持久化到DB中；一般随着浏览器关闭而消亡，也可持久化，但是会带来安全问题
- 安全性：无；Cookie伪造问题
- 对服务器的压力：Session会占用相当多的资源从而影响性能，可考虑使用Session集群；Cookie则较小
- 大小限制：Cookie客户端的限制是4KB；Session无限制
- 数据类型上的支持：Session支持更多数据类型
- 生命周期：浏览区关闭时，Cookie一般会消亡，而Session还保存在服务端

## SpringBoot

SpringBoot是一个轻量级、快速开发框架，整合了常用的第三方依赖整合（原理：通过Maven子父工程的方式），简化XML配置，全部采用注解形式，内置Http服务器（Jetty和Tomcat），最终以java应用程序(Main函数)进行执行。

#### SpringBoot核心特征
- Springboot项目为独立运行的spring项目，java -jar xx.jar即可运行
- 内嵌Servlet容器(可以选择内嵌: tomcat，jetty等服务器)
- 提供了starter的pom配置简化maven的配置
- 自动配置Spring容器中的bean。当不满足实际开发场景，可自定义bean的自动化配置
- 准生产的应用监控(基于:ssh, http, telnet对服务器运行的项目进行监控)
- Springboot无需做出xml配置,也不是通过代码生成来实现(通过条件注解)

#### SpringBoot相比于Spring框架有什么优势
SpringMVC是基于Servlet的一个MVC框架，主要解决WEB开发的问题，因为Spring的配置非常复杂，各种xml，properties处理起来比较繁琐。于是为了简化开发者的使用，Spring社区创造性地推出了SpringBoot，它遵循约定优于配置，极大降低了Spring使用门槛，但又不失Spring原本灵活强大的功能。SpringBoot具有如下优点：简化编码；简化配置；简化部署；简化监控

#### 如何重新加载SpringBoot上的更改，而无需重新启动服务器
使用DEV工具来实现。SpringBoot有一个开发工具（DevTools）模块，它有助于提高开发人员的生产力。Java开发人员面临的一个主要挑战是将文件更改自动部署到服务器并自动重启服务器。开发人员可以重新加载Spring Boot上的更改，而无需重新启动服务器。这将消除每次手动部署更改的需要。

#### SpringBootActuator
SpringBoot Actuator是Spring启动框架中的重要功能之一。SpringBoot监视器可帮助您访问生产环境中正在运行的应用程序的当前状态。有几个指标必须在生产环境中进行检查和监控。即使一些外部应用程序可能正在使用这些服务来向相关人员触发警报消息。监视器模块公开了一组可直接作为HTTP URL访问的REST端点来检查状态。

#### YAML是什么
YAML是一种人类可读的数据序列化语言。它通常用于配置文件。与属性文件相比，如果我们想要在配置文件中添加复杂的属性，YAML文件就更加结构化，而且更少混淆。

#### 如何实现SpringBoot的安全性
为了实现Spring Boot的安全性，我们使用spring-boot-starter-security依赖项，并且必须添加安全配置。它只需要很少的代码。配置类将必须扩展WebSecurityConfigurerAdapter并覆盖其方法。

#### 如何实现SpringBoot分页
使用pagehelper。PageHelper.startPage会拦截下一个sql，也就是pageMapper.getAll()的SQL。并且根据当前数据库的语法，把这个SQL改造成一个高性能的分页SQL，同时还会查询该表的总行数

#### 你对SpringBoot Batch的理解
SpringBoot Batch提供可重用的函数，这些函数在处理大量记录时非常重要，包括日志/跟踪，事务管理，作业处理统计信息。通过优化和分区技术，可以实现大批量和高性能批处理作业。

#### SpringBoot的启动方式
- Main函数
- Maven命令行
- 打包后放到Tomcat中启动

#### SpringBoot支持的日志框架
SpringBoot支持Java Util Logging, Log4j2, Lockback作为日志框架，如果你使用Starters 启动器，SpringBoot将使用 Logback作为默认日志框架。

#### SpringBoot配置加载顺序
properties文件->YAML文件->系统环境变量->命令行参数

#### SpringBoot2的新特性
- 配置变更
- JDK 版本升级
- 第三方类库升级
- 响应式 Spring 编程支持
- HTTP/2 支持
- 配置属性绑定

#### SpringBoot中采用注解配置与XML配置的理解
采用注释配置相比XML配置具有一定的优势，例如：注释配置可以充分利用Java的反射机制获取类结构信息，从而减少配置工作；配置信息伴随Java代码有利于提高程序内聚性，避免开发过程中频繁在程序文件和配置文件间切换。Spring2.5的一大增强就是引入了很多注释类，现在已经可以使用注释配置完成大部分XML配置的功能。虽然采用注解配置便于开发，但是不代表可以完全替代XML配置。原因如下：  
- 采用注释配置不一定先天性由于XML配置，如果Bean的依赖关系是固定的并且不会在部署时再调整，那么注释配置优于XML配置。如果依赖关系在部署时会发生调整，那么采用XML配置较好，因为依赖配置需要动态修改代码并重编译
- 如果Bean不是自己编写的类，那么配置注释将无法部署实施，此时XML将会是唯一的部署方式
- 注释配置往往是类级别的，而XML配置则可以表现得更加灵活。比如相比于@Transaction事务注释，使用aop/tx命名空间的事务配置更加灵活和简单。

#### SpringBoot常见注解

###### @Autowired
Spring2.5引入了@Autowired注释，它可以对类成员变量、方法(setter)及构造函数进行标注，完成自动装配的工作。Spring通过一个BeanPostProcessor对@Autowired进行解析，所以要让@Autowired起作用必须事先在Spring容器中声明AutowiredAnnotationBeanPostProcessor这个Bean。例如：  
```xml
<bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/>
```
当Spring容器启动时，AutowiredAnnotationBeanPostProcessor将扫描Spring容器中所有Bean，当发现Bean中拥有@Autowired注释时就找到和其匹配（默认按类型匹配）的Bean，并注入到对应的地方。当@Autowired作用于setter方法或构造方法上时，@Autowired将与查找方法参数中引用类型匹配的Bean自动注入。值得注意的的是，@Autowired自动注入时，如果找不到匹配的Bean或有多个匹配的Bean时将会抛出异常，解决方法是使用@Autowired(required = false)与使用@Qualifier注释指定注入Bean的名称从而消除歧义，例如：  
```xml
@Autowired
public void setOffice(@Qualifier("office")Office office) {
    this.office = office;
}
```
@Qualifier("office")中的office是Bean的名称，所以@Autowired和@Qualifier结合使用时，自动注入的策略就从byType转变成byName。

###### @Resource
@Resource与@Autowired作用类似，区别在于@Autowired按照byType自动注入，而@Resource默认按照byName自动注入。Spring将@Resource注释的name属性解析为Bean的名字，而type属性则解析为Bean的类型。如果既不指定name也不指定type属性，这时将通过反射机制使用byName自动注入策略。要使@Resource生效还需在Spring容器中注册一个负责处理这些注释的BeanPostProcessor：<bean class="org.springframework.context.annotation.CommonAnnotationBeanPostProcessor"/>  
```java
package com.baobaotao;
import javax.annotation.Resource;
public class Boss {
    @Resource
    private Car car;    // 自动注入类型为 Car 的 Bean
    @Resource(name = "office")
    private Office office;  // 自动注入 bean 名称为 office 的 Bean
}
```

###### @Inject
@Inject等价于默认的@Autowired，只是没有required属性。  


###### @Component
使用@Component可以完全消除配置文件中对Bean的定义，@Component标注于类上并配合@Autowired即可。需要注意的是，要在配置文件中额外配置<context:component-scan/>用于扫描组件

###### @Scope
用于标注类从而指定单例还是原型

###### @SpringBootApplication
@SpringBootApplication注解是SpringBoot的核心注解，实际上由诸多注解组成的组合注解.其中主要的三个注解分别为：@SpringBootConfiguration;@EnableAutoConfiguration;@ComponentScan。

###### @SpringBootConfiguration
@SpringBootConfiguration继承于@Configuration，标注当前类是配置类，并会将当前类内声明的一个或多个以@Bean注解标记的方法的实例纳入到Spring容器中，并且实例名就是方法名。

###### @EnableAutoConfiguration
@EnableAutoConfiguration注解就是SpringBoot能自动进行配置的魔法所在。主要是通过此注解，能把所有符合自动配置条件的bean的定义加载到Spring容器中。

###### @ComponentScan
@ComponentScan注解默认情况下会扫描@SpringBootApplication所在包及其子包下被@Component，@Controller，@Service，@Repository等注解标记的类并纳入到spring容器中进行管理。具体功能如下：  
- 自定扫描路径下边带有@Controller，@Service，@Repository，@Component注解加入Spring容器
- 通过includeFilters加入扫描路径下没有以上注解的类加入spring容器
- 通过excludeFilters过滤出不用加入spring容器的类

###### @Controller
在SpringMVC中，控制器Controller负责处理由DispatcherServlet分发的请求，它把用户请求的数据经过业务处理层处理之后封装成一个Model，然后再把该Model返回给对应的View进行展示。使用@Controller标记一个类是Controller，然后使用@RequestMapping和@RequestParam等一些注解用以定义URL请求和Controller方法之间的映射，这样的Controller就能被外界访问。此外Controller不会直接依赖于HttpServletRequest和HttpServletResponse等HttpServlet对象，它们可以通过Controller的方法参数灵活的获取。 

###### @RestController
@RestController的编写方式依赖注解组合，@RestController被@Controller和@ResponseBody标注，表示@RestController具有两者的注解语义，因此在注解处理时@RestController比@Controller多具有一个@ResponseBody语义，这就是@RestController和@Controller的区别。

###### @RequestMapping
@RequestMapping用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。一个实例如下：  
```java
@RequestMapping(value = "/helloworld", method = {RequestMethod.GET, RequestMethod.POST},
	consumes = "application/json", produces = "application/json", params = "myparam=myvalue",
	headers = "Refer=http://www.fkit.org/")
public String helloworld(Model model) {
	model.addAttribute("message", "Hello World");
	return "welcome";
}
```
- value指定请求路径
- method指定该方法仅处理某些HTTP请求，可同时支持多个HTTP请求
- consumes指定方法处理请求的提交内容类型
- produces指定返回的内容类型，该类型bi必须是request请求头(Accept)中所包含的类型
- params指定request中必须包含某些参数值时，才让该方法处理
- headers指定request中必须包含指定的header值，才让该方法处理

###### @RequestBody
@RequestBody注解允许request的参数在reqeust体中，常常结合前端POST请求，进行前后端交互。

###### @ResponseBody
@ResponseBody是作用在方法上的，@ResponseBody表示该方法的返回结果直接写入HTTP response body中，一般在异步获取数据时使用【也就是AJAX】。在使用@RequestMapping后，返回值通常解析为跳转路径，但是加上 @ResponseBody后返回结果不会被解析为跳转路径，而是直接写入HTTP response body中。比如异步获取json数据，加上 @ResponseBody后，会直接返回json数据。

###### @RequestParam
@RequestParam 用来接收URL中的参数,如/param?username=001,可接收001作为参数，例如：  
```java
@RequestMapping("/users")
@ResponseBody
public String param(@RequestParam String username){
    return "user" + username; 
}
```

###### @RequestHeader
@RequestHeader用于将请求的头信息区数据映射到功能处理方法的参数上。


###### @PostConstruct和@PreDestroy
Spring允许在Bean在初始化完成后以及Bean销毁前执行特定的操作，既可以通过实现InitializingBean/DisposableBean接口来定制初始化之后/销毁之前的操作方法，也可以通过<bean>元素的init-method/destroy-method属性指定初始化之后/销毁之前调用的操作方法。@PostConstruct和@PreDestroy则对应于上述过程。然而不同于上述方式，使用@PostConstruct和@PreDestroy注释却可以指定多个初始化/销毁方法，那些被标注@PostConstruct或@PreDestroy注释的方法都会在初始化/销毁时被执行。

###### @Configuration
@Configuration中所有带@Bean注解的方法都会被动态代理，因此调用该方法返回的都是同一个实例。从定义来看，@Configuration注解本质上还是@Component，因此<context:component-scan/>或者@ComponentScan都能处理@Configuration注解的类。

###### @Service
@Service表示被标注的类是业务层Bean。@Service("userService")注解是告诉Spring，当Spring要创建UserServiceImpl的的实例时，bean的名字必须叫做"userService"。

###### @Repository
@Repository对应数据访问层Bean，@Repository(value="userDao")注解是告诉Spring，让Spring创建一个名字叫“userDao”的UserDaoImpl实例。

###### @CookieValue
@CookieValue用于将请求的Cookie数据映射到功能处理方法的参数上。


## SpringCloud
Spring Cloud为开发人员提供了快速构建分布式系统中一些常见模式的工具（例如配置管理，服务发现，断路器，智能路由，微代理，控制总线）。分布式系统的协调导致了样板模式, 使用Spring Cloud开发人员可以快速地支持实现这些模式的服务和应用程序。他们将在任何分布式环境中运行良好，包括开发人员自己的笔记本电脑，裸机数据中心，以及Cloud Foundry等托管平台。Spring Cloud专注于提供良好的开箱即用经验的典型用例和可扩展性机制覆盖，并具有以下特征：  
- 分布式/版本化配置
- 服务注册和发现
- 路由
- service-to-service调用
- 负载均衡
- 断路器
- 分布式消息传递
