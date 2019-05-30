### 一.IOC和DI
#### 1.IOC(Inversion of Control)
其思想是反转资源获取的方向。传统的资源查找方式要求组件向容器发起请求查找资源，作为回应，容器适时的返回资源；而应用了IOC之后，则是容器主动的将资源推送给它所管理的组件，组件所要做的仅是选择一种合适的方式来接收资源。这种行为也被成为查找的被动形式。       

#### 2.DI(Dependency Injection)
IOC的另一种表达方式：即组件的一些预先定义好的方式(例如：setter方法)接受来自如容器的资源注入。相对于IOC而言，这种表述更直接。    
    
![image.png](/image/spring/1-1.png)

#### 3.IOC的前世今生            
使用这样一个需求，生成HTML或者PDF格式的不同类型的报表为例，说明IOC的演变。
①分离接口设计      

![image.png](/image/spring/1-2.png)

②采用工厂模式      

![image.png](/image/spring/1-3.png)


③采用反转控制      

![image.png](/image/spring/1-4.png)      

### 二.spring配置bean     
#### 1.在spring的IOC容器里面配置Bean。        
在xml文件中通过bean节点来配置bean     
```xml
<bean id="helloWorld" 
         class="com.desperado.helloworld.HelloWorld >
</bean>
```
id:Bean的名称。        
- 在IOC容器中必须是唯一的.      
- 若id没有指定，Spring会自动将全限定性类名作为Bean名称。      
- id可以指定多个名字，名字之间可以用逗号、分号、或空格隔开。       

#### 2.Spring容器       
- 在Spring IOC容器读取Bean配置创建Bean实例之前，必须对它进行实例化。只有在容器实例化之后，才可以从IOC容器里面获取Bean实例并使用。     
- Spring提供了两种类型的IOC容器实现         
  * BeanFactory：IOC容器的基本实现。     
  * ApplicationContext：提供了更多的高级特性，是BeanFactory的子接口。       
  * BeanFactory是Spring框架的基础设施，面向Spring本身。ApplicationContext面向使用Spring框架的开发者，几乎所有的应用场合都直接使用ApplicationContext而非底层的BeanFactory。      
  * 无论使用何种方式，配置文件是相同的。     

#### 3.ApplicationContext       
- ApplicationContext的主要实现类：     
  * ClassPathXmlApplicationContext:从类路径下面加载配置文件。       
  * FileSystemXmlApplicationContext:从文件系统中加载配置文件。      
- ConfigurableApplicationContext扩展于ApplicationContext，新增加两个主要方法：refresh()和close(),让ApplicationContext具有启动、刷新和关闭上下文的能力。    
- ApplicationContext在初始化上下文时就实例化所有单例的Bean。      
- WebApplicationContext是专门为WEB应用而准备的，它允许从相对于WEB根目录的路径中完成初始化工作.。    

![image.png](/image/spring/1-5.png)

- 调用ApplicationContext的getBean()方法可以获取到Bean。     

#### 4.依赖注入的三种方式          
- 属性注入          
  属性注入即通过setter方法注入Bean的属性值或依赖的对象。     
  属性注入使用<property>元素，使用name属性指定Bean的属性名称，value属性或<value>子节点指定属性值。       
  属性注入是实际应用中最常用的注入方式。      
```xml
<bean id="helloWorld"   class="com.desperado.helloworld.HelloWorld">
      <property name="userName" value="desperado"></property>
</bean>
```
- 构造器注入        
  通过构造方法注入Bean的属性值或依赖的对象，它保证了Bean实例在实例化后就可以使用。       
  构造器注入在<constructor-arg>元素里声明属性，<constructor-arg>中没有name属性。      
  * 按索引匹配入参       
```xml
<bean id="car" class ="com.desperado.helloworld.Car">
      <constructor-arg value="奥迪" index="0"></constructor-arg>  
      <constructor-arg value="奔驰" index="1"></constructor-arg>  
      <constructor-arg value="5000" index="2"></constructor-arg>  
</bean>
```
  * 按匹配参数注入
```xml
<bean id="car" class ="com.desperado.helloworld.Car">
      <constructor-arg value="奥迪" type="java.lang.String"></constructor-arg>  
      <constructor-arg value="奔驰" type="java.lang.String"></constructor-arg>  
      <constructor-arg value="5000" type="java.lang.double"></constructor-arg>  
</bean>
```
- 工厂方法注入(很少使用，不推荐)     

