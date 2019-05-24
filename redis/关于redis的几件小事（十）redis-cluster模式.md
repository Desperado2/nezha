redis cluster是redis提供的集群模式。   
### 1.redis cluster的架构  
①可以有多个master node，每个master node 都可以挂载多个slave node。  

②读写分离的架构，对应每个master node来说，写就写到master node，读就从master node对应的slave node去读。  

③高可用。每个master node都有多个 slave node，如果master node挂了，redis cluster机制就会自动将某个slave node切换成 master node。

所以redis cluster是 多master + 读写分离 + 高可用 的。只要基于edis cluster去搭建集群，就可以达到搭建 replication复制+主从架构+读写分离+哨兵集群+高可用的 集群架构。  

![redis cluster的架构  示意图.png](/image/redis/10-1cluster架构.webp)

### 2.redis cluster和replication+sentinal
①输入数据量很少，主要是承载高并发的场景，单机就可以了。   

②replication只有一个master node节点，多个slave node节点，所有节点上面的数据是一样的，所有它可以提高读请求的吞吐量，然后可以搭建一个Sentinal集群去保证高可用，但是他无法去扩容数据的存储量。   

③redis cluster，可以有多个master node去存储不同的数据，所有他适合数据量特别大的场景，而且它可以不用Sentinal集群就可以保证高可用。  

### 3.数据分布算法  
#### (1)redis cluster机制  
①自动将数据分片，每个master node上面存放一部分数据。  

②提供内置的高可用支持，部分master node不可用时，还是可以工作的。  

③在redis cluster架构中，每个redis要开发两个端口，比如一个是6379，那么另一个就是加10000之后的端口号，比如16379。16379端口是用来进行节点间通信的，也就是cluster bus集群总线，cluster bus的通信用来进行故障检测、配置更新、故障转移授权等操作。   

④cluster bus用来一种二进制的协议，主要用于节点间进行高效的数据交换，占用更少的网络带宽和处理时间。

####(2)最古老的hash算法  
**原理** ：来了一个key之后，计算hash值，然后对master node节点数量取模，将数据哈希到不同的节点。  
**存在问题** :会导致大量缓存重建的问题。这种情况下，一旦一个master宕机了，所有的请求过来之后，就会对新的节点数量(原节点数量-1)去取模，然后去相应的node取数据，这样会导致请求走不到原本路由到的实例上面去，导致大量的key瞬间全部失效。  

![)最古老的hash算法  示意图.png](/image/redis/10-2最古老hash.webp)

#### (3)一致性hash算法(自动缓存迁移)

**原理** ：将所有master node落在一个圆环上面，然后，有一个key过来之后。同样就是hash值，然后会用hash值在圆环对应的各个点上(每个点都有一个hash值)去对比，看hash值落在那个位置，落在圆环上面以后，就会顺时针旋转去寻找距离自己最近的一个节点，数据的存储于读取都在该节点进行。   
**优势** ：保证了任何一个master宕机，只会影响之前在那个master上面的数据，因为照着顺时针走，全部在之前的master上面找不到了，master也宕机了，就会继续顺着顺时针走到下一个master节点去。这样就只会有一部分数据丢失。  
**存在问题** ：假如有3个master，那么就会丢失1/3的数据，这也是很多的。还会存在 缓存热点的问题。 **缓存热点问题** ：可能在某个hash区间内存在大值特别多，那么就会导致大量的数据进入同一个master，造成该master出现瓶颈。  

![一致性hash算法示意图.png](/image/redis/10-3一致性hash.webp)

#### (4)一致性哈希(自动缓存迁移)+虚拟节点(自动负载均衡)  
为了解决上面的问题，在一致性哈希的基础上增加了虚拟节点方案来处理。  
**原理** ：给每个master都做了一部分的虚拟节点，这部分虚拟节点也分布在这个圆环上面，那么在每个区间内，大量的数据就会均匀的分布在不同节点上。  

![一致性哈希+虚拟节点示意图.png](/image/redis/10-4虚拟节点hash.webp)

#### (5)hash slot算法
**原理** ：①redis cluster有固定的16384个hash slot，每个key计算CRC16值，然后多
16384取模，可以获取key对应的hash slot。  
②redis cluster中每个master节点都会持有一部分hash slot。  
③增加一个master，就讲其他master的hash slot移动一部分给新加入的master。  
④减少一个master，就将他的hash slot移动到其他master上面去。  
⑤移动hash slot的成本是非常低的。   
⑥客户端的api是可以指定hash tag来让数据走同一个hash slot的。  

![hash slot算法 示意图.png](/image/redis/10-5hashslot.webp)

### 4.redis cluster的核心原理   
#### (1)节点间的内部通信机制
**1.基础通信原理**  
①redis cluster节点间采取gossip协议进行通信。   
跟集中式不同，不是将集群元数据(节点信息、故障等等)集中存储在某个节点上，而是相互之间不断通信，保持整个集群所有节点的数据是完整的。  

②维护集群的元数据的两种方式对比   
A.集中式  
**优点** ：元数据的更新和读取，时效性非常好，一旦元数据出现了变更，立即就更新到集中式的存储中。  
**缺点** ：所有的元数据的更新压力全部集中在一个地方，可能导致元数据的存储有压力。  

![集中式 示意图.png](/image/redis/10-6集中式.webp)

B.gossip  
**优点** ：元数据的更新比较分散，不是集中在同一个地方，更新请求会陆陆续续到达所有节点上去更新，有一定的延时，降低了压力。   
**缺点** ：元数据更新有延时，可能会导致集群的一些操作会有一些滞后。  

![gossip示意图.png](/image/redis/10-7gossip.webp)

