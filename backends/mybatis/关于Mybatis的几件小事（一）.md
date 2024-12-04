### 一、Mybatis简介
#### 1.Mybatis简介
- MyBatis是支持定制化SQL、存储过程以及高级映射的优秀的持久层框架。  
- MyBatis避免了几乎所有的JDBC代码和手动设置参数以及获取结果集。     
- MyBatis可以使用简单的XML或注解用于配置和冤死映射，将接口和Java的POJO(Plain Old Java Objects,普通的java对象)映射成数据库中的记录。

#### 2.为什么使用MyBatis
- MyBatis是一个半自动化的持久化层框架。     
- JDBC存在的问题      
    1. SQL夹在Java代码块中，耦合度高导致硬编码内伤。        
    2. 维护起来不容易且实际开发需求中Sql是有变化的，频繁修改的情况太多。

- Hibernate和JPA存在的问题      
    1. 长难的负载SQL，对于Hibernate而言处理复杂。      
    2. 内部产生的SQL，不容易做特殊优化。     
    3. 基于全映射的全自动框架，大量字段的POJO进行部分映射时比较困难，导致数据库性能下降。    

- 对于开发而言，核心Sql还是需要自己优化的。    
- sql和java代码分开，功能边界清晰，一个专注业务、一个专注数据。

#### 3.SqlSession
- SqlSession的实例不是线程安全的，因此不能被共享。       
- SqlSession每次使用完成后需要正确关闭，这个关闭操作是必须的。     
- SqlSession可以直接调用方法的id进行数据库操作，但是一般还是推荐使用SqlSession获取到DAO接口的代理，执行代理对象的方法，可以更安全的进行类型检查操作。

### 二、全局配置文件
- MyBatis的配置文件包含了影响MyBatis行为的设置和属性信息，文档的顶层结构如下：    
    - configuration 配置
        - properties属性
        - settings 属性
        - typeAliases属性
        - typeHandler类型处理器
        - objectFactory对象工厂
        - plugins插件
        - environments环境
            - environment环境变量
                - transactionManager 事务管理器
                - dataSource 数据源
        - databaseIdProvider 数据库厂商标识
        - mappers映射器

#### 1.properties属性
dbconfig.properties文件内容
```properties
driver=com.mysql.jdbc.cj.Driver
url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8   
username=root
password=root
```
在全局配置文件中进行如下配置      
```xml
<properties resource="dbconfig.properties"></properties>
```
- 如果属性在不止一个地方进行了配置，那么MyBatis将按照下面的顺序来加载：    
     - 在properties元素体内指定的属性首先被读取。      
    - 然后根据properties元素中的resource属性读取类路径下属性文件或根据url属性指定的路径读取属性文件，并覆盖已读取的同名属性。      
    - 最后读取作为方法参数传递的属性，并覆盖以读取的同名属性。      

#### 2.settings设置   
- 这是MyBatis中极为重要的参数设置，会改变MyBatis的运行时行为。     
```xml
<settings>
      <setting name="mapUnderscoreToCamelCase" value="true" />
</settings>
```

设置参数|描述|有效值|默认值
:-|:-|:-|:-
cacheEnabled|改配置影响的所有映射器中配置的缓存的全局开关|true \| false |true
lazyLoadingEnabled|延迟加载的全局开关，开启后，所有关联的对象都会延时加载。特定关联关系中可通过设置fetchType属性来覆盖本配置|true \| false | false
useColumnLabel|使用列标签代替列名。不同的驱动在这方面会有不同的表现，具体可参考相关驱动文档或通过测试这两种不同的模式来观察所用驱动的结果| true \| false | true
defaultStatementTimeout|设置超时时间，决定驱动等待数据库响应的秒数| 任意合理的int值|null(不进行限制)   
mapUnderscoreToCamelCase|是否开启自动驼峰命名规则映射| true \| false | false

#### 3.typeAliases别名处理器
- 类型别名是为Java类型设置一个短的名字，可以方便我们引用某个类    
```xml
<typeAliases>
    <typeAlias type="com.desperado.entity.Employee" alias="employee"/>
    <typeAlias type="com.desperado.entity.Departemnt" alias="department"/>
</typeAliases>
```
- 类很多的情况下，可以批量设置别名这个包下的每一个类创建一个默认的别名，就是简单类名小写。    
```xml
<typeAliases>
      <package name="com.desperado.entity" /> 
</typeAliases>
```

