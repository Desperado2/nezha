### 一.文件上传
#### 1.文件上传
- SpringMVC为文件上传提供了直接的支持，这种类型是通过即插即用的MultipartResolver技术的。Spring用Jakarta Commons FileUpload技术实现了一个MultipartResolver实现类:CommonsMultipartResolver.    
- SpringMVC上下文中默认没有装配MultipartResolver，因此默认情况下不能处理文件的上传工作，如果想使用Spring的文件上传功能，需现在上下文中配置MultipartResolver。   
#### 2. 配置MultipartResolver
- defaultEncoding：必须和用户JSP的pageEncoing属性一致，以便正确解析表单的内容。   
- 为了让CommonsMulitpartResolver正确工作，必须先将Jakarta Commons FileUpload及Jakarta Commons io的类包添加到类路径下。   

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <propert name="defaultEncoding" value="UTF-8"></property>
    <property name="maxUploadSize" value="524880"></property>
</bean>
```

### 二.拦截器
#### 1.自定义拦截器
SpringMVC也可以使用拦截器对请求进行拦截处理，用户可以自定义拦截器来实现特定的功能，自定义的拦截器必须实现HandlerInterceptor接口。接口中有三个方法：     

  - preHandle():这个方法在业务处理器请求之前被调用，在该方法中对用户请求request进行处理。如果你决定该拦截器对请求进行拦截处理好还要调用其他的拦截器，或者是业务处理器去进行处理，则返回true；如果程序员决定不需要再调用其他的组件去处理请求，则返回false。    

  - postHandle():这个方法在业务处理器处理完请求后，但是DispatcherServlet向客户端返回响应前被调用，在该方法中对用户请求request()进行处理。

  - afterCompletion()：这个方法在DispatcherServlet完全处理完请求后被调用，可以在该方法中进行一些资源清理的操作。

#### 2.拦截器方法执行顺序

![image.png](/image/spring/5-1.png)

#### 3.配置自定义拦截器

```xml
<mvc:interceptors>
      <!--  拦截所有资源 -->
      <bean class="com.desperado.interceptors.FirstInterceptor"></bean>
      <!--拦截指定资源-->
      <mvc:interceptor>
              <mvc:mapping path="/emps"/>
              <bean class="com.desperado.interceptors.SecondInterceptor"></bean>
      </mvc:interceptor>
      <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"></bean>
</mvc:interceptors>
```

### 4.多个拦截器的执行顺序

![image.png](/image/spring/5-2.png)

#### 5.拦截器preHandle()方法返回false时候的执行顺序

![image.png](/image/spring/5-3.png)

#### 6.一道经典面试题
拦截器和过滤器有什么区别：     
1. 拦截器是基于java的反射机制的，而过滤器是基于函数回调的。     
2. 拦截器不依赖于Servlet容器，而过滤器依赖与Servlet容器。    
3. 拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求其作用。   
4. 拦截器可以访问action的上下文、值栈里面的对象，而过滤器不能访问。     
5. 在action的声明周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次、(这里的调用一次，是对于构造函数而言的，而doFilter会对匹配的请求做持续的处理。)   
6. 拦截器可以获取IOC容器中的Bean，而过滤器不行。    

执行顺序

![image.png](/image/spring/5-4.png)

### 三.异常处理
#### 1.异常处理
- Spring MVC通过HandlerExceptionResolver处理程序的异常，包括Handler映射、数据绑定以及目标方法执行时发生的异常。   
- SpringMVC提供的HandlerExceptionResolver的实现类  
      - AbstractHandlerExceptionResolver      
      - AbstractHandlerMethodExceptionResolver
      - ExceptionHandlerExceptionResolver
      - AnnotationMethodHandlerExceptionResolver
      - DefaultHandlerExceptionResolver
      - ResponseStatusExceptionResolver
      - SimpleMappingExceptionResolver
      - HandlerExceptionResolverComposite

#### 2.HandlerExceptionResolver
- DispatcherServlet默认装配的HandlerExceptionResolver：     
    - 没有使用<mvc:annotation-driver />
         -AnnotationMethodHandlerExceptionResolver
         -DefaultHandlerExceptionResolver
         -ResponseStatusExceptionResolver

    - 使用了<mvc:annotation-driver />
        -ExceptionHandlerExceptionResolver
        -DefaultHandlerExceptionResolver
        -ResponseStatusExceptionResolver

#### 3.ExceptionHandlerExceptionResolver
- 主要处理Handler中用@ExceptionHandler注解定义的方法。    
- @ExceptionHandler注解定义的优先级问题:如果发生的是NullPointerException，但是声明的异常有RuntimeException和Exception，此时会根据异常的最近继承关系找到继承深度最浅的那个@ExceptionHandler注解方法，即标记了RuntimException的方法。   
- ExceptionHandlerMethodResolver内部若找不到@ExceptionHandler注解的话，会找@ControllerAdvice中的@ExceptionHandler注解方法

#### 4.ResponseStatusExceptionResolver
- 在异常及异常父类中找到@ResponseStatus注解，然后使用这个注解的属性进行处理。  
- 定义一个@ResponseStatus注解修饰的异常类。   
 ```java
