### 1.Cache aside pattern
这是最经典的 缓存+数据库 读写模式，操作如下：   
①读的时候，先读缓存，缓存没有就读数据库，然后将取出的数据放到缓存，同时返回请求响应。  

②更新的时候，先删除缓存，然后更新数据库。  

### 2.为什么是删除缓存，而不是更新缓存呢？  
①因为很多时候，缓存中放的并不是简单的从数据中取出来值，可能要进行一个状态的替换，一些数据的计算，还有可能要进行数据的组合等等。   

②二八法则，即20%的数据，占用了80%的访问量，这个更新的数据，可能是冷门数据，好久也访问不了不了一次，这样只会占用缓存内存。lazy思想，数据等你第一次要的时候再去加载，避免没必要的内存和时间浪费。  

### 3.最初级的缓存不一致问题以及解决方案
**问题** ：先修改数据库，再删除缓存，如果缓存删除失败了，那么就会早上数据库和缓存数据不一致。  
**解决思路** ：先删除缓存，再修改数据库，如果删除缓存成功了，如果修改数据库失败了，那么数据库中是旧数据，缓存中是空的，那么数据不会不一致。因为读的时候缓存没有，则读数据库中旧数据，然后更新到缓存中

### 4.比较复杂的数据不一致问题分析  
**问题** ：数据发生了变更，先删除了缓存，然后要去修改数据库，此时还没修改。  
一个请求过来，去读缓存，发现缓存空了，去查询数据库，查到了修改前的旧数据，放到了缓存中。数据变更的程序完成了数据库的修改。  
这样就又导致了数据不一致
### 5.为什么上亿流量高并发场景下，缓存会出现这个问题？
因为只有在对一个数据在并发的进行读写的时候，才可能会出现这种问题。
### 6.数据库与缓存更新与读取操作进行异步串行化  
为了解决上面的并发读写问题，可以考虑将更新和读取操作进行串行化。  
①更新数据的时候，根据数据的唯一标识，将操作路由之后，发送到一个jvm内部的队列里面去。  

②读取数据的时候，如果发现缓存中没有，那么将从数据库读取数据的操作和更新缓存的操作一起路由到同一个JVM内部的队列中去。   

③一个队列对应一个工作线程，然后线程从队列里面去取请求进行操作。  

这样就可以将同一个key的操作进行串行化，一个数据变更的操作，先执行，删除缓存，然后再去更新数据库，但是还没完成更新，此时如果一个读请求过来，读到了空的缓存，那么可以先将缓存更新的请求发送到队列中，此时会在队列中积压，然后同步等待缓存更新完成。

这里有一个优化点，一个队列中，其实多个更新缓存请求串在一起是没意义的，因此可以做过滤，如果发现队列中已经有一个更新缓存的请求了，那么就不用再放个更新请求操作进去了，直接等待前面的更新操作请求完成即可。

待那个队列对应的工作线程完成了上一个操作的数据库的修改之后，才会去执行下一个操作，也就是缓存更新的操作，此时会从数据库中读取最新的值，然后写入缓存中。

如果请求还在等待时间范围内，不断轮询发现可以取到值了，那么就直接返回; 如果请求等待的时间超过一定时长，那么这一次直接从数据库中读取当前的旧值。

### 7.高并发的场景下，该解决方案要注意的问题
①读请求长时间阻塞问题  
由于读请求进行了非常轻度的异步化，所以一定要注意读超时的问题，每个读请求必须在超时时间范围内返回。  
该解决方案，最大的风险点在于说，可能数据更新很频繁，导致队列中积压了大量更新操作在里面，然后读请求会发生大量的超时，最后导致大量的请求直接走数据库。  
务必通过一些模拟真实的测试，看看更新数据的频繁是怎样的。

②读请求并发量过高  
还有一个风险，就是突然间大量读请求会在几十毫秒的延时hang在服务上，看服务能不能抗的住，需要多少机器才能抗住最大的极限情况的峰值。  
但是因为并不是所有的数据都在同一时间更新，缓存也不会同一时间失效，所以每次可能也就是少数数据的缓存失效了，然后那些数据对应的读请求过来，并发量应该也不会特别大。

③多服务实例部署的请求路由    
可能这个服务部署了多个实例，那么必须保证说，执行数据更新操作，以及执行缓存更新操作的请求，都通过nginx服务器路由到相同的服务实例上。  

④热点商品的路由问题。导致请求的倾斜  
万一某个商品的读写请求特别高，全部打到相同的机器的相同的队列里面去了，可能造成某台机器的压力过大。  
就是说，因为只有在商品数据更新的时候才会清空缓存，然后才会导致读写并发，所以更新频率不是太高的话，这个问题的影响并不是特别大。  
但是的确可能某些机器的负载会高一些。