- 也可以使用@Alias注解为其指定一个别名
```java
@Alias("emp")
public class Employee{}
```

-MyBatis已经为许多常见的Java类型内建了相应的别名，他们都是大小写不敏感的，起别名时要避免这些。    
别名|映射的类型名|别名|映射的类型名|别名|映射的类型名
-|-|-|-|-|-
_byte|byte|string|String|data|Data
_long|long|byte|Byte|decimal|Decimal
_short|short|long|Long|bigdecimal|BigDecimal
_int|int|short|Short|object|Object
_integer|int|int|Integer|map|Map
_double|double|integer|Integer|hashmap|HashMap
_float|float|double|Double|list|List
_boolean|boolean|float|Float|arraylist|ArrayList
-| |boolean|Boolean|collection|Collection
- ||||iterator|Iterator


#### 4.typeHandlers类型处理器
无论MyBatis在预处理语句(PreparedStatement)中设置一个参数时，还是从结果集中取出一个值时，都会用类型处理器将获取的值以合适的方式转换成Java类型。    
类型处理器|Java类型|JDBC类型
-|-|-
BooleanTypeHandler|java.lang.Boolean,boolean|数据库兼容的BOOLEAN
ByteTypeHandler|java.lang.Byte,byte|数据库兼容的NUMERIC或BYTE
ShortTypeHandler|java.lang.Short,short|数据库兼容的NUMERIC或SHORT INTEGER
IntegerTypeHandler|java.lang.Integer,int|数据库兼容的NUMERIC或INTEGER
LongTypeHandler|java.lang.Long,long|数据库兼容的NUMERIC或LONG INTEGER
FloatTypeHandler|java.lang.Float,float|数据库兼容的NUMERIC或FLOAT
DoubleTypeHandler|java.lang.Double,double|数据库兼容的NUMERIC或DOUBLE
BigDecimalTypeHandler|java.math.BigDecimal|数据库兼容的NUMERIC或DECIMAL
StringTypeHandler|java.lang.String |CHAR、VARCHAR

#### 5.日期类型的处理
- 在JDK1.8之前，通过使用JSR310规范的Joda-Time来操作。1.8已经实现全部的JSR310规范了。     
- 日期时间处理上，可以使用MyBatis基于JSR310编写的各种时间类型处理器
- MyBatis3.4以前的版本需要我们收到去注册这些处理器，以后的版本会自动注册。  
 
```xml
<typeHandlers>
    <typeHandler handler="org.apache.ibatis.type.InstantTypeHandler"/>
    <typeHandler handler="org.apache.ibatis.type.LocalDateTimeTypeHandler"/>
    <typeHandler handler="org.apache.ibatis.type.LocalDateTypeHandler"/>
    <typeHandler handler="org.apache.ibatis.type.LocalTimeTypeHandler"/>
    <typeHandler handler="org.apache.ibatis.type.OffsetDateTimeTypeHandler"/>
    <typeHandler handler="org.apache.ibatis.type.OffsetTimeTypeHandler"/>
    <typeHandler handler="org.apache.ibatis.type.ZonedDateTimeTypeHandler"/>
    <typeHandler handler="org.apache.ibatis.type.YearTypeHandler"/>
    <typeHandler handler="org.apache.ibatis.type.MonthTypeHandler"/>
    <typeHandler handler="org.apache.ibatis.type.YearMonthTypeHandler"/>
    <typeHandler handler="org.apache.ibatis.type.JapaneseDateTypeHandler"/>
</typeHandlers>
```

#### 6.自定义类型处理器
- 可以重写类型处理器或创建自己的类型处理器来处理不支持的或非标准的类型。
- 创建一个类型处理器如下：   
    1. 实现org.apache.ibatis.type.TypeHandler接口或者继承org.apache.ibatis.type.BaseTypeHandler
    2. 指定其映射某个JDBC类型(可选操作)
    3. 在mybatis全局配置文件中注册。

