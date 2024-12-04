### 1.redis是用来干嘛的？
>Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache and message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs, geospatial indexes with radius queries and streams. Redis has built-in replication, Lua scripting, LRU eviction, transactions and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster.

通过上面redis官网的说明可以看出，redis是一个可以对内存数据结构进行存储的东西，它可以用作数据库、缓存和消息代理。它支持数据结构，如字符串，散列，列表，集合，带有范围查询的排序集，位图，超级日志，具有半径查询和流的地理空间索引。Redis具有内置复制，Lua脚本，LRU驱逐，事务和不同级别的磁盘持久性，并通过Redis Sentinel提供高可用性并使用Redis Cluster自动分区。  
在项目中主要用来用作数据的缓存，将数据缓存在redis中，减轻对底层数据库的访问压力，获得更高的并发和更快的请求响应速度。  

### 2.在项目中如何使用?
我们知道，项目中的数据一般情况下都是存在于数据库当中的，而且数据库的并发性能不是特别高，如果同时接收到大量的请求，数据库可能就会崩掉，而且sql查询会消耗一定的时间，增加请求的响应时间，所以不用缓存会出现系统无法支撑大量的并发情况，请求响应时间会变长等问题。  
如果我们在第一次访问某个数据的时候，比如根据一个订单id获取订单的详细信息，将这条数据再返回的时候，也放到缓存里面去，那么下次就可以直接在缓存里面返回数据，不必去查询数据库，这样就可以减轻数据库的访问压力，而且缓存在内存中，势必要比直接访问数据库的速度要快很多，这样也就减小了请求的响应时间，redis在项目中就主要使用来解决数据的缓存问题。

### 3.为啥要使用缓存？
使用缓存的目的主要有两个：  
（1）高性能
比如说有一个很复杂的sql数据查询，这个查询要耗费大量的时间，如果每次都直接取数据查询，那必然会对请求响应时间造成很大的影响，如果能在第一次查询完毕之后，将其直接保存在缓存当中，下次查询的时候，直接在缓存中拿走现成的数据，这样就会大大缩短请求的响应时间。

（2）高并发
我们知道数据库能承受的并发是有限的，那么在流量高峰期(比如，抢购、打折、秒杀等等)，会有大量的请求进入我们的系统，比如查询某个商品的详情，如果我们没有缓冲，那么给次查询都要走数据库，假如我们的数据库每秒只能接受2000个请求，结果一秒钟进来了5000个请求，那么数据库就直接崩掉了，毫无高并发可言，而如果我们中间具有缓存服务，那么在第一个用户查询商品详情时(或者提前将放好)我们可以直接将商品的详情信息数据放到缓存里面，这样在后续用户查询时就可以直接走缓存，不走数据库，缓存是基于内存的，它的访问速度快，并发高；因此就可以提供一个高并发的支持。

### 4.用了缓存会出现什么问题？
主要常见的有下面三个问题  
1）缓存与数据库双写不一致  
2）缓存雪崩  
3）缓存穿透  
4）缓存并发竞争

