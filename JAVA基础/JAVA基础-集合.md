# Java基础-集合

![image](https://user-gold-cdn.xitu.io/2020/6/28/172fa9609cb9bf3f?imageslim)

## Collection接口

### LinkList和ArrayList

LinkList基于双向链表实现，增删改效率快，查询性能差，因为插入的时候只需要记录前后的节点的地址，所以插入快，而查询的时候需要遍历整个链表，所以性能差。

ArrayList基于数组实现，初始化的时候默认初始化一个容量为10的数组，增删改效率慢，查询效率高，因为插入的时候需要移动后面的元素。越靠近ArrayList头部的插入速度越慢，因为需要移动的元素更多。ArrayList时基于索引定位到元素的，所以无论什么位置时间复杂度都是O（1）。

实际测试中，ArrayList和LinkList尾部插入性能差不多，而在LinkList中部插入的性能则相比ArrayList相差10倍，因为LinkList插入时，需要首先找到这个位置在哪，如果index比size的一半要小，就从头遍历，如果比size的一半要大，则从尾部开始遍历，所以当插入位置为中间的时候，需要遍历的位置最多，效率越低。



### ArrayList和Vector

Vector是线程安全的，初始容量是10，速度慢，底层数据结构是数组，扩容因子为1，每次扩容为当前容量的一倍。Vector是通过在add,remove,size,get等方法加Sychroniezd关键字来保证同步的，当一个线程在使用该方法时，其他线程会堵塞。

虽然Vector的方法是线程安全的，但是当复合操作Vector中的方法时，仍会有线程不安全的风险。需要对Vector单独加锁。

ArrayList扩容为当前容量的1.5倍+1。为线程不安全。扩容时，会将老数组中的元素拷贝一份到新数组中。



### LinkedList和ArrayDequeue和Stack

三个集合都能作为栈使用。

stack继承于vector，单个方法是线程安全的。

arrayDequeue和linkedList都实现了dequeue接口，是一个双向队列，但是ArrayDequeue内部是由head,tail,object数组组成，由于是由数组实现的双向队列，对于随机访问会有较高的性能，可以通过索引直接查询对应的值，而LinkedList是由链表组成，对于删除插入性能都比较好，因为只需要记录prev和next，如果想要实现线程安全的话，可以用Collections工具类中的SychronizedXXX方法获得ArrayDequeue或LinkedList的线程安全类型。

### Set

HashSet底层基于HashMap实现，有序且不能重复，Set的add方法时基于HashMap中的Entry去判断是否包含重复，如果与原集合中原有Entry的key相同，则会把旧Key覆盖成新Key。

LinkHashSet根据元素的hashCode值决定元素的位置，但是他同时使用链表来维护元素的次序。这样就能在使用的时候，看起来是按插入的顺序保存的，遍历该集合的时候，LinkedHashSet会按元素插入的顺序去遍历。

LinkedHashSet在迭代的时候，遍历的性能比HashSet要好，但是插入的性能稍逊色与HahSet。

​                  TreeSet类

TreeSet是SortedSet的唯一实现类，TreeSet通过CompareTo方法去实现Set中元素大小的有序。



## Map

### HashMap

HashMap 1.8之前通过数组和链表实现，1.8之后通过数组和红黑树实现，HashMap会把Key对应的Hash值放入到Entry<key,value>中,然后创建一个Entry[]table的数组存放这个entry,默认长度为16，HashMap用数组去存放散列值的地址，用链表去存放地址值，结合数组寻址容易，链表插入容易的优势。1.8扩容时不重新计算哈希值，巧妙采用和扩容后容量进行与操作来计算新的索引位置。

为什么要用红黑树取代链表，因为没有办法保证hash的值在桶中可以均匀分布，当插入大量元素时，会存在hash冲突，hashmap中是通过链表去解决hash冲突的，如果大量元素都集中到一个桶内，调用get()方法时，就要去遍历该hash对应的所有链表，那遍历的时候相当于遍历了整个链表，效率低，在1.8的时候通过红黑树的数据结构取代了链表。如果桶中链表的元素个数大于8时，会从链表结构转为红黑树结构，因为红黑树的平均查找长度为Ologn，如果为8时，平均查找长度为3，如果继续使用链表，平均查找长度为4，这才有转换为红黑树的必要。若桶中链表元素个数小于等于6时，会还原成链表。

#### 为什么转化的阈值选择为6和8

1. 根据泊松分布，如果采用的是分布均匀的hashcode算法，基本不会用到红黑树的结构，当使用的是随机hashcode时，链表长度为8的概率千万分之六，基本为不可能发生的概率。
2. 长度为8时，红黑树的平均查找时间为3而链表为4，长度为6时，红黑树的平均查找时间为2.6，而链表平均查找时间为3，考虑了数据结构转换的损耗，所以选择在8的时候才进行转换。
3. 选择6跟8作为临界点是为了有效的防止链表和树的频繁转换，中间有个差值7缓冲。不然如果hashMap频繁的插入删除，链表个数在8徘徊，就有可能频繁的进行链表和树的转换，浪费性能。

####hashMap的环形链表，

为什么hashmap扩容是2的倍数，因为hashmap的根据hash获取数组下标的算法是hashcode和数组的按位与计算，二进制的每位都是1,3,7。。，所以扩容只能是2^n-1,只有这样才能实现hashcode的均匀分布。

hashmap怎么判断key是否重复，是通过equals和hashcode方法一起判断的，只有都相等时，才认为是相同元素

put过程，先通过hash寻找对应的index,然后判断是否发生hash冲突，如果冲突插入链表头部，记录nextnode为之前的链表头，如果链表长度超过8，转为红黑树结构，如果没冲突，直接插入数组。如果数组达到扩容因子，则进行rehash操作。

get过程，通过hash寻找对应的index，然后判断是否发生hash冲突，没有直接命中返回，如果有的话，通过equals比较，如果长度超过8则比较的是红黑树结构Ologn\

，如果不超过比较的是链表结构On。

####jdk1.8对hashmap的优化

1. 1.8之前hashmap由数组和链表实现，1.8后由数组链表和红黑树实现。
2. 1.8插入时，扰动函数只扰动了一次，高十六位和第十六位做异或运算，1.8之前扰动函数扰动了四次，更加消耗性能。
3. 1.8扩容后，不需要重新进行rehash，因为根据扩容都是2的倍数的特性，扩容后的位置要不在原来的位置，要不就在原索引+oldCap的地方，只需要根据hash值扩容后新增的那一位判断是0还是1，如果是0的话在原来位置，如果是1的话在原索引+oldCap
4. 发生hash冲突时，1.8之前是头插法，1.8之后是尾插法，头插法并发扩容时下容易形成循环链表，而1.8之前不采用尾插法是考虑了需要遍历到链表的末端，而1.8优化了数据结构，当链表长度为8时转化为红黑树，限制了链表的遍历长度，所以采用了尾插法，用来避免循环链表的问题。

####hashMap环形链表问题

1. 由于entry里面的next，在多线程操作下，会出现问题。
2. 当前entry的next指向b，但是rehash后，当前entry指向null，当遍历entry链表时，该next就会丢失掉。

### TreeMap和HashMap区别

TreeMap底层使用红黑树和entry实现，每个Entry都会当成是红黑树上的一个节点，性能比HashMap要低，当TreeMap添加元素时，通过循环找到新增Entry的插入位置，比较耗性能；当TreeMap取出元素时，需要遍历树才能找到合适的Entry，复杂度为Ologn。TreeMap中的所有Entry总是按照Key根据指定排序规则保存有序的状态。

可以通过TreeMap中重写compare方法，当判断元素重复时，强制返回一个非0的数，让他可以重复。

利用TreeMap中的tailMap方法实现一致性哈希，对2^32-1取模，找到treeMap节点中比他小且最近的一个节点，则为对应的node节点。

### LinkHashMap和HashMap区别

LinkHashMap额外维护了一个双向链表，按顺序记录了所有的entry，进行迭代时，是迭代双向链表而不是迭代数组中的单向链表。在调用get和put的时候，把元素丢到双向队列的尾巴。

怎么利用LinkHashMap实现LRU，因为调用get时会把元素丢到双向队列的尾巴，那么该队列的头部就是使用最少的元素，LinkHashMap还提供了removeEldestEntry方法，能够将每次添加新条目时移除最旧条目的实现，默认返回false。重写removeEldestEntry

#### hash冲突解决办法

1. 开放地址法 
2. 链地址法
3. 再哈希法，发生hash冲突后，调用别的hash函数重新计算。
4. 公共溢出区域法，建立基本表和溢出表，凡是和基本表发生冲突的都丢进溢出表。

### HashTable

HashTable是线程安全的，内部put，remove等方法都用了sychronied修饰，但是性能不好，每次同步执行时都要锁住整个结构。

### ConcurrentHashMap

ConcurrentHashMap使用的分段锁的技术来实现线程安全，当进行并发操作的时候只会将存放entry链表的segment锁住，而不会锁住整个结构，并发的粒度取决于hashMap当前数组容量的大小

对数组的引用，node里面的val和next做了volatile修饰，保证写入的可见性。

对sizeCtrl进行CAS操作，保证多线程任务分片。

Java8为了更好的性能，放弃了分段锁，改用CAS和Sycharonized控制并发操作

怎么保证size()方法的一致性。

1. 遍历所有的segment。
2. 统计所有的segment的修改次数跟元素总数。‘
3. 判断segment的修改总数跟上一次统计的对比有没有变化，如果有变化重新统计，直到超过阈值，如果没有变化，结束统计。返回元素的总数。
4. 如果统计次数超过阈值，对所有的segment加锁。重新进行统计。
5. 因为此时已经对所有的segment加锁，所以跟上一次的对比肯定不会发生变化。
6. 释放锁，结束统计。

