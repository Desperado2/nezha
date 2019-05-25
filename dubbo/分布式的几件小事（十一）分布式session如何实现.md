### 1.分布式会话是什么？   
首先，我们知道浏览器有个cookie，在一段时间内这个cookie都存在，然后每次发请求过来都带上一个特殊的jsessionid cookie，就根据这个东西，在服务端可以维护一个对应的session域，里面可以放点儿数据。    

其次，单系统的时候session是不存在问题的。    
分布式系统出现session问题的场景：     
比如系统部署了3台机器，用户登录的时候，负载均衡到了其中的一台机器上面，登录成功在这台机器上面创了session，保存了用户的登录信息，然后用户进行了两外一个操作，这时候通过负载均衡到了另外一台机器上面去，这台机器上面是没有客户的session的，那么就会出现用户没有登录的情况，又会让用户去登陆，这样就会造成系统无法使用。    

![image.png](/image/dubbo/11-1分布式会话场景1.png)

### 2.如何解决    
①**tomcat+redis方案**             
这个其实还挺方便的，就是使用session的代码跟以前一样，还是基于tomcat原生的session支持即可，然后就是用一个叫做Tomcat RedisSessionManager的东西，让所有我们部署的tomcat都将session数据存储到redis即可。

在tomcat的配置文件中，配置一下
```
<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />

<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
         host="{redis.host}"
         port="{redis.port}"
         database="{redis.dbnum}"
         maxInactiveInterval="60"/>

```

还可以用基于redis哨兵支持的redis高可用集群来保存session数据，都是ok的.   
```
<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />
<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
	 sentinelMaster="mymaster"
	 sentinels="<sentinel1-ip>:26379,<sentinel2-ip>:26379,<sentinel3-ip>:26379"
	 maxInactiveInterval="60"/>

```

②**spring session+redis方案**          
分布式会话的东西和容器耦合在一起，比如要将tomcat容器迁移成jetty，就需要修改session同步的配置，这样会比较麻烦。     

可以使用spring的解决方案spring session，使用方法如下     
1.在pom中引入依赖
```
<dependency>
  <groupId>org.springframework.session</groupId>
  <artifactId>spring-session-data-redis</artifactId>
  <version>1.2.1.RELEASE</version>
</dependency>

<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>2.8.1</version>
</dependency>

```
2.配置spring配置文件
```
<bean id="redisHttpSessionConfiguration"
     class="org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration">
    <property name="maxInactiveIntervalInSeconds" value="600"/>
</bean>

<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <property name="maxTotal" value="100" />
    <property name="maxIdle" value="10" />
</bean>

<bean id="jedisConnectionFactory"
      class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory" destroy-method="destroy">
    <property name="hostName" value="${redis_hostname}"/>
    <property name="port" value="${redis_port}"/>
    <property name="password" value="${redis_pwd}" />
    <property name="timeout" value="3000"/>
    <property name="usePool" value="true"/>
    <property name="poolConfig" ref="jedisPoolConfig"/>
</bean>

```
3.修改web.xml      
```
<filter>
    <filter-name>springSessionRepositoryFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSessionRepositoryFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

```

使用示例： 
```
@Controller
@RequestMapping("/test")
public class TestController {

@RequestMapping("/putIntoSession")
@ResponseBody
    public String putIntoSession(HttpServletRequest request, String username){
        request.getSession().setAttribute("name",  “leo”);

        return "ok";
    }

@RequestMapping("/getFromSession")
@ResponseBody
    public String getFromSession(HttpServletRequest request, Model model){
        String name = request.getSession().getAttribute("name");
        return name;
    }
}

```
上面的代码就是ok的，给sping session配置基于redis来存储session数据，然后配置了一个spring session的过滤器，这样的话，session相关操作都会交给spring session来管了。接着在代码中，就用原生的session操作，就是直接基于spring sesion从redis中获取数据了。

实现分布式的会话，有很多种很多种方式，我说的只不过比较常见的两种方式，tomcat + redis早期比较常用；近些年，重耦合到tomcat中去，通过spring session来实现。