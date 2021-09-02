[TOC]

## 多线程简介

### 进程、线程、协程区别

**进程**：具有独立的内存空间，一个应用至少有一个进程，上下文进程间的切换开销比较大，但是相对比较稳定安全。

**线程**：通过共享内存空间进行线程间的通信，切换上下文很快，资源开销较少，但是相比进程不够稳定容易丢失数据，一个进程至少拥有一个线程。

**协程**：是用户态的轻量级线程，协程的调度完全由用户控制，切换协程调度时，会把寄存器上下文和栈保存到其他地方，切回来时，先恢复先前保存的寄存器上下文和栈，直接操作栈则基本没有内核切换的开销，可以不加锁的访问全局变量。

### 守护线程

UserThread和DeamonThread，只要存在一个非守护线程未关闭，守护线程全部工作，只有当最后一个非守护线程关闭后，JVM会跟守护线程一起关闭。守护线程最简单的例子就是GC线程。用户可以在声明线程时通过setDeamon方法指定该线程为守护线程。

### 线程的实现

- 实现Runnable接口
- 实现Callable接口，Future返回执行结果。
- 继承Thread类，实现run方法。start方法会创建线程，直接调run方法不会创建线程。

### 并发编程三要素

- **原子性**：即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

- **可见性:  **  可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

- **有序性:  **即程序执行的顺序按照代码的先后顺序执行。happens before原则。

## 锁

### 介绍

锁是不同线程竞争资源时控制不同线程执行方式的工具，只有当线程获得锁时才能够访问同步的代码，否则需要等待其他线程释放锁后去竞争锁。

|              | 自旋锁                                                       | 互斥锁                                         |                   偏向锁                    | 轻量级锁                                                     | 重量级锁              |
| ------------ | ------------------------------------------------------------ | ---------------------------------------------- | :-----------------------------------------: | ------------------------------------------------------------ | --------------------- |
| **实现原理** | while循环调用CAS，通过cas.get()实现可重入                    | 基于CAS和AQS实现                               | 偏向锁会偏向第一个获取他的线程，通过CAS实现 | Mark Word中的部分字节CAS更新指向线程栈中的Lock Record        | *使用互斥量mutex阻塞* |
| **适用场景** | 持有锁时间比较短，非多线程少处理器情况                       |                                                |      无竞争且只有一个线程使用锁的情况       | 不存在锁竞争的场景                                           | 自旋大概率失败        |
| **优点**     | 不会发生上下文切换，执行速度快                               | 上下文切换，堵塞等待节约资源，可以选择是否公平 |       持有偏向锁的线程不需要进行同步        | 减少锁和释放锁所带来的的性能消耗                             | 避免自旋带来的cpu损耗 |
| **缺点**     | 浪费CPU资源，非公平锁                                        | 存在上下文切换的开销，执行性能差               |   大多数锁都是被不同线程访问，偏向锁多余    | 如果*锁竞争激烈*，那么轻量级将很快膨胀为重量级锁，那么维持轻量级锁的过程就成了浪费 | 线程堵塞耗费性能      |
| **变种**     | TicketLock，线程分配自增ID实现公平锁，排队号放在ThreadLocal中。自适应自旋锁。CLHlock，MCSlock |                                                |                                             |                                                              |                       |



### 乐观锁和悲观锁

**乐观锁**： 在修改时认为操作不出现多线程问题，但是更新时会判断其他线程在这之前有没有对线程修改，通常通过版本号和CAS控制。AtomicStampedReference来解决ABA问题。标记变量的版本号。-XX:PreBlockSpin控制自旋次数，挂起当前线程。

**悲观锁：**修改记录时，用排它锁控制其他线程对该记录的修改。只有当获取到排它锁的线程才能去修改该记录。sychronized实现是通过字节码，在前后文加monitorenter和monitorexit实现。

### 偏向锁、轻量级锁和重量级锁

