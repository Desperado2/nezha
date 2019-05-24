如果你用redis缓存技术的话，肯定要考虑如何用redis来加多台机器，保证redis是高并发的，还有就是如何让Redis保证自己不是挂掉以后就直接死掉了，redis高可用

redis高并发：主从架构，一主多从，一般来说，很多项目其实就足够了，单主用来写入数据，单机几万QPS，多从用来查询数据，多个从实例可以提供每秒10万的QPS。

redis高并发的同时，还需要容纳大量的数据：一主多从，每个实例都容纳了完整的数据，比如redis主就10G的内存量，其实你就最对只能容纳10g的数据量。如果你的缓存要容纳的数据量很大，达到了几十g，甚至几百g，或者是几t，那你就需要redis集群，而且用redis集群之后，可以提供可能每秒几十万的读写并发。

redis高可用：如果你做主从架构部署，其实就是加上哨兵就可以了，就可以实现，任何一个实例宕机，自动会进行主备切换。
下面详细介绍
### 一.redis如何通过读写分离来承载读请求QPS超过10万+？  
#### 1.redis高并发跟整个系统高并发的关系   
redis要搞高并发，那就要把底层的缓存搞好，让更少的请求直接到数据库，因为数据库的高并发实现起来是比较麻烦的，而且有些操作还有事务的要求等等，所以很难做到非常高的并发。  
redis并发做的好对于整个系统的并发来说还是不够的，但是redis作为整个大型的缓存架构，在支撑高并发的架构里面是非常重要的一环。  
要实现系统的高并发，首先缓存中间件、缓存系统必须要能够支撑起高并发，然后在经过良好的整体缓存架构设计(多级缓存、热点缓存),才能真正支撑起高并发。  
### 2.redis不能支撑高并发的瓶颈  
redis不能支撑高并发的瓶颈主要是单机问题，也就是说只有一个单一的redis，就算机器性能再怎么好，也是有上限的。  
### 3.如何支撑更高的并发  
单机的redis不可能支撑太高的并发量，要想支持更高的并发可以进行 **读写分离** 。对于缓存来说，一般都是支撑读高并发的，写的请求是比较少的，因此可以基于主从架构进行读写分离。  

配置一个master(主)机器用来写入数据，配置多个slave(从)来进行数据的读取，在master接收到数据之后将数据同步到slave上面即可，这样slave可以配置多台机器，就可以提高整体的并发量。  
![基于主从的读写分离简单示意图.png](/image/redis/5-1主从分离.webp)



### 二.redis replication以及master持久化对主从架构的安全意义  
#### 1.redis replication原理。  
一个master节点下面挂若干个slave节点，写操作将数据写到master节点上面去，然后在master写完之后，通过异步操作的方式将数据同步到所有的slave节点上面去，保证所有节点的数据是一致的。
#### 2.redis replication的核心机制  
（1）redis采用异步方式复制数据到slave节点，不过redis2.8开始，slave node会周期性地确认自己每次复制的数量。  
（2）一个master node 可以配置多个 salve node。  
（3）slave node也可以连接其他的slave node。  
（4）slave node做复制分时候，是不会阻塞master node的正常工作的。  
（5）slave node在做复制的时候，也不会阻塞自己的操作，它会用旧的数据来提供服务；但是复制完成的时候，需要删除旧的数据，加载新的数据，这个时候会对外暂停提供服务。  
（6）slave node主要用来进行横向扩容，做读写分离，扩容的slave node 可以提高吞吐量。  
#### 3.master持久化对主从架构的安全意义  
如果采用这种主从架构，那么必须要开启master node的持久化。不建议使用slave node作为master node的热备份，因为如果这样的话，如果master一旦宕机，那么master的数据就会丢失，重启之后数据是空的，其他的slave node要是来复制数据的话，就会复制到空，这样所有节点的数据就都丢了。  
要对备份文件做多种冷备份，防止整个机器坏了，备份的rdb数据也丢失的情况。

### 三.redis主从复制原理、断点续传、无磁盘化复制、过期key处理  
#### 1.主从复制原理  
①当启动一个slave node的时候，它会发送一个**PSYNC** 命令给master node。   

