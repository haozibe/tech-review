### 分布式自增ID

1. redis用incr命令生产。
2. 雪花算法snowFlaker
3. mongo的objectId
4. zookeeper生成id

### 分布式事务实现方式

- 两阶段提交 RM TM分别负责本地事务提交 整体事务提交
- 二阶段提交事务
- 分布式事务的二阶段提交
- 消息事务

### 分布式寻址算法

- hash
- 一致性hash + 虚节点
- hash槽



### 分布式一致性算法

- paxos
- raft