如果不存在多线程竞争，当前线程通过CAS把线程ID更新到对象的MARK WORD中后，获取到锁，下一次获取锁时就不需要再执行CAS操作了，节约了性能，这就是偏向锁。

如果存在多线程竞争，当代码进入同步块时，当前线程会在栈帧中创建一个锁记录Lock Record区域，同时将锁对象的对象头MARK WORD拷贝到锁记录中，再尝试用CAS将MARK WORD更新为指向锁记录的指针。如果更新成功就成功获得锁，如果更新失败，JVM会首先检查锁对象的MARK WORD是否指向当前线程的锁记录。如果是则说明当前线程拥有锁，可以再次进入同步快，如果不是则说明其他线程占用锁，如果多个线程竞争锁，超过一定次数，轻量级锁会膨胀成重量级锁。

自适应自旋，如果一个线程多次自选都获取不到锁，就会降低他下次自旋的时间

### 死锁

当一个线程A持有自己的锁的同时去请求线程B的锁，而线程B持有自己的锁的同时去请求线程A的锁，这个时候就造成死锁。怎么解决死锁，通过join方法，保证线程按顺序获得锁。

用jstat -l id检测死锁，如果有Found x deadlock,表示当前进程有x个死锁。

#### 产生死锁的条件：

1. 互斥：一个资源只能被一个线程使用。
2. 请求与保持条件：一个进程因请求资源而堵塞时，对已获得的资源保持不放。
3. 不剥夺条件：线程已获得的资源，在未使用完前不释放。
4. 循环等待条件：若干个线程之间形成头尾相接的循环等待资源关系。

#### 解决办法：

1. **加锁顺序**。

2. **银行家算法：** 定义空闲资源池 通过试探分配 然后通过系统安全性算法判断是否安全 

   系统安全算法：先看进程池能否有进程可以用分配后的剩余资源执行完毕，若无，则会产生死锁，系统不安全，不分配。若有，则假设回收已分配给他的资源，将进程标记成已完成，再看其他进程能否以当前资源执行完毕，若存在，当前系统处于安全状态，并且根据可完成进程的分配顺序生成安全序列。

3. **加锁时限**：如果线程长时间没有获得所需要的锁，则释放所有锁等过一段时间再重试。

4. **死锁检测：**线程每获得一个锁在map中记录下锁的信息。

产生死锁后解决：

1. 通过jstack查看是否有死锁。
2. 通过Jconsole跟进程号确定死锁。

### wait和notify、notifyAll区别

wait立刻释放锁，notify、notifyAll会等线程执行剩余代码后才会释放锁。

### ReentrantLock和Sychronize区别

都是可重入锁，基于计数器，当前线程进入一次都计数器+1，只有当计数器为0时，才释放锁。

ReentrantLock可以选择是公平锁还是非公平锁，Sychronized更便利，ReentrantLock需要手动加锁和释放锁，锁粒度ReentrantLock比Synchronized要细。ReentrantLock可以通过Condition类实现分组唤醒需要唤醒的线程。ReentrantLock提供了一种能中断等待锁的线程机制。

ReentrantLock通过循环调用CAS操作实现加锁。都是基于AQS实现的。



## Sychronized

### 字节码实现

#### 修饰代码块时 

修饰类时采用的是monitorEnter和moniterExite。执行完方法，或者抛异常都会释放锁，体现是出现多次monitorexit。

#### 修饰方法时

修饰方法时，字节码体现的是ACC_SYCHRONIZED的访问标志，体现是否为同步方法，当方法被调用时，先判断是否是ACC_SYCHRONIZED标志，如果是的话去获取对象的monitor对象才进入方法体，当方法执行完时，会释放掉monitor对象。

#### 锁膨胀过程