- 注入属性值细节   
  * 字面值：可用字符串表示的值，可以通过<value>元素标签或value属性进行注入。     
  * 基本数据类型及其封装类、String等类型都可以采取字面值的注入。     
  * 若字面值中包含特殊字符，可以使用<![CDATA[]]>把字面值包裹起来。     

  
#### 5.引用其他Bean    
- 组成应用程序的Bean经常需要相互协作以完成应用程序的功能，要使Bean能够相互访问，就必须在Bean配置文件中指定对Bean的引用。   
- 在Bean的配置文件中，可以通过<ref>元素或ref属性为Bean的属性或构造器参数指定对Bean的引用。   
- 也可以在属性或构造器里包含Bean的声明，这样的Bean成为内部Bean。 

   
```xml
<bean id="service" class="com.desperado.helloworld.Service"></bean> 

<bean id="action" class="com.desperado.helloworld.Action">
      <property name="service" ref="service"></property>
</bean>
```


#### 6.内部bean
- 当Bean实例仅仅给一个特定的属性使用时，可以将其声明为内部Bean，内部Bean声明直接包含在<property>或者<constructor-arg>元素里，不需要设置任何id或name属性。     
- 内部Bean不能使用在任何其他部分。 

   
#### 7.null值和级联属性     
- 可以使用专用的<null />元素标签为Bean的字符串或其它对象类型的属性注入null值。     
- Spring支持级联属性的配置。      


#### 8.集合属性      
- 在Spring中可以通过一组内置的xml标签(例如<list>,<set>,<map>)来配置集合属性。     
- 配置java.util.List类型的属性，需要指定<list>标签，在标签里包含一些元素。这些标签可以通过<value>指定简单的常量值。这些标签可以通过<value>指定简单的常量值，通过<ref>指定对其他Bean的引用。通过<bean>指定内置Bean的定义，通过<null/>指定空元素，甚至可以内嵌其他集合。    
- 数组的定义和List一样，都使用<list>     
- 配置java.util.Set需要使用<set>标签，定义元素的方法与List一样。     
- java.lang.Map通过<map>标签定义，<map>标签里可以使用多个<entry>作为子标签，每个条目包含一个键和一个值。    
- 必须在<key>标签里定义键。    
- 因为键和值的类型没有限制，索引可以自由的为它们指定<value>,<ref>,<bean>或<null>元素。     
- 可以将Map的键和值作为<entry>的属性定义：简单常量使用key和value定义；Bean引用通过key-ref和value-ref属性定义。    
- 使用<props>定义java.util.Properties，该标签使用多个<prop>作为子标签，每个<prop>标签必须定义key属性。     


#### 9.使用utility scheme定义集合      
- 使用基本的集合标签定义集合时，不能将集合作为独立的Bean定义，导致其他Bean无法引用该集合，所以无法再不同Bean之间共享集合。    
- 可以使用util schema里的集合标签定义独立的集合Bean。需要注意的是，必须在<beans>根元素里添加util schema定义。     


#### 10.使用P命名空间     
- 为了将XML文件的配置越来越多的XML文件采用属性而非子元素配置信息。    
- Spring从2.5开始引入了一个新的p命名空间，可以通过<bean>元素属性的方式配置Bean的属性。    
  -使用p命名空间后，基于XML的配置方式将进一步简化。   

  
### 三.XML配置里的Bean自动装配   
- Spring IOC容器可以自动装配Bean，需要做的仅仅是在<bean>的autowire属性里指定自动装配的模式。    
- byType(根据类型自动装配):若IOC容器中有多个与目标Bean类型一致的Bean。在这种情况下，Spring将无法判定哪个Bean最适合该属性，索引不能执行自动装配。      
- byName(根据名称自动装配):必须将目标Bean的名称和属性名设置的完全相同。     
- constructor(通过构造器自动装配):当Bean中存在多个构造器时，此种自动装配方式将会很复杂，不推荐使用。


**缺点**      
- 在Bean配置文件里设置autowire属性进行自动装配将会装配Bean的所有属性，然后，若只希望装配个别属性时，autowire属性就不够灵活了。    
- autowire属性要么根据类型自动装配，那么根据名称自动装配，不能两者兼而有之。    
- 一般情况下，在实际的项目之中很少使用自动装配功能，因为和自动装配功能所带来的好处比起来，明确清晰的配置文档更好。   


### 四.Bean之间的关系：继承和依赖   

