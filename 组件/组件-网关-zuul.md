### 原理

通过一个统一的Servlet入口拦截所有请求没然后通过内置的ZuulFilter对请求做拦截和过滤处理。ZuulFilter和servlet.Filter原理类似，ZuulFilter时ZuulServlet处理请求的时候调用的。

维护了FilterRegistry饿汉式单例（Filter的注册中心）

