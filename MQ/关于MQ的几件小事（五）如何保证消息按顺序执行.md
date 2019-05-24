### 1.为什么要保证顺序
消息队列中的若干消息如果是对同一个数据进行操作，这些操作具有前后的关系，必须要按前后的顺序执行，否则就会造成数据异常。举例：
比如通过mysql binlog进行两个数据库的数据同步，由于对数据库的数据操作是具有顺序性的，如果操作顺序搞反，就会造成不可估量的错误。比如数据库对一条数据依次进行了 插入->更新->删除操作，这个顺序必须是这样，如果在同步过程中，消息的顺序变成了 删除->插入->更新，那么原本应该被删除的数据，就没有被删除，造成数据的不一致问题。
### 2.出现顺序错乱的场景
（1）rabbitmq
①一个queue，有多个consumer去消费，这样就会造成顺序的错误，consumer从MQ里面读取数据是有序的，但是每个consumer的执行时间是不固定的，无法保证先读到消息的consumer一定先完成操作，这样就会出现消息并没有按照顺序执行，造成数据顺序错误。
![rabbitmq消息顺序错乱第一种情况示意图.png](https://upload-images.jianshu.io/upload_images/8494967-e450c6cb00e84866.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

②一个queue对应一个consumer，但是consumer里面进行了多线程消费，这样也会造成消息消费顺序错误。
![abbitmq消息顺序错乱第二种情况示意图.png](https://upload-images.jianshu.io/upload_images/8494967-65a77852d22d0833.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（2）kafka
①kafka一个topic，一个partition，一个consumer，但是consumer内部进行多线程消费，这样数据也会出现顺序错乱问题。
![kafka消息顺序错乱第一种情况示意图.png](https://upload-images.jianshu.io/upload_images/8494967-8cc85c5a6cc9bbf5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

②具有顺序的数据写入到了不同的partition里面，不同的消费者去消费，但是每个consumer的执行时间是不固定的，无法保证先读到消息的consumer一定先完成操作，这样就会出现消息并没有按照顺序执行，造成数据顺序错误。
![kafka消息顺序错乱第二种情况示意图..png](https://upload-images.jianshu.io/upload_images/8494967-ad745af0ef9c38a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3.保证消息的消费顺序
（1）rabbitmq
①拆分多个queue，每个queue一个consumer，就是多一些queue而已，确实是麻烦点；这样也会造成吞吐量下降，可以在消费者内部采用多线程的方式取消费。
![一个queue对应一个consumer](https://upload-images.jianshu.io/upload_images/8494967-12a89bd74e2f5135.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

②或者就一个queue但是对应一个consumer，然后这个consumer内部用内存队列做排队，然后分发给底层不同的worker来处理
![一个queue对应一个consumer，采用多线程.png](https://upload-images.jianshu.io/upload_images/8494967-5edc7ed5df03d12a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

（2）kafka
①确保同一个消息发送到同一个partition，一个topic，一个partition，一个consumer，内部单线程消费。
![单线程保证顺序.png](https://upload-images.jianshu.io/upload_images/8494967-7deff1fc07849dc8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


②写N个内存queue，然后N个线程分别消费一个内存queue即可
![多线程保证顺序.png](https://upload-images.jianshu.io/upload_images/8494967-8fd1fac60635c2c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上一篇《[如何防止数据队列数据丢失](https://www.jianshu.com/p/8ed16edc73e4)》
下一篇《[消息积压在消息队列里怎么办](https://www.jianshu.com/p/07b2169bef49)》
