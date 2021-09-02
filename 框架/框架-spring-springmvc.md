## SpringMVC的流程

发送请求到DispatchServlet拦截器，交给handlerMapping，handlerMapping根据绝对路径映射到对应的Controller，处理完Controller的业务逻辑后，包装成ModelAndView返回给ViewResolver解析器渲染页面。

### springmvc和strust2的区别

strust2是基于action的，springmvc是基于方法的，springmvc里面的请求变量是独享的，strust2是共享的。



### 手写springmvc dispatchServlet

~~~java
public class MyDispatchServlet extends Servlet{
  private Map<String,Method> handlerMapping = new HashMap<>();
  @Override
  public void init(){
    //1. 加载配置文件
    loadProperties(config.getInitParamter("contextConfiguration"));
    //2. 初始化所有关联的类，扫描用户设定包下面所有的类
    scanner(properties.getProperty("scanPackage"));
    //3. 拿到扫描的类，通过反射机制，实例化，并且放到ioc容器中
    doInstance();
    //4. 初始化HandlerMapping（"将url和method对应上"）
    initHandlerMapping();
  }
}
~~~