#### 7.plugins插件
- 插件是mybatis提供的一个非常强大的机制，我们可以通过插件来修改mybatis的一些核心机制。插件通过动态机制代理，可以介入四大对象的任何一个方法的执行。   
    - Executor(update,query,flushStatements,commit,rollback,getTransaction,close,isClosed)
    - ParameterHandler(getParameterObject,setParameters)

    - ResultSetHandler(handlerResultSets,handlerOuutputParameters)

    - StatementHandler(prepare,parameterize,batch,update,query)

#### 8.environments环境
- MyBatis可以配置多种环境，比如开发、测试和生产环境需要有不同的配置。   
- 每种环境使用一个environment标签配置并指定唯一标志符。   
- 可以通过environments标签中的default属性指定一个环境的标识符来快速的切换环境。   

**1.指定具体环境**   
- id：指定当前环境的唯一标识
- transactionManager和dataSource必须都要有

```xml
<environments default="development">
    <environment id="development">
          <transactionManager type="JDBC" />
          <dataSource type="POOLED">
                  <property name="driver" value="${dept.driver}" / >
                  <property name="url" value="${dept.url} "/ >
                  <property name="username" value="${dept.username}" / >
                  <property name="password" value="${dept.password}" / >
          <dataSource>
    </environmet>

    <environment id="test">
          <transactionManager type="JDBC" />
          <dataSource type="POOLED">
                  <property name="driver" value="${test.driver}" / >
                  <property name="url" value="${test.url} "/ >
                  <property name="username" value="${test.username}" / >
                  <property name="password" value="${test.password}" / >
          <dataSource>
    </environmet>
</environments>
```

**2.transactionManager**
type值取值有三种类型 JDBC|MANAGED|自定义
- JDBC：使用了JDBC的事务提交和回滚设置，依赖于从数据源得到的连接来管理事务范围。对应JdbcTransactionFactory    
- MANAGED：不提交或回滚一个连接，让容器来管理事务的整个生命周期。对应ManagedTransactionFactory
- 自定义：实现TransactionFactory接口，type=全类名/别名。

**3.dataSource**
type有四种取值 UNPOOLED|POOLED|JNDI|自定义
- UNPOOLED：不使用连接池，对应UnpooledDataSourceFactory。   
- POOLED：使用连接池，对应PooledDataSourceFactory。  
- JNDI：在EJB或应用服务器这类容器中查找指定的数据源。   
- 自定义：实现DataSourceFactory接口，定义数据源的获取方式。

#### 9.databaseIdProvider环境
- MyBatis可以根据不同的数据库厂商执行不同的语句。   
  
```xml
<databaseIdProvider type="DB_VENDOR">
      <property name="MySQL" value="mysql"/>
      <property name="Oracle" vale="oracle"/>
      <property name="SQL Server" value="sqlserver" />
</databaseIdProvider>

<select Id="getUserById" result="User"
        parameterType="Integer" databaseId="mysql">
      select * from user where id =#{id}
</select>
```

- Type:DB_VENDOR:使用MyBatis提供的VendorDatabaseIdProvider解析数据库厂商标识。也可以实现DatabaseIdProvider接口来自定义。
- Property- name：数据库厂商标识。
- property-value：为标识起一个别名，方便SQL语句使用DatabaseId属性引用。
- DB_VENDOR 会通过DatabaseMetaData类的getDatabaseProductName()返回的字符串进行设置。由于通常情况下这个字符串都非常长而且相同产品的不同版本会返回不同的值，所以最好通过设置属性别名来使其变短。   
- MyBatis匹配规则如下:  
    1. 如果没有配置databaseIdProvider标签，那么databaseId=null。   
    2. 如果配置了databaseIdProvider标签，使用标签配置的name去匹配数据库信息，匹配上设置databaseId=属性指定的值，否则依旧为null。
    3. 如果databaseId不为null，他只会找到配置databaseId的sql语句。    
    4. mybatis会加载不带databaseId属性和带有匹配当前数据库databaseId属性的所有语句。如果同时找到带有databaseId和不带databaseId的相同语句，则后者会被舍弃。

#### 10.mapper映射
- mapper逐个注册SQL映射文件

```xml
<mappers>
    <mapper resource="mybatis/mapper/PersonDao.xml" />
    <mapper url="file:///D:/userDao.xml" />
    <mapper class="com.desperado.dao.UserDao" />
</mappers>
 ```   
- 使用批量注册。这种方式要求SQL映射文件名必须和接口名相同并且在同一目录下。