#### 1.继承Bean配置      
- Spring允许继承Bean的配置，被继承的Bean称为父bean，继承这个父bean的bean称为子bean。      
- 子Bean从父Bean中继承配置，包括Bean的属性配置。    
- 子Bean也可以覆盖从父Bean继承过来的配置。      
- 父Bean可以作为配置模板，也可以作为bean实例。若只想把父bean作为模板，可以设置<bean>的abstract属性为true，这样Spring将不会实例化这个Bean。     
- 并不是<bean>元素里的所有属性都会被继承。比如autowire、abstract等。      
- 也可以忽略父Bean的class属性，让子Bean指定自己的类，而共享相同的属性配置，但此时abstract必须设为true。  
  
  
#### 2.依赖Bean配置      
- Spring允许用户通过depends-on属性设定Bean前置依赖的Bean，前置依赖的Bean会在本Bean实例化之前创建好。      
- 如果前置依赖于多个Bean，则可以通过逗号、空格的方式配置Bean的名称。    


### 五.bean的作用域      
- 在Spring中，可以在<bean>元素的scope属性里设置Bean的作用域。      
- 默认情况下，Spring只为每个在IOC容器里面声明的Bean创建唯一一个实例，整个IOC容器范围内都能共享该实例：所有后续的getBean()调用和Bean引用都将返回这个唯一的Bean实例。该作用域被称为singleton，它是所有bean的默认作用域。       

| 类别          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| singleton     | 在SpringIOC容器中仅存在一个Bean实例，Bean以单实例的方式存在  |
| prototype     | 每次调用getBean()时都会返回一个新的实例                      |
| request       | 每次HTTP请求都会创建一个新的Bean，该作用域仅适用于WebApplicationContext环境 |
| session       | 同一个HTTP session共享一个Bean，不同的HTTP Session使用不同的Bean。该作用域仅适用于WebApplicationContext环境。 |
| globalSession | 一般用于protlet应用环境，该作用域仅适用于WebApplicationContext环境。 |



### 六.使用外部属性文件    
1.在配置文件里配置Bean时，有时需要在Bean的配置里混入系统部署的细节信息(例如：文件路径，数据源配置信息等)。而这些部署细节实际上需要和Bean配置相分离。     

2.Spring提供了一个PropertyPlaceholderConfigurer的BeanFactory后置处理器，这个处理器允许用户将Bean配置的部分内容外移到属性文件中。可以在Bean配置文件里使用形式为 \$\{var\} 的变量，PropertyPlaceholderConfigurer从属性文件里加载属性，并使用这些属性来替换变量。     

3.Spring还允许在属性文件中使用${propName},以实现属性直接的相互作用。    

4.注册PropertyPlaceholderConfigurer       
Spring2.0的配置方式：      
```xml
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
      <property name="location" value="classpath:jdbc.properties"></property>
</bean>
```
Spring2.5之后：可通过<context:property-placeholder>元素简化：       
- <beans>中添加context Schema定义     
- 在配置文件中加入如下配置  
```xml
<context:property-placeholder location="classpath:db.properties" />
```


### 七.SpEL
#### 1.Spring表达式语言(简称SpELl):是一个支持运行时查询和操作对象图的强大的表达式语言。     
- 语法类似于EL：SpELl使用#{...}作为界定符，所有再打括号中的字符都被认为是SpEL        
- SpEL为bean的属性进行动态赋值提供了便利。     
- 通过SpELl可以实现：     
  - 通过bean的id对bean进行引用。       
  - 通过方法以及引用对象的中的属性      
  - 计算表达式的值        
  - 正则表达式的匹配

  
#### 2.SpEL的字面量      
- 字面量的表示      
  - 整数：\<property name="count" value="#{5}"/>
  - 小数：\<property name="count" value="#{87.9}"/>    
  - 科学计数法：\<property name="count" value="#{1e4}"/>        
  - String可以使用单引号或者双引号作为字符串的界定符号：\<property name="count" value="#{'chuck'}"/>.     
  - booean: \<property name="count" value="#{false}"/>       

  
#### 3.引用Bean、属性和方法      
- 应用其他对象      
  通过value属性和SpELl配置bean之间的关系      
```xml
<property name="prefix" value="#{prefixGenerator}"/>       
```
- 引用其他对象的属性       
  通过value属性和SpELl配置suffix属性值为另一个Bean的suffix属性值       
```xml
<property name="suffix" value="#{sequenceGenerator2.suffix}"/>       
```
- 调用其他方法，还可以链式操作      
  通过value属性和SpELl配置suffix属性值为另一个Bean的方法的返回值      