②如果这个slave node是重新连接master node，那么master node 仅仅会复制给slave部分缺失的数据；如果是第一次连接master node，那么就会触发一次 **full resynchronization**。  

③开始 full resynchronization的时候，master会启动一个**后台线程** ，开始生成一份RDB快照文件，同时还会将从客户端新接收到的所有写命令缓存在内存当中。  

④master node将生成的RDB文件发送给slave node，**slave现将其写入本地磁盘，然后再从磁盘加载到内存当中**。然后master node会将内存中缓存的写命令发送给slave node，**slave node也会同步这部分数据** 。  

⑤slave node如果跟master node因为网络故障断开了连接，**会自动重连** 。  

⑥master如果发现有多个slave node来重新连接，**仅仅会启动一个rdb save操作** ，用一份数据服务所有slave node。
#### 2.主从复制的断点续传  
从redis2.8开始支持断点续传。如果在主从复制的过程中，网络突然断掉了，那么可以接着上次
复制的地方，继续复制，而不是从头复制一份。  
**原理**：
master node会在内存中创建一个backlog，master和slave都会保存一个replica offset还有一个master id，offset就保存在backlog中。如果master和slave网络连接断掉了，slave会让master从上次的replica offset开始继续复制。但是如果没有找到offset，就会执行一次 full resynchronization操作。  
#### 3.无磁盘化复制  
无磁盘化复制是指，master直接再内存中创建RDB文件，然后发送给slave，不会在自己本地磁盘保存数据。 
**设置方式**     
配置 repl-diskless-sync和repl-diskless-sync-delay参数。  
repl-diskless-sync:该参数保证进行无磁盘化复制。  
repl-diskless-sync-delay：该参数表示等待一定时长再开始复制，这样可以等待多个slave节点从新连接上来。

#### 4.过期key处理  
**slave不会过期key** ，只有等待master过期key。  
如果master过期了一个key，或者淘汰了一个key，那么master会模拟**发送一条del命令** 给slave，slave接到之后会删除该key。

### 四.redis replication的完整流运行程和原理的再次深入剖析
#### 1.复制的完整流程
①slave node启动，仅仅保存了master node的信息，包括master node的host和ip，但是数据复制还没有开始。 master node的host和ip是在redis.conf文件里面的slaveOf中配置的 。

②slave node内部有一个定时任务，每秒检查是否有新的master node要连接个复制，如果发现，就跟master node建立socket网络连接。  

③slave node发送ping命令给master node。  

④如果master设置了requirepass，那么slave node必须发送master auth的口令过去进行口令验证。  

⑤master node第一次执行全量复制，将所有数据发送给slave node。  

⑥master node持续降写命令，异步复制给slave node。

#### 2.数据同步的机制
指的是slave第一次连接master时的情况，执行的是全量复制。  
①master和slave都会维护一个offset   
master会在自身不断累加offset，slave也会在自身不断累加offset。slave每秒都会上报自己的offset给master，同时master也会保存每个slave的offset。  
这个倒不是说特定就用在全量复制的，主要是master和slave都要知道各自的数据的offset，才能知道互相之间的数据不一致的情况

②backlog  
master node有一个backlog在内存中，默认是1M大。  
master node给slave node复制数据时，也会将数据在backlog中同步一份。  
backlog主要是用来做全量复制中断时候的增量复制的。  

③master run id  
redis通过info server 可以查看到master run id。  
**用途**：slave根据其来定位唯一的master。  
**为什么不用host+ip** ： 因为使用host+ip来定位master是不靠谱的，如果master node重启或者数据出现了变化，那么slave应该根据不同的master run id进行区分，run id不同就需要做一次全量复制。  
如果需要不更改run id重启redis，可以使用**redis-cli debug reload** 命令。

#### 3.全量复制流程与机制  
①master执行bgsave，在本地生成一份RDB文件。  

②master node将RDB快照文件发送给slave node，如果RDB文件的复制时间超过60秒(repl-timeout),那么slave node就会任务复制失败，可以适当调整这个参数。  

③对于千兆网卡的机器，一般每秒传输100M，传输6G文件很可能超过60秒。  