```xml
<mappers>
    <package name="com.desperado.dao" />
</mappers>
```

### 三、映射文件
映射文件指导着MyBatis如何进行数据库的增删改查，有着非常重要的意义：
   - cache 命名空间的二级缓存配置。
   - cache-ref 其他命名空间缓存配置的引用。
   - resultMap 自定义结果集映射。
   - parameterMap 已废弃，老式风格的参数映射
   - sql 抽取可重用语句块
   - insert 映射插入语句
   - update 映射更新语句
   - delete 映射删除语句
   - select 映射查询语句

#### 1.insert、update、delete元素
属性|说明
:-|:-
id|命名空间中的唯一标志符
parameterType|将要传入语句的参数的完全限定类名或别名。这个属性是可选的，因为MyBatis可以通过TypeHandler推断出具体传入语句的参数类型，默认为unset。
flushCache|将其设置为true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都被清空，默认值true(对应插入、删除和更新语句)。
timeout|这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认为unset(依赖驱动)
statementType|STATEMENT，PREPARED或CALLABLE的一个。这会让MyBatis分别饰演Statement、PreparedStatement或CallableStatement，默认值：PREPARED
useGeneratorKeys|(仅对insert和update有效)这会让MyBatis使用JDBC的getGeneratorKeys方法取出由数据库内部生成的主键，默认值false
keyProperty|(仅对insert和update有效)唯一标记一个属性，MyBatis会通过getGeneratedKeys的返回值或者通过insert语句的selectKey子元素设置它的键值，默认unset
keyColumn|(仅对insert和update有效)通过生成的键值设置表中的列名，这个设置紧张模型数据库是必须的，当主键列不是表中的第一列的时候需要设置，如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。
databaseId|如果配置了databaseIdProvider，MyBatis会加载所有的不带databaseId或匹配当前databaseId的语句；如果带或者不带的语句都有，则不带的会被忽略。    

#### 2.主键生成方式
- 若数据库支持自动生成主键的字段(比如mysql和SQL server)，则可以设置useGeneratedKeys="true",然后再把keyProperty设置到目标属性上。   
```xml
<insert id="insertCustomer" databaseId="mysql"     
    useGeneratedKeys="true" keyProperty="id">
    insert into customer(name,age,email) values (#{name},#{age},#{email})
</insert>
```

-  对于不支持自增型主键的数据库(例如：oracle)，则可以使用selectKey子元素，selectKey元素会首先运行，id会被设置，然后插入语句会被调用。
```xml
<insert id="insertCustomer" databaseId="oracle" parameterType=“customer”>

    <selectKey order="BEFORE" keyProperty="id" resultType="_int" >
          select crm_seq.nextval from dual
    </selectKey>
    insert into customer(id,name,age,email) values (#{id},#{name},#{age},#{email})
</insert>
```
- SelectKey

属性|说明
:-|:-
keyProperty|selectKey语句结果应该被设置的目标属性
keyColumn|匹配属性的返回结果集中的列名称
resultType|结果的类型，MyBatis通常可以推断出来，但是为了更加确定写上也不会有什么问题。MyBatis允许任何简单类型用作主键的类型，包括字符串。
order|可以被设置为BEFORE或AFTER，如果设置为BEFORE，那么它会首先选择主键，设置keyProperty然后执行插入语句。如果设置为After，那么先执行插入语句，然后是selectKey执行。
statementType|与前面相同，MyBatis支持STATEMENT，PREPARED和CALLABLE语句的映射类型，分布代表PreparedStatement和CallableStatement

#### 3.参数传递
- 单个参数：可以接受级别类型、对象类型、集合类型的值。这种情况下MyBatis可以直接使用这个参数，不需要做其他处理。    
- 多个参数：任意多个参数，都会被MyBatis重新包装成一个Map传入。map的key是param1、param2等等，值就是传入的参数值。
- 命名参数：为参数使用@Param注解起一个名字，Mybatis会将这些参数封装进map时，使用指定的名字作为key
- POJO：当这些参数属于POJO时，可以直接传递POJO
- Map：也可以封装多个参数为map，直接传递map。

#### 4.参数处理
- 参数也可以指定一个特殊的数据类型：   

