### IOC

#### 1. Spring support two types implementation

   (1)  BeanFactory: Spring内部使用，加载配置文件的时候，不会创建其中对象。在获取对象时才创建。

   (2)  ApplcationContext: BeanFactory的子接口。加载配置文件时就创建对象。

   (3)  ApplcationContext的实现类 

![image-20210407143553768](https://raw.githubusercontent.com/PeterHanJ/notes/main/spring5/image-20210407143553768.png)



#### 2.IOC操作bean管理（基于xml)
   * 创建对象时，默认执行无参数构造方法创建对象

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