④master node在生成RDB文件时，会将所有新接到的写命令缓存在内存中，在slave node保存了RDB文件之后，再将这些写命令复制个slave node。

⑤查看client-output-buffer-limit slave 参数，比如[client-output-buffer-limit slave 256MB 64MB 60],表示在复制期间，内存缓存去持续消耗超过64M，或者一次性超过256MB，那么停止复制，复制失败。  

⑥slave node接收到RDB文件之后，清空自己的数据，然后重新加载RDB文件到自己的内存中，在这个过程中，基于旧数据对外提供服务。  

⑦如果slave node开启了AOF，那么会立即执行BRREWRITEAOF，重新AOF

rdb生成、rdb通过网络拷贝、slave旧数据的清理、slave aof rewrite，很耗费时间

如果复制的数据量在4G~6G之间，那么很可能全量复制时间消耗到1分半到2分钟
#### 4.增量复制流程与机制
①如果全量复制过程中，master和slave网络连接断掉，那么slave重新连接master会触发增刊复制。  

②master直接从自己的backlog中获取部分丢失是数据，发送给slave node。  

③master就是根据slave发送的psync中的offset来从backlog中获取数据的。
#### 5.心跳
master和slave互相都会发送heartbeat信息。  
master默认每隔10秒发送一次，slave node默认每隔1秒发送一次。
#### 6.异步复制
master每次接收到写命令之后，现在内部写入数据，然后异步发送给slave node

### 五.redis主从架构下如何才能做到99.99%的高可用性？
#### 1.什么是99.99%高可用?
>**高可用性**（英语：high availability，缩写为 HA），IT术语，指系统无中断地执行其功能的能力，代表系统的可用性程度。是进行系统设计时的准则之一。高可用性系统与构成该系统的各个组件相比可以更长时间运行。
高可用性通常通过提高系统的容错能力来实现。定义一个系统怎样才算具有高可用性往往需要根据每一个案例的具体情况来具体分析。
其度量方式，是根据系统损害、无法使用的时间，以及由无法运作恢复到可运作状况的时间，与系统总运作时间的比较。计算公式为: ![image.png](/image/redis/5-4计算公式.webp)
A（可用性），MTBF(平均故障间隔)，MDT(平均修复时间)
在线系统和执行关键任务的系统通常要求其可用性要达到5个9标准(99.999%)。

可用性|年故障时间
-|-
99.9999% | 32秒
| 99.999% | 5分15秒 |
| 99.99% | 52分34秒 |
| 99.9% | 8小时46分 |
| 99% | 3天15小时36分 |


#### 2.redis不可用
redis不可以包含了单实例的不可用，主从架构的不可用。  
不可用的情况：  
①主从架构的master节点挂了，如果master节点挂了那么缓存数据无法再写入，而且slave里面的数据也无法过期，这样就导致了不可用。  

②如果是单实例，那么可能因为其他原因导致redis进程死了。或者部署redis的机器坏了。  

**不可用的后果** ：首先缓存不可用了，那么请求就会直接走数据库，如果涌入大量请求超过了数据库的承载能力，那么数据库就挂掉了，这时候如果不能及时处理好缓存问题，那么由于请求过多，数据库重启之后很快就又会挂掉，直接导致整个系统不可用。

#### 3.如何实现高可用  
①保证每个redis都有备份。  
②保证在当前redis出故障之后，可以很快切换到备份redis上面去。  
为了解决这个问题，引入下面的哨兵机制。

### 六.redis哨兵架构的相关基础知识的讲解  
#### 1.什么是哨兵？
哨兵(Sentinal)是redis集群架构当中非常重要的一个组件，它主要有一下功能：  
①**集群监控** ，负责监控redis master和slave进程是否正常工作。  
②**消息通知** ，如果某个redis实例有故障，那么哨兵负责发送消息作为报警通知给管理员。  
③**故障转移** ，如果master挂掉了，会自动转移到slave上。  
④**配置中心** ，如果故障发生了，通知client客户端连接到新的master上面去。  

