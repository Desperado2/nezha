#### 0.AOP简介
- AOP(Aspect-Oriented Programming，面向切面编程):是一种新的方法论，是穿透OOP的补充。   
- AOP的主要编程对象是切面(aspect)，而切面模块化横切关注点。    
- 在使用AOP编程时，仍然需要定义功能功能，但可以明确的定义这个功能在哪里，以什么方式应用，并且不必修改受影响的类。这样一来横切关注点就被模块化到特殊的对象(切面)里。    
- AOP的好处：
      - 每个事物逻辑位于同一层，代码不分散，便于维护和升级。     
      - 业务模块更简洁，只包含核心业务代码。     
  
#### 1.AOP术语
- 切面(Aspect)：横切关注点(跨越应用程序多个模块 的功能)被模块化的特殊对象。     
- 通知(Advice)：切面必须要完成的工作。     
- 目标(Target)：被通知的对象。     
- 代理(Proxy)：向目标对象应用通知之后创建的对象。     
- 连接点(Joinpoint)：程序执行的某个特定位置：如某个方法调用前、调用后、方法抛出异常后等。连接点由两个信息确定：方法表示的程序执行点；相对点表示的方位。     
- 切点(pointcut)：每个类都拥有多个连接点。AOP通过切点定位到特定的连接点。类比：连接点相当于数据库中的记录，切点相当于查询条件。切点和连接点不是一对一的关系，一个切点匹配多个连接点，切点通过org.springframework.aop.Pointcut接口进行描述，它使用类和方法作为连接点的查询条件。   
#### 2.AspectJ
Java社区里最完善最流行的AOP框架。     
在Spring2.0以上版本中，可以使用基于AspectJ注解或基于XML配置的AOP。    

#### 3.在Spring中启用AspectJ注解支持    
- 要在Spring应用中使用AspectJ注解，必须早classpath下包含AspectJ类库
  ;aoplliance.jar、aspectj.weaver.jar和spring-aspects.jar.      
- 将aop Schema添加到<beans>根元素中。     
- 要在Spring IOC容器中启用AspectJ注解支持，只要在Bean配置文件中定义一个空的XML元素<aop:aspectj-autoproxy>         
- 当Spring IOC容器侦测到Bean配置文件中的<aop:aspectj-autoproxy>元素时，会自动为与AspectJ切面匹配的Bean创建代理。    
- 
#### 4.用AspectJ注解声明切面
- 要在Spring中声明AspectJ切面，只需要在IOC容器中将切面声明为Bean实例。当在SpringIOC容器中初始化AspectJ切面之后，SpringIOC容器就会为那些与AspectJ切面相匹配的Bean创建管理。      
- 在AspectJ注解中，切面只是一个带有@Aspect注解的Java类。      
- 通知是标注有某种注解的简单的Java方法。      
- AspectJ支持5种类型的通知注解：     
    - @Before:前置通知，在方法执行之前执行。   
    - @After:后置通知，在方法执行之后执行。    
    - @AfterRunning:返回通知，在方法返回结果之后执行。     
    - @AfterThrowing:异常通知，在方法抛出异常之后。     
    - @Around:环绕通知，围绕着方法执行。

