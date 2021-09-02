

### serialization层



### 核心功能

- 服务监控
- 服务注册/注销
- 服务调用
- 服务定义
- 远程通信与信息交换
- 服务订阅与取消

### 服务调用过程

![img](https://user-gold-cdn.xitu.io/2020/4/22/1719f7ddccfef74a?imageslim)

![img](http://5b0988e595225.cdn.sohucs.com/images/20190614/9117ca82c507452db11b380e45ef9a94.jpeg)

![img](https://images2017.cnblogs.com/blog/1147548/201709/1147548-20170928143017294-1702265087.png)

1. 服务发送方发布服务到注册中心。
2. 消费方从注册中心订阅自己需要的服务。
3. 消费方调用已注册的可用服务。



![img](https://images2017.cnblogs.com/blog/1147548/201709/1147548-20170928143227981-1007327667.png)

### 协议支持

- Dubbo协议
- Hessian协议
- HTTP协议
- RMI协议
- WebService协议
- Thrift协议
- Memcached协议
- Redis协议



### Hessian协议原理

#### 执行过程

1. HessianSkeleton把客户端请求的流反序列化，得到对应方法名称和参数。
2. 服务类把执行方法返回值序列化到输出流。
   ![img](https://img-my.csdn.net/uploads/201205/11/1336727054_2599.jpg)



### Protobuf协议原理



### dubbo如何实现负载均衡

继承AbstractLoadedBalance类。dubbo中的invoker等同于springcloud的service

- RandomLoadedBalance
- ConsistenceHashLoadedBalance
- BestAvaliableLoadedBalance
- LeastAliveLoadedBalance
- RoundRobinLoadedBalance
- shortestResponseLoadedBalance




### dubbo如何控制并发和限流

####ExecuteLimitFilter

执行过程：

1. 首先会去获得服务提供者每服务每方法最大可并行执行请求数
2. 如果每个服务每个方法最大并行可执行请求数大于0，则创建一个rpcStatus实例。
3. 通过rpcStatus实例获取一个信号量，用tryAquired获取该信号量，如果返回false直接抛异常。
4. 如果获取到信号量，执行rpcStatus中的静态方法beginCount,对url和方法进行并发计数。
5. 调用服务。
6. 调用结束后计数调用rpcStatus静态方法endCount,计数结束。
7. 释放信号量。



- TPSLimter



### dubbo序列化方式



### dubbo的心跳机制



## 架构设计

### 架构分层

- Service
- Config
- proxy
- protocol
- registry
- cluster
- monitor
- exchange
- transport
- serialization

## SPI机制



## 注册中心



## 远程通信



## RPC



## 集群

