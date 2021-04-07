### IOC

#### 1. Spring support two types implementation

 (1) BeanFactory: Spring内部使用，加载配置文件的时候，不会创建其中对象。在获取对象时才创建。

 (2) ApplcationContext: BeanFactory的子接口。加载配置文件时就创建对象。

 (3) ApplcationContext的实现类   ![image-20210407143553768](spring5\image-20210407143553768.png)



