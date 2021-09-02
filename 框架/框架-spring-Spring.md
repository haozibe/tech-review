# Spring

## 什么是Spring

spring是一系列开发程序解决方案的框架，其中最重要的功能是IOC跟AOP。

### springApplicationContext和BeanFactory区别

spring的上下文就是ApplicationContext，SpringApplicationContext实现了BeanFactory接口，增加了一些拓展的功能，包括国际化，事件，资源访问等，用户可以根据实际场景运用不同的ApplicationContext对象。

BeanFactory是延迟加载，SpringApplication是一次性加载。beanFactory

### spring注入的方式

1. 接口注入
2. setter方式注入
4. 构造方法注入
5. 注解方法注入

### spring的单例实现方式

spring对bean的单例实现时通过注册表方式实现的，在一个currentHashMap中维护一张bean的注册表。

## 解决循环依赖

1. 采用无参数构造器。
2. 用setter的方式注入属性。
3. spring的三级缓存机制，singletonObject，earlySingletonObjects,SingletonFactories。


## bean的生命周期

- spring对bean进行实例化。

- spring将值和对bean的引用注入到bean中。

- spring通过Aware接口，将容器信息注入到bean。

- 通过BeanPostProcessor。进行进一步构造，会在InitialzationBean前后执行对应方法，bean对象会被传递进来，这个时候可以对bean对象进行处理。

- Initialzation。这个阶段也可以在bean正式构造完成前增加自定义逻辑，但是跟前置处理不同，这个函数不会把bean对象传递进来，因此该过程无法处理对象本身，只能增加一些额外的逻辑。

- BeanPostInitialzation，后置处理。

- DisposableBean，bean将一直驻留在应用上下文给应用使用，直到应用上下文被销毁，如果bean实现了接口，会在销毁前调用distory方法。

  ~~~mermaid
  graph TD
  A[实例化] --> B[调用BeanNameAware的setBeanName方法] 
  B --> C[调用BeanFactoryAware的setBeanFactory方法]
  C --> D[调用ApplicationContextAware的setApplicationContext方法]
  D --> E[调用BeanPostProcess的postBeforeInitialization方法]
  E --> F[调用initialization方法调用定制初始化方法]
  F --> G[调用BeanPostProcess的PostAfterInitailzation方法]
  G --> H[bean准备就绪]
  H --> I[调用dispostableBean的destory方法 定制destory]
  
  ~~~
  
  
  
  
  
  调用ApplicationContextAware的setApplicationContext方法 -> 调用BeanPostProcess的postBeforeInitializtion->InitializationBean的afterProperties->调用定制初始化方法->调用BeanPostProcess的postAfterInitialization方法->bean准备就绪->调用dispostableBean的destory方法->调用定制的destory方法

## bean的作用域

- singleton 单例，Spring IOC容器只会存在一个共享的bean实例，无论引用他的bean有多少个，始终指向同一个对象。

- prototype 原型，每次通过spring容器获取bean时，都会初始化一个新的对象，每个对象都有自己的属性和状态。

- session 在一次http session中，容器会返回该bean的同一个实例。而对不同的session创建新的实例，该bean实例仅在当前的session中有效。

- global session 在一个全局的http session中，容器会返回该bean的同一个实例，仅在使用protlet context时有效。

- request 在一次http请求中，容器会返回该bean的同一个实例。而对不同的Session请求则会创建新的实例，该bean实例仅在Session中有效。


### spring是如何管理bean的事件的

1. 通过initalzingBean和disposeableBean回调方法，分别在初始化和销毁时。
2. 针对特殊行为的其他Aware接口。
3. 配置文件中的custom init和destory方法。
4. @PostConstruct和@PreDestory。



## Spring IOC和工厂模式区别

IOC模式要比工厂模式要更灵活，因为对象在IOC中是动态生成的，如果对象中属性有变更，可以使用spring的热拔插模式对class进行替换，不必把程序停止。

### SpirngIoc的实现原理。

