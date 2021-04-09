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
   * 创建对象时，默认执行无参数构造方法创建对象。否则声明argument来匹配构造方法。如果参数的类型不同，可以很明确的区分的时候，不需要设定类型或者index。否则，可以通过类型或者index来匹配构造方法。

     ```xml
     <!--用类型来匹配-->
     <bean id="exampleBean" class="examples.ExampleBean">
         <constructor-arg type="int" value="7500000"/>
         <constructor-arg type="java.lang.String" value="42"/>
     </bean>
     
     <!--用index来匹配-->
     <bean id="exampleBean" class="examples.ExampleBean">
         <constructor-arg index="0" value="7500000"/>
         <constructor-arg index="1" value="42"/>
     </bean>
     
     <!--用参数名来匹配-->
     <bean id="exampleBean" class="examples.ExampleBean">
         <constructor-arg name="years" value="7500000"/>
         <constructor-arg name="ultimateAnswer" value="42"/>
     </bean>
     ```

     > Keep in mind that, to make this work out of the box, your code must be compiled with the debug flag enabled so that Spring can look up the parameter name from the constructor. If you cannot or do not want to compile your code with the debug flag, you can use the [@ConstructorProperties](https://download.oracle.com/javase/8/docs/api/java/beans/ConstructorProperties.html) JDK annotation to explicitly name your constructor arguments. The sample class would then have to look as follows:

     ```java
     package examples;
     
     public class ExampleBean {
     
         // Fields omitted
     
         @ConstructorProperties({"years", "ultimateAnswer"})
         public ExampleBean(int years, String ultimateAnswer) {
             this.years = years;
             this.ultimateAnswer = ultimateAnswer;
         }
     }
     ```

     

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

     关于循坏依赖(Circular dependencies)，spring官方文档说明如果是两个类的构造方法中存在循坏依赖，会导致BeanCurrentlyInCreationException。建议是将其中一些class改为setters注入。

     > One possible solution is to edit the source code of some classes to be configured by setters rather than constructors. Alternatively, avoid constructor injection and use setter injection only. In other words, although it is not recommended, you can configure circular dependencies with setter injection.
     >
     > Unlike the typical case (with no circular dependencies), a circular dependency between bean A and bean B forces one of the beans to be injected into the other prior to being fully initialized itself (a classic chicken-and-egg scenario).

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

   * Configure a `java.util.Properties` instance

     The Spring container converts the text inside the `<value/>` element into a `java.util.Properties` instance by using the JavaBeans `PropertyEditor` mechanism.

      ```xml
      <bean id="mappings"
      class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
        
        <!-- typed as a java.util.Properties -->
        <property name="properties">
            <value>
                jdbc.driver.className=com.mysql.jdbc.Driver
                jdbc.url=jdbc:mysql://localhost:3306/mydb
            </value>
        </property>
      </bean>
      ```
     
   * The `idref` element
     
     `idref`和`ref`的区别：`idref`传递的是一个string value,和`value`类似。只是这个字符串必须是容器中存在的bean的id。而`ref`传递的是一个对象的应用`reference`
     
     
     
     The `idref` element is simply an error-proof way to pass the `id` (**`a string value - not a reference`**) of another bean in the container to a `<constructor-arg/>` or `<property/>` element. The following example shows how to use it:
     
     ```
     <bean id="theTargetBean" class="..."/>
     
     <bean id="theClientBean" class="...">
         <property name="targetName">
             <idref bean="theTargetBean"/>
         </property>
     </bean>
     ```
     
     The preceding bean definition snippet is exactly equivalent (at runtime) to the following snippet:
     
     ```
     <bean id="theTargetBean" class="..." />
     
     <bean id="client" class="...">
         <property name="targetName" value="theTargetBean"/>
     </bean>
     ```
     
     > The first form is preferable to the second, because using the `idref` tag lets the container validate at deployment time that the referenced, named bean actually exists. In the second variation, no validation is performed on the value that is passed to the `targetName` property of the `client` bean. Typos are only discovered (with most likely fatal results) when the `client` bean is actually instantiated. If the `client` bean is a [prototype](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes) bean, this typo and the resulting exception may only be discovered long after the container is deployed.

   * Collections and Collection Merging

     ```xml
     <bean id="moreComplexObject" class="example.ComplexObject">
         <!-- results in a setAdminEmails(java.util.Properties) call -->
         <property name="adminEmails">
             <props>
                 <prop key="administrator">administrator@example.org</prop>
                 <prop key="support">support@example.org</prop>
                 <prop key="development">development@example.org</prop>
             </props>
         </property>
         <!-- results in a setSomeList(java.util.List) call -->
         <property name="someList">
             <list>
                 <value>a list element followed by a reference</value>
                 <ref bean="myDataSource" />
             </list>
         </property>
         <!-- results in a setSomeMap(java.util.Map) call -->
         <property name="someMap">
             <map>
                 <entry key="an entry" value="just some string"/>
                 <entry key ="a ref" value-ref="myDataSource"/>
             </map>
         </property>
         <!-- results in a setSomeSet(java.util.Set) call -->
         <property name="someSet">
             <set>
                 <value>just some string</value>
                 <ref bean="myDataSource" />
             </set>
         </property>
     </bean>
     ```

     The value of a map key or value, or a set value, can also be any of the following elements:

     ```
     bean | ref | idref | list | set | map | props | value | null
     ```

     The following example demonstrates collection merging:

     ```xml
     <beans>
         <bean id="parent" abstract="true" class="example.ComplexObject">
             <property name="adminEmails">
                 <props>
                     <prop key="administrator">administrator@example.com</prop>
                     <prop key="support">support@example.com</prop>
                 </props>
             </property>
         </bean>
         <bean id="child" parent="parent">
             <property name="adminEmails">
                 <!-- the merge is specified on the child collection definition -->
                 <props merge="true">
                     <prop key="sales">sales@example.com</prop>
                     <prop key="support">support@example.co.uk</prop>
                 </props>
             </property>
         </bean>
     <beans>
     ```

     