```xml
<property name="suffix" value="#{sequenceGenerator2.toString()}"/>       
```
- 方法的链式操作      
```xml
<property name="count" value="#{sequenceGenerator2.toString().toUpperCase()}"/>       
```
- 调用静态方法或静态属性:通过T()调用一个类的静态方法，它将返回一个Class Object，然后再调用相应的方法或属性。       
```xml
<property name="initValue" value="#{T(java.lang.Math).PI}"></property>
```


#### 4. SpEL支持的运算符号      
- 数学运算符:+，-，*，/，%，^：       
- 加好还可以用作字符串的连接。     
- 比较运算符:<,>,==,<=,>=,lt,gt,eq,le,ge     
- 逻辑运算符:and,or,not,|
- if-else运算符:?:(ternary),?:(Elvis)       
- if-else的变体      
- 正则表达式:matches


### 八.Bean的生命周期 

#### 1.IOC容器中Bean的生命周期方法    
- Spring IOC容器可以管理Bean的生命周期，Spring允许在Bean生命周期的特定点执行定制的任务。       
- SpringIOC容器对Bean的生命周期进行管理的过程：    
    1. 通过构造器或工厂方法创建Bean实例。      
    2. 为Bean的属性设置值和对其他Bean的引用。      
    3. 调用Bean的初始化方法。      
    4. Bean可以使用了。     
    5. 当容器关闭了，调用Bean的销毁方法。    
- 在Bean的声明里设置init-method和destroy-method属性，为Bean指定初始化和销毁方法。  

  
#### 2.创建Bean后置处理器    
- Bean后置处理器允许在调用初始化方法前后对Bean进行额外的处理。     
- Bean后置处理器对IOC容器里的所有Bean实例逐一处理，而非单一实例。其典型应用是：检查Bean属性的正确性或根据特定的标准更改Bean的属性。    
- 对Bean后置处理器而言，需要实现BeanPostProcessor接口。在初始化方法被调用前后，Spring将把每个Bean实例分别传递给上述接口的postProcessAfterInitialization()方法和postProcessBeforeInitialization()方法。       


#### 3.添加后置处理器后Bean的生命周期
SpringIOC容器对Bean的生命周期进行管理的过程：     
    1. 通过构造器或工厂方法创建Bean实例。     
    2. 为Bean的属性设置值和对其他Bean的引用。    
    3. 将Bean实例传递给Bean后置处理器的postProcessBeforeInitialization()方法。    
    4. 调用Bean的初始化方法。     
    5. 将Bean的实例传递给Bean的后置处理器的postProcessAfterInitialization()方法。      
    6. Bean可以正常使用了。     
    7. 当容器关闭时，调用Bean的销毁方法。 
	
	
#### 4.通过调用静态工厂方法创建Bean      
- 调用静态工厂方法创建Bean是将对象创建的过程封装到静态方法中。当客户端需要对象时，只需要简单地调用静态方法，而不用关心创建对象的细节。    
- 要声明通过静态方法创建的Bean，需要在Bean的class属性里面这么拥有该工厂的方法的类，同时在factory-method属性里指定工厂方法的名称。最后使用<constrctor-arg>元素为该方法传递方法参数。


#### 5.通过调用实例工厂方法创建Bean   
- 实例工厂方法：将对象的创建过程封装到另外一个对象实例的方法里。当客户端需要请求对象时，只需要简单的调用该实例方法而不需要关系对象的创建细节。     
- 要声明通过实例工厂方法创建的Bean     
    - 在bean的factory-bane属性里指定拥有该工厂方法的bean。     
    - 在factory-method属性里指定该工厂方法的名称。     
    - 使用constructor-arg元素为工厂方法传递方法参数。

	
#### 6.实现FactoryBean接口在SpringIOC容器中配置Bean    
- Spring中有两种类型的Bean，一种是普通Bean，另一种是工厂Bean，即FactoryBean。    
- 工厂Bean跟普通Bean不同，其返回的对象不是指定类的一个实例，其返回的是该工厂Bean的getObject方法所返回的对象。     


### 九.组件装配    
#### 1.在classpath中扫描组件
- 组件扫描：spring能够从classpath下自动扫描，侦测和实例化具有特定注解的组件。   
- 特定组件包括：      
    1. @Component:基本注解，标识了一个受Spring管理的组件。     
    2. @Responsitory:标识持久层组件。     
    3. @Service:标识服务层(业务层)组件。      
    4. @Controller:标识表现出组件。   