#### 5.前置通知   
- 在方法执行之前执行的通知。      
- 前置通知使用@Before注解，并将切入点表达式的值作为注解值。    
```java
@Aspect
public class CalculatorLoggingAspect{
    private Log log = LogFactory.getLog(this.getClass());

    @Before("execution(* ArithmenticCalculator.add(..))")
     public void logBefore(){
            log.info("The method add() begins");
    }
}
```
- @Aspect注解标识这个类是一个切面。     
- @Before注解标识这个方法是个前置通知。    
- 切点表达式表示执行 ArithmeticCalculator接口的add()方法。    
- \* 代表匹配任意修饰符及任意返回值。    
- 参数列表中的  . .  表示匹配任意数量的参数。          
- 
#### 6.利用方法签名编写AspectJ切入点表达式。      
最典型的的切入点表达式是根据方法的签名来匹配各种方法的。     
- execution * com.desperado.spring.ArithmeticCalculator.*(..):匹配ArithmeticCalculator中声明的所有方法，第一个 * 代表任意修饰符以及任意返回值；第二个 * 代表任意方法； .. 匹配任意数量的参数。若目标类与接口与该切面在同一个包中，可以省略包名。   
- execution public * com.desperado.spring.ArithmeticCalculator.*(..):匹配ArithmeticCalculator接口的所有公共方法。    
- execution public double com.desperado.spring.ArithmeticCalculator.*(..):匹配ArithmeticCalculator接口的所有返回double类型数值的公共方法。    
- execution public double  com.desperado.spring.ArithmeticCalculator.*(double,..):匹配ArithmeticCalculator接口的所有第一个参数类型为double类型的公共方法。     
- execution public double  com.desperado.spring.ArithmeticCalculator.*(double,double):匹配ArithmeticCalculator接口的所有参数类型为double,double的公共方法。    
- 
#### 7.合并切入点表达式    
在AspectJ中，切入点表达式可以通过操作符&&，||，!结合起来。    
```java
@Pointcut("execution(* *.add(int,..)) || execution(* *.sub(int,..))")
private void loggingOperation(){}

@Before("loggingOperation()")
public void logBefore(JoinPoint joinPoint){
    log.info("");
}
```
- 可以在通知方法中声明一个类型为JoinPoint的参数，然后就能访问连接细节，如方法名称和参数。     
#### 8.后置通知  
后置通知是在连接点完成之后执行的，即连接点返回结果或抛出异常的时候，下面的后置通知记录了方法的终止。    
- 一个切面可以包括一个或多个通知。    
```java
@Aspect
public class CalculatorLoggingAspect{
     private Log log = LogFactory.getLog(this.getClass());

    @After("execution(* ArithmenticCalculator.add(..))")
     public void logBefore(){
            log.info("The method add() begins");
    }
}
```
#### 9.返回通知
无论连接点是正常返回还是抛出异常，后置通知都会执行，如果只想在连接点返回的时候记录日志，应使用返回通知大厅后置通知。   
```java
@Aspect
public class CalculatorLoggingAspect{
     private Log log = LogFactory.getLog(this.getClass());

    @AfterReturning("execution(* ArithmenticCalculator.add(..))")
     public void logBefore(){
            log.info("");
    }
}
```
- 在返回通知中，只要将returning属性添加到@AfterReturing注解中，就可以访问连接点的返回值。    
- 必须必须在通知方法的签名中添加一个同名参数，在运行时，Spring AOP会通知这个参数传递返回值。    
- 原始的切点表达式需要出现在pointcut属性中     
```java
@AfterReturning(pointcut="execution(* *.*(..))",returning ="result")
public void logAfterReturning(JoinPoint joinPoint,Object result){
     log.info("The method "+joinPoint.getSignature().getName()+" () ends with "+ result);
}
```

#### 10.异常通知
- 只在连接点抛出异常时才执行异常通知。    
- 将throwing属性添加到@AfterThrowing注解中，也可以访问连接点抛出的异常，Throwable是所有错误和异常类的超类。所以在异常通知方法可以捕获到任何错误和异常。     
- 如果只对某种特殊的异常类型感兴趣，可以将参数声明为其他异常的参数类型；然后通知就只在抛出这个类型及其子类的异常才被执行。     
```java
@AfterThrowing(pointcut="execution(* *.*(..))",throwing ="e")
public void logAfterReturning(JoinPoint joinPoint,ArithmeticExection e){
     log.info("A exception "+e+" has been throwing ");
}
```

#### 11.环绕通知    
- 环绕通知是所有通知类型中功能最强大，能够全面地控制连接点。甚至可以控制是否执行连接点。    
- 对于环绕通知来说，连接点的参数类型必须是ProceedingJoinPoint。它是JoinPoint的子接口，运行控制何时执行，是否执行连接点。   
- 在环绕通知中需要明确调用ProceedingJoinPoint的proceed()方法来执行被代理的方法。如果忘记这样做就会导致通知被执行了，但目标方法没有被执行。    
- 注意：环绕通知的方法需要返回目标方法执行之后的结果，即调用joinPoint.proceed()的返回值，否则会出现空指针异常。
```java
@Around(pointcut="execution(* *.*(..))")
public void logAround(proceedingJoinPoint joinPoint) throws Throwable{
     log.info("A method begins");

    try{
        joinPoint.proceed();
        log.info("The methods ends");
    }catch(Throwable e){
        log.info("An Exception has been throwing");
        throw e;
    }
}
```

#### 12.指定切面的优先级
- 在同一个连接点上应用不止一个切面时，除非明确指定，否则他们的优先级是不确定的。    
- 切面的优先级可以通过实行Ordered接口或利用@Order注解指定。    
- 实行Ordered接口，getOrder()方法的返回值越小，优先级越高。    
- 若使用@Order注解，序号出现在注解中     
```java
@Aspect
@Order(0)
public class Aspect1{}

@Aspect
@Order(1)
public class Aspect2{}
```

#### 13.重用切入点定义
- 在编写AspectJ切面时，可以直接再通知注解中书写切入点表达式，但同一个切点表达式可能会在多个通知中重复出现。    
- 在AspectJ切面中，可以通过@Pointcut注解将一个切入点声明为简单的方法，切入点的方法体通常是空的，因为将切入点定于与应用程序逻辑混在一起是不合理的。    
- 切入点方法的方法控制符同时也控制这这个切入点的可见性。如果切入点要在多个切面中使用，最好将它们集中在一个公共的类中。在这种情况下，它们必须被声明为public。在引入这个切入点时，必须将类名也包括在内，如果累没有与这个切面放在同一个包中，还必须包含包名。   
- 其他通知可以通过方法名称引入该切入点。     
```java
   @Pointcut("execution(* *.*(..))")
    private void loggingOperation(){}
    
    @Before("loggingOperation()")
    public void logBefore(JoinPoint joinPoint){
        log.info("The method begins ");
    }
    
    @AfterReturning("loggingOperation()",returning="result")
    public coid logAfterReturning(JoinPoint joinPoint,Object result){
        log.info("The method ends");
    }
    
    @AfterThrowing("loggingOperation()",throwing="result")
    public coid logAfterReturning(JoinPoint joinPoint,Object result){
        log.info("An exception has been throwing");
    }
```

