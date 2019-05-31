### 一.SpringMVC表单标签He处理静态资源
#### 1.Spring的表单标签
通过SpringMVC的表单标签可以实现将模型数据中的属性和HTML表单元素相绑定，以实现表单数据更便捷编辑和表单值的回显。    

##### 1.form标签
- 一般情况下，通过GET请求获取表单页面，而通过POST请求提交表单页面，因此获取表单页面和提交表单页面的URL是相同的。只要满足该最佳条件的契约，<form:form>标签就无需通过action属性指定表单提交的URL。   
- 可以通过modelAttribute属性指定绑定的模型属性，若没有指定该属性，则默认从request域对象中获取command的表单bean。如果该属性值也不存在，则会发生错误。 
- SpringMVC提供了多个表单组件标签，如\<form:input/>、\<form:select/>等，用以 绑定表单字段的属性值，它们的共有属性如下：
  - path：表单字段，对应html元素的name属性，支持级联属性。  
  - htmlEscape：是否对表单值的HTML特殊字符进行转换，默认值为true。    
  - CSSClass：表单组件对应的CSS样式类名。    
  - cssErrorClass：表单组件的数据存在错误时，采取的CSS样式。
  - form:input、form:password、form:hidden、form:textarea：对应HTML表单的text、password、hidden、textarea标签。    
  - from:radiobutton：单选框组件标签，当表单bean对应的属性值和value值相等时，单选框被选中。   
  - form:radiobuttons：单选框组标签，用于构造多个单选框。    
      - items:可以是一个List、String[]或Map。   
      - itemValue：指定radio的value值。可以是集合中bean的一个属性值。   
      - itemLabel：指定radio的label值。  
      - delimiter：多个单选框可以通过delimiter指定分隔符。
  - form:checkbox：复选框组件，用于构造单个复选框。   
  - form:checkboxs：用于构造多个复选框。  
  - form:errors：显示表单组件或数据校验所对应的错误

#### 2.处理静态资源
- 优雅的Rest风格的资源URL不希望带.html或.do等后缀。      
- 若将DispatcherServlet请求映射配置为  /,则SpringMVC将捕获WEB容器的所有请求，包括静态资源的请求，SpringMVC会将它们当初一个普通请求处理，因找不到对应处理器将导致错误。   
- 可以在SpringMVC的配置文件中配置<mvc:default-servlet-handler />的方式解决静态资源的问题：   
    - <mvc:default-servlet-handler />将在SpringMVC上下文中定义一个DefaultServletHttpRequestHandler，它会对进入DispatcherServlet的请求进行筛查，如果发现是没有经过映射的请求，就将该请求交由WEB应用服务器默认的Servlet处理。如果不是静态资源的请求，才由DispatcherServlet继续处理。    
    - 一般WEB应用服务器默认的Servlet的名称都是default。若所使用的WEB服务器的默认Servlet名称不对default，则需要通过default-servlet-name属性显示指定。

### 二.数据转换
#### 1.数据绑定流程
  1. SpringMVC主框架将ServletRequest对象及目标方法的入参实例传递给WebDataBinderFactory实例，以创建DataBinder实例对象。     
   2. DataBinder调用装配在SpringMVC上下文的ConversionService组件进行数据类型转换、数据格式化工作。将Servlet中的请求信息填充到入参对象中。   
   3. 进行Validator组件对已经绑定了请求消息的入参对象进行数据合法性校验，并最终生成数据绑定结果BindingData对象。   
   4. SpringMVC抽取BindingResult中的入参对象和校验错误对象，将它们赋给处理方法的响应入参。
   5. SpringMVC通过反射机制对目标处理方法进行解析，将请求消息绑定到处理方法的入参中，数据绑定的核心部件是DataBinder。   

![image.png](/image/spring/4-1.png)

 #### 2.数据转换
- springMVC上下文内建了很多转换器，可完成大多数Java类型的转换工作。     
- SpringMVC自带的转换器都在org.springframework.core.convert.support路径下面。   

