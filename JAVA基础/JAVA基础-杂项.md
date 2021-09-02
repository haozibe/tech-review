### Object的9种方法

1. getClass() 获取对应的class
2. clone() 浅复制
3. notify() 释放该类上指定等待的线程
4. notifyAll() 释放该类上所有等待的线程
5. wait() 释放锁
6. equals()
7. hashcode() 重写equals要重写hashcode
8. finalized() 标记优先进行回收
9. toString()

### 为什么重写equal要重写hashcode

Object里面的equal其实也是基于引用的对比，如果x与y时不同的对象，hashcode时不相同的，为了维持这个原则。

### 基本数据类型长度

byte boolean -128~127 256个数字占一个字节 8bit

short char -32768~32761 占两个字节 16bit

int float -2147483648~2147485647 占四字节 32bit

Long Double 64位 占8字节

### String中的hashcode如何实现

以31为权，每一位为字符的ASCII值进行运算，用自然溢出取模。

~~~java
public int hashcode(){
  int h = hash;
  if( h==0 && value.length >0){
    char val[] = value;
    for(int i =0;i<val.length;i++){
      h=31*h + val[i];
    }
    hash = h;
  }
  return h;
}
~~~



### 对象初始化

1. 分配内存地址
2. 对象初始化
3. 将对象指向分配的内存地址

JVM对类初始化时，会进行同步控制，可以利用这个特性构造完美的单例模式。

### 如何排查内存泄漏

1. Jps [options] [hostid]或者ps -eg|grep java找到当前java应用的进程id
2. 找到需要监控的id,用jstat去查看。
3. 用jmap命令导出对象文件，里面可以看到对象内存占用大小，找到内存占用异常的对象，去代码中排查该对象引起内存泄漏的原因。

### 优雅停机

基于JVM的ShutDownHook,spring-actuator的shutdown接口



### 双亲委派机制

![img](https://img-blog.csdnimg.cn/20190617122458779.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1b3lhbmdfamF2YQ==,size_16,color_FFFFFF,t_70)

### 什么情况下会破坏双亲委派模型

1. 第一次破坏jdk1.2之前
2. 第二次破坏,SPI,基础API调用用户代码。引入Thread Context ClassLoader
3. 第三次破坏,OSGI.

#### 举例

原生的JDBC中Driver驱动本身只是一个接口，并没有具体的实现，具体的实现是由不同数据库类型去实现的。例如，MySQL的mysql-connector-.jar中的Driver类具体实现的。 原生的JDBC中的类是放在rt.jar包的，是由启动类加载器进行类加载的，在JDBC中的Driver类中需要动态去加载不同数据库类型的Driver类，而mysql-connector-.jar中的Driver类是用户自己写的代码，那启动类加载器肯定是不能进行加载的，既然是自己编写的代码，那就需要由应用程序启动类去进行类加载。于是乎，这个时候就引入线程上下文件类加载器(Thread Context ClassLoader)。有了这个东西之后，程序就可以把原本需要由启动类加载器进行加载的类，由应用程序类加载器去进行加载了。