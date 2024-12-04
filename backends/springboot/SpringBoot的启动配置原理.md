## 一、启动流程
1. 创建SpringApplication对象        

```java
public class SpringApplication {
 
    public SpringApplication(Class... primarySources) {
        this((ResourceLoader)null, primarySources);
    }

    public SpringApplication(ResourceLoader resourceLoader, Class... primarySources) {
        this.sources = new LinkedHashSet();
        this.bannerMode = Mode.CONSOLE;
        this.logStartupInfo = true;
        this.addCommandLineProperties = true;
        this.addConversionService = true;
        this.headless = true;
        this.registerShutdownHook = true;
        this.additionalProfiles = new HashSet();
        this.isCustomEnvironment = false;
        this.resourceLoader = resourceLoader;
        Assert.notNull(primarySources, "PrimarySources must not be null");
        // 保存主配置类
        this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));
        // 判断当前是否一个web应用
        this.webApplicationType = WebApplicationType.deduceFromClasspath();                                                            
         // 从类路径下找到META-INF/spring.factories配置的所有ApplicationContextInitializer；然后保存起来。
        this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
        // 从类路径下找到META-INF/spring.factories配置的所有ApplicationListener
        this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
        //从多个配置类中找到有main方法的主配置类
        this.mainApplicationClass = this.deduceMainApplicationClass();
    }
```

2. 运行run方法

```java
    public ConfigurableApplicationContext run(String... args) {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        ConfigurableApplicationContext context = null;
        Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList();
        this.configureHeadlessProperty();
        // 获取SpringApplicationRunListener; 从类路径下META-INF/spring.factories获取
        SpringApplicationRunListeners listeners = this.getRunListeners(args);
        // 回调所有的获取SpringApplicationRunListener.starting()方法
        listeners.starting();

        Collection exceptionReporters;
        try {
            // 封装命令行参数
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            // 准备环境
            ConfigurableEnvironment environment = this.prepareEnvironment(listeners, applicationArguments);
            this.configureIgnoreBeanInfo(environment);
            // 创建环境完成后回调SpringApplicationRunListener.environmentPrepared();表示环境准备完成
            Banner printedBanner = this.printBanner(environment);
            // 创建ApplicationContext；决定创建web的IOC还是普通的IOC
            context = this.createApplicationContext();
            exceptionReporters = this.getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[]{ConfigurableApplicationContext.class}, context);
            // 准备上下文环境；将environment保存到IOC中，而且applyInitializers();
            // 而且applyInitializers():回调之前保存的所有ApplicationContextInitializer的initialize方法
            // 回调所有的SpringApplicationRunListener的contextPrepared();
            this.prepareContext(context, environment, listeners, applicationArguments, printedBanner);
            // prepareContext运行完成以后回调所有的SpringApplicationRunListener的contextLoaded();
            // 扫描容器；IOC容器初始化（如果是web应用还会创建嵌入的Tomcat）；
            // 扫描，创建，加载所有组件的地方(配置类，组件，自动配置)
            this.refreshContext(context);
            // 从IOC容器中获取所有的ApplicationRunner和CommandLineRunner进行回调
            // ApplicationRunner先回调，CommandLineRunner再回调
            this.afterRefresh(context, applicationArguments);
            stopWatch.stop();
            if (this.logStartupInfo) {
                (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch);
            }

            listeners.started(context);
            this.callRunners(context, applicationArguments);
        } catch (Throwable var10) {
            this.handleRunFailure(context, var10, exceptionReporters, listeners);
            throw new IllegalStateException(var10);
        }

        try {
            listeners.running(context);
            // 整个SpringBoot应用启动完成以后返回启动的IOC容器
            return context;
        } catch (Throwable var9) {
            this.handleRunFailure(context, var9, exceptionReporters, (SpringApplicationRunListeners)null);
            throw new IllegalStateException(var9);
        }
    }
```

3. 事件监听机制        
需要配置在META-INF/spring.factories中的事件监听器。       
**ApplicationContextInitializer**        

```java
public class CustomApplicationContextInitializer implements 
        ApplicationContextInitializer<ConfigurableApplicationContext> {

    @Override
    public void initialize(ConfigurableApplicationContext configurableApplicationContext) {
        System.out.println("ApplicationContextInitializer.....initialize");
    }
}
```

**SpringApplicationRunListener**          

