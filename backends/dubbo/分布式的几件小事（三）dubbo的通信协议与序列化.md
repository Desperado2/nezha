### 1.dubbo的通信协议    
①dubbo协议    
Dubbo缺省协议采用单一长连接和NIO异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况。    
**特点** ：    
- dubbo缺省协议，使用的是基于netty+hessian的tbremoting交互。   
- 连接个数：单连接。   
- 连接方式：长连接。  
- 传输协议：TCP。   
- 传输方式：NIO异步传输。   
- 使用范围：传入传出数据包较小，消费者数据比提供者多，单一消费者无法压满提供者，尽量不要用来传输超大数据。   
- 使用场景：常规远程服务方法调用。   
>为什么要消费者比提供者个数多：
>因dubbo协议采用单一长连接，
>假设网络为千兆网卡(1024Mbit=128MByte)，
>根据测试经验数据每条连接最多只能压满7MByte(不同的环境可能不一样，供参考)，
>理论上1个服务提供者需要20个服务消费者才能压满网卡。

>为什么不能传大包：
>因dubbo协议采用单一长连接，
>如果每次请求的数据包大小为500KByte，假设网络为千兆网卡(1024Mbit=128MByte)，每条连接最大7MByte(不同的环境可能不一样，供参考)，
>单个服务提供者的TPS(每秒处理事务数)最大为：128MByte / 500KByte = 262。
>单个消费者调用单个服务提供者的TPS(每秒处理事务数)最大为：7MByte / 500KByte = 14。
>如果能接受，可以考虑使用，否则网络将成为瓶颈。

>为什么采用异步单一长连接：
>因为服务的现状大都是服务提供者少，通常只有几台机器，
>而服务的消费者多，可能整个网站都在访问该服务，
>比如Morgan的提供者只有6台提供者，却有上百台消费者，每天有1.5亿次调用，
>如果采用常规的hessian服务，服务提供者很容易就被压跨，
>通过单一连接，保证单一消费者不会压死提供者，
>长连接，减少连接握手验证等，
>并使用异步IO，复用线程池，防止C10K问题。

②RMI协议    
RMI协议采用JDK的java.rmi.*来实现，采用的是 阻塞式短连接和标准化的JDK序列化方式。   

**特点** ：   
- Java标准化的远程调用协议。  
- 连接个数：多连接。  
- 连接方式：短连接。   
- 传输协议：TCP。   
- 传输方式： 同步传输。   
- 序列化： Java标准二进制序列化。   
- 适用范围：传入传出数据包大小混合，消费者与提供者个数差不多，可传文件。   
- 适用场景： 常规远程服务方法调用，与原生RMI服务互操作。   

③Hessian协议   
Hessian协议用于集成Hessian的服务，Hessian底层采用Http通讯，采用Servlet暴露服务，Dubbo缺省内嵌jetty作为服务器。   
**特点** ：    
- 基于Hessian的远程调用协议。    
- 连接个数：多连接。   
- 连接方式：短连接。   
- 传输协议：HTTP。    
- 传输方式：同步传输。    
- 序列化： Hessian二进制序列化。    
- 适用场景：传入传出参数数据包较大，提供者比消费者个数多，提供者压力较大，可传文件。   
  -适用场景：页面传输，文件传输，或与原始Hessian服务互操作。    

④Http协议   
采用Spring的HttpInvoker实现。    
**特点**    
- 基于Http表单的远程调用协议，     
- 连接个数：多连接。    
- 连接方式：短连接。   
- 传输协议：HTTP。   
- 传输方式：同步传输。   
- 序列化：表单序列化JSON。    
- 适用范围：传入传出的数据报大小混合，提供者比消费者个数多，可以浏览器查看，可用表单或url传参数，不支持传输文件。    
- 适用场景：需要用时给其他程序和浏览器JS使用的服务。    

⑤WebService协议     

基于CXF的frontend-simple和transports-http实现。    
** 特点**   
- 基于WebService的远程调用协议。  
- 连接个数：多连接   
- 连接方式：短连接  
- 传输协议：HTTP   
- 传输方式：同步传输    
- 序列化：SOAP文本序列化    
- 适用场景：系统集成，跨语言调用    

⑥thrif协议    
Thrift是Facebook捐给Apache的一个RPC框架，当前 dubbo 支持的 thrift 协议是对 thrift 原生协议的扩展，在原生协议的基础上添加了一些额外的头信息，比如service name，magic number等。    

### 2.dubbo的序列化方式    
dubbo实际基于不同的通信协议，支持hessian、java二进制序列化、json、SOAP文本序列化多种序列化协议。但是hessian是其默认的序列化协议。   
①Hessian序列化。     
hessian是一种跨语言的高效二进制的序列化方式，但这里实际不是原生的hessian2序列化，而是阿里修改过的hessian lite，它是dubbo RPC默认启用的序列化方式。

②Java二进制序列化。   
主要是采用JDK自带的java序列化实现，性能很不理想。

③Json序列化      
目前有两种实现，一种是采用的阿里的fastjson库，另一种是采用dubbo中自已实现的简单json库，一般情况下，json这种文本序列化性能不如二进制序列化。   

④SOAP文本序列化。    
简单对象访问协议是交换数据的一种协议规范，是一种轻量的、简单的、基于XML的协议，它被设计成在WEB上交换结构化的和固化的信息。