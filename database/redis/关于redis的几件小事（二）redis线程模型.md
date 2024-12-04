### 1.memcached和redis有什么区别？
（1）Redis支持服务器端的数据操作  
redis和memcached相比，redis拥有更多的 **数据结构并且支持更丰富的数据操作** ，通常在memcached里面，你需要将数据拿到客户端来进行类型的修改然后在set回去，这样就严重增加了网络IO的次数和数据体积。在redis里面，这些操作可以在服务端完成，所以这些复杂的操作就和一般的GET/SET一样高效。所以，如果需要缓存能支持更复杂的结构和操作，那么redis是不错的选择 。  
（2）内存使用率  
如果使用简单的 key-value 存储的话，Memcached的内存利用率会更高，而如果Redis采用 hash 结构来做 key-value 存储，由于其组合式的压缩，其内存利用率会高于Memcached。  
（3）性能  
由于redis只使用单核，而Memcached可以使用多核，所以平均每一个核上redis在存储小数据时比Memcached性能更好。而在100K以上的数据中，Memcached性能要高于redis。  
（4）集群模式  
memcached没有原生的集群模式，需要依靠客户端来实现集群中分片写入数据；redis原生支持cluster模式，官方支持redis cluster集群模式。

对比点|memcached|redis
-|-|-
是否支持服务端操作|不支持|支持
数据结构类型|简单|复杂多样
内存使用率|简单 key-value 存储，利用率高|采用hash结构存储，内存利用率高  
性能|存储大数据性能高|存储小数据性能高
集群模式|没有原生支持|原生支持cluster模式


### 2.redis的线程模式？  
要了解redis的线程模式，必须先了解下面几个概念  
（1）文件事件处理器  
①redis是基于reactor模式开发了网络事件处理器，这个处理器叫做 文件事件处理器(file event Handler)。这个文件事件处理器是单线程的，所以redis才叫做单线程模式，采用IO多路复用机制去同时监听多个socket，根据socket上的时间来选择对应的事件处理器来处理这个事件。  

②如果被监听的socket准备好执行accept、read、write、close等操作的时候，跟操作对应的文件事件就会产生，这个时候文件处理器就会调用之前关联好的的事件处理器来处理这个事件。  

③文件事件处理器是单线程模式运行的，但是通过IO多路复用机制监听多个socket，可以实现高性能的网络通信模型，又可以跟内部其他单线程的模块进行对接，保证了redis内部的线程模型的简单性。  

④文件事件处理器的结构包含四个部分：多个socket、IO多路复用程序、文件事件分派器、事件处理器(命令请求处理器、命令回复处理器、连接应答处理器，等等)。  

⑤多个socket可能并发的产生不同的操作，每个操作对应不同的文件 事件，但是IO多路复用程序会监听多个socket，但是会将socket放到一个队列中去处理，每次从队列中取出一个socket给事件分派器，事件分派器把socket给对应的事件处理器。  

⑥然后一个socket的事件处理完了之后，IO多路复用程序才会将队列中的下一个socket给事件分派器。事件分派器会根据每个socket当前产生的事件，来选择对应的事件处理器来处理。  

（2）文件事件  
①当socket变得可读时(比如客户端对redis执行write操作，或者close操作)，或者有新的可以应答的socket出现时(客户端redis执行connect操作)，socket就会产生一个AE_READABLE事件。  

②当socket变得可写的时候(客户端对redis执行read操作)，socket就会产生一个AE_WRITABLE事件。  

③IO多路复用程序可以同时监听AE_READABLE和AE_WRITABLE两种事件，要是一个socket同时差生了这两种事件，那么文件分配器优先处理AE_READABLE事件，然后才是AE_WRITABLE事件。  

（3）文件事件处理器  
如果是客户端要连接redis，那么会为socket关联连接应答处理器。  
如果是客户端要写数据到redis，那么会为socket关联命令请求处理器。  
如果是客户端要从redis读数据，那么会为socket关联命令回复处理器。  

![redis内存模式简单示意图.png](/image/redis/2-1内存模式.webp)

（4）客户端与redis通信的一次流程  
①在redis启动初始化的时候，redis会将连接应答处理器跟AE_READABLE事件关联起来，接着如果一个客户端跟redis发起连接，此时redis会产生一个AE_READABLE事件，然后由连接应答处理器来处理跟客户端建立连接，创建客户端响应的socket，同时将这个socket的AE_READABLE事件跟命令请求处理器关联起来。  

②当客户端向redis发起请求的时候(不管是读请求还是写请求，都一样)，首先就会在socket产生一个AE_READABLE事件，然后由对应的命令请求处理器来处理。这个命令请求处理器就会从socket中读取请求的相关数据，然后执行操作和处理。  

③接着redis这边准备好了给客户端的响应数据之后，就会将socket的AE_WRITABLE事件跟命令回复处理器关联起来，当客户端这边准备好读取相应数据时，就会在socket上产生一个AE_WRITABLE事件，会由相应的命令回复处理器来处理，就是将准备好的响应数据写入socket，供客户端读取。  

④命令回复处理器写完之后，就会删除这个socket的AE_WRITABLE事件和命令回复处理器的关联关系。  

![一次通信过程.png](/image/redis/2-2一次通信.webp)

### 3.为什么单线程redis还可以支撑高并发？  
（1）纯内存操作。  
（2）核心是基于非阻塞的IO多路复用机制  
（3）单线程避免了多线程上下文切换的开销。