1. Resource定位，就是通过xml配置文件或者javaconfig配置类，找到描述bean对象的文件，才好完成后面对象创建与管理。
2. BeanDefinition的解析和注册，承继上面的找到bean对象描述信息后，我们需要在内存中用命名为BeanDefinition的对象去封装他。以BeanName为key，BeanDefinition为value注册到一个concurrentMap中。获取Bean的class对象，反射生成的实例对象，配置bean的那些依赖的对象或者属性。getBean会导致循环依赖
3. IOC的依赖注入，

## Spring AOP

什么是spring aop，aop指的是面向切面编程，通过aop我们可以对一个函数或一段函数声明切面，对该切面执行前中后进行自己的一些业务逻辑处理。aop一般运用在一些与业务无关的代码里面，常见的应用有：权限管理、日志管理、事务管理等。

### 静态代理

如果想要代理一个接口的多个实现类的话，需要声明多个代理类

~~~java
public interface InterfaceA{
  public void doSomeThing();
}
public class RealInstance implements InterfaceA{
  @Override
  public void doSomeThing(){
    syso("do something");
  }
}
public class ProxyImpl implement InterfaceA{
  private InterfaceA instance;
  public ProxyImpl(){
    instance = new RealInstance();
  }
  public void doSomeThing(){
    syso("proxy before do some thing");
    instance.doSomeThing();
    syo("proxy after do some thing");
  }
}
~~~



## aop原理

jdk反射：通过反射机制生成代理类的字节码文件，调用具体方法前调用InvokeHandler。代理的对象至少要实现一个接口。

~~~java
public interface Man{
  public void findObject();
}
public class Zhang implements Man{
  @override
  public void findObject(){
    Syso("find you");
  }
}
public class ManHandler implements InvokerHandler{
  private Man man;
  public ManHandler(Man man){
    this.man = man;
  }
  @Override
  public Object invoke(Object proxy,Method method,Object[] args) throws Throwable{
    before();
    method.invoker(man,null);
    after();
    return null;
  }
  public void before(){
    syso("before find you")；
  }
  public void after(){
    syso("after find you");
  }
}
public static void mian(String args[]){
  Man man = new Zhang();
  ManHandler manHandler = new ManHandler(man);
  Man proxyMan = (Man)Proxy.newProxyInstance(man.getClass().getClassLoader()
                                            ,new Class[](Man.class),manHandler);
  System.out.println(proxyMan.getClass.getName());
  proxyMan.findObject();
}
~~~



CGLIB工具：基于字节码实现，动态生成类的代理子类，所以被代理的方法不能声明为final。

### 事务通知类型

| 通知类型 | 接口               | 描述                     |
| -------- | ------------------ | ------------------------ |
| Around   | MethodInterceptor  | 拦截对目标方法的调用     |
| Before   | MethodBeforeAdvice | 在目标方法调用前调用     |
| After    | AfterReturnAdvice  | 在目标方法后调用         |
| Throws   | ThrowsAdvice       | 当目标方法抛出异常时调用 |

## spring-tx事务

原理： 基于数据库的事务，使用@Transaction注解，通过spring-aop为类跟方法生成代理，正常提交，异常回滚。

### 类型

- 声明式事务
- 编程式事务

### spring事务隔离级别

1. Read-uncommited 
2. Read-commided
3. Repeated-read
4. Serailizable 

### spring事务的传播行为

什么是事务的传播行为：事务传播行为用来描述，由一个被用事务传播行为修饰的方法被嵌套进另一个方法时，该方法的事务该如何传播。

1. PROPAGATION_REQUIRED 如果当前没有事务，新建一个事务，如果已经存在一个事务，加入他。
2. PROPAGATION_SUPPORTS 支持当前事务，如果当前没有事务，就以非事务方式执行。
3. PROPAGATION_MANDATORY 使用当前事务，如果没有事务，抛出异常
4. PROPAGATION_REQUIRES_NEW 新建事务，如果当前存在事务，把当前事务挂起
5. PROPAGATION_NOT_SUPPORTED 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起
6. PROPAGATION_NEVER 以非事务方式执行，如果当前存在事务，则抛出异常。
7. PROPAGATION_NEST 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作





