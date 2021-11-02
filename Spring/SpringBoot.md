## WebMVCAutoConfiguration

> Spring Boot provides auto-configuration for Spring MVC that works well with most applications.
>
> The auto-configuration adds the following features on top of Spring’s defaults:
>
> - Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.
> - Support for serving static resources, including support for WebJars (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.static-content)).
> - Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans.
> - Support for `HttpMessageConverters` (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.message-converters)).
> - Automatic registration of `MessageCodesResolver` (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.message-codes)).
> - Static `index.html` support.
> - Automatic use of a `ConfigurableWebBindingInitializer` bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-web-applications.spring-mvc.binding-initializer)).
>
> If you want to keep those Spring Boot MVC customizations and make more [MVC customizations](https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`.
>
> If you want to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, and still keep the Spring Boot MVC customizations, you can declare a bean of type `WebMvcRegistrations` and use it to provide custom instances of those components.
>
> If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`, or alternatively add your own `@Configuration`-annotated `DelegatingWebMvcConfiguration` as described in the Javadoc of `@EnableWebMvc`.



```java
@Configuration(proxyBeanMethods = false)   //Lite 模式, configure中调用@bean方法不会经过增强代理。直接返回新对象。
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
```

```java
	@Bean
	@ConditionalOnMissingBean(HiddenHttpMethodFilter.class)
	@ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled") //默认关闭，因为页面表单只支持post,所以这里用于让表单提交支持restful.如果直接发送put,delete请求，则这个不会影响到。
	public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
		return new OrderedHiddenHttpMethodFilter();
	}
```

## 开启矩阵变量

```java
//springboot默认不支持，;在url中会被移除
WebAutoConfiguration
      @Configuration(proxyBeanMethods = false)
      @Import(EnableWebMvcConfiguration.class)
	 @EnableConfigurationProperties({ WebMvcProperties.class,
			org.springframework.boot.autoconfigure.web.ResourceProperties.class, WebProperties.class })
	  @Order(0)
	  public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer, ServletContextAware {
    	 configurePathMatch //这个方法中的 urlPathHelper里面设定  removeSemicolonContent = true
              UrlPathHelper urlPathHelper = new UrlPathHelper();
                removeSemicolonContent = true;   //这里会把;移除。 注意，jsessionid一定会被移除
                     /**
                         * Remove ";" (semicolon) content from the given request URI if the
                         * {@linkplain #setRemoveSemicolonContent removeSemicolonContent}
                         * property is set to "true". Note that "jsessionid" is always removed.
                         * @param requestUri the request URI string to remove ";" content from
                         * @return the updated URI string
                         */
                        public String removeSemicolonContent(String requestUri) {
                            return (this.removeSemicolonContent ?
                                    removeSemicolonContentInternal(requestUri) : removeJsessionid(requestUri));
                        }

//要开启，需要自定义一个 WebMvcConfigurer，重写configurePathMatch,设定urlPathHelper.removeSemicolonContent 为false
    @Bean
    public WebMvcConfigurer getWebMvcConfigurer(){
        return new WebMvcConfigurer() {
            @Override
            public void configurePathMatch(PathMatchConfigurer configurer) {
                UrlPathHelper helper = new UrlPathHelper();
                helper.setRemoveSemicolonContent(false);
                configurer.setUrlPathHelper(helper);
            }
        };
    }
          
          
  //controller @MatrixVariable应用
  @GetMapping("/matrix/{p1}/{p2}")
    public Map getMatrixParam(@MatrixVariable(value = "name",pathVar = "p1") String name1,
                              @MatrixVariable(value = "name",pathVar = "p2") String name2,
                              @MatrixVariable(value = "interest", pathVar="p1") List<String> interest1,
                              @MatrixVariable(value = "interest", pathVar="p2") List<String> interest2,
                              @PathVariable("p1") String path1,
                              @PathVariable("p2") String path2) {

        Map<String,Object> map = new HashMap<>();
        map.put("p1", name1);
        map.put("p2", name2);
        map.put("interest1", interest1);
        map.put("interest2", interest2);
        map.put("path1", path1);
        map.put("path2", path2);
        return map;
    }         
```

## 自定义参数绑定

```java
```





Spring Boot 1.4之后取消了 ConfigurationProperties 的 locations 属性，无法指定属性资源的位置。


两种替代方案
第一种：
使用 @Component 注册为组件，然后使用 @PropertySource 指定资源位置。

@Component
@ConfigurationProperties(prefix = "book")
@PropertySource(value={"classpath:book.properties"})
public class Book {}
1
2
3
4
缺点：只能使用 application.properties，不能使用 application.yml

第二种：
新建 application-book.yml 文件，然后在 application.yml 中开启该属性文件

spring:
  profiles:
    active: book
1
2
3
***缺点：要按 SpringBoot 的规定的格式配置，即 application-XXX.yml ***