③10000端口   
每个节点都有一个专门用于节点间通信的端口号，就是自己提供服务的端口号+10000。每个节点每隔一段时间都会往另外几个节点发送ping消息，同时其他节点接收到ping之后会返回pong消息。

④节点间交换的信息  
包含故障信息，节点的增加和移除，hash slot信息等等。  

**2.gossip协议**  
gossip协议，即流言协议。gossip协议包含多种消息，包括ping,pong,meet,fail等等。   
①ping： 每个节点都会频繁的给其他节点发送ping，其中包括自己的状态还有自己维护的集群元数据，互相通过ping交换元数据。  

②meet：某个节点发送meet给新加入的节点，让其加入进群中，然后新节点就会开始与其他节点进行通信。   

③pong：作为ping和meet的响应，包含自己的状态和其他信息，也可以用于信息广播和更新。  

④fail：某个节点判断另一个节点fail之后，就会发送fail消息给其他节点，通知其他节点，指定的节点宕机了。  

**3.深入ping消息**   
①ping很频繁，而且要携带一些元数据，所以会加重网络负担。   

②每个节点每秒会执行10次ping，每次会选择5个最久没有通信的其他节点去通信。   

③如何发现某个节点通信时延达到了 cluster_node_timeout/2,那么立即发送ping，避免数据交换延时太长，落户的时间太多。   

④可以调节 cluster_node_timeout的值，如果调节比较大。就会降低ping发送的概率。   

⑤每次ping都会带上自己的信息。还要带上1/10其他节点的信息，发送出去进行交换。    

⑥至少包含3个其他节点的信息，最多包含总节点-2个其他节点的信息。

#### (2)面向集群的jedis内部实现原理
**1.基于重定向的客户端**   
redis-cli c，自动重定向   

①请求重定向   
客户端可能会挑选任意一个redis实例去发送命令，每个redis实例接收到命令之后，都会接受key对应的hash slot，如果在本地就在本地处理，否则返回moved给客户端，让客户端进行重定向。  
cluster keyslot mykey ，可以查看一个key对应的hash slot是什么。  
用redis-cli的时候，可以加入-c参数，支持自动的请求重定向，redis-cli接收到moved之后，会自动重定向到对应的节点执行命令。   

②计算hash slot   
计算hash slot的算法，就是根据key计算CRC16值，然后对16384取模，拿到对应的hash slot。  
用hash tag可以手动指定key对应的slot，同一个hash tag下的key，都会在一个hash slot中，比如set mykey1:{100}和set mykey2:{100}。

③hash slot查找  
节点间通过gossip协议进行数据交换，就知道每个hash slot在那个节点上面了

**2.smart jedis**   
①什么是smart jedis：   
基于重定向的客户端，很消耗网络IO，因为大部分情况下，可能都会出现一次请求重定向，才能找到正确的节点。

所以大部分的客户端，比如java redis客户端，就是jedis，都是smart的。

本地维护一份hashslot -> node的映射表，缓存，大部分情况下，直接走本地缓存就可以找到hashslot -> node，不需要通过节点进行moved重定向。  

②JedisCluster的工作原理   
A：在JedisCluster初始化的时候，就会随机选择一个node，初始化hash slot到node的映射表，同时为每个节点创建一个JedisPool连接池。   
B：每次基于JedisCluster执行操作，首先会在本地计算key的hash slot，然后在本地映射表中找到节点。  
C：如果那个node真好还是持有那个hash slot，那么就OK。  
D：如果JedisCluster API返回的是moved，那么利用该节点的数据，更新本地的hash slot 和node的映射表。   
E：重复上面的步骤，知道找到对应的节点，如果重试超过5次，就会报错，抛出JedisClusterMaxRedirectionException异常。   

jedis老版本，可能会出现在集群某个节点故障还没完成自动切换恢复时，频繁更新hash slot，频繁ping节点检查活跃，导致大量网络IO开销。

jedis最新版本，对于这些过度的hash slot更新和ping，都进行了优化，避免了类似问题。

③hash slot迁移和ask重定向   
A：如果hash slot正在进行迁移，那么会返回ask重定向给jedis，  
B：jedis接收到ask重定向之后，会重新定位到目标节点去执行。  
C：但是因为ask发生在hash slot迁移过程中，所以收到ask是不会更新hash slot本地缓存。   
D：已经可以确定说hash slot已经迁移完了，moved是会更新本地hash slot到node的映射缓存的。  

#### (3)高可用与主备切换原理
redis cluster的高可用原理，几乎和哨兵时类似的。   
**1.判断节点宕机**   
①如果一个节点认为另一个节点道济，那么就是pfail，主观宕机。   

②如果多个节点都认为另外一个 节点宕机了，那就是fail，客观宕机。   

③在cluster-node-timeout内，某个几点一直没有返回pong，那么就认为pfail。   

④如果一个节点认为某个节点pfail了，那么会在gossip ping消息中，发送给其他节点，如果超过半数的节点都认为pfail了，那就好变成fail。   

**2.从节点过滤**  
①对宕机的master node，从其所有的slave node中，选择一个切换成master node。   

②检查每个slave node与master node断开连接的时机，如果超过了cluster-node-timeout * cluster-slave-validity-factor，那么这个节点就没有资格切换成 master node，直接被过滤。   

**3.从节点选举**   
①每个从几点，都根据自己对master复制数据的offset，来设置一个选举时间，offset越大的从节点，选举时间越靠前，优先进行选举。   

②所有的master node开始slave选举投票，给要进行选举的slave进行投票，如果大部分master node(N/2 + 1)都投票给了某个从节点，那么选举通过，那个从节点可以切换成master node。  

③从节点执行主备切换，从节点切换为主节点。  

**4.与哨兵进行比较**   
整个流程跟哨兵相比，非常类似，所以说，redis cluster功能强大，直接集成了replication和sentinal的功能