- 对于扫描到的组件，Spring有默认的命名策略：使用非限定类名，第一个字母小写。也可以在注解中通过value属性值标识组件的名称。
- 当在组件类上使用了特定的注解之后，还需要在Spring的配置文件声明<context:component-scan>：       
   - base-package属性制定一个要扫描的基类包，Spring容器将会扫描这个基类包及其子包中的所有类。    
   - 当需要扫描多个包时，可以使用逗号隔开。     
   - 如果仅希望扫描特定的类而非基类包下的所有类，可使用resource-pattern属性过滤特定的类。       

   - <context:include-filter>子节点表示要包含的目标类。    
    - <context:exclude-filter>子节点表示要排除在外的目标类。      
    - <context:component-scan>下面可以拥有若干个<context:include-filter>和<context:exclude-filter>子节点。     

```xml
 <context:component-scan base-package="com.desperado.helloworld" 
                         resource-pattern="autowire/*.class />
```
- <context:include-filter>和<context:exclude-filter>子节点支持多种类型的过滤表达式。     

| 类别       | 示例                        | 说明                                                         |
| ---------- | --------------------------- | ------------------------------------------------------------ |
| annotation | com.desperado.XxxAnnotation | 所有标注了XXXAnnotation的类。该类型采用目标；类是否标注了某个注解进行过滤。 |
| assinable  | com.desperado.XxxService    | s所有继承或扩展XXXService的类。该类型采用目标类是否继承或扩展某个特定类进行过滤。 |
| aspectj    | com.desperado..*Service+    | 所有类名以Service结尾的类以及继承或扩展它们的类。该类型采用AspectJ表达式进行过滤 |
| regex      | com.desperado.anno\..*      | 所有com.desperado.anno包下面的类。该类型采用正则表达式根据类的类名进行过滤。 |
| custom     | com.desperado.XxxTypeFilter | 采用XXXTypeFilter通过代码的方式定义过滤规则。该类必须实现org.springframework.core.type.TypeFilter接口 |

- <context:component-scan>元素还会自动注册AutowiredAnnotationBeanPostProcessor实例，该实例可以自动装配具有@Autowired和@Resource、@Inject注解的属性。     


#### 2.使用@Autowired自动装配Bean
@Autowired注解自动装配具有兼容类型单个Bean属性。     
  - 构造器，普通字段(即使是非public)，一切具有参数的方法都可以应用@Autowired注解。    
  - 默认情况下，所有使用@Autowired注解的属性都需要被设置。当Spring找不到匹配的Bean装配属性时，会抛出异常，若某一属性允许不被设置，可以设置@Autowired注解的required属性为false。     
  - 默认情况下，当IOC容器里存在多个类型兼容的Bean时，通过类型的自动装配将无法工作，此时可以在@Qualifier注解里提供Bean的名称。Spring允许对方法的入参标准@Qualifiter已指定Bean的名称。     
  - @Autowired注解也可以应用在数组类型的属性上，此时Spring将会把所有匹配的Bean进行自动装配。    
  - @Autowired注解也可以应用在集合属性上，此时Spring读取该集合的类型信息，然后自动装配所有与之兼容的Bean。     
  - @Autowired注解用在java.util.Map上时，若该Map的键值为String，那么Spring将自动装配与之Map值类型兼容的Bean，此时Bean的名称作为键值。 


  #### 3.使用@Resource或@Inject自动装配Bean    
- Spring还指出@Resource和@Inject注解，这两个注解和@Autowired注解的功用类似。    
- @Resource注解要求提供一个Bean名称的属性，若该属性为空，则自动采用标注处的变量或方法名作为Bean的名称。    
- @Inject和@Autowired注解一样也是按类型匹配注入的Bean，但没有required属性。    
- 建议使用@Autowired注解



### 十.泛型依赖注入
Spring4.x中可以为子类注入子类对应的泛型类型的常用变量的引用。    

   ![image.png](/image/spring/1-6.png)




### 十一.整个多个配置文件
Spring允许通过<import>将多个配置文件引入到一个文件中，进行配置文件的集成。这样在启动Spring容器时，仅需要指定这个合并好的配置文件就可以。      
import元素的resource属性支持Spring的标准的路径资源 
     

| 地址前缀   | 示例                                         | 对应资源类型                                        |
| ---------- | -------------------------------------------- | --------------------------------------------------- |
| classpath: | classpath:spring-mvc.xml                     | 从类路径下加载资源，classpath:和classpath:/是等价的 |
| file:      | file:/conf/security/spirng-shiro.xml         | 从文件系统目录中装载资源，可采用绝对或相对路径      |
| http://    | http:///www.desperado.com/resource/beans.xml | 从web服务器中加载资源                               |
| ftp://     | ftp://www.desperado.com/resource/beans.xml   | 从FTP服务器中加载资源。                             |