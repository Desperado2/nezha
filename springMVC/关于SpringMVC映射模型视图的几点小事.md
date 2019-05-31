### 一.SpringMVC概述
- SpringMVC为展现层提供的基于MVC设计理念的优秀的Web框架，是目前最主流的MVC框架之一。       
- SpringMVC通过一套MVC注解，让POJO成为处理请求的控制器，而无需实现任何接口。    
- 支持RESTFUL风格的URL。   
- 采用了松散耦合可插拔组件结构，更具灵活性和扩展性。      

### 二.使用@RequestMapping映射请求    
#### 1.使用@RequestMapping映射请求   
- SpringMVC使用@RequestMapping注解为控制器指定可以出来那些URL请求。       
- 在控制器的类定义及方法定义处都可以进行标注。      
     - 类定义处：提供初步的请求映射信息。相对于WEB应用的根目录。    
     - 方法处：提供进一步的细分映射信息。相对于类定义处的URL。若类定义处未标注@RequestMapping，则方法标记处的URL相对于WEB应用的根目录。      
- DispatcherServlet截获请求后，就通过控制器上@RequestMapping提供的映射信息确定请求所对应的处理方法。       

#### 2.映射请求参数、请求方法或请求头    

- @RequestMapping除了可以使用请求URL映射请求外，还可以使用请求方法、请求参数及请求头映射请求。        
- @RequestMapping的value、method、params及headers分别表示请求URL、请求方法、请求参数和请求头的映射条件，他们之间是与的关系，联合使用可以使请求映射更加精准化。     
- params和headers支持简单的表达式。     
     - param1:表示请求必须包含名为param1的请求参数。     
     - !param1:表示请求不能包含名为param1的请求参数。    
     - param1 != value1:表示请求包含名为param1的请求参数，但其值不能为value1.       
     - {"param1=value","param2"}:请求必须包含名为param1和param2 两个请求参数，且parma1参数的值必须为value1。   

#### 3.使用@RequestMapping映射请求的匹配符
  - Ant风格资源地址支持3种匹配符。     
    - ? : 匹配文件名中的一个字符。     
    - \* :匹配文件名中的任意字符。   
    - \** :匹配多层路径。     
  - @RequestMapping还支持Ant风格的URL。    
    - /user/*/createUser:匹配/user/aaa/createUser、/user/bbb/createUser等url。     
    - /user/**/createUser：匹配/user/createUser、/user/aaa/createUser、/user/aaa/bbb/createUser等URL。    
    - /user/createUser??: 匹配/user/createUseraa、/user/createUserbb等URL。         

#### 4.使用@PathVariable映射URL绑定的占位符
- 带占位符的URL是Spring 3.0新增的功能，该功能在SpringMVC向Rest目标过程中具有重要意义。    
- 通过@PathVeriable可以将URL中的占位符参数绑定到控制器处理方法的入参中：URL中的{xxx}占位符可以通过@PathVeriable("xxx")绑定到操作方法的入参中。

### 三.映射请求参数   
#### 1.请求处理方法签名      
- springMVC通过分析处理方法的签名，将HTTP请求信息绑定到处理方法的相应入参中。    
- SpringMVC对控制器处理方法签名的限制是很宽松的，几乎可以按喜欢的任何方式对方法进行签名。    
- 必要时可以对方法及方法入参标注相应的主键(@PathVariable、@RequestParam、@RequestHeader等)，SpringMVC框架会将HTTP请求的信息绑定到相应的方法入参中，并根据方法的返回值类型做出相应的后续处理。      

#### 2.使用@RequestParam绑定请求参数值。     
- 在处理方法入参处使用@RequestParam可以把请求参数传递给请求方法。
  - value：参数名。    
  - required：是否必须，默认为true，表示请求参数中必须包含对应的参数，若不存在，将抛出异常。     

#### 3.使用@RequestHeader绑定请求头的属性值    
  - 请求头包含了若干个属性，服务器可据此获知客户端的信息，通过@RequestHeader即可将请求头中的属性值绑定到处理方法的入参中。    

#### 4.使用@CookieValue绑定请求参数中的Cookie的值    
- @CookieValue可让处理方法入参绑定某个cookie值。    

#### 5.使用POJO对象绑定请求参数值     
- SpringMVC会按请求参数名和POJO属性名称进行自动匹配，自动为该对象填充属性，支持级联属性。

#### 6.使用Servlet API作为入参
- 使用HttpServletRequest、HttpServletResponse、HttpSession、java.lang.Principal、Locale、InputStream、OutputStream、Reader、Writer

