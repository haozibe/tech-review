![avatar](https://camo.githubusercontent.com/e871b5d002a9699e7a2d9fa0178af5c72f0743e0/68747470733a2f2f6e6574666c69782e6769746875622e636f6d2f487973747269782f696d616765732f687973747269782d6c6f676f2d7461676c696e652d3835302e706e67)

## 基本原理

![img](https://static.oschina.net/uploads/space/2018/0129/174915_Bvjy_2663573.png)

### 工作流程

1. 构造一个HystrixCommand或者HystrixObservableCommand对象，用于封装请求，并在构造方法配置请求被执行需要的参数。
2. 执行命令，四种执行命令的方式。
3. 判断是否开启了请求缓存，如果开启了，且缓存有效，则直接使用缓存响应请求。
4. 判断熔断器是否打开，如果打开，直接跳到第八步。
5. 判断线程池、队列、信号量是否已满，已满直接跳到第八步。
6. 执行HystrixObservableCommand.construct()或HystrixCommand.run()，如果执行失败跳到第八步。
7. 统计熔断器监控指标。
8. 执行fallback备用逻辑。
9. 返回请求响应。

### 舱壁模式

类似船体设计时设置隔离仓，如果只有一个仓体进水，将该舱体封闭隔离，将异常封闭在该舱体内。

### 断路器HystrixCircuitBreaker

- 周期（HystrixCommandProperties.default_metricsRollingStatisticalWindow = 1000ms)内，总请求超过一定量（HystrixCommandProperties.circuitBreakerRequestVolumeThreshold = 20）
- 错误请求占总请求数超过一定比例（HystrixCommandProperties.circuitBreakerErrorThresholdPercentage = 50%）

## 信号量

### 信号量超时

withExecutionIsolationThreadTimeoutMiliseconds

Hystrix在任务启动时会启动会另外一个线程HystrixTime去监控任务。如果在Timeout时间内，任务未完成，对于线程池模式，会把执行任务的线程设置为中断，如果是信号量模式，Hystrix不会对执行任务的线程做任何操作。然后再使用HystrixTime线程去执行fallback逻辑。

对于信号量超时模式，如果发生超时，Hystrix任务并不会结束，任务结束还是得依赖run方法执行。对于线程池超时模式，如果发生超时，Hystrix任务可以结束，不依赖与run方法执行情况，但是run方法可能还会在执行，这依赖于run方法中的代码逻辑。