#### 2.哨兵的核心知识 
①哨兵本身是分布式的，需要作为一个集群去运行，个哨兵协同工作。  
②故障转移时，判断一个master宕机了，需要大部分哨兵同意才行。  
③即使部分哨兵挂掉了，哨兵集群还是能正常工作的。  
④哨兵至少需要3个实例，来保证自己的健壮性。  
⑤哨兵+redis主从结构，是无法保证数据零丢失的，只会保证redis集群的高可用。  
⑥对应哨兵+redis主从这种架构，再使用之前，要做重复的测试和演练。  

#### 3.为什么哨兵集群部署2个节点无法正常工作？
哨兵集群必须部署2个以上的节点。如果集群仅仅部署了2个哨兵实例，那么quorum=1(执行故障转移需要同意的哨兵个数)。  
![2节点示意图.png](/image/redis/5-2两个哨兵示意图.webp)

如图，如果这时候master1宕机了，哨兵1和哨兵2中只要有一个认为master1宕机了就可以进行故障转移，同时哨兵1和哨兵2会选举出一个哨兵来执行故障转移。  
同时这个时候需要majority(也就是所有集群中超过一半哨兵的数量)，2个哨兵那么majority就是2，也就说需要至少2个哨兵还运行着，才可以进行故障转移。  
但是如果整个master和哨兵1同时宕机了，那么就只剩一个哨兵了，这个时候就没有majority来运行执行故障转移了，虽然两外一台机器还有一个哨兵，但是1无法大于1，也就是无法保证半数以上，因此故障转移不会执行。

#### 4.经典的3节点哨兵集群

![三节点示意图.png](/image/redis/5-3三个哨兵.webp)


Configuration: quorum = 2，majority=2

如果M1所在机器宕机了，那么三个哨兵还剩下2个，S2和S3可以一致认为master宕机，然后选举出一个来执行故障转移

同时3个哨兵的majority是2，所以还剩下的2个哨兵运行着，就可以允许执行故障转移

### 七.redis哨兵主备切换的数据丢失问题：异步复制、集群脑裂  
#### 1.两种数据丢失的场景
①异步复制导致的数据丢失  
因为从master到slave的数据复制过程是异步的，可能有部分数据还没来得及复制到slave上面去，这时候master就宕机了，那么这部分数据就丢失了。  

②集群脑裂导致的数据丢失  
**什么是脑裂**：脑裂，也就是说，某个master所在机器突然脱离了正常的网络，跟其他slave机器不能连接，但是实际上master还运行着。  
此时哨兵可能就会认为master宕机了，然后开启选举，将其他slave切换成了master。  
这个时候，集群里就会有两个master，也就是所谓的脑裂。  

此时虽然某个slave被切换成了master，但是可能client还没来得及切换到新的master，还继续写向旧master的数据可能也丢失了。  
因此旧master再次恢复的时候，会被作为一个slave挂到新的master上去，自己的数据会清空，重新从新的master复制数据

#### 2.解决异步复制的脑裂导致的数据丢失  
要解决这个问题，就需要配置两个参数：  
**min-slaves-to-write 1** 和 **min-slaves-max-lag** ：  
表示 要求至少有一个slave 在进行数据的复制和同步的延迟不能超过10秒。  
如果一旦所有的slave数据同步和复制的延迟都超过了10秒，那么这个时候，master就会在接受任何请求了。  
①减少异步复制的数据丢失  
有了min-slaves-max-lag这个配置，就可以确保说，一旦slave复制数据和ack延时太长，就认为可能master宕机后损失的数据太多了，那么就拒绝写请求，这样可以把master宕机时由于部分数据未同步到slave导致的数据丢失降低的可控范围内。

（2）减少脑裂的数据丢失
如果一个master出现了脑裂，跟其他slave丢了连接，那么上面两个配置可以确保说，如果不能继续给指定数量的slave发送数据，而且slave超过10秒没有给自己ack消息，那么就直接拒绝客户端的写请求。
这样脑裂后的旧master就不会接受client的新数据，也就避免了数据丢失。

上面的配置就确保了，如果跟任何一个slave丢了连接，在10秒后发现没有slave给自己ack，那么就拒绝新的写请求。
因此在脑裂场景下，最多就丢失10秒的数据

### 八.redis哨兵的多个核心底层原理的深入解析（包含slave选举算法）

