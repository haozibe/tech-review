# RockerMq组成

![img](https://cai-java.oss-cn-hangzhou.aliyuncs.com/java/RocketMQ%E6%9E%B6%E6%9E%84%E5%9B%BE.jpg)



### Rocketmq的nameServer

节点之间互不通信，节点的信息有可能不一致，但是通过消息容错机制进行处理。每个broker向所有的nameServer发送消息。



## Rocketmq事务

![](https://images2018.cnblogs.com/blog/471426/201808/471426-20180806134355388-167315766.png)

RocketMq4.3.0版本开始支持事务消息，实现类似于二阶段提交，消息系统标志MessageSysFlag中定义的:TRANSACTION_PREPARED_TYPE、TRANSACTION_COMMIT_TYPE、TRANSACTION_ROLLBACK_TYPE

![img](https://cai-java.oss-cn-hangzhou.aliyuncs.com/java/RocketMQ%E4%BA%8B%E5%8A%A1%E6%B6%88%E6%81%AF%E5%8E%9F%E7%90%86.jpg)

### RocketMQ零拷贝

RocketMQ的消息采用顺序写到commitlog文件，然后利用consume queue文件作为索引；RocketMQ采用零拷贝mmap+write的方式来回应Consumer的请求；
同样kafka中存在大量的网络数据持久化到磁盘和磁盘文件通过网络发送的过程，kafka使用了sendfile零拷贝方式；