```xml
#{property,javaType=int,jdbcType=NUMERIC}

#{height,javaType=double,jdbcType=NUMERIC,numericScale=2}
```

   -javaType通常可以从参数对象中去确定。
   -如果null被当做值来传递，对应所有可能为空的列，jdbcType需要被设置。 
    -对于数值类型，还可以设置小数点后保留的位数
    -mode属性允许指定IN、OUT和INOUT属性。如果参数为OUT或INOUT，参数对象属性的真实值会被改变，就像在获取输出参数时所期望的那些。

- 参数位置支持的属性       
javaType、jdbcType、mode、numericScale、resultMap、typeHandler、jdbcTypeName

- \#{key}：获取参数的值，预编译到SQL中，安全。    
- ${key}：获取参数的值，拼接到SQL中，有SQL注入问题。

#### 5.select语句
属性|说明
:-|:-
id|命名空间中的唯一标志符
parameterType|将要传入语句的参数的完全限定类名或别名。这个属性是可选的，因为MyBatis可以通过TypeHandler推断出具体传入语句的参数类型，默认为unset。
resultType|返回的期望类型的完全限定名或别名。注意如果是集合，那应该是集合可以包含的类型，而不是集合本身，本属性和resultMap不能同时使用。
flushCache|将其设置为true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都被清空，默认值false。
useCache|将其设置为true，将会导致本条语句的结果被二级缓存进行缓存，默认值：true
timeout|这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认为unset(依赖驱动)
fetchSize|影响驱动程序每次批量返回的结果行数。默认值unset(依赖驱动)
statementType|STATEMENT，PREPARED或CALLABLE的一个。这会让MyBatis分别饰演Statement、PreparedStatement或CallableStatement，默认值：PREPARED
resultSetType|FORWARD_ONLY、SCROLL_SENSITIVE或者SCROLL_INSENSITIVE中的一个，默认值为unset(依赖驱动)
databaseId|如果配置了databaseIdProvider，MyBatis会加载所有的不带databaseId或匹配当前databaseId的语句；如果带或者不带的语句都有，则不带的会被忽略。  
resultOrdered|这个设置仅针对嵌套结果select语句适用；如果为true，就假设包含了嵌套结果集或是分组，这样当返回一个主结果行，就不会发生有对前面结果集引用的情况，这就使得在获取嵌套的结果集的时候不至于导致内存不足，默认值false
resultSets|这个设置仅对多结果集的情况适用，它将雷人语句执行后返回的结果集并给每个结果集一个名称，名称是逗号分隔的。

#### 6.自动映射
- 全局setting设置   
      -autoMappingBehavior默认是PARTIAL，开启自动映射的功能。唯一的要求是列名和javaBean属性名一致。    
      -如果autoMappingBehavior设置为null会取消自动映射。       
      -数据库命名规范，POJO属性符号驼峰命名法，如customer_name -> customerName,可以改期自动驼峰命名规则映射功能。       

- 自定义resultMap，实现高级结果集映射。

#### 7.resultMap
- constructor  类在实例化时，用来注入结果到构造方法中。    
      -idArg  id参数，标记结果作为id可以提高整体性能。          
      -arg   注入到构造方法的一个普通结果

- id   一个ID结果，标记结果作为id可以帮助提高整体性能。     
- result  注入到字段或JavaBean属性的普通结果
- association   一个复杂的类型关联；许多结果将包成这种类型。       
        -嵌入结果映射，结果映射自身的关联。
- collection  复杂类型的集。     
        -嵌入结果映射，结果映射自身的集。
- discriminator  使用结果值来决定使用哪个结果映射。  
        -case基于某些值的结果映射。

#### 8.Id&Result
- id和result映射一个单独列的值到简单数据类型的属性或字段。    

属性|说明
:-|:-
property|映射到列结果的字段或属性。
column|数据表的列名
javaType|一个Java类的完全限定名或一个类型别名
jdbcType|JDBC类型是仅仅需要对插入、更新和删除操作可能为空的列进行处理
typeHandler|类型处理器