#### 14.引入通知
- 引入通知是一种特殊的通知类型，它通过为接口提供实现类，允许对象动态地实现接口，就像对象已经在运行时扩展了实现类一样。     

   ![image.png](/image/spring/2-1.png)

- 引入通知可以使用两个实现类MaxCalculatorImpl和MinCalculatorImpl，让ArithmeticCalculatorImpl动态地实现MaxCalculator和MinCalculator接口。而这与从MaxCalculatorImpl和MinCalculatorImpl中实现多继承的效果相同，但却不需要修改ArithmeticCalculatorImpl的源代码。     
- 引入通知也必须在切面中声明。    
- 在切面中，通过为任意字段添加@DeclareParents注解来引入声明。    
- 注解类型的value属性表示哪些类是当前引入通知的目标。value属性值也可以是一个AspectJ类型的表达式，以将一个即可引入到多个类中。defaultImpl属性中指定这个接口使用的实现类。    
```java
public class CalculatorLoggingAspect implements Ordered {
    private Log log = LoggerFactory.getLog(this.getClass());
    
    @DeclareParents(value="* *.Arithmetic",defaultImpl=MaxCalculatorImpl.class)
    private MaxCalculator maxCalculator;
    
    @DeclareParenta(value="* *.Arithmetic",defaultImpl=MinCalculatorImpl.class)
    private MinCalculator minCalculator;
    
    MinCalculator minCalculator = (MinCalculator) ctx.getBean("airthmeticCalculator");
    minCalculator.min(1,2);
}
```

#### 15.用基于XML的配置声明切面    
- 除了使用AspectJ注解声明切面，Spring也支持在Bean配置文件中声明切面，这种声明是通过aop schema中的XML元素完成的。    

- 正常情况下，基于注解的声明要优先于基于XML的声明。通过AspectJ注解，切面可以与AspectJ兼容，而基于XML的配置则是Spring专有的，由于AspectJ得到越来越多的AOP框架支持，所以以注解风格编写的切面将会有更多重用的集合。     

**基于XML---声明切面**     

- 当使用XML声明切面时，需要在<beans>根元素中导入aop schema。     

- 在Bean配置文件中，所有的Spring AOP配置都必须定义在<aop:config>元素内部。对于每个切面而言，都需要创建一个<aop:aspect>元素来为切面实现引用后端Bean实例。   

- 切面Bean必须有一个标识符，供<aop:aspect>元素引用。    
```xml
<bean id="aspect1" class="com.desperado.Aspect1"></bean>

<aop:config>
    <aop:aspect id="loggingAspect" ref="aspect1"></aop:aspect>
</aop:config>
```
**基于XML ---- 声明切入点**  
- 切入点使用<aop:pointcut>元素声明。   
- 切入点必须电仪在<aop:aspect>元素下，或者直接定义在<aop:config>元素下。
    - 定义在<aop:aspect>元素下：只对当前切面有效。   
    - 定义在<aop:config>元素下：对所有切面都有效。    
- 基于XML的AOP配置不允许在切入点表达式中通名称引用其他切入点。    
```xml
<aop:config>
    <aop:pointcut id="testOperation" expression="execution(* com.desperado.bean.Arithmetic*.*(..))"/>
</aop:config>
```
**基于XML-----声明通知**     
- 在aop Schema中，每种通知类型都对应一个特定的XML元素。    
- 通知元素需要使用<pointcut-ref>来引入切入点，或用<pointcut>直接嵌入切入点表达式。method属性指定切面类中通知方法的名称。    
```xml
<aop:config>
    <aop:pointcut id="testOperation" expression="execution(* com.desperado.bean.Arithmetic*.*(..))"/>

    <aop:aspect id="loggingAspect" ref="calculatorLOggingAspect">
          <aop:after method="logBefore" pointcut-ref="textOperation">
    </aop:aspect>
</aop:config>
```

**声明引入**     
可以利用<aop:declare-parents>元素在切面内部声明引入。     
```xml
<aop:config>
   
    <aop:aspect id="loggingAspect" ref="calculatorLOggingAspect">
          <aop:after method="logBefore" pointcut-ref="textOperation">
          <aop:declare-parents  
                types-matching="com.desperado.Arithmetic*"
                implement-interface="com.desperado.Mincalculator"
                default-impl="com.desperado.MinCalculatorImpl"/>
    </aop:aspect>
</aop:config>
```
