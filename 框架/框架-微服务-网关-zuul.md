### 服务分组

将业务上紧密相关的服务归到同一个组，该组中服务的使用放，都通过这个API网管进行访问。

### 原理

通过统一的servlet入口，用ZuulFilter进行拦截。

