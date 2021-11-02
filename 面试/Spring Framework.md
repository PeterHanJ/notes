## What is IOC

In [software engineering](https://en.wikipedia.org/wiki/Software_engineering), **inversion of control** (**IoC**) is a programming principle. IoC inverts the [flow of control](https://en.wikipedia.org/wiki/Control_flow) as compared to traditional control flow. In IoC, custom-written portions of a [computer program](https://en.wikipedia.org/wiki/Computer_program) receive the flow of control from a generic [framework](https://en.wikipedia.org/wiki/Software_framework). A [software architecture](https://en.wikipedia.org/wiki/Software_architecture) with this design inverts control as compared to traditional [procedural programming](https://en.wikipedia.org/wiki/Procedural_programming): in traditional programming, the custom code that expresses the purpose of the program [calls](https://en.wikipedia.org/wiki/Function_call#Main_concepts) into reusable libraries to take care of generic tasks, but with inversion of control, it is the framework that calls into the custom, or task-specific, code.

We can achieve IOC with 

- Dependency Lookup

  **Advantage**: Can control the exactly resource and the timing when needed.

  **Disadvantage**: Most rely the framework or container API. So you need have these extra lookup code in your business logic. 

  The way servlet / ejb component gets jdbc data source object form registry through jndi lookup operation is called dependency lookup

- Dependency Injection

  Container/framework will assign the dependency when it ready.

  **Advantage**: resource need not spend time to get dependent value and it can use dependent value at the moment resource is ready

  **Disadvantage**: both necessary and unnecessary values will be injected

  1. by constructor

  2. by setter

     

Hollywood Principle states, "Don't Call Us, We'll Call You." It's closely related to the [Dependency Inversion Principle](https://deviq.com/principles/dependency-inversion-principle)

The advantages of this architecture are:

- decoupling the execution of a task from its implementation
- making it easier to switch between different implementations
- greater modularity of a program
- greater ease in testing a program by isolating a component or mocking its dependencies, and allowing components to communicate through contracts

We can achieve Inversion of Control through various mechanisms such as: Strategy design pattern, Service Locator pattern, Factory pattern, and Dependency Injection (DI).

## ObjectFactory, FactoryBean, BeanFactory?

BeanFactory is a IOC container

FactoryBean is a bean to help create bean, especially for creation a complicate bean. 

```java
If a bean implements this
interface, it is used as a factory for an object to expose, not directly as a
bean instance that will be exposed itself.
    
FactoryBean objects participate in the containing BeanFactory's
synchronization of bean creation. There is usually no need for internal
synchronization other than for purposes of lazy initialization within the
FactoryBean itself (or the like).
```



## Bean initialization/destroy order?

Initialization (都是无参的)

1. @PostCounstructor  
2. InitializingBean interface， afterPropertiesSet()
3. init method

- xml <bean init-method="init"/>
- Java Annotation @bean(initMethod="init")
- JAVA API AbstractBeanDefinition#setInitMethodName(String)

Destroy (Perform when context close)

1. @PreDestroy
2. DisposableBean interface, destroy()
3. destroy method
   - xml <bean destroy="destroy()"/>
   - Java Annotation @bean(destroy="destroy")
   - Java API AbstractionBeanDefinition#setDestroyMethodName(String)



## Different by setter injection and constructor injection

```
// 依赖查找并且创建 Bean
// 构造函数注入
// 1. 只有无参构造函数，当然不会有任何注入
// 2. 有参构造函数，spring会选择最大匹配注入的构造函数,如果匹配参数个数一样，用第一个匹配到的构造函数注入
// 3. 匹配按类型->名称 进行查找
// 4. 构造器注入是按参数顺序来的，必须依次找到对应的bean,否则就不会用这个构造器注入。 setter的注入顺序却并不确定
```



## @Autowired and @Resource

会忽略掉静态字段