## 一、JDBC方式
1. 引入starter。       

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
```

2. 配置application.properties 

```properties
spring.datasource.driver-class-name=com.mysql.jdbc.cj.Driver
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test
spring.datasource.username=root
spring.datasource.password=root
```

3. 配置后默认使用org.apache.tomcat.jdbc.pool.DataSource作为数据源；数据源的相关配置都在org.springframework.boot.autoconfigure.jdbc.DataSourceProperties里面。

```java
@ConfigurationProperties(
    prefix = "spring.datasource"
)
public class DataSourceProperties implements BeanClassLoaderAware, InitializingBean {
    private ClassLoader classLoader;
    private String name;
    private boolean generateUniqueName;
    private Class<? extends DataSource> type;
    private String driverClassName;
    private String url;
    private String username;
    private String password;
    private String jndiName;
    private DataSourceInitializationMode initializationMode;
    private String platform;
    private List<String> schema;
    private String schemaUsername;
    private String schemaPassword;
    private List<String> data;
    private String dataUsername;
    private String dataPassword;
    private boolean continueOnError;
    private String separator;
    private Charset sqlScriptEncoding;
    private EmbeddedDatabaseConnection embeddedDatabaseConnection;
    private DataSourceProperties.Xa xa;
    private String uniqueName;
.....
}
```

4. 自动配置原理        
根据org.springframework.boot.autoconfigure.jdbc.DataSourceConfiguration，根据配置去创建数据源，默认使用tomcat连接池。        

5. SpringBoot默认支持的数据源类型：     
  
  - "com.zaxxer.hikari.HikariDataSource",    
  - "org.apache.tomcat.jdbc.pool.DataSource",    
  - "org.apache.commons.dbcp2.BasicDataSource"

6. 可以使用spring.datasource.type指定数据源类型。因为springboot在创建数据源的时候就是根据这个来选择要创建的数据源的类型的。

```java
abstract class DataSourceConfiguration {
    DataSourceConfiguration() {
    }

    protected static <T> T createDataSource(DataSourceProperties properties, Class<? extends DataSource> type) {
        return properties.initializeDataSourceBuilder().type(type).build();
    }

    @Configuration
    @ConditionalOnMissingBean({DataSource.class})
    @ConditionalOnProperty(
        name = {"spring.datasource.type"}
    )
    static class Generic {
        Generic() {
        }

        @Bean
        public DataSource dataSource(DataSourceProperties properties) {
            return properties.initializeDataSourceBuilder().build();
        }
    }

    @Configuration
    @ConditionalOnClass({BasicDataSource.class})
    @ConditionalOnMissingBean({DataSource.class})
    @ConditionalOnProperty(
        name = {"spring.datasource.type"},
        havingValue = "org.apache.commons.dbcp2.BasicDataSource",
        matchIfMissing = true
    )
    static class Dbcp2 {
        Dbcp2() {
        }

        @Bean
        @ConfigurationProperties(
            prefix = "spring.datasource.dbcp2"
        )
        public BasicDataSource dataSource(DataSourceProperties properties) {
            return (BasicDataSource)DataSourceConfiguration.createDataSource(properties, BasicDataSource.class);
        }
    }

    @Configuration
    @ConditionalOnClass({HikariDataSource.class})
    @ConditionalOnMissingBean({DataSource.class})
    @ConditionalOnProperty(
        name = {"spring.datasource.type"},
        havingValue = "com.zaxxer.hikari.HikariDataSource",
        matchIfMissing = true
    )
    static class Hikari {
        Hikari() {
        }

        @Bean
        @ConfigurationProperties(
            prefix = "spring.datasource.hikari"
        )
        public HikariDataSource dataSource(DataSourceProperties properties) {
            HikariDataSource dataSource = (HikariDataSource)DataSourceConfiguration.createDataSource(properties, HikariDataSource.class);
            if (StringUtils.hasText(properties.getName())) {
                dataSource.setPoolName(properties.getName());
            }

            return dataSource;
        }
    }

    @Configuration
    @ConditionalOnClass({org.apache.tomcat.jdbc.pool.DataSource.class})
    @ConditionalOnMissingBean({DataSource.class})
    @ConditionalOnProperty(
        name = {"spring.datasource.type"},
        havingValue = "org.apache.tomcat.jdbc.pool.DataSource",
        matchIfMissing = true
    )
    static class Tomcat {
        Tomcat() {
        }

