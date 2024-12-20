# Table of contents

* [操作系统](linux)
	* [linux常用命令](linux/linux常用命令.md)

* [编程语言](language)

	* [关于Java的几件小事](Java)
		* [关于JVM](Java/jvm)
			* [(一)类的加载](Java/jvm/（一）JVM如何加载class文件.md)
			* [(二)什么是反射](Java/jvm/（二）什么是反射.md)
			* [(三)什么是ClassLoader](Java/jvm/（三）什么是ClassLoader.md)
			* [(四)JAVA内存模型](Java/jvm/（四）java内存模型.md)
		* [关于GC](Java/gc)
			* [(一)垃圾回收算法](Java/jvm/（一）垃圾回收算法.md)
			* [(二)新生代垃圾收集器](Java/jvm/（二）新生代垃圾收集器.md)
			* [(三)老年代垃圾收集器](Java/jvm/（三）老年代垃圾收集器.md)
		* [关于线程与并发](Java/thread)
			* [(一)线程与进程](Java/jvm/（一）线程与进程.md)
			* [(二)线程的创建方式](Java/jvm/（二）线程的创建方式.md)
			* [(三)线程的函数操作](Java/jvm/（三）线程的函数操作.md)
			* [(四)synchronized](Java/jvm/（四）synchronized.md)
			* [(五)ReentrantLock](Java/jvm/（五）ReentrantLock.md)
			* [(六)JMM内存可见性](Java/jvm/（六）JMM内存可见性.md)
			* [(七)CAS](Java/jvm/（七）关于CAS.md)
			* [(八)Java线程池](Java/jvm/（八）Java线程池.md)  

		* [关于常用类库](Java/utils)
			* [(一)Java异常体系](Java/utils/（一）Java异常体系.md)
			* [(二)Collection体系](Java/utils/（二）Collection体系.md)
			* [(三)ArrayList与LinkedList与Vector](Java/utils/（三）ArrayList与LinkedList与Vector.md)
			* [(四)HashMap与HashSet](Java/utils/（四）HashMap与HashSet.md)
			* [(五)ConcurrentHashMap](Java/utils/（五）ConcurrentHashMap.md)


* [网络](network)
	* [(一)scoket相关](network/关于scoket.md)
	* [(二)TCP与UDP](network/TCP与UDP.md)
	* [(三)TCP的三次握手与四次挥手](network/TCP的三次握手与四次挥手.md)
	* [(四)TCP的滑窗机制](network/TCP的滑窗机制.md)
	* [(五)HHTP与HTTPS](network/HHTP与HTTPS.md)
	

    
* [数据结构&算法](algorithms)
    * [算法时间复杂度分析](algorithms/算法时间复杂度分析.md)
	* [字符串匹配算法](algorithms/字符串匹配算法.md)


* [数据库](database)  

	* [关于mysql的几件小事](database/mysql)
		* [(一)关于MySQL索引的几件小事](database/mysql/关于MySQL索引的几件小事.md)
		* [(二)关于mysql事务的几件小事](database/mysql/关于mysql事务的几件小事.md)
		* [(三)数据库如何拆分](database/mysql/分库分布的几件小事（一）数据库如何拆分.md)
		
		* [(四)如何进行分库分表的数据迁移](database/mysql/分库分布的几件小事（二）如何进行分库分表的数据迁移.md)
		
		* [(五)可以动态扩容缩容的分库分表方案](database/mysql/分库分布的几件小事（三）可以动态扩容缩容的分库分表方案.md)
		
		* [(六)分库分表的id主键生成](database/mysql/分库分布的几件小事（四）分库分表的id主键生成.md)
		
		* [(七)MYSQL读写分离](database/mysql/分库分布的几件小事（五）MYSQL读写分离.md)

	* [关于MQ的几件小事](MQ)
		* [(一)消息队列的用途、优缺点、技术选型](MQ/关于MQ的几件小事（一）消息队列的用途、优缺点、技术选型.md)
		* [(二)如何保证消息队列的高可用](MQ/关于MQ的几件小事（二）如何保证消息队列的高可用.md)
		* [(三)如何保证消息不重复消费](MQ/关于MQ的几件小事（三）如何保证消息不重复消费.md)
		* [(四)如何保证消息不丢失](MQ/关于MQ的几件小事（四）如何保证消息不丢失.md)
		* [(五)如何保证消息按顺序执行](MQ/关于MQ的几件小事（五）如何保证消息按顺序执行.md)
		* [(六)消息积压在消息队列里怎么办](MQ/关于MQ的几件小事（六）消息积压在消息队列里怎么办.md)
		* [(七)如果让你设计一个MQ，你怎么设计](MQ/关于MQ的几件小事（七）如果让你设计一个MQ，你怎么设计.md)


	* [关于redis的几件小事](redis)

		* [(一)redis的使用目的与问题](redis/关于redis的几件小事（一）redis的使用目的与问题.md)
		
		* [(二)redis线程模型](redis/关于redis的几件小事（二）redis线程模型.md)
		
		* [(三)redis的数据类型与使用场景](redis/关于redis的几件小事（三）redis的数据类型与使用场景.md)
		
		* [(四)redis的过期策略以及内存淘汰机制](redis/关于redis的几件小事（四）redis的过期策略以及内存淘汰机制.md)
		
		* [(五)redis保证高并发以及高可用](redis/关于redis的几件小事（五）redis保证高并发以及高可用.md)
		
		* [(六)redis的持久化](redis/关于redis的几件小事（六）redis的持久化.md)
		
		* [(七)redis缓存雪崩与穿透](redis/关于redis的几件小事（七）redis缓存雪崩与穿透.md)
		
		* [(八)缓存与数据库双写时的数据一致性](redis/关于redis的几件小事（八）缓存与数据库双写时的数据一致性.md)
		
		* [(九)redis的并发竞争问题](redis/关于redis的几件小事（九）redis的并发竞争问题.md)
		
		* [(十)redis-cluster模式](redis/关于redis的几件小事（十）redis-cluster模式.md)
		* [(十一)Redis缓存保持数据类型.md](redis/Redis缓存保持数据类型.md)