### 四、处理模型数据
#### 1.SpringMVC提供输出模型数据的途径   
- ModelAndView：处理方法返回值类型为ModelAndView时，方法体即可通过该对象添加数据模型。   
- Map及Model：入参为org.springframework.ui.Model、org.springframework.ui.ModelMap或java.util.Map时，处理方法返回时，Map中的数据会自动添加到模型中。     
- @SessionAttributes：将模型中的某个属性暂存到HttpSession中，以便多个请求之间可以共享这个属性。    
- @ModelAttribute：方法入参标注该注解后，入参的对象就会放到数据模型中。 

#### 2.ModelAndView
- 控制器处理方法的返回值如果为ModelAndView，则其既包含视图信息，也包含模型数据信息。     
- 添加模型数据。   
    - ModelAndView addObject(String attributeName,Object attributeValue)
    - ModelAndView addAllObject(Map<String,?> modeMap)

- 设置视图
    - void setView(View view);
    - void setViewName(String viewName)

#### 3.Map及Model
- SpringMVC在内部使用了一个org.springframework.ui.Model接口存储模型数据。  
- 具体步骤： 
    - SpringMVC在调用方法前会创建一个隐含的模型对象作为模型数据的存储容器。    
    - 如果方法的入参为Map或Model类型，SpringMVC会将隐含模型的引用传递给这些入参。在方法体内，开发者可以通过这个入参对象访问到模型中的所有数据，也可以向模型中添加新的属性数据。    

#### 4.SessionAttributes
- 若希望在多个请求之间共用摸个模型属性数据。则可以在控制器类上标注一个@SessionAttributes，SpringMVC将在模型中对应的属性暂存到HttpSession中。   
- @SessionAttributes除了可以通过属性名指定需要放到会话中的属性外，还可以通过模式属性的对象类型指定那些模型属于需要放到会话中。    
     - @SessionAttributes(types=User.class)会将隐含模型中所有类型为User.class的属性添加到会话中。    
     - @SessionAttributes(value={"user1","user2"},)会将隐含模型中的名称为user1和user2的属性添加到会话中。
     -@SessionAttributes(value{"user1"},types={User.class}) 可以同时进行指定。    

#### 5.ModelAttributes
- 在方法定义上使用@ModelAttributes注解：SpringMVC在调用目标处理方法前，会先逐个调用在方法级别上标注了@ModelAttributes的方法。    
- 在方法的入参使用@ModelAttributes注解：
    - 可以从隐含对象中获取隐含的模型数据中获取对象，再将请求参数绑定到对象中，再传入入参。
    - 将方法入参对象添加到模型中。      

#### 6.由@SessionAttributes引发的异常
- 如果在处理类定义处标准了@SessionAttributes("xxx"),则尝试从会话中获取该属性，并将其赋值给入参，然后再用请求消息填充该入参对象。如果在会话中找不到对应的属性，就会抛出HttpSessionRequiredException异常。    
- 如果避免，可以在类中通过该属性名称的getter方法手动向隐含模型中添加一个名称为属性名的模型属性。   

### 五.视图和视图解析器
 #### 1.SpringMVC如何解析视图      

![image.png](/image/spring/3-1.png)

 #### 2.视图和视图解析器
- 请求处理方法执行完成后，最终返回一个ModelAndView对象。对应那些返回String，View或ModelMap等类型的处理，SpringMVC也会在内部将它们装配成一个ModelAndView对象，它包含了逻辑名和模型对象的视图。    
- SpringMVC借助视图解析器得到最终的视图对象，最终的视图可以是JSP，也可能是Excel、JFreechart等各种表现形式的视图。   
- 对于最终究竟采用何种视图对象对模型数据进行渲染。处理器不关心，处理器各种重点聚焦在生模型数据的工作上，从而实现MVC的重复解耦。   

#### 3.视图
- 视图的作用是渲染模型数据，将模型里的数据以某种形式呈现个客户。   
- 为了实现视图模型和具体实现技术的解耦，Spring在org.springframework.web.servlet包中定义了一个高度抽象的View接口。    
- 视图对象由视图解析器负责实例化。由于视图是无状态的，所以他们不会有线程安全的问题。   

#### 4.常用的视图实现类

