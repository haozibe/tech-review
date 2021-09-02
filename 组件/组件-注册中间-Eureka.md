### Eureka原理

![text](https://github.com/Netflix/eureka/raw/master/images/eureka_architecture.png)

服务消费者启动时，会发送一个register请求给Eureka-Server，从服务注册中心获取服务注册清单。Eureka维护一份只读的服务注册清单（List<Application>）返回给客户端，该清单30s刷新一次。eureka在启动时会创建一个定时任务，注册成功后，eureka会维护一个心跳(heartBeat)，(TimerTask)每隔一段时间（默认是60s），将清单中超时（默认90秒）的服务从清单中剔除。客户端服务正常下线时，会发送一个下线的rest请求给Eureka，注册中心收到该请求后，会将该服务的状态置为down，同时把该下线事件传播下去。服务中心互相注册，当有注册的信息发到peer中服务中心的一个节点时，他会转发给其他节点，保证服务信息的一致。

ReadWriterMap和Register（ConcurrentHashMap）

```
ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>> registry
```

第一层String是AppName，

第二层String是InstanceId 默认 

\${spring.cloud.client.hostname}:\${spring.application.name}:${server.port}

Eureka如果在一段时间内（90s）没有收到该实例的心跳，会在清单中移除该实例，但是当网络分区故障发生时，微服务和Eureka之间无法正常通信，而微服务本身是运行的，此时不该移除该服务，所以引入了保护机制，保护工作机制是指如果在一段时间（15分钟）内，百分之85的服务节点都没有正常的心跳，Eureka就认为客户端和注册中心出现了网络故障，自动进入保护机制：

1. Eureka不在移除长时间没有收到心跳而应该过期的服务。
2. 仍然接受新服务的注册和查询请求，但是不会同步到其他节点，保证当前节点可用。
3. 网络稳定后，会将节点信息同步到其他节点。

不建议改变默认的心跳跟失效时间，因为可以由此判断客户端跟服务端之间的是否存在广泛的连接异常。

eurekaClient使用的是jersey和原生的servlet实现跟eurekaServer的连接c，可以通过重写实现类自定义。

netflix的monitor的timer unavailable-replicas



### CAP理论

C consistence

![text](https://img.alicdn.com/5476e8b07b923/TB1a8dGqb1YBuNjSszeXXablFXa)



P Partition Tolerance

![text](https://img.alicdn.com/5476e8b07b923/TB1QyQRqb1YBuNjSszeXXablFXa)

### Eureka包含组件

Ribbon、Jersey、Guava、Hystrix

### 两种场景：eureka集群节点宕机和eureka客户端节点宕机

请求宕机的eureka节点会进行熔断返回null。会存在一定概率注册不上。

如果eureka集群都宕机了，也不影响服务之间的通信。因为本地保留着服务清单。

eureka集群之间通信出现异常，但是客户端能够正常连接上eureka节点。

### eureka的register_rule





### eureka的lastDirtyTimestamp



### eureka的事件机制



###eureka中的ribbon

DiscoveryEnbaleServer

用私有静态内部类实现单例懒加载



### eureka的分区

1. region 地区
2. zone 机房，可以用zone来区分开发环境和测试环境。

### eureka三层缓存

1. ReadWriteMap，有服务提供者注册服务或维持心跳时，会修改ReadWriterMap,会定时把ReadWriterMap的数据更新到ReadOnlyMap中。ReadOnlyMap,当有服务调用者查询服务实例列表时，默认会从ReadOnlyMap读取，这样可以减少ReadWriterMap读写锁的竞争。
2. ResponseCache缓存。用ReadWriteMap和ReadOnlyMap。useReadOnlyCache开启ReadOnlyMap,配合Timer的timerTask.schedule定时调用,自定义的Key
3. Client客户端本地的服务清单RegisterList缓存。client snapshot

### 主要功能

1. 失效剔除 剔除90秒内超时没有续约的服务 evit
2. 自我保护 跟ZK、Consule最大的区别，ZK如果服务有超过一半不可用会导致整个集群不可用而瘫痪。
3. 服务注册 register
4. 服务同步 Fetch
5. 服务续约 renew

### Eureka和Zk区别

Eureka是基于AP设计的，首先保证的是高可用性，某个节点挂掉不会影响集群的正常运行，每个节点之间都是平等的，如果一段时间内百分之85%的节点都没有正常的心跳会自动进入保护机制。

ZK是基于CP设计的，保证的是高一致性，如果master节点长时间和其他节点失去联系时，需要进行重新选举，选举时间较长通常为30s到120s，这段时间内集群是不可用的，会导致选举期间注册中心瘫痪的风险，这时不能容忍的。

|     维度      | eureka                                                       | nacos                                                        | consul                                                       |
| :-----------: | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
|   社区生态    | 社区热度高                                                   | 社区热度较低、中文文档多                                     | 社区热度较高                                                 |
|     性能      | 8核16G服务端定时轮询查询更新，当服务数量过多(8000-10000)时，压力过大 | 8核16G单个节点9000以上的连接时会出现硬件负载过高场景         | 暂无                                                         |
| cap分布式特性 | AP可用性以及分区容错性;可用性：无主从节点，一个节点挂了自动切换其他节点,但可能会导致不同的注册中心节点的服务列表不一致，可以接受 | AP或CP                                                       | CP：作为分布式集群来说，是CP                                 |
|   维护状态    | 1.X停止维护；2.0版本闭源                                     | 近期才开源；只到0.9.0.release;                               | 一直在跟着spring cloud维护，目前到2.1.1                      |
|   重点功能    | 1.springcloud-starter服务，开箱即用2.有控制台                | 1、支持springcloud 服务发现、配置管理2、有web ui控制台3、通过了大规模的性能测试4、注册中心与配置中心 | 有控制台；缺点：暂时没有对consul集群的配置方式，即服务无法注册多个consul节点 |
|   注册使用    | service 注册时向eureka server发送自身服务信息；service每30秒向eureka server续约服务；service 每30秒拉取服务列表至本地缓存。 | 同样是使用服务不断上报心跳的方式保持在注册中心的活跃性；服务发现。一、不断从服务端查询可用服务实例。二、从已变服务队列中通知服务持有者的查询任务，服务发生变更时更新本地服务列表。 | 既支持consul主动检查服务的情况，也支持服务主动向consul发送心跳报告自己的健康状况。 |
|     其他      |                                                              | k8s集成，dubbo集成均支持                                     |                                                              |



| Feature                | Consul                 | ZK                      | NACOS           | Eureka        |
| ---------------------- | ---------------------- | ----------------------- | --------------- | ------------- |
| 服务健康检查           | 服务状态，内存，硬盘等 | （弱）长连接，keepAlive | 可配置          | 可配置        |
| 多数据中心             | 支持                   | -                       | -               | -             |
| kv储存服务             | 支持                   | 支持                    | 支持            | -             |
| 一致性                 | raft                   | ZAB                     | raft            | -             |
| cap                    | CP                     | CP                      | CP+AP           | CA            |
| 使用接口（多语言能力） | 支持http和dns          | 客户端                  | http/grpc       | http(sidecar) |
| watch支持              |                        |                         |                 |               |
| 自身监控               | metrics                | -                       | metrics         | metrics       |
| 安全                   | Acl/https              | acl                     | https支持（弱） | -             |
| Spring cloud集成       | 支持                   | 支持                    | 支持            | 支持          |

### 参数调优

#### 服务端

(官方不建议调整时间间隔参数，用于鉴别服务端和客户端之间的连接问题)

针对Client下线没有通知Eureka的问题，可以调整EvictionTask的频率，默认60s，调整为5s。

针对响应缓存responseCache的问题，可以根据情况考虑关闭readOnlyCacheMap;

或者调整readWriteCacheMap的过期时间。

#### 客户端

调整Ribbon刷新时间（客户端） ribbon.ServerListRefreshInterval:5000 默认30s

针对新服务上线，客户端拉取信息不及时的问题，可以考虑调高拉取Server注册信息的频率默认30秒改为5s.

极端情况下，会存在11秒钟的服务不可用。需要引入重试机制,Spring-retry，可以避免服务主动下线的时间。

下线服务可以通过手动发送REST请求/eurekaUnRegister给Eureka进行实例的下线,如果实现了actuator可以通过发送Post请求修改状态为DOWN下线。

~~~java
ribbon.OkToRetryOnAllOperations:true
ribbon.MaxAutoRetriesNextServer:3
ribbon.MaxAutoRetries:1
ribbon.ReadTimeout:3000
ribbon.ConnectTimeOut:3000
ribbon.retryableStatusCodes:404,500,503
spring.cloud.loadBalancer.retry.enable:true
~~~

### Nacos

- 支持实时推送，达到秒级服务发现。

- 多层容灾机制，尽量保证服务发现中心宕机不影响应用调用。