        @Bean
        @ConfigurationProperties(
            prefix = "spring.datasource.tomcat"
        )
        public org.apache.tomcat.jdbc.pool.DataSource dataSource(DataSourceProperties properties) {
            org.apache.tomcat.jdbc.pool.DataSource dataSource = (org.apache.tomcat.jdbc.pool.DataSource)DataSourceConfiguration.createDataSource(properties, org.apache.tomcat.jdbc.pool.DataSource.class);
            DatabaseDriver databaseDriver = DatabaseDriver.fromJdbcUrl(properties.determineUrl());
            String validationQuery = databaseDriver.getValidationQuery();
            if (validationQuery != null) {
                dataSource.setTestOnBorrow(true);
                dataSource.setValidationQuery(validationQuery);
            }

            return dataSource;
        }
    }
}
```

7. 自定义数据源类型。      

```java
    @ConditionalOnMissingBean(DataSource.class)
    @ConditionalOnProperty(name="spring.datasource.type")
    static class Generic{

        @Bean
        public DataSource dataSource(DataSourceProperties properties){
            //使用DataSourceBuilder创建数据源，利用反射创建响应type的数据源，并绑定相关属性
            return properties.initializeDataSourceBuilder().build();
        }
    }
```

8. 自动运行建表语句原理         
自动运行建表语句依赖于org.springframework.boot.autoconfigure.jdbc.DataSourceInitializer这个类，在应用启动的时候，会去寻找字段建表语句并运行。         

```java

class DataSourceInitializer {
    private static final Log logger = LogFactory.getLog(DataSourceInitializer.class);
    private final DataSource dataSource;
    private final DataSourceProperties properties;
    private final ResourceLoader resourceLoader;

    DataSourceInitializer(DataSource dataSource, DataSourceProperties properties, ResourceLoader resourceLoader) {
        this.dataSource = dataSource;
        this.properties = properties;
        this.resourceLoader = (ResourceLoader)(resourceLoader != null ? resourceLoader : new DefaultResourceLoader());
    }

    DataSourceInitializer(DataSource dataSource, DataSourceProperties properties) {
        this(dataSource, properties, (ResourceLoader)null);
    }

    public DataSource getDataSource() {
        return this.dataSource;
    }

    public boolean createSchema() {
        List<Resource> scripts = this.getScripts("spring.datasource.schema", this.properties.getSchema(), "schema");
        if (!scripts.isEmpty()) {
            if (!this.isEnabled()) {
                logger.debug("Initialization disabled (not running DDL scripts)");
                return false;
            }

            String username = this.properties.getSchemaUsername();
            String password = this.properties.getSchemaPassword();
            this.runScripts(scripts, username, password);
        }

        return !scripts.isEmpty();
    }

    public void initSchema() {
        List<Resource> scripts = this.getScripts("spring.datasource.data", this.properties.getData(), "data");
        if (!scripts.isEmpty()) {
            if (!this.isEnabled()) {
                logger.debug("Initialization disabled (not running data scripts)");
                return;
            }

            String username = this.properties.getDataUsername();
            String password = this.properties.getDataPassword();
            this.runScripts(scripts, username, password);
        }

    }

    private boolean isEnabled() {
        DataSourceInitializationMode mode = this.properties.getInitializationMode();
        if (mode == DataSourceInitializationMode.NEVER) {
            return false;
        } else {
            return mode != DataSourceInitializationMode.EMBEDDED || this.isEmbedded();
        }
    }

    private boolean isEmbedded() {
        try {
            return EmbeddedDatabaseConnection.isEmbedded(this.dataSource);
        } catch (Exception var2) {
            logger.debug("Could not determine if datasource is embedded", var2);
            return false;
        }
    }

   //查找要自动运行的语句
    private List<Resource> getScripts(String propertyName, List<String> resources, String fallback) {
        if (resources != null) {
            return this.getResources(propertyName, resources, true);
        } else {
            String platform = this.properties.getPlatform();
            List<String> fallbackResources = new ArrayList();
            fallbackResources.add("classpath*:" + fallback + "-" + platform + ".sql");
            fallbackResources.add("classpath*:" + fallback + ".sql");
            return this.getResources(propertyName, fallbackResources, false);
        }
    }

   