@ResponseStatus(HttpStatus.UNAUTHORIZED)
public class UnAuthorizedException extends RuntimeException{}
 ```

- 若在处理器方法中抛出了上面定义的异常：     
由于触发的异常带有@ResponseStatus注解，因此会被ResponseStatusExceptionResolver解析到。最后响应HttpStatus.UNAUTHORIZED代码给客户端。      

#### 5.DefaultHandlerExceptionResolver对一些特殊的异常进行处理，比如：    
NoSuchRequestHandlingMethodException      
HttpRequestMethodNotSupportedException      
HttpMediaTypeNotSupportedException       
HttpMediaTypeNotAcceptableException
等。     

#### 6.SimpleMappingExceptionResolver
如果希望对所有异常进行统一处理，可以使用该解析器，它将异常类名映射为视图名，即发生异常是使用对应的视图报告异常

### 四.Spring的运行流程
1. 请求到达Spring DispatcherServlet的url-pattern。    
2. 判断SpringMVC中是否存在对应的映射    
3. 如果不存在判断是否配置了<mvc:default-servlet-handler />
4. 如果配置了就去找目标资源，如果没有配置，返回404页面。   
5. 如果存在则从HandlerMapping获取handlerExecutionChain对象。   
6. 获取HandlerAdapter对象。   
7. 调用拦截器的preHandle方法。   
8. 调用目标Handler的目标方法得到ModelAndView对象。   
9. 调用拦截器的postHandle方法。   
10. 判断是否存在异常
11. 存在异常，由HandlerExceptionResolver组件处理异常得到新的ModelAndView对象。   
12. 不存在异常，由ViewResolver组件根据ModelAndView对象得到实际的View
13. 渲染视图    
14. 调用拦截器的afterCompletion方法。   

![image.png](/image/spring/5-5.png)

### 五.在Spring的环境下使用SpringMVC
#### 1.Bean被创建两次?
Spring的IOC容器不应该扫描SpringMVC中的Bean，对应的SpringMVC的IOC容器不应该扫描Spring中的Bean。    

#### 2.在SpringMVC配置文件中引用业务层的Bean
- 多个SpringIOC容器之间可以设置为父子关系，以实现良好的解耦。
- SpringMVC WEB层容器可作为"业务层"Spring容器的子容器，即WEB层容器可以引用业务层容器的Bean，而业务层容器却访问不到WEB层容器的Bean。 

![image.png](/image/spring/5-6.png)

### 六、Spring和Struts2对比
1. SpringMVC的入口是Servlet，而Struts2是Filter.    
2. SpringMVC会比Struts2快点，SpringMVC是基于方法设计的，而Struts2是基于类，每次发一个请求都会实实例一个Action
3. SpringMVC使用更加简洁，开发效率高。   
4. Struts2的OGNL表达式使页面开发效率相比SpringMVC要高一点。