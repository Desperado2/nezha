#### 1.Spring中的事务管理
- 作为企业级应用程序框架，Spring在不同的事务管理API之上定义了一个抽象层。而应用程序开发人员不必了解底层的事务管理API，就可以使用Spring的事务管理机制。    
- Spring既支持编程式事务管理，也支持声明式的事务管理。   
- 编程事务管理：将事务管理代码嵌入到业务方法中来控制事务的提交和回滚。在编程式管理事务时，必须在每个事务操作中包含额外的事务管理代码。   
- 声明式事务管理：大多数情况下比编程式事务管理更好用。它将事务管理代码从业务方法中分离出来，以声明的方式来实现事务管理。事务管理作为一种横切关注点，可以通过AOP方法模块化。Spring通过SpringAOP框架支持声明式事务管理。    

#### 2.Spring中的事务管理器
- Spring从不同的事务管理API中抽象了一整套的事务机制。开发人员不必了解底层的事务API，就可以利用这些事务机制。有了这些事务机制，事务管理代码就能独立于特定的事务技术了。   
- Spring的核心事务管理抽象是TransactionManager管理封装了一组独立于技术的方法。无论使用Spring的那种事务管理策略(编程式或声明式)，事务管理器都是必须的。    
- DataSourceTransactionManager:在应用程序中只需要处理一个数据源，而且通过JDBC存储。   
- JtaTransactionManager:在JavaEES应用服务器上用JTA进行事务管理。    
- HibernateTransactionManager：用Hibernate框架存取数据库。   
- 事务管理器以普通的Bean形式声明在SpringIOC容器中。     

#### 3.用事务通知声明式地管理事务
- 事务管理是一种横切关注点。   
- 为了在Spring 2.x中启用声明式事务管理，可以通过tx Schema中定义的<tx:advice>元素声明事务通知。为此必须事先将这个Schema定义添加到<beans>根元素中去。    
- 声明了事务通知后，就需要将它与切入点关联起来，由于事务通知是在<aop:config>元素外部声明的，索引它无法直接与切入点产生关联，所以必须在<aop:config>元素中声明一个增强器通知与切入点关联起来。   
- 由于Spring AOP是基于代理的方法，索引只能增强公共方法。因此，只有公有方法才能通过Spring AOP进行事务管理。
```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
         <property name="dataSource" ref="dataSource"></property>
    </bean>
    
    <tx:advice id="bookShopTxAdvice" transaction-manager="transactionManager"></tx:advice>
    
    <aop:config>
        <aop:pointcut id="bookShopOperation" expression="execution(* *.BookShopService.*(..))"></aop:pointcut>
        <aop:advisor advice-ref="bookShopTxAdvice" pointcut-ref="bookShopOperation"/>
    </aop:config>
```

#### 4.用@Transactional注解声明式的管理事务   
- 除了在带有切入点，通知和增强器的Bean配置文件中声明事务外，Spring还允许简单地用@Transactional注解来标注事务方法，。    
- 为了将方法定义为支持事务处理的，可以为方法添加@Transactional注解。根据Spring AOP基于代理机制，只能标注公有方法。    
- 可以在方法或者类级别上添加@Transactional注解。当把这个注解应用到类上时，这个类中的所有公共方法都会被定义成支持事务处理的。  
- 在Bean配置文件中只需要启用<tx:annotation-driven>元素，并为之指定事务处理器就可以了。  
- 如果事务处理器的名称是transactionManager，就可以在<tx:annotation-driven>元素中省略transaction-manager属性。这个元素会自动检测该名称的事务处理器。   
```xml
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    
    <context:component-scan base-package="com.desparado.service"></context:component-scan>
    
    <tx:annotation-driven/>
```

#### 5.事务传播属性
- 当事务方法被另一个事务方法调用时，必须指定事务应该如何传播，例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。    
- 事务的传播行为可以由传输属性指定，Spring定义了7种传播行为。 