        //运行语句
    private void runScripts(List<Resource> resources, String username, String password) {
        if (!resources.isEmpty()) {
            ResourceDatabasePopulator populator = new ResourceDatabasePopulator();
            populator.setContinueOnError(this.properties.isContinueOnError());
            populator.setSeparator(this.properties.getSeparator());
            if (this.properties.getSqlScriptEncoding() != null) {
                populator.setSqlScriptEncoding(this.properties.getSqlScriptEncoding().name());
            }

            Iterator var5 = resources.iterator();

            while(var5.hasNext()) {
                Resource resource = (Resource)var5.next();
                populator.addScript(resource);
            }

            DataSource dataSource = this.dataSource;
            if (StringUtils.hasText(username) && StringUtils.hasText(password)) {
                dataSource = DataSourceBuilder.create(this.properties.getClassLoader()).driverClassName(this.properties.determineDriverClassName()).url(this.properties.determineUrl()).username(username).password(password).build();
            }

            DatabasePopulatorUtils.execute(populator, dataSource);
        }
    }
}

```

所以。如果ROM要初始化一些数据库脚本，可以按照规则，将要初始化的数据库脚本命名为schema-\*.sql 、data-*\.sql这种格式，比如schema.sql,sachema-all.sql等，也可以在配置文件中指定位置。  

```yml
schema:
    - classpath:department.sql
```

## 二、整合Druid数据源
1. 引入druid依赖

```xml
   		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid-spring-boot-starter</artifactId>
		</dependency>
```

2. 编写druid的配置类        

```java
//导入druid数据源
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
        return new DruidDataSource();
    }

    //配置druid管理监控

    //1.配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean = new ServletRegistrationBean(
                new StatViewServlet(),"/druid/*"
        );
        Map<String,String> initParams = new HashMap<>();
        initParams.put("loginUsername","admin");
        initParams.put("loginPassword","123456");
        //允许所有访问
        initParams.put("allow","");
        initParams.put("deny","192.168.12.34");
        
        bean.setInitParameters(initParams);
        return bean;
    }
    
    //2.配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStateFilter(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());
        
        Map<String,String> initParams = new HashMap<>();
        initParams.put("exclusions","*.js,*.css,/druid/*");
        bean.setInitParameters(initParams);
        
        bean.setUrlPatterns(Arrays.asList("/*"));
        return bean;
    }
}

```

## 三、整合Mybatis
1. 引入mybatis依赖       

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-mybaties</artifactId>
		</dependency>
```

2.  配置数据源属性(同上)

3. 自定义MyBatis的配置规则         
要想自定义mybatis的匹配规则只需要容器中添加一个ConfigurationCustomizer即可。      

```java
@Configuration
public class MyBatisConfig {

    @Bean
    public ConfigurationCustomizer configurationCustomeizer(){

        return new ConfigurationCustomizer(){
                
            @Override
            public void customize(Configuration configuration){
                configuration.setMapUnderscoreToCamelCase(true);
            }
        };

    }
}

```

4. 使用MapperScan批量扫描所有Mapper接口     
 
```java
@MapperScan(value="com.desperado.mapper")
@SpringBootApplication
public class ProjectDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(ProjectDemoApplication.class, args);
	}
}
```

5. 使用配置文件扫描        

```properties
mybatis.config-locations=classpath:mybatis/mybatis-config.xml
mybatis.mapper-locations=classpath:mybatis/mapper/*.xml
```

## 四、整合SpringData JPA

![image.png](/image/sp/2-1.png)

1. 编写一个实体类和数据表进行映射，并且配置好映射关系。        

```java
// 使用JPA注解配置映射关系
@Entity   // 标识这是一个JPA的实体类(和数据表映射的类)
@Table(name = "tbl_user")  // 指定和数据库对应的表，如果省略默认表名就是类名小写
public class User {
    
    @Id  // 标识这是一个主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)//指定主键的生成方式
    private Integer id;
    
    @Column(name = "name",length = 50) //指定和数据表对应的列
    private String name;
    
    @Column     // 如果忽略名称，那么需要字段名称和数据表字段名称一致
    private String email;
}
```

2. 编写一个Dao接口来操作实体类对应的数据表。        

```java
// 继承JpaRepository来完成对数据库的操作
public interface UserRepository extends JpaRepository<User,Integer> {
    
}
```

3. 基本配置

```properties
# 更新或者创建数据表结构
spring.jpa.hibernate.ddl-auto=update
# 控制台显示sql
spring.jpa.hibernate.show-sql=true
```