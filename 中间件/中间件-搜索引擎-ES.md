### 原理

牺牲了插入更新的性能，提高查询的性能。为每一个filed都设置了索引。

将磁盘的东西尽量搬进内存，减少磁盘随机读取次数，组合各种奇技淫巧的压缩算法，用及其苛刻的态度使用内存。

### 海量数据中怎么提高效率

1. Filesystem cache

   Es搜索引擎是严重依赖于FileSytem cache,如果分配更多的内存，尽量让内存可以容纳所有的index segment file索引数据文件。

2. 数据预热

   热点数据需要做专门的缓存预热子系统，每隔一段时间提前访问下，让数据进入filesytem cache中

3. 冷热分离

   

### 倒排索引

1. 倒排索引分成term index ->term dictionary  -> postinglist
2. Term index记录了term的前缀的引用，通过term index可以快速定位到term dictionary的某个offset，然后再从这个位置往后顺序寻找
3. term index不需要存下索引term，利用FST压缩技术，可以使term index缓存到内存中，从term dictionary找到block位置之后，再去磁盘寻找term，减少磁盘的随机读次数。
4. 利用roaring bitmaps压缩对应的posting list文档id,要求posting list必须是有序的。
5. 大块用bit，小块用short，id范围以65535为界限，保证每一块都在65535范围内，4096来区分大小快。
6. 联合索引用调表的数据结构快速做与运算或者利用bitset按位做与运算



### 集群

ElasticSearch启动时会利用多播寻找集群中的其他节点，并与之建立连接。

在集群中，一个节点被选举成为主节点。这个节点负责管理集群的状态，当集群的拓扑结构改变时把索引分片分派到相应的节点中。