![img](https://user-gold-cdn.xitu.io/2018/1/15/160f78651f12426e?imageslim)

#### 锁消除和锁粗化

锁粗化就是告诉我们任何事情都有个度，有些情况下我们反而希望把很多次锁的请求合并成一个请求，以降低短时间内大量锁请求、同步、释放带来的性能损耗。

锁削除是指虚拟机即时编译器在运行时，对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁进行削除。锁削除的主要判定依据来源于逃逸分析的数据支持，如果判断到一段代码中，在堆上的所有数据都不会逃逸出去被其他线程访问到，那就可以把它们当作栈上数据对待，认为它们是线程私有的，同步加锁自然就无须进行。 

#### 对象在内存中的布局

<img src="https://img-blog.csdnimg.cn/2019060422371035.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhbmR5Q0NDYXRpb24=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:100%;" />



#### monitor对象如何实现

C语言，mutex互斥锁，loop中循环修改。

#### mutex属性

PTHREAD_MUTEX_LOCK 堵塞

PTHREAD_MUTEX_TIMED_NP，这是缺省值，也就是普通锁。当一个线程加锁以后，其余请求锁的线程将形成一个等待队列，并在解锁后按优先级获得锁。这种锁策略保证了资源分配的公平性。

PTHREAD_MUTEX_RECURSIVE_NP，嵌套锁，允许同一个线程对同一个锁成功获得多次，并通过多次unlock解锁。如果是不同线程请求，则在加锁线程解锁时重新竞争。
　　自己再解析一下：嵌套锁也叫递归锁，在同一个线程中，由于业务问题，需要递归一块上锁代码，可以使用这个属性。

PTHREAD_MUTEX_ERRORCHECK_NP，检错锁，如果同一个线程请求同一个锁，则返回EDEADLK，否则与PTHREAD_MUTEX_TIMED_NP类型动作相同。这样就保证当不允许多次加锁时不会出现最简单情况下的死锁。在不同线程的处理业务的场景下，线程是最小处理单位，可以设置这个属性，避免线程内死锁。还能满足线程资源分配公平性

PTHREAD_MUTEX_ADAPTIVE_NP，适应锁，动作最简单的锁类型，仅等待解锁后重新竞争。

```
// 创建互斥锁
HANDLE CreateMutexA(
  LPSECURITY_ATTRIBUTES lpMutexAttributes, // 指向安全属性的指针

  // True ：创建MutexObject的线程，立刻拥有Mutex的控制权。
  // False：必须使用等待函数waitForSigleObject。
  BOOL                  bInitialOwner, 
  LPCSTR                lpName  // 指定互斥体对象的名字，可以用vbNullString创建一个未命名的互斥体对象。

);

// 释放控制权
BOOL ReleaseMutex(
  HANDLE hMutex
);


// 一般用于跨进程获取已经创建的Mutex对象
HANDLE OpenMutexW(
  // 访问的方式：
  // SYNCHRONIZE 允许互斥体对象同步使用
  // MUTEX_ALL_ACCESS 请求对互斥体的完全访问
  DWORD   dwDesiredAccess, 
  BOOL    bInheritHandle,  // true,子进程能够继承句柄
  LPCWSTR lpName           // 指定互斥体对象的名字
);
```

### AQS的原理

内部维护一个volatile修饰的成员变量state,表示有多少个线程持有锁，线程获得锁时，state+1，释放锁时，state-1,线程可以多次获取锁,state的更新时基于CAS实现的。内部还维护了一个链表，用来储存等待获取锁的线程的信息，每个节点代表的是等待着的线程的信息，这是个FIFO的链表。

acquire方法获取锁，如果获取失败通过addWaiter方法将当前线程写入队列，写入前会将当前线程包装成Node对象，通过CAS写入队列。如果CAS写入失败，则会调用enq方法写入队列，先判断队列是否为空，为空的话先初始化，然后将Node写入队尾。相当于自旋加CAS保证一定能写入队列。

获取锁的方式有三种：

1. 当前没有其他线程获取锁，当前线程去获取锁，直接获取成功，state+1；
2. 本身线程已经获取到锁了，需要再次获取锁，state+1；
3. 其他线程获取到锁了，当前线程信息维护到链表中，等待其他线程释放锁。



acquired方法，获取资源，获取失败CAS自旋插入队列尾部等待唤醒。

pack方法，等待。被唤醒只有两种情况，unpack和interrupt。

release方法，释放资源

### join方法

保证执行的顺序，先当前线程中调用其他线程的join方法，需要等待其他线程执行完毕。

### 进程和线程

线程是运行在进程内的，java支持多线程也支持通过Process,ProcessBuilder和Runtime.exec()的多进程。对操作系统来说，一个任务就是一个进程，一个进程至少要有一个线程。

### 线程的调度算法

抢占式，根据线程优先级、线程饥饿情况等数据算出一个总的优先级并分配下一个时间片给某个线程执行。

### 线程的生命周期

线程的状态：新建 就绪 运行 堵塞 死亡

创建 线程被new出来-> 就绪 调用了start方法等待获取到cpu使用权-> 运行 获得cpu使用权 开始执行run ->堵塞 等待堵塞 同步堵塞 其他堵塞-> 就绪 -> 运行 -> 销毁

Notify()，wait()，sleep()，NotifyAll()，join()，interrupted()

Lock，sychronized，lock，CountDownLatch，CyclicBarrirer，Semaphore

Volatile实现原理，内存屏障，读取时直接从主寸读取值，更新时立刻同步到主存。

### CountDownLatch和CyclicBarrier区别

CountDownLatch，执行countDown之后，线程还会继续执行。countdown主线程会等待其他线程执行完毕后才执行。可以不限制等待线程的数量，而会限制countDown的操作数。

CycilicBarrier会限制等待线程的数量。

CyclicBarrier执行cyc.await之后，线程必须等待所有线程执行完毕后才继续执行后续任务。

CountDownLatch计数器不能重置，CyclicBarrier计数器可以通过reset重置。

### CAS

ABA: AutomicStampedReference和AutomicMarktableReference

### 锁

锁是不同线程竞争资源时控制不同线程执行方式的工具，只有当线程获得锁时才能够访问同步的代码，否则需要等待其他线程释放锁后去竞争锁。

乐观锁,在修改时认为操作不出现多线程问题，但是更新时会判断其他线程在这之前有没有对线程修改，通常通过版本号和CAS控制。

悲观锁，修改记录时，用排它锁控制其他线程对该记录的修改。只有当获取到排它锁的线程才能去修改该记录。sychronized实现是通过字节码，在前后文加monitorenter和monitorexit实现。

volatile是通过内存屏障实现。

### volatile

可以避免指令的重排序，可以保证可见性、顺序性，不能保证原子性。类似i++操作。

一部分操作能保证原子性，例如long、double。jvm都对这部分做了优化，不会说如果没声明volatile多线程读到不一样的数据。

As-if-serial

Happen-before



### 对象初始化的过程：

给对象分配内存空间

初始化对象数据，调用构造方法

将对象指向到分配好的内存空间地址



![这里写图片描述](https://img-blog.csdn.net/20160228114034746)

## 线程安全的单例模式

~~~java
public class Singleton{
    public static volatile Singleton singleton;
    pubic Singleton(){};
    public static Singleton getInstance(){
        if(singleton == null){
            sychroniezd(Singleton.class){
                if(singleton == null){
                    singleton == new Singleton();
                }
            }
        }
        return singleton;
    }
}
~~~

- 为什么要用volatile去修饰singletion，为了避免指令重排序，在分配内存空间时，正常的顺序是先分配内存空间，然后再赋值，但是由于指令重排序，有可能会先分配赋值，然后再分配内存空间，这个时候其他线程去判断singletion是否为空的时候，会判断到singleton已经赋值，但是实际上singleton的构造还未结束，其他线程去调用该实例时会报错。

## 队列

1. ArrayBlockingQueue（有界队列）
2. DelayQueue scheduleThreadPool
3. LinkBlockingQueue FixThreadPool
4. SychronousQueue CacheThreadPool
5. PriorityQueue
6. LinkBlockingDeque

## 线程池



- 创建方式(为什么不推荐用Executors创建，一个无界队列可能导致oom，一个拒绝策略使用不当，任务满时不是预期结果)
  1. Executors.newCachedThreadPool()
  2. Executors.newFixedThreadPool()
  3. Executors.newScheduledThreadPool()
  4. Executors.newSingleThreadPool()
  5. ThreadPoolExecutors
- 常用参数有哪些：
  1. corePoolSize 最小线程数
  2. maxPoolSize 最大线程数
  3. KeepAliveTime 维持空闲时间
  4. allowCoreThreadTimeout 是否允许线程空闲退出
  5. queueCapacity 任务队列容量
  6. unit 存活时间的单位
  7. handler 超出线程范围和队列数量的处理程序
  9. workqueue，选择任务队列的结构，一般用LinkBlockingQueue（无界队列）和SychronousQueue

### 线程池的拒绝策略

1. AbordPolicy 直接抛异常，放弃任务（默认）
2. DiscardPolicy 不抛异常，丢弃任务
3. DiscardOldestPolicy 不抛异常，丢弃队列中等待最久的任务
4. CallerRunsPolicy 尝试用主线程的去执行任务的execute方法

### 如何自己实现一个线程池

1. 声明一个Queue用于保留线程。

2. 声明一个Queue用于保存队列。

3. 设置Max和min变量，用于控制线程池的大小。

4. 设置KeepAliveTime用于控制

5. 声明拒绝策略，用于处理任务溢出逻辑。

    



### Fork/Join框架




## 多线程常遇问题

### SimpleDateFormat

SimpleDateFormat是线程不安全的，是因为里面的Calender对象的属性不是原子性的，当多线程操作的时候，进行parse时会出现线程不安全问题。要避免设为static，如果设为static，推荐用threadLocal处理。

~~~java
static ThreadLocal<SimpleDateFormat> sdf = new ThreadLocal<SimpleDateFormat>(){
    @Override
    proteceted SimpleDateFormat initValue(){
        new SimpleDateFormat("yyyy-MM-dd ss");
    }
}
new Thread() -> sdf.get().parse("2019-01-23 23");
~~~

## CPU MESI协议

MODIFY 

高速寄存器

缓冲区

flush和refresh



## 内存屏障

moniter

## ThreadLocal

创建每个线程中变量的副本，每个线程访问时，都是访问副本的对象，互不干涉。实现原理，是在每个线程成维护一个Map，Key值为ThreadLocal值，value为变量的值，线程去获取变量时，用ThreadLocal去取value。由于map的key为了避免内存溢出，使用weak reference声明的，只要发生gc就会回收掉key，而value是强引用指向map中，所以有可能会发生内存泄漏，key被回收了,value没有办法被回收。

### ThreadLocal内存泄漏问题，如何防止

ThreadLocal本身不存值，但是会作为key让线程从ThreadLocalMap中获取value，Map使用了弱引用，但是弱引用只是针对key，每个key都弱引用指向threadLocal，当ThreadLocal实例置位null后，下一次gc会进行回收，但是value值不会被回收，因为存在一条从current thread连接过来的强引用，只有当前thread结束后，current thread才不会存在于栈中，强引用断开。所以ThreadLocalMap的生命周期是跟着current thread走的，如果thread Local设为Null且线程还未结束这段时间内，value值泄漏了。

为了避免这种情况，在每次用完threadlocal后，要先将value值remove掉，再将threadlocal设为null。

### 线程池中使用ThreadLocal的问题

由于线程池中，线程会