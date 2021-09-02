### mongodb

mongodb不支持事务，通过精简事务达到性能提高。

### mongodb原理

默认引擎为MMAPV1，还有一个新引擎wiredTiger可选。collection底层是partition的单位。

对update,insert,read性能较高，锁级别为表锁，并发表现不好，文档大小固定为2的次幂



3.2以上版本默认为wiredTiger，支持粒度更细的锁，在并发下表现更好，所有write都是文档级别的锁，而不是collection级别的

### mongodb索引



### mongodb分片

movechunk文件夹，会在balance平衡分片数据时产生临时文件，目前需要手动删除。

分片如果停止了，会返回错误。

分片集群至少一个ConifgSvr，一个分片，一个Mongos组成。

configSvr配置服务器： 保存集群和分片的元数据 --congifSvr指定为配置服务器

mongos进程 供应用程序连接，充当客户端，聚合分片的数据



### 为什么使用分片

1. 增加使用的RAM
2. 增加可用磁盘空间
3. 减轻单台服务器的负载
4. 处理单个mongodb无法承受的吞吐量