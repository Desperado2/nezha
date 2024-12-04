## 一、Spring抽象缓存   
1. Spring从3.1开始定义了org.springframework.cache.Cache和org.springframework.cache.CacheManager接口来统一不同的缓存技术；并支持使用JCache(JSR-107)注解简化开发。         
 
2. Cache接口为缓存的组件规范定义，包含缓存的各种操作集合。         

3. Cache接口下Spring提供了各种Cache的实现，如RedisCache、EhCacheCache、ConcurrentMapCache等。           

4. 每次调用需要缓存功能的方法时，Spring会检查指定参数的指定的目标方法是否已经被调用过；如果有就直接从缓存中获取方法调用后的结果，如果没有就调用方法并缓存结果后返回给用户，下次直接从缓存中获取。         

## 二、缓存组件和注解
注解|说明
:-|:-
Cache|缓存接口，定义缓存操作。实现有:RedisCache、EhCacheCache、ConcurrentMapCache等。     
CacheManager|缓存管理器，管理各种缓存组件
@Cacheable|主要针对方法配置，能够根据方法的请求参数对其结果进行缓存
@CacheEvict|清空缓存
@CachePut|保证方法被调用的时候讲结果缓存
@EnableCaching|开启基于注解的缓存
keyGenerator|缓存数据时key生成策略
Serialize|缓存数据时value序列化策略

## 三、@Cacheable/@CachePut/@CacheEvict主要参数

参数|说明|例子
:-|:-|:-
value|缓存的名称，在Spring配置文件中定义，必须指定至少一个|@Cacheable(value="mycache")或者@Cacheable(value={"cache1","cache2"})
key|缓存的key，可以为空，如果指定要按照SpEL表达式编写，如果不指定，则缺省按照方法的所有参数进行组合|@Cache(value="testCache",key="#userName")
condition|缓存的条件，可以为空，使用SpEL编写，返回true或者false，只有为true才进行缓存/清除缓存|@Cacheable(value="testCache",condition="#userName")
allEntries(@(CacheEvict))|是否清空所有缓存内容，缺省为false，如果指定为true，则方法调用后将立即清空所有缓存|@CacheEvict(value="testCache",allEntries=true)
beforeInvocation(@CacheEvict)|是否在方法执行前就清空，缺省为false，如果指定为true，则方法在还没有执行的时候就会清空缓存，缺省情况下，如果方法执行抛出异常，则不会清空缓存|@CacheEvict(value="testCache",beforeInvocation=true)

## 四、Spring对消息的支持
1. spring-jms提供了对JMS的支持。      
2. spring-rabbit提供了对AMQP的支持。   
3. 需要ConnectionFactory的实现来连接消息代理.      
4. 提供JmsTemplate、RabbitTemplate来发送消息。     
5. @JmsListener(JMS)、@RabbitListener(AMQP)注解在方法上监听消息代理发布的消息。      
6. @EnableJms、@EnableRabbit开启支持。
7. 使用JmsAutoConfiguration、RabbitAutoConfiguration进行自动配置

## 五、Spring与检索
SpringBoot通过整合Spring Data ElasticSearch提供了非常便捷的检索功能支持。      
1. 引入spring-boot-starter-data-elasticsearch依赖。       
2. 安装对应版本的ElasticSearch。     
3. 进行配置。     
4. 使用自动配置的ElasticSearchRepository、Client进行相应的操作。     

## 六、Springboot与任务
### 1.异步任务
在Java应用中，绝大多数情况下都是通过同步的方式来实现交互处理的；但是在处理与第三方系统交互的时候，容易造成响应迟缓的情况，之前大部分都是使用多线程来完成此类任务，其实，在Spring 3.x之后，就已经内置了@Async来解决这个问题。      
@EnableAsync注解开启异步支持。      
@Async标识这是一个异步执行的方法。       

### 2. 定时任务
Spring提供了异步任务调度的方式，提供了TaskExecutor、TaskScheduler接口。      
@EnableScheduling 启用定时任务支持。      
@Schedlued  标识定时任务。      
任务的调用时间使用cron表达式进行指定。      

Cron表达式说明  

字段|允许值|允许的特殊字符
:-|:-|:-
秒|0-59|, - * /
分|0-59|, - * /
小时|0-23|, - * /
日期|1-31| , - * ? / L W C
月份|1-12或者JAN-DEC|, - * /
星期|1-7或者SUN-SAT|, - * ? / L C #
年(可选)|空，1970-2099|, - * /


特殊字符含义说明

特殊字符|含义
 :-|:-
,|枚举
-|区间
*|任意
/|步长
?|日/星期冲突匹配
L|最后
W|工作日
C|和calender联系后计算过的值
#|星期，4#2 表示第二个星期三

### 3. 邮件任务
- 邮件发送依赖于spring-boot-starter-mail。     
- 使用MailSenderAutoConfiguration进行自动配置。     
- 定义MailProperties内容，配置在application.yml中。   
- 自动装配JavaMailSender。    

## 七、SpringBoot与安全
### 1.安全
Spring Security是针对Spring项目的安全框架，也是SpringBoot底层安全模块默认的技术选型。它可以实现强大的web安全控制。对于安全控制，只需要引入spring-boot-starter-security模块，进行少量的配置即可。    
@EnableWebSecurity   开启WebSecurity模式           
WebSecurityConfigurerAdapter   自定义Security策略。      
AuthenticationManagerBuilder   自定义认证策略。        

- "认证"，是建立一个它声明的主体的过程(一个“主体”，一般是值用户，设备或一些可以在程序中执行动作的其他系统)。    
- "授权"，指确定一个主体是否允许在应用程序执行一个动作的过程。为了抵达需要授权的点，主体的身份已经有认证过程建立。       

### 2.Web安全
1. CSRF(Cross-site request forgery)跨站请求伪造。    
    - HttpSecurity启用csrf功能。    
2. 登录/注销
    - HttpSecurity配置
3. remember me(记住我)
    - 表单添加remember-me的checkbox
    - 配置启用remember-me功能
4. Thymeleaf提供的SpringSecurity标签支持
    - 需要引入thymeleaf-extras-springsecurity4     
    - sec:authentication="name" 获取当前用户的用户名。     
    - sec:authorize="hasRole('ADMIN')" 当前用户必须拥有ADMIN权限时才会显示标签内容。

## 八、SpringBoot与监控管理
### 1. 监控管理
通过引入spring-boot-starter-actuator，可以使用SpringBoot为我们提供的标准生产环境下的应用监控和管理功能，可以通过HTTP、JMX、SSH协议来进行操作，自动得到审计、健康指标信息等。   

监控和管理端点
端点名|描述
:-|:-
actuator|所有Endpoint端点，需加入spring HATEOAS支持
autoconfig|所有自动配置的信息
beans|所有Bean的信息
configprops|所有配置属性
dump|线程状态信息
env|当前环境信息
health|应用健康状况
info|当前应用信息
metrics|应用的各项指标
mappings|应用@RequestMapping映射路径
shutdown|关闭当前应用(默认关闭)
trace|追踪信息(最新的http请求)

### 2. 定制端点信息
- 定制端点一般通过endpoints+端点名+属性名来设置。   
- 修改端点id(endpoints.beans.id=mybeans)
- 开启远程应用关闭功能(endpoints.shutdown.enabled = false)
- 关闭端点(endpoints.beans.enabled=false)
- 开启所需端点
    - endpoints.enabled=false
    - endpoints.beans.enabled=true
- 定制端点访问路径(management.context-path=/manager)
- 关闭http端点(management.port=-1)
