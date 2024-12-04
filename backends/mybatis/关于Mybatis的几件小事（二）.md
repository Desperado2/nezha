### 一、MyBatis缓存机制
#### 1.简介
- Mybatis包含了一个非常强大的查询缓存的特性，它可以非常方便地配置和定制。
- 缓存key极大提高查询效率
- MyBatis系统中默认定义了两次缓存
- 默认情况下，只有一级缓存(SqlSession级别的缓存，也称为本地缓存)开启。  
- 二级缓存需要手动开启和配置，它是基于namespace级别的缓存。
- 为了提高扩展性，MyBatis定义了缓存接口cache。可以通过实现Cache接口来自定义二级缓存。

#### 2.一级缓存
- 一级缓存，即本地缓存，作用域默认为SqlSession。当Session flush或close后，该Session中的所有Cache将被清空。   
- 本地缓存不能被关闭，但可以调用clearCache()来清空本地缓存，或者改变缓存的作用域。  
- 在mybatis3.1之后，可以配置本地缓存的作用域，在mybatis.xml中配置localCacheScope属性。
-MyBatis利用本地缓存防止循环引用和加速重复嵌套查询，默认值为SESSION，这种情况下会缓存一个会话中执行的所有查询。若设置值为STATEMENT，本地会话仅用在语句执行上，对相同SqlSession的不同调用将不会共享数据。
- 同一个会话期间只要查询过的数据都会被保存在当前SqlSession的一个Map中
      - key：hashCode+查询的SqlId+编写的sql查询语句+参数
	  
#### 3.一级缓存失效情况
- 不同的SqlSession对应不同的一级缓存
- 同一个SqlSession但是查询条件不同
- 同一个SqlSession两次执行期间执行了任何一次增删改操作。
- 同一个SqlSession两次查询期间手动清空了缓存。

#### 4.二级缓存
- 二级缓存，全局作用域缓存
- 二级缓存默认不开腔，需要手动配置。
- MyBatis提供二级缓存的接口以及实现，缓存实现要求POJO实现Serializable接口。
- 二级缓存在SqlSession关闭或提交之后才会生效。
- 使用步骤：
    1. 全局配置文件中开启二级缓存
    2. 需要使用二级缓存的映射文件处使用cache配置缓存
    3. POJO实现Serializable接口。

#### 5.缓存相关属性
- eviction="FIFO" ：缓存回收策略
        - LRU(最近最少使用)：移除最长时间不被使用的对象。
        - FIFO(先进先出)：按对象进入缓存的顺序来移除对象。
        - SOFT(软引用)：移除基于垃圾回收期状态和软引用规则的对象。   
        - WEAK(弱引用)：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
        - 默认的是LRU

- flushInterval：刷新间隔，单位毫秒
        - 默认情况下不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新。

- size：引用数目，正整数
        - 代表缓存最多可以存储多少个对象，太大容易导致内存溢出

- readOnly：只读，true/false
        - true：只读缓存；会给所有调用者返回缓存对象的相同实例。这些对象不能被修改。这提供了很重要的性能优势。        
        - false：读写缓存，会返回缓存对象的拷贝(通过序列化)。这会慢一些，但是安全，默认为false。

#### 6.缓存设置
- 全局setting的cacheEnable
      - 配置二级缓存的开关，一级缓存一直是打开的。

- select标签的useCache属性
      - 配置这个select是否使用二级缓存。一级缓存一直是使用的。

- sql标签的flushCache属性：
       - 增删改默认为true，sql执行后，会清空一级和二级缓存。查询默认false。

- sqlSession.claerCache()
        - 用来清除一级缓存

- 当在某一个作用域(一级缓存/二级缓存/Namespace)进行了增删改操作化，默认该作用域下所有select中的缓存将被clear。

#### 7.使用第三方缓存之后的查询流程
1. 查询二级缓存。     
2. 如果二级缓存没有，查询一级缓存。    
3. 一级缓存也没有，查询数据库。

![image.png](/image/spring/m1-1.png)

### 二、MyBatis工作原理示意图

![image.png](/image/spring/m1-2.png)

### 三、MyBatis插件开发
#### 1.MyBatis插件
- MyBatis在四大对象的创建过程中，都会有插件进行介入，插件可以利用动态代理机制一层层的包装目标对象，而实现在目标对象指向目标方法之前进行拦截的效果。
- MyBatis允许在已映射语句执行过程中的某一点进行拦截调用。  
- 默认情况下，MyBatis允许使用插件来拦截的方法包括：
Executor(update,query,flushStatements,commit,rollback,getTransaction,close,isClosed)         
       ParameterHandler(getParameterObject,setParameters) 
 ResultSetHandler(handlerResultSets,handlerOuutputParameters) 
 StatementHandler(prepare,parameterize,batch,update,query)

