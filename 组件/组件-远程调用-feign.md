### Feign实现原理

基于OKHttp进行封装，结合了Ribbon和Hystrisx做负载均衡和服务熔断，基于动态代理实现的，对接口定义了FeignClient注解，Feign就会针对这个接口创建一个动态代理对象。Feign的动态代理是根据RequestMapping等注解来动态构造出你要请求的服务的地址。

针对这个地址、发起请求、解析响应。

可以替换底层的http实现，选择用okhttp或者httpClient,默认是httpClient



实现过程：

1. 通过@EnableFeignClients注解开启FeignClient
2. 根据Feign的规则实现接口，并加@FeignClient注解
3. 程序启动后，会扫描所有用@FeignClient注解声明的接口，将信息注入到容器中。
4. 当接口的方法被调用的时候，生成JDK的代理，来生成具体的RequestTemplate
5. RequestTempalte再生成具体的Request
6. Request交给Client去处理，其中Client可以是HttpUrlConnection也可以是OKHttp等
7. Client被封装到LoadBalanceClient类中，结合Ribbon做负载均衡。



