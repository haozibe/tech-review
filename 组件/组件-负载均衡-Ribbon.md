

### Ribbon原理

![img](http://springforall.ufile.ucloud.com.cn/static/img/e9998448e095176c2454fafd66ea57ab1511332)

### 功能

- 负载均衡
- 容错
- 多协议（HTTP，TCP，UDP）支持异步和反应模型
- 缓存和批处理

### 过程

1. 获取用@LoadedBalanced注解标记的restTemplate。
2. 给restTemplate注册一个拦截器filter，每当该restTemplate进行http请求时，拦截器进行拦截。
3. filter中ILoadBalancer通过获取该次请求远程服务集群的所有清单。
4. 通过特定的负载均衡策略，获取到请求的具体服务地址。
5. 请求目标地址，返回结果。

### ILoadBalancer

![img](https://images2018.cnblogs.com/blog/166781/201802/166781-20180221142042985-1539286285.png)

#### IPing检验服务是否存活

- PingUrl 真实的去ping 某个url，判断其是否alive
- PingConstant 固定返回某服务是否可用，默认返回true，即可用
- NoOpPing 不去ping,直接返回true,即可用。
- DummyPing 直接返回true，并实现了initWithNiwsConfig方法。
- NIWSDiscoveryPing，根据DiscoveryEnabledServer的InstanceInfo的InstanceStatus去判断，如果为InstanceStatus.UP，则为可用，否则不可用。

#### IRule选取服务的规则

- BestAvailableRule 选择最小请求数
- ClientConfigEnabledRoundRobinRule 轮询
- RandomRule 随机选择一个server
- RoundRobinRule 轮询选择server
- RetryRule 根据轮询的方式重试
- WeightedResponseTimeRule 根据响应时间去分配一个weight ，weight越低，被选择的可能性就越低
- ZoneAvoidanceRule 根据server的zone区域和可用性来轮询选择

#### ServerList服务集群的列表



#### 方法

1. addServer()

2. markServerDown()

3. choseServer()

4. getReachableServer()

5. getAllServer()

   

### 负载均衡策略：

1. RandomRule 随机
2. RoundRobinRule轮询策略
3. WeightedResponseTimeRule 加权策略
4. BestAvailable请求数最少策略
5. hash哈希
6. 一致性哈希