```java
public class CustomSrpingApplicationRunListener implements SpringApplicationRunListener {
    
    // 必须有的构造器
    public CustomSrpingApplicationRunListener(SpringApplication application,String[] args){
        
    }
    @Override
    public void starting() {
        System.out.println("SpringApplicationRunListener...starting....");
    }

    @Override
    public void environmentPrepared(ConfigurableEnvironment environment) {
        Object o = environment.getSystemEnvironment().get("os.name");
        System.out.println("SpringApplicationRunListener...environmentPrepared...."+o);
    }

    @Override
    public void contextPrepared(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener..contextPrepared");
    }

    @Override
    public void contextLoaded(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener..contextLoaded");
    }

    @Override
    public void started(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener..started");
    }

    @Override
    public void running(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener..running");
    }

    @Override
    public void failed(ConfigurableApplicationContext context, Throwable exception) {
        System.out.println("SpringApplicationRunListener..failed");
    }
    
}

```

配置(META-INF/spring.factories)    

```factories
org.springframework.context.ApplicationContextInitializer=com.desperado.demo.CustomApplicationContextInitializer
org.springframework.boot.SpringApplicationRunListener=com.desperado.demo.CustomSrpingApplicationRunListener
```

只需要放在IOC容器中的监听器。       
**ApplicationRunner**           

```java
@Component
public class CustomApplicationRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) {
        System.out.println("ApplicationRunner ... run....");
    }
}
```

**CommandLineRunner**         

```java
@Component
public class CustomCommandLineRunner implements CommandLineRunner {

    @Override
    public void run(String... args) {
        System.out.println("commandLineRunner ... run ...");
    }
}
```

## 二、自定义starter
1. 要使用到的注解        
@Configuration 指定类是一个配置类        
@ConditionalOnXXX 在指定条件成立的情况下自动配置生效、                 
@AutoConfigureAfter  在特定自动装配class之后(指定自动配置类的顺序)         
@AutoConfigureBefore  在特定自动装配class之前(指定自动配置类的顺序)          
@AutoConfigureOrder 指定顺序            
@Bean  给容器中添加组件         
@ConfigurationPropertie 结合相关xxxProperties类来绑定相关的配置。       
@EnableConfigurationProperties 让xxxProperties生效加入到容器中。          

2. 加载方式      
自动配置类要能加载，将需要启动就加载的自动配置类，配置在META-INF/spring.factories文件中。       

3. 启动器模式       
启动器只用来做依赖导入，专门写一个自动配置模块。启动器依赖自动配置，使用时只需要引入启动器(starter)。      

4. 启动器规则         
启动器就是个空jar文件，仅提供辅助性依赖管理，这些依赖可能用于自动装配或者其他库。        

命名规范：      
    - 推荐使用以下命名规范     
    - 官方命名空间          
           &emsp;- 前缀：spring-boot-starter-        
           &emsp;- 模式：spring-boot-starter-模块名
    - 自定义命名空间       
         &emsp; - 后缀：-spring-boot-starter      
         &emsp; - 模式：模块名-spring-boot-starter

5. 编写一个启动器模块       
1). 启动器模块       

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.desperado.starter</groupId>
    <artifactId>desperado-spring-boot-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <!-- 启动器 -->
    <dependencies>
        <!-- 引入自动配置模块 -->
        <dependency>
            <groupId>com.desperado.starter</groupId>
            <artifactId>desperado-spring-boot-starter-autoconfigurer</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>

    </dependencies>

</project>
```

2). 自动配置模块       

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.desperado.starter</groupId>
    <artifactId>desperado-spring-boot-starter-autoconfigurer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath/>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <!--  引入spring-boot-starter；所有starter的基本配置 -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

    </dependencies>

</project>

```

3). 编写配置文件类         

```java
@ConfigurationProperties(prefix = "desperado.custom")
public class CustomProperties {
    
    private String prefix;
    
    private String suffix;

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getSuffix() {
        return suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }
}
```

4). 进行配置文件的处理       

```java
public class CustomService {
    CustomProperties customProperties;

    public CustomProperties getCustomProperties(){
        return customProperties;
    }

    public void setCustomProperties(CustomProperties customProperties){
        this.customProperties = customProperties;
    }

    public String sayHello(String name){
        return customProperties.getPrefix()+"-"+name+customProperties.getSuffix();
    }
}
```

5). 编写自动配置类         

```java
@ConditionalOnWebApplication
@EnableConfigurationProperties(CustomProperties.class)
public class CustomServiceAutoConfiguration {
    
    @Autowired
    CustomProperties customProperties;
    
    @Bean
    public CustomService customService(){
        CustomService customService = new CustomService();
        customService.setCustomProperties(customProperties);
        return customService;
    }
}

```