#### 9.association
- 复杂对象类型
- POJO中的属性可能会是一个对象
- 可以使用联合查询，并以级联属性的方式封装对象。 
```xml
<resultMap type="com.desperado.bean.Lock" id="myLock">
    <id column="id" property="id" />
    <id column="lockName" property="lockName" />
    <id column="key_id" property="key.id" />
    <id column="keyName" property="key.keyName" />
</resultMap>
```
- 使用association标签定义对象的封装规则
- association嵌套结果集
```xml
<resultMap type="com.desperado.bean.Lock" id="myLock2">
    <id column="id" property="id" />
    <result column="lockName" property="lockName" />
    <association property="key" javaType="com.desperado.bean.Key">
        <id column="key_id" property="id" />
        <result column="keyName" property="keyName" />
    </association>
</resultMap>

```

- association分段查询     
    -select:调用目标的方法查询当前属性的值
    -column：将指定列的值传入目标方法

```xml
<resultMap type="com.desperado.bean.Lock" id="myLock3">
    <id column="id" property="id" />
    <result column="lockName" property="lockName" />
    <association property="key" 
        select="com.desperado.dao.KeyMapper.getKeyById"  
        column="key_id">
    </association>
</resultMap>
```

- association分段查询和延迟查询
开启延迟加载和属性按需加载       
```xml
<settings>
    <select name="lazyLoadingEnabled" value="true" />
    <setting name="aggressiveLazyLoading" value="false"  />
</settings>
```

#### 10.Collection
- 集合类型&嵌套结果集
-不使用collection进行联合查询
```xml
<select id="getDeptById" resultMap="MyDept">
      select d.id d_id,
             d.dept_name d_deptName,
             e.id e_id,
             e.last_name e_lastName,
             e.email e_email,
             e.gender e_gender,
             e.dept_id e_deptId
      from depertment d 
      left join employee e on e.dept_id = d.id
      where d.id = #{id}
</select>
```
  -使用collection之后
```xml
<resultMap type="com.desperado.bean.Department" id="MyDept">
    <id column="d_id" property="id" />
    <result column="d_deptName" property="deptName" />
    <collection property="emps" 
        ofType="com.desperado.bean.Employee"  
        columnPrefix="e_">
         <id column="id" property="id" />
         <id column="lastName" property="lastName" />
         <id column="email" property="email" />
         <id column="gender" property="gender"/>
    </collection>
</resultMap>
```

- 分布查询&延迟加载
```xml
<resultMap type="com.desperado.bean.Department" id="MyDeptStep">
    <id column="id" property="id" />
    <result column="dept_name" property="deptName" />
    <collection property="emps" 
        select="com.desperado.dao.EmployeeMapper.getEmpsByDeptId"  
        column="id">
    </collection>
</resultMap>
```

#### 11.扩展-多列值封装map传递
- 分步查询的时候通过指定column，将对应的列分数据传递过去，有时候需要传递多个参数。 
- 使用{key1=column1,key2=column2}的形式。  
```xml
<resultMap type="com.desperado.bean.Department" id="MyDeptStep">
    <id column="id" property="id" />
    <result column="dept_name" property="deptName" />
    <collection property="emps" 
        select="com.desperado.dao.EmployeeMapper.getEmpsByDeptId"  
        column="{deptId=id}">
    </collection>
</resultMap>
```
- association或者collection标签的fetchType=eager/lazy可以覆盖全局的延迟加载策略，指定理解加载(eager)或者延迟加载(lazy)

### 四、动态SQL
#### 1.动态SQL简介
- 动态SQL是MyBatis强大特性之一。极大的简化拼装SQL的操作。   
- 动态SQL元素和使用JSTL或其它类似基于XML的文本处理器相似。   
- MyBatis采用功能强大的基于OGNL的表达式来简化操作。   

#### 2.Multi-db vendor 支持
- 若在mybatis配置文件中配置了databaseIdProvider，则可以使用"_databaseId"变量，这样就可以根据不同的数据库厂商构建特定的语句
```xml
<databaseIdProvider type="DB_VENDOR">
    <property name="MySQL" value="mysql" />
    <property name="Oracle" value="oracle" />
</databaseIdProvider>
```

```xml
<select id="getEmpPage" resultType="com.desperado.bean.Employee">
       <if test="_databaseId =='mysql' ">
          select * from employee where limit 0,5
       </if>

       <if test="_databaseId == 'oracle' ">
            select * from (select e.*,rownum as r1 from employee e where rownum &lt;=5) where r1 &gt;= 1;
       </if>  
</select>
```