#### 3.自定义类型转换器
- ConversionService是Spring类型转换体系的核心接口。   
- 可以利用ConversionServiceFactoryBean在Spring的IOC容器中定义一个ConversionService。Spring将自动识别出IOC容器中的ConversionService，并在Bean属性配置及SpringMVC处理方法入参绑定等场合使用它进行数据的转换。   
- 可以通过ConversionServiceFactoryBean的converters属性注册自定义的类型转换器。   

```xml
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <list>
                <bean class="com.desperado.UserConverter"></bean>
            </list>
        </property>
    </bean>
```

#### 4.Spring支持的转换器
- Spring定义了3种类型的转换器接口，实现任意一个转换器接口都可以作为自定义转换器注册到ConversionServiceFactoryBean中：     
  - Converter<S,T>：将S类型对象转为T类型对象。   
  - ConverterFactory：将相同系列多个“同质”Converter封装在一起。如果希望将一种类型的对象转换为另一种类型及其子类的休息可使用该转换器工厂类。 
  - GenericConverter：会根据源类对象及目标类对象所在的宿主类中的上下文信息进行类型转换。   
- <mvc:annotation-driver conversion-service="conversionService" />会将自定义的ConversionService注册到SpringMVC上下文中。

```xml    
<mvc:annotation-driver conversion-service="conversionService" />

<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <list>
                <bean class="com.desperado.UserConverter"></bean>
            </list>
        </property>
    </bean>
```
#### 5.关于mvc:annotation-driver
- <mvc:annotation-driver />会自动注册RequestMappingHandlerMapping、RequestMappingHandlerAdapter与ExceptionHandlerExceptionResolver这三个Bean。   
- 还提供一下支持：  
    - 支持使用ConversionService实例对表单参数进行类型转换。  
    - 支持完成@NumberFormat、@DateTimeFormat注解完成数据类型的格式化。   
    - 支持使用@Valid注解对JavaBean实例进行JSR303验证。   
    - 支持使用@RequestBody和@ResponseBody注解。     

#### 6.@InitBinder
- 由@InitBinder标识的方法，可以对WebDataBinder对象进行初始化。WebDataBinder是DataBinder的子类，用于完成由表单字段到JavaBean属性的绑定。   
- @InitBinder方法不能有返回值，它必须声明为void。   
- @InitBinder方法的参数通常是WebDataBinder。   

### 三.数据格式化
#### 1.数据格式化
- 对属性对象的输入/输出进行格式化。   
- Spring在格式化模块定义了一个实现ConversionService接口的FormattingConversionService实现类，因此它既具有类型转换的功能，又拥有格式化的功能。    
- FormattingConversionService拥有一个FormattingConversionServiceFactoryBean工厂类，后者用于在Spring上下文构造前者。
- FormattingConversionServiceFactoryBean内部注册了：  
    - NumberFormatAnnotationFormatterFactory：支持对数字类型的属性使用@NumberFormat注册。   
    - JodaDataTimeFormatAnnotationFormatterFactory：支持对日期类型的属性使用@DateTimeFormat注解。
- 装配了FormattigConversionServiceFactoryBean后，就可以在SpringMVC入参绑定及模型数据输出时使用注解驱动了。
- <mvn:annotation-driver />默认创建的ConversionService实例为FormattingConversionServiceFactoryBean。

#### 2.日期转换器
@DateTimeFormat注解可对java.util.Date
java.util.Calendar、java.long.Long时间类型进行标注。     
  - pattern属性：类型为字符串。指定解析/格式化字段数据的模式，如“yyyy-MM-dd hh:mm:ss”  
  - iso属性：类型为DateTimeFormat.ISO。指定解析/格式化字段数据的ISO模式，包括四种模式:ISO.NONE(默认，不使用)，ISO.DATE(yyyy-MM-dd)、ISO.TIME(hh:mm:ss.SSSZ)、ISO.DATE_TIME(yyyy-MM-dd hh:mm:ss.SSSZ)
  - style属性：字符串类中。通过样式指定日期时间的格式，由两位字符组成，第一位表示日期的格式，第二位表示时间的格式：S:短日期/时间格式、M:中日期/时间格式、L:长日期/时间格式、F:完整日期/时间格式、-:忽略日期或事件格式。