#### 1.sdown和odown两种状态  
sdown是主观宕机，就一个哨兵如果自己觉得一个master宕机了，那么就是主观宕机

odown是客观宕机，如果quorum数量的哨兵都觉得一个master宕机了，那么就是客观宕机

sdown达成的条件很简单，如果一个哨兵ping一个master，超过了is-master-down-after-milliseconds指定的毫秒数之后，就主观认为master宕机

sdown到odown转换的条件很简单，如果一个哨兵在指定时间内，收到了quorum指定数量的其他哨兵也认为那个master是sdown了，那么就认为是odown了，客观认为master宕机。

#### 2.哨兵集群的字段发现机制  
①哨兵相互之间的发现，是通过redis的pub/sub系统实现的，每个哨兵都会往 **\_\_sentinel__:hello** 这个channel里面发送一个消息，这时候其他的哨兵都可以消费这个消息，并感知其他哨兵的存在。  

②每个两秒钟，每个哨兵都会往自己监控的某个master+slave对应的**\_\_sentinel__:hello** channel里面发送一个消息，内容是自己的host、ip和run id还有对这个master的监控配置。  

③每个哨兵也会去监听自己监控的每个master+slave对应的**\_\_sentinel__:hello**  channel，r然后去感知到同样在监听这个master+slave的其他哨兵的存在。  

④每个哨兵还会根据其他哨兵交换对master的监控配置，互相进行监控配置的同步。  

#### 3.slave配置的自我纠正  
哨兵会负责自动纠正slave的一些配置，比如slave如果要成为潜在的master候选人，哨兵会确保slave在复制现有master数据；如果slave连接到了一个错误的master上，比如故障转移之后，那么哨兵会确保他们连接到正确的master上来。  
#### 4.选举算法  
如果一个master被认为odown了，而且majority数量的哨兵都允许了主备切换，那么某个哨兵就会执行主备切换，此时首先要选举一个slave出来。选举会考虑到一下情况：   
①slave跟master断开连接的时长  
②slave的优先级   
③slave复制数据的offset   
④slave的run id   

首先，如果一个slave跟master断开连接已经超过了 down-after-millisecondes 的10倍，外加master宕机的时长，那么slave就被认为不适合选举为master了。  
即：断开连接时间 > down-after-milliseconds * 10 + milliseconds_since_master_is_in_SDOWN_sate.   

对应剩下的slave按照如下规定排序：   
①首先，按照slave的优先级进行排序，slave priority越低，优先级就越高。  
②如果优先级相同，那么就看replica offset，那个slave复制了越多的数据，offset越靠后，优先级就越高。  
③如果上面都想同，那就选择run id最小的那个slave。  

#### 5.quorum和majority  
每次一个哨兵做主备切换，首先需要quorum数量的哨兵认为odown，然后选举出一个哨兵来做主备切换，这个哨兵还要得到majority数量哨兵的授权，才能正式执行切换。  

如果quorum < majority ,比如5个哨兵，majority就是3(超过半数)，quorum设置为2，那么就需要3个哨兵授权就可以执行切换。   

如果 quorum >= majority，那么必须quorum数量的哨兵都授权才可以进行切换，比如5个哨兵，quorum是5，那么必须5个哨兵都同意授权，才可以进行切换。

#### 6.configuration epoch

哨兵会对一套redis master+slave进行监控，有相应的监控的配置。

执行切换的那个哨兵，会从要切换到的新master（salve->master）那里得到一个configuration epoch，这就是一个version号，每次切换的version号都必须是唯一的。

如果第一个选举出的哨兵切换失败了，那么其他哨兵，会等待failover-timeout时间，然后接替继续执行切换，此时会重新获取一个新的configuration epoch，作为新的version号。

7、configuraiton传播

哨兵完成切换之后，会在自己本地更新生成最新的master配置，然后同步给其他的哨兵，就是通过之前说的pub/sub消息机制。

这里之前的version号就很重要了，因为各种消息都是通过一个channel去发布和监听的，所以一个哨兵完成一次新的切换之后，新的master配置是跟着新的version号的。

其他的哨兵都是根据版本号的大小来更新自己的master配置的。