#### 2.插件开发步骤
1. 编写插件实现Interceptor接口，并使用@Intercepts注解完成插件签名。    
```java
@Intercepts({
    @Signature(type=StatementHandler.class,method="prepare",
              args={Connection.class})
})
public class MyPlugin implements Interceptor{}
```

2. 在全局配置文件中注册插件
```xml
<plugins>
    <plugin interceptor="com.desperado.plugin.MyPlugin">
          <property  name="username" value="tomcat"/>
     </plugin>
</plugins>
```

#### 3.插件原理
1. 按照插件注册声明，按照插件配置顺序调用插件plugin方法，生成被拦截对象的动态代理。     
2. 多个插件依次生成目标对象的代理对象，层层包裹，先声明的先包裹，形成代理链。     
3. 目标想法执行时依次从外到内执行intercept方法。     
4. 多个情况下，我们往往需要在某个插件中分离出来目标对象，可以借助MyBatis提供的SystemMateObject类来进行获取最后一层的h以及target属性的值。

#### 4.Interceptor接口
- intercept：拦截目标方法执行。   
- plugin：生成动态代理对象，可以使用MyBatis提供的Plugin类的wrap方法。
- setProperties：注入插件配置时设置的属性。   

#### 5. 常用代码
```java
//1.分离代理对象。由于会形成多次代理，所以需要通过while循环分离出最终被代理的对象，从而方便提取信息。
MetaObject metaObject = SystemMetaObject.forObject(target);
while(metaObject.hasGetter("h")){
      Object h = metaObject.getValue("h");
      metaObject = SystemMetaObject.forObject(h);
}
//2.获取到代理对象中包含的被代理的真实对象
Object obj = metaObject.getValue("target");
//3.获取被代理对象的MetaObject方便进行信息提取
metaObject forObject  = SystemMetaObject.forObject(obj);
```
### 四、其他操作
#### 1.分页操作
- 分页可以使用PageHelper插件。
- 使用步骤：
        - 导入相关的依赖，pagehelper.jar和jsqlparser.jar.       
        - 在Mybatis全局配置文件中配置分页插件      
        - 使用PageHelper提供的方法进行分页       
        - 可以使用更强大的PageInfo封装返回结果。     

#### 2.批量操作
- 默认的openSession()方法没有参数，使用它创建的SqlSession具有如下特性：
      - 会开启一个事务(也就是 不自动提交)         
      - 连接对象会从由活动环境配置的数据源实例得到。      
      - 事务隔离级别将会使用驱动或数据源的默认设置。       
      - 预处理语句不会被复用，也不会批量处理更新。    

- openSession方法的ExecutorType类型的参数，枚举类型，取值如下：
        - SIMPLE：这个执行器类型不做特殊的事情(这是默认装配的)。它为每个语句的执行创建一个新的预处理语句。    
        - REUSE：这个执行器类型会复用预处理语句。       
        - BATCH：这个执行器会批量执行所有更新语句。       

- 批量操作就是使用MyBatis提供的BATCH类型的Executor进行的，它的底层就是通过暂存sql的方式进行的。可以保存sql到一定数量后发给数据库执行一次。

- 与Spring整合中。推荐额外配置一个可以专门用来执行批量操作的sqlSession，需要使用的时候，注入配置的批量操作的SqlSession，通过它获取到mapper映射器进行操作。  

- 批量操作是在session.commit()以后才发送sql语句给数据库进行执行的。  
- 途观想让其提前执行，可以使用sqlSession.flushStatements()方法，让其执行刷到数据库中进行执行。

#### 3.存储过程
- 调用存储过程
1. select标签中statementType="CALLABLE". 
2. 标签体中调用语法 
```xml
<select id="callProcedure" statementType="CALLABLE">
    call procedure_name(#{param1},#{param2})
</select>
```
- mybatis对存储过程的游标提供了一个JdbcType=CURSOR的支持，可以智能的把游标读取到的数据，映射到我们声明的结果集中。

#### 4.自定义TypeHandler处理枚举
- 可以通过自定义TypeHandler的形式来设置参数或者取出结果集的时候自定义参数封装策略。  
- 步骤：
      1. 实现TypeHandler接口或者继承BaseTypeHandler。     
      2. 使用@MappedType定义处理的Java类型，使用@MappedJdbcTypes定义jdbcType类型。
      3. 在定义结果集标签或者参数处理的时候声明使用自定义TypeHandler进行处理或者在全局配置TypeHandler要处理的javaType
 