#### 3.数组格式化
@NumberFormat可对类似数字类型的属性进行标注，它拥有两个互斥的属性：    
   - style：类型为NumberFormat.Style。用于指定样式类型，包括三种:Style.NUMBER(正常数字类型)、Style.CURRENCY(货币类型)、Style.PERCENT(百分比类型)    
   - pattern：类型为String，自定义样式，如pattern="#,###"

### 四.数据校验
#### 1.JSR 303
- JSR303是Java为Bean数据合法性校验提供的标准框架，它已经包含在JavaEE6.0中。   
- JSR303通过在Bean属性上标注类似于@NotNull、@Max等标准的注解指定校验标准，并通过标准的验证接口对Bean进行验证。  

![image.png](/image/spring/4-2.png)

#### 2.SpringMVC数据校验
- Spring4.0拥有自己独立的数据校验框架，同时支持JSR303标准的校验框架。   
- Spring在进行数据绑定时，可同时调用校验框架完成数据校验工作。在SpringMVC中，可直接通过注解驱动的方式进行数据校验。    
- Spring的LocalValidatorFactoryBean，即可将其注入到需要数据校验的Bean中，   
- Spring本身并没有提供JSR303的实现，索引必须将JSR303的实现者的jar包放到类路径下。   
- <mvc:annotation-driver />会默认装配好一个LocalValidatorFactoryBean，通过在处理方法的入参上标注@Valid注解即可让SpringMVC在完成数据绑定后执行数据校验的工作。   
- SpringMVC是通过对处理方法签名的规约来保持校验结果的：前一个表单/命令对象的校验结果保存到税后的入参中，这个保存校验结果的入参必须是BindingResult或者Errors类型。

#### 3.提示消息的国际化
- 每个属性在数据绑定和数据校验发送错误时，都会生成一个对应的FieldError对象。    
- 当一个属性检验失败后，校验框架会为该属性生成4个消息代码，这些代码以校验注解类名为前缀，结婚ModelAttribute、属性名及属性类型名生成多个对应的消息代码。    
- 当时用SpringMVC标签显示错误消息时，SpringMVC会查看WEB上下文是否装配了对应的国际化消息，如果没有，则显示默认的错误消息，否则使用国际化消息。
- 若数据类型转换或数据格式转换时发生错误，或该有的参数不存在，或调用处理方法时发生错误，都会在隐含模型中创建错误消息。其错误代码前缀说明如下：   
    -required：必要的参数不存在。    
    -typeMismatch：在数据绑定时，发生数据类型不匹配的问题。    
    -methodInvocation：SpringMVC在调用处理方法时发生了错误。   

- 注册国际化资源文件
```xml
<bean id="messageSource" 
          class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="i18n"></property>
    </bean>
```

### 五.处理JSON
- HttpMessageConverter<T>是Spring3.0新添加的一个接口，负责将请求信息转换为一个对象(类型为T)，将对象(类型为T)输出为响应信息。    
- HttpMessageConverter<T>接口定义的方法：
    -Boolean canRead(Class<?> clazz,MediaType mediaType):指定转换器可以读取的对象类型，即转换器是否可将请求信息转换为clazz类型的对象，同时指定支持MIME类型。  
      -Boolean canWrite(Class<?> clazz,MediaType mediaType):指定转换器是否可以clazz类型的对象写到响应流中，响应流支持的媒体类型在MediaType中定义。    
      -List<MediaType> getSupportMediaType():该转换器支持的媒体类型。    
      -T read(Class<? extends T> clazz,HttpInputMessage inputMessage):将请求信息流转换为T类型的对象。     
      -void write(T t,MediaType contentType,HttpOutputMessage):将T类型的对象写到响应流中，同时指定相应的媒体类型为contentType。

![image.png](/image/spring/4-3.png)


- HttpMessageConverter<T>的实现类

