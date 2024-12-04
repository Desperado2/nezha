### 1.dubbo负载均衡策略    
①**random loadbalance 策略**     
默认情况下，dubbo是random loadbalance 随机调用实现负载均衡，可以对provider不同实例设置不同的权重，会按照权重来进行负载均衡，权重越大分配的流量越高，一般就用这个默认的就可以了。    

②**roundrobin loadbalance策略**    
这个策略默认会将请求均匀的分布到各个provider上面，但是如果各个机器的性能不一样，很容易到杭州性能差 的机器负载过高。   

③**leastactive loadbalance策略**    
自动感知机器性能，如果某个机器性能差，那么这个机器接收到的请求就会越少。接收到的请求越少，机器就越不活跃，那么不活跃的机器就会接到更少的请求。    

④**consistanthash loadbalance策略**   
一致性hash算法，相同参数的请求一定发送到同一个provider上面去，provider挂掉的时候，会基于虚拟节点均匀分配剩余的请求，抖动不会太大。   

### 2.dubbo的集群容错策略    
①**failover cluster策略**     
调用一个provider失败，自动切换到其他的provider上面去调用，默认策略，常见于读操作。    

②**failfast cluster策略**    
一次调用provider失败就立即失败，常见于写操作。     

③**failsafe cluster策略**    
出现异常时忽略掉，常见于不重要的接口调用，比如日志记录。    

④**faliback cluster策略**   
失败后后台自动记录请求，然后定时重发，比较适合写消息队列这种操作。    

⑤**forking cluster策略**    
并行调用多个provider，只要有一个成功就立即返回。   

⑥**broadcast cluster策略**    
逐个调用所有的provider。    

### 3.dubbo的动态代理策略    

默认使用javassist动态字节码生成，创建代理类。     
可以通过spi机制扩展配置自己的动态代理策略。