| 传播属性      | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| REQUIRED      | 如果有事务在运行，当前的方法就在这个事务内部运行，否则，就启动一个新的事务，并在自己的事务内运行。 |
| REQUIRED_NEW  | 当前的方法必须启动新事务，并在它自己的事务内运行，如果有事务正在运行，应该将它挂起 |
| SUPPORTS      | 如果有事务在运行，当前的方法就在这个事务内运行，否则它可以不允许在事务中 |
| NOT_SUPPORTED | 当前的方法不应该运行在事务中，如果有运行的事务，将它挂起     |
| MANDATORY     | 当期的方法必须运行在事务内部，如果没没有正在运行的事务，就抛出异常 |
| NEVER         | 当前的方法不应该运行在事务中，如果有运行的事务，就跑出异常   |
| NESTED        | 如果有事务在运行，当前的方法就应该在这个事务的嵌套事务内运行，否则，就启动一个新的事务，并在它自己的事务内运行 |
- 事务传播属性的配置
  在@Transactional注解中的属性中进行配置   
```java
@Transactional(propagation=Propagation.REQUIRES_NEW)
```    

- 在xml中可以在<tx:method>中进行配置    

```xml
<tx:advice id="bookShopTxAdvice" transaction-manager="transactionManager">
    <tx:attributes>
          <tx:method name="purchase" propagation="REQUIRES_NEW"/>
    </tx:attributes>
</tx:advice>
    
```


#### 6.设置隔离事务属性
- 用@Transactional注解声明式地管理事务时可以在@Transactional的isolation属性中设置隔离级别

```java
@Transactional(propagation=Propagation.REQUIRES_NEW,isolation=Isolation.READ_COMMITTED)

```

- 在Spring 2.x事务通知中，可以在<tx:method》元素中指定隔离级别

```xml
<tx:advice id="bookShopTxAdvice" transaction-manager="transactionManager">
    <tx:attributes>
          <tx:method name="purchase" propagation="REQUIRES_NEW" 
              isolation="READ_COMMITTED"/>
    </tx:attributes>
</tx:advice>
```

#### 7.设置回滚事务属性
- 默认情况下只有未检查异常(RuntimeException和Error类型的异常)会导致事务回滚，二受检查异常不会。     
- 事务的回滚规则可以通过@Transactional注解的rollbackFor和noRollbackFor属性来定义，这两个属性被声明为Class[]类型的，因此可以为这两个属性指定多个异常类。   
      - rollbackFor：遇到时必须进行回滚。    
      - noRollbackFor：一组异常类，遇到时必须不会滚
 
```java
@Transactional(propagation=Propagation.REQUIRED_NEW,
                isolation=Isolation.READ_COMMITTED,
                rollbackFor=（IOException.class,SQLException.class）,
                noRollbackFor=ArithmeticException.class)  
public void purchase(String isbn,String username){}
```
- 在Spring 2.x事务通知中，可以在<tx:method》原始种指定回滚规则，如果有不止一种异常，用逗号隔开    
 ```xml
<tx:advice id="bookShopTxAdvice" transaction-manager="transactionManager">
    <tx:attributes>
          <tx:method name="purchase" propagation="REQUIRES_NEW" 
              isolation="READ_COMMITTED"
              rollback-for="java.io.IOException,java.sql.SQLException" 
              no-rollback-for="java.lang,ArithmeticException"/>
    </tx:attributes>
</tx:advice>
 ```
 
#### 8.超时和只读属性
- 由于事务可以在行和表上获得锁，因此长事务会占用资源，并对整体性能产生影响。   
- 如果一个事务只读取数据但不做修改，数据库引擎可以对这个事务进行优化。   
- 超时事务属性：事务在强制回滚之前可以保持多久。这样可以防止长期运行的事务占用资源。   
- 只读事务属性：表示这个事务只读取数据库但不更新数据库，这样可以帮助数据库引擎优化事务。 
- 超时和只读属性可以在@Transactional注解中定义，超时属性以秒为单位。   
```java
@Transactional(propagation=Propagation.REQUIRED_NEW,
                isolation=Isolation.READ_COMMITTED,
                rollbackFor=（IOException.class,SQLException.class）,
                noRollbackFor=ArithmeticException.class,
                readOnly=true,tomeout=30)  
public void purchase(String isbn,String username){}
```    

- 在Spring 2.x事务通知中，超时和只读属性可以在<tx:method>元素中进行指定
```xml
<tx:advice id="bookShopTxAdvice" transaction-manager="transactionManager">
    <tx:attributes>
          <tx:method name="purchase" propagation="REQUIRES_NEW" 
              isolation="READ_COMMITTED"
              rollback-for="java.io.IOException,java.sql.SQLException" 
              no-rollback-for="java.lang,ArithmeticException"  
              read-only=true
              timeout=30/>
    </tx:attributes>
</tx:advice>
```