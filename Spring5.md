### IOC Container

#### 1. Spring support two types implementation

   (1)  BeanFactory: Spring内部使用，加载配置文件的时候，不会创建其中对象。在获取对象时才创建。

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// most flexible 
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

   (2)  ApplcationContext: BeanFactory的子接口。加载配置文件时就创建对象。

   (3)  ApplcationContext的实现类 

![image-20210407143553768](https://raw.githubusercontent.com/PeterHanJ/notes/main/spring5/image-20210407143553768.png)



#### 2.IOC操作bean管理（基于xml)
   * 创建对象时，默认执行无参数构造方法创建对象

   * Instantiation with a Static Factory Method

     ```xml
<bean id="clientService"
class="examples.ClientService"
factory-method="createInstance"/>
     ```
     
     ```java
     public class ClientService {
         private static ClientService clientService = new ClientService();
         private ClientService() {}
     
         public static ClientService createInstance() {
             return clientService;
         }
     }
     ```
     
   * Instantiation by Using an Instance Factory Method
     
     ```xml
     <!-- the factory bean, which contains a method called createInstance() -->
     <bean id="serviceLocator" class="examples.DefaultServiceLocator">
         <!-- inject any dependencies required by this locator bean -->
     </bean>
     
     <!-- the bean to be created via the factory bean -->
     <bean id="clientService"
         factory-bean="serviceLocator"
         factory-method="createClientServiceInstance"/>
     ```
     
     ```java
     public class DefaultServiceLocator {
     
         private static ClientService clientService = new ClientServiceImpl();
     
         public ClientService createClientServiceInstance() {
             return clientService;
         }
     }
     ```
     
     > In Spring documentation, “factory bean” refers to a bean that is configured in the Spring container and that creates objects through an [instance](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class-instance-factory-method) or [static](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class-static-factory-method) factory method. By contrast, `FactoryBean` (notice the capitalization) refers to a Spring-specific [`FactoryBean` ](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-factorybean)implementation class.
     
   * DI注入 （set方法注入，有参构造方法注入）

   * p名称空间注入

     ```xml
     <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:p="http://www.springframework.org/schema/p"
            xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
     ```
     
   * 字面量
     * null值
     ```xml
     <property name="age">
              <null/>
     </property>
     ```
     
     * 属性值还有特殊符号（转义或写入CDATA)     
     ```xml
     <property name="address">
          <value><![CDATA[<SG>]]></value>
          <!--<value>&lt;SG&gt;</value>-->
     </property>
     ```