* [后端开发](backend)
	* [关于spring的几件小事](backend/spring)
		* [(一)关于spring中bean配置的几件小事](backend/spring/关于spring中bean配置的几件小事.md)
		* [(二)关于spring中AOP的几件小事](backend/spring/关于spring中AOP的几件小事.md)
		* [(三)关于spring中事务管理的几件小事](backend/spring/关于spring中事务管理的几件小事.md)

	* [关于springMVC的几件小事](backend/springMVC)	
		* [(一)关于SpringMVC映射模型视图的几点小事](backend/springMVC/关于SpringMVC映射模型视图的几点小事.md)
		* [(二)关于SpringMVC的几件小事](backend/springMVC/关于SpringMVC的几件小事.md)
		* [(三)关于SpringMVC拦截器和异常的几件小事](backend/springMVC/关于SpringMVC拦截器和异常.md)
		

	* [关于mybatis的几件小事](backend/mybatis)	
		* [(一)关于Mybatis的几件小事](backend/mybatis/关于Mybatis的几件小事（一）.md)
		* [(二)关于Mybatis的几件小事](backend/mybatis/关于Mybatis的几件小事（二）.md)
		
	* [关于SpringBoot的几件小事](backend/springboot)
		* [(一)关于SpringBoot的自动配置和启动过程](backend/springboot/关于SpringBoot的自动配置和启动过程.md)		
		* [(二)关于SpringBoot的数据访问](backend/springboot/SpringBoot的数据访问.md)		
		* [(三)关于SpringBoot的启动配置原理](backend/springboot/SpringBoot的启动配置原理.md)		
		* [(四)关于SpringBoot与缓存、消息、检索、任务、安全与监控](backend/springboot/SpringBoot与缓存、消息、检索、任务、安全与监控.md)		




	* [关于分布式的几件小事](backend/dubbo)

		* [(一)为什么要把系统拆分成分布式的](backend/dubbo/分布式的几件小事（一）为什么要把系统拆分成分布式的.md)
		
		* [(二)dubbo的工作原理](backend/dubbo/分布式的几件小事（二）dubbo的工作原理.md)
		
		* [(三)dubbo的通信协议与序列化](backend/dubbo/分布式的几件小事（三）dubbo的通信协议与序列化.md)
		
		* [(四)dubbo负载均衡策略和集群容错策略](backend/dubbo/分布式的几件小事（四）dubbo负载均衡策略和集群容错策略.md)
		
		* [(五)dubbo的spi思想是什么](backend/dubbo/分布式的几件小事（五）dubbo的spi思想是什么.md)
		
		* [(六)dubbo如何做服务治理、服务降级以及重试](backend/dubbo/分布式的几件小事（六）dubbo如何做服务治理、服务降级以及重试.md)
		
		* [(七)分布式系统接口的幂等性如何保证](backend/dubbo/分布式的几件小事（七）分布式系统接口的幂等性如何保证.md)
		
		* [(八)分布式服务接口请求的顺序性如何保证](backend/dubbo/分布式的几件小事（八）分布式服务接口请求的顺序性如何保证.md)
		
		* [(九)zookeeper都有哪些使用场景](backend/dubbo/分布式的几件小事（九）zookeeper都有哪些使用场景.md)
		
		* [(十)分布式锁是啥](backend/dubbo/分布式的几件小事（十）分布式锁是啥.md)
		
		* [(十一)分布式session如何实现](backend/dubbo/分布式的几件小事（十一）分布式session如何实现.md)
		
		* [(十二)分布式事务](backend/dubbo/分布式的几件小事（十二）分布式事务.md) 


* [大数据](bigdata)  

	* [数据同步](bigdata/数据同步)  

		* [离线数据同步](bigdata/数据同步/离线数据同步)

			* [基于Hadoop体系的离线数据同步](bigdata/数据同步/离线数据同步/基于Hadoop体系的离线数据同步.md)  

			* [基于Apache DolphinScheduler的数据同步](bigdata/数据同步/离线数据同步/基于Apache%20DolphinScheduler的数据同步.md)

			* [基于自定义脚本的数据同步](bigdata/数据同步/离线数据同步/基于自定义脚本的数据同步.md)
			
		* [实时数据同步](bigdata/实时数据同步)  

			* [一种小资源情况下RDS数据实时同步StarRocks方案](bigdata/数据同步/增量数据同步/一种小资源情况下RDS数据实时同步StarRocks方案.md)  
		* [DataX数据同步无响应](bigdata/数据同步/DataX数据同步无响应.md)  
	
	
* [其他](others)

	* [图片相关知识](others/图片格式.md)  
	* [二维码相关知识](others/QR码.md)  
	* [Shape文件处理工具](others/Shape文件处理工具.md)  
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	