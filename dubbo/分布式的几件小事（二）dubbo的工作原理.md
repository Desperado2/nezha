### 1.dubbo的工作原理    
①整体设计    

![image.png](/image/dubbo/2-1dubbo整体架构.jpg)    

图例说明：

- 图中左边淡蓝背景的为服务消费方使用的接口，右边淡绿色背景的为服务提供方使用的接口，位于中轴线上的为双方都用到的接口。  

- 图中从下至上分为十层，各层均为单向依赖，右边的黑色箭头代表层之间的依赖关系，每一层都可以剥离上层被复用，其中，Service 和 Config 层为 API，其它各层均为 SPI。  

- 图中绿色小块的为扩展接口，蓝色小块为实现类，图中只显示用于关联各层的实现类。   

- 图中蓝色虚线为初始化过程，即启动时组装链，红色实线为方法调用过程，即运行时调时链，紫色三角箭头为继承，可以把子类看作父类的同一个节点，线上的文字为调用的方法。

②dubbo的十层架构说明   
- **service 接口层** ：给服务提供者、服务消费者去实现。   

- **config 配置层** ：对外的配置接口，以ServiceConfig，REferenceConfig为中心，可以直接初始化配置类。也可以通过spring解析配置生成配置类。    

- **proxy服务代理层** ：服务接口透明代理，生成服务的客户端Stub和服务端Skeleton，以ServiceProxy为中心，扩展接口为ProxyFactory。   

- **registry注册中心层** :封装注册地址的注册与发现，以服务URL为中心，扩展接口为RegistryFactory，Registry，RegistryService。   

- **cluster路由层** ：封装多个提供者的路由与负载均衡，并桥接注册中心，以Invoker为中心，扩展接口为cluster，Directory，Router，LoadBalance。   

- **monitor监控层** ：RPC调用次数和调用时间监控，以Statistics为中心，扩展接口为MonitorFactory，Monitor，MonitorService。   

- **protocol远程调用层** ：封装RPC调用，以Invocation，Result为中心，扩展接口为Protocol，Invoker，Exportor。  

- **exchange信息交换层** ：封装请求响应模式，同步转异步，以Request，Response为中心，扩展接口为Exchanger，ExchangeChannel，ExchangeClient，ExchangeServer。   

- **transport网络传输层**:抽象mina和netty为统一接口，以Message为中心，扩展接口为Channel，Transporter，Client，Server,Codec。   

- **serialize数据序列化层** ：可以复用的一些工具，扩展接口为Serialization，ObjectInput，ObjectOutput，ThreadPool。   

③关系说明    
- 在RPC中，Protocol是核心层，也就是只要有Protocol+Invoker+Exporter就可以完成非透明的RPC调用，然后在Invoker的过程上Filter拦截点。   

- 图中的Consumer和Provider是抽象概念，只是想让看图者更直观的了解哪些类分属于客户端与服务器端，不用 Client 和 Server 的原因是 Dubbo 在很多场景下都使用 Provider, Consumer, Registry, Monitor 划分逻辑拓普节点，保持统一概念。   

- Cluster是外围概念，所有cluster的目的是将多个Invoker伪装成一个Invoker，这样其他人只要关注Protocol层Invoker即可，加上Cluster或者去掉Cluster对其他层都不会造成影响，因为只有一个提供者时，是不需要Cluster的。   

- Proxy层封装了所有接口的透明化代理，而在其它层都以Invoker为中心，只有暴露给用户使用时，才用Proxy将Invoker转为接口，或者将接口实现为Invoker，也就是去掉Proxy层的RPC是可以run的，只是不那么透明，不那么看起来向本地服务一样调用远程服务。   

- Remoting实现是dubbo协议的实现，如果你选择RMI协议，整个Remoting都用不上，Remoting内部再划分为transport传输层和exchange交换层，Transport层只负责单向消息传输，是对mina，netty，Grizzly的抽象，它可以扩展UDP传输，二Exchange层是在传输层之上封装了Request-Response语义。  

- Registry和Monitor实际上不算一层，而是一个独立的节点，只是为了全局概览，用层的方式画在了一起。    

### 2.dubbo的一次调用过程    

![image.png](/image/dubbo/2-2调用过程.jpg)  

暴露服务时序   

![image.png](/image/dubbo/2-3暴露服务时序.jpg)


暴露服务时序    

![image.png](/image/dubbo/2-4引用服务时序.jpg)

简易通信过程示意图   

![image.png](/image/dubbo/2-5dubbo的一次调用过程.png)


### 3.注册中心挂了可以继续通信吗？

可以，因为刚开始初始化的时候，消费者会将提供者的地址等信息拉取到本地缓存，所以注册中心挂了可以继续通信