| 大类          | 视图类型                     | 说明                                                         |
| ------------- | ---------------------------- | ------------------------------------------------------------ |
| URL视图解析器 | InternalResourceView         | 将JSP或其它资源封装成一个视图，是InternalResourceViewResolver默认使用的视图实现类 |
| URL视图解析器 | JstlView                     | 如果JSP文件中使用了JSTL国际化标签的功能，则需要使用该视图类  |
| 文档视图      | AbstractExcelView            | excel文档视图的抽象类。该视图基于POI构造excel文档            |
| 文档视图      | AbstractPdfView              | PDF文档视图的抽象类，该视图类基于IText构造PDF文档。          |
| 报表视图      | configurableJsperReportsView | 使用JasperReports报表技术的视图                              |
| 报表视图      | JasperReportsCsvView         | 使用JasperReports报表技术的视图                              |
| 报表视图      | JasperReportsMultiFormatView | 使用JasperReports报表技术的视图                              |
| 报表视图      | JasperReportsHtmlView        | 使用JasperReports报表技术的视图                              |
| 报表视图      | JasperReportsPdfView         | 使用JasperReports报表技术的视图                              |
| 报表视图      | JasperReportsXlsView         | 使用JasperReports报表技术的视图                              |
| JSON视图      | MappingJacksonJsonView       | 将模型数据通过Jackson开源框架的ObjectMapper以JSON方式输出    |

#### 5.视图解析器
- SpringMVC为逻辑视图名的解析提供了不同的策略，可以早SpringWEB上下文中配置一种或多种解析策略，并指定他们之间的先后顺序。每一种映射策略对应一个具体的视图解析器实现类。   
- 视图解析器的作用比较单一：将逻辑视图解析为一个具体的试图对象。    
- 所有的视图解析器都必须实现ViewResolver接口。    

#### 6.常用的视图解析器实现类
| 大类             | 视图类型                    | 说明                                                         |
| ---------------- | --------------------------- | ------------------------------------------------------------ |
| 解析为Bean的名字 | BeanNameViewResolver        | 将逻辑视图名解析为一个Bean，Bean的id等于逻辑视图名。         |
| 解析为URL文件    | InternalResourceViewResolve | 将视图对象解析为一个URL文件，一般使用该解析器将视图名映射成为一个保存在WEB-INF目录下面的程序文件。 |
| 解析为URL文件    | JasperReportsViewResolver   | JasperReports是一个基于java的开源报表工具，该解析器将视图名解析为报表文件对应的URL |
| 模板文件视图     | FreeMarkerResolver          | 解析基于FreeMarker模板技术的模板文件                         |
| 模板文件视图     | VelocityViewResolver        | 解析基于Velocity模板技术的模板文件                           |
| 模板文件视图     | VelocityLayoutViewResolver  | 解析基于Velocity模板技术的模板文件                           |

- 可以选择使用一种视图解析器或混用多种视图解析器     
- 每个视图解析器都实现了Ordered接口并开发出一个order属性，可以通过order属性指定解析器的优先顺序，order越小优先级越高。    
- SpringMVC会按照视图解析器顺序的优先顺序对逻辑视图进行解析，知道解析成功并返回视图对象，否则将抛出ServletException异常。  

#### 7.InternalResourceViewResolver
- JSP是最常见的视图技术，可以使用InternalResourceViewResolver作为视图解析器。  
- 若项目中使用了JSTL，则SpringMVC会自动把视图由InternalResourceView转为JstlView。   
- 若使用JSTL的fmt标签需要在SpringMVC的配置文件中配置国际化资源文件。
- 若希望直接响应通过SpringMVC渲染的页面，可以使用mvc:view-controller标签实现。

#### 8.Excel视图
- 若希望使用Excel展示数据文件，仅需要扩展SpringMVC提供的AbstractExcelView或AbstractJExcelView即可。实现buildExcelDocument()方法，在方法中使用模型数据对象构建Excel文档就可以了。   
- AbstractExcelView基于POI API，而AbstractJExcelView是基于JExcelAPI的。    
- 视图对象需要配置IOC容器中的一个Bean，使用BeanNameViewResolver作为视图解析器即可。   
- 若希望直接再浏览器中直接下载Excel文档，则可以设置响应头Content-Disposition的值为attachment;filename=xxx.xls。      

#### 9.关于重定向
- 一般情况下，控制器方法返回字符串类型的值会被当成逻辑视图名处理。  
- 如果返回的字符串中带forward:或redirect:前缀时，SpringMVC会对他们进行特殊处理：将forward:和redirect:当成指示符，其后的字符串作为URL来处理。   
    - redirect:success.jsp：会完成一个到success.jsp的重定向操作。   
    - forward:success.jsp：会完成一个到success.jsp的转发操作。     