| 实现类                               | 功能说明                                                     |
| :----------------------------------- | :----------------------------------------------------------- |
| StringHttpMessageConverter           | 将请求信息转换为字符串                                       |
| FormatHttpMessageConverter           | 将表单数据读取到MutilValueMap中                              |
| XmlAwardFormHttpMessageConverter     | 扩展于FormHttpMessageConverter，如果部分表单属性是XML数据，可用该转换器进行读取 |
| ResourceHttpMessageConverter         | 读写org.springframework.core.io.Resource对象                 |
| BufferedImageHttpMessageConverter    | 读写BufferedImage对象                                        |
| ByteArrayHttpMessageConverter        | 读取二进制数据                                               |
| SourceHttpMessageConverter           | 读写java.xml.transform.Source类型的数据                      |
| MarshallingHttpMessageConverter      | 通过Spring的Marshaller和Unmarshaller读写XML消息              |
| Jaxb2RootElementHttpMessageConverter | 通过JAXB2读写XML消息，将请求消息转换到标注XmlRootElement和XmlType的类中 |
| MappingJacksonHttpMessageConverter   | 利用Jackson开源包的ObjectMapper读写JSON数据                  |
| RssChannelHttpMessageConverter       | 能够读写RSS种子消息                                          |
| AtomFeedHttpMessageConverter         | 和RssChannelMessageConverter能够读写RSS种子消息              |

- DispatcherServlet默认装配RequestMappingHandlerAdapter，而RequestMappingHandlerAdapter默认装配如下HttpMessageConverter：    
    1. ByteArrayHttpMessageConnverter   
    2. StringHttpMessageConverter
    3. ResourceHttpMessageConverter
    4. SourceHttpMessageConverter<T>
    5. AllEncompassingFormHttpMessageConverter   
    6. Jaxb2RootElementHttpMessageConverter

- 加入Jackson jar包后，RequestMappingHandlerAdapter装配的HttpMessageConverter如下：  
    1. ByteArrayHttpMessageConnverter   
    2. StringHttpMessageConverter
    3. ResourceHttpMessageConverter
    4. SourceHttpMessageConverter<T>
    5. AllEncompassingFormHttpMessageConverter   
    6. Jaxb2RootElementHttpMessageConverter
    7. MappingJackson2HttpMessageConverter

- 使用HttpMessageConverter<T> 将请求信息转化并绑定到处理方法的入参中或将响应结果转为对应类型的响应信息，Spring提供了两种途径：
      - 使用@RequestBody或@ResponseBody对处理方法进行标注。    
      - 使用HttpEntity<T>或ResponseEntity<T>作为处理方法的入参或返回值。    

- 当控制器处理方法使用到@RequestBody或@ResponseBody或者HttpEntity<T>或ResponseEntity<T>时，Spring首先根据请求头或响应头的Accept属性选择匹配的HttpMessageConverter，进而根据参数类型或泛型类型的过滤得到匹配的HttpMessageConverter，若找不到可用的HttpMessageConverter将报错。   

- @RequestBody和@ResponseBody不需要成对出现。

### 六.国际化
#### 1.国际化概述
- 默认情况下，SpringMVC根据Accept-Language参数判断客户端的本地化类型。   
- 当接收到请求时，SpringMVC会在上下文中查找一个本地化解析器(LocalResolver)，找到后使用它获取请求所对应的本地化类型信息。   
- SpringMVC还允许装配一个动态更改本地化类型的拦截器，这样通过指定一个请求参数就可以控制单个请求的本地化类型。    

#### 2.本地化解析器和本地化拦截器
- AcceptHeaderLocalResolver：根据HTTP请求头的Accept-Language参数确定本地化类型，如果没有显式定义本地化解析器，SpringMVC使用该解析器。     

- CookieLocalResolver：根据指定的Cookie值确定本地化类型。   

- SessionLocalResolver：根据Session中特定的属性确定本地化类型。   

- LocalChangeInterceptor:从请求参数中获取本次请求对应的本地化类型

```xml
 <bean id="localeResolver" 
          class="org.springframework.web.servlet.i18n.SessionLocaleResolver">
    </bean>
    <mvc:interceptors>
        <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"></bean>
    </mvc:interceptors>
```

#### 3.SessionLocaleResolver和LocaleChangeInterceptor工作原理

![image.png](/image/spring/4-4.png)