





![image-20210819181129006](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210819181129006.png)

# book

软件架构设计：大型网站技术架构与业务架构融合之道



# Hash Table

可以根据一个key值来直接访问数据，因此查找速度快

**实现**

1. 数组+链表

2. 数组+二叉树

key传入散列函数计算得到数据存储位置

**哈希冲突**

解决

1. 开放寻址法

   冲突找下一个, 直到找到空位置

   Java中的ThreadLocal就是利用了开放寻址法

2. 拉链法

   链表解决

3. ...

**哈希表扩容**

哈希表位置占满或哈希表被占的位置较多，哈希冲突的概率也就变高了，所以很有必要进行扩容

负载因子   被占位置与总位置的百分比

hashmap 是0.75

新创建一个数组是原来的2倍，然后把原数组的所有Entry都重新Hash一遍放到新的数组

**哈希函数是核心**

直接定址法，数字分析法，折叠法，随机数法和除留余数法等等



# java 

## 反射

## 内部类

![](https://uploadfiles.nowcoder.com/images/20190221/242025553_1550728055483_BA9669C5826A238ACEC0BD86755FA5DB)

![](https://uploadfiles.nowcoder.com/images/20180701/3807435_1530425536125_D49BCBCCF82CF58C566E12F1E3130070)









## Collections

set(集）、list(列表包含 Queue）和 map(映射)

### 1.Map

#### 1.HashMap

无序, 允许一个key为null多个值为null, 线程不安全(Collections.synchronizedMap()或ConcurrentHashMap)

==初始化的时候给一个大致的数值， 避免Map进行频繁的扩容。==

##### 原理

数组+链表/数组+红黑树

Node是HashMap的一个内部类， 实现了 Map.Entry接口， 本质是就是一个映射（键值对）

```java
int threshold; //所能容纳的key-value对极限  threshold = length * loadFactor
final float loadFactor; //负载因子
int modCount; //hashmap内部结构变化次数  主要用于迭代的快速失败 例如put新kv 覆盖旧value不算
int size; //实际存在键值对数量
```

Node[]table的初始长度是16, 

Load Factor为负载因子默认0.75 (如果内存多且对时间要求高,可降低,内存紧张时间要求不高可调大, 可以大于一)

threshold超过就扩容为原本两倍

哈希桶数组长度必须为2的n次方(一定是合数)

1.8链表大于8转为红黑数,利用其快速增删改查提高性能

##### 功能实现-方法

###### 1.根据key获取哈希桶数组索引位置

```java
//方法一：
static final int hash(Object key) {//jdk1.8 & jdk1.7
    int h;
    //h = key.hashCode()为第一步 取hashCode值
    //h ^ (h >>> 16)为第二步  高位参与运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
//方法二：
static int indexFor(int h, int length) { //jdk1. 7的源码jdk1.8没有这个方法，但是实现原理一样的
    return h & (length-1); //第三步取模运算
}
```

取key的HashCode值、高位运算、取模运算。

通过h & (length-1)来得到该对象的保存位， 而HashMap底层数组的长度总是2的n次方，这是HashMap 在速度上的优化。当length为2的n次方, h & (length-1)等价于对length取模, 即 h%length ,但是&比%效率高

###### 2.put

![image-20210730092815945](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210730092815945.png)

###### 3.扩容机制

```java
void resize(int newCapacity) { //传入新的容噩
    Entry[] oldTable = table; //引用扩容前的Entry数组
    int oldCapacity = oldTable. length; 
    if (oldCapacity == MAXIMUM_CAPACITY) { //扩容前的数组大小如果已经达到最大2^30了
        threshold = Integer.MAX_VALUE; / /修改阈值为int的最大值(2^31-1),这样以后就不会扩容了 
            return;
    }    
    Entry[] newTable = new Entry[newCapacity];//初始化一个新的Entry数组
    transfer(newTable);//把数据转移到新数组里
    table = newTable;//HashMap的table属性引用新的Entry数组
    threshold= (int)(newCapacity * loadFactor);//修改阈值
} 
```

```java
void transfer(Entry[] newTable) { 
    Entry[] src = table; //src引用了旧的Entry数组
    int newCapacity = new Table.length; 
    for (int j = 0; j < src. length; j++) { //遍历旧的Entry数组
        Entry<K,V> e = src[j]; //取得旧Entry数组的每个元素
        if (e != null) { 
            src[j] = null;//释放旧Entry数组的对象引用(for循环后，旧的Entry数组不再引用任何对象）
            do { 
                Entry<K,V> next= e.next;
                int i= indexFor(e.hash, newCapacity); //'! 重新计算每个元素在数组中的位置
                e.next = newTable[i]; //标记[1]
                newTable[i] = e; //将元素放在数组上
                e = next;//访问下一个Entry链上的元素
            }while (e!=null);
        }
    }
}
```

newTable[i]的引用赋给了e.next, 也就是使用了单链表的头插入方式， 同一位置上新元索总会袚放在链表的头部位置这样先放在一个索引上的元索终会袚放到Entry链的尾部（如果发生了Hash冲突的话） ， 这一点和Jdk1.8有区别。
在旧数组中同一条Entry链上的元索， 通过重新计算索引位置后， 有可能袚放到了新数组的不同位置上。

###### 3.线程安全性

多线程可能死循环

#### 2.LinkedHashMap

有序, 保存插入顺序

#### 3.TreeMap

保存记录默认根据key升序排序, 也可以指定排序比较器

用排序推荐treemap, key必须实现comparable或在构造treemap传入自定义的comparator

#### 4.HashTable

线程安全, 不推荐用

kv不难为null

#### 5.ConcurrentHashMap

##### JDK1.7实现

ReentrantLock+Segment+HashEntry

数组+Segment+分段锁的方式实现

###### 1.Segment(分段锁)

ConcurrentHashMap 中的分段锁称为 Segment，它即类似于 HashMap 的结构，即内部拥有一个 Entry 数组，数组中的每个元素又是一个链表,同时又是一个 ReentrantLock（Segment 继承了 ReentrantLock）。

concurrencyLevel：并行级别、并发数、Segment 数，默认是 16，可以初始化时设置, 初始化后不可扩容

###### 2.内部结构

ConcurrentHashMap 使用分段锁技术，将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问，能够实现真正的并发访问

![image-20210730110304597](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210730110304597.png)

ConcurrentHashMap 定位一个元素的过程需要进行两次 Hash 操作。
第一次 Hash 定位到 Segment，第二次 Hash 定位到元素所在的链表的头部

###### 3.该结构的优劣势

* 劣势

  Hash 的过程要比普通的 HashMap 要长

* 好处

  写操作的时候可以只对元素所在的 Segment 进行加锁即可，不会影响到其他的Segment，这样，在最理想的情况下，ConcurrentHashMap 可以最高同时支持 Segment 数量大小的写操作（刚好这些写操作都非常平均地分布在所有的Segment 上）。
  所以，通过这一种结构，ConcurrentHashMap 的并发能力可以大大的提高。

###### 4.构造函数分析

```java
//通过指定的容量，加载因子和并发等级创建一个新的ConcurrentHashMap
public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    //对容量，加载因子和并发等级做限制
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    //限制并发等级不可以大于最大等级
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    // 下面即通过并发等级来确定Segment的大小
    //sshift用来记录向左按位移动的次数
    int sshift = 0;
    //ssize用来记录Segment数组的大小
    int ssize = 1;
    //Segment的大小为大于等于concurrencyLevel的第一个2的n次方的数
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }

    this.segmentShift = 32 - sshift;
    //segmentMask的值等于ssize - 1（这个值很重要）
    this.segmentMask = ssize - 1;
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    //c记录每个Segment上要放置多少个元素  初始大小
    int c = initialCapacity / ssize;
    //假如有余数，则Segment数量加1
    if (c * ssize < initialCapacity)
        ++c;
    int cap = MIN_SEGMENT_TABLE_CAPACITY;
    while (cap < c)//每个segment也是2的n次方
        cap <<= 1;

    //创建第一个Segment，并放入Segment[]数组中，作为第一个Segment
    Segment<K,V> s0 =
        new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                         (HashEntry<K,V>[])new HashEntry[cap]);
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
    this.segments = ss;
}
```

构造函数的第3个参数concurrenyLevel，是“并发度”，也就是Segment数组的大小。这个值一旦在构造函数中设定，之后不能再扩容。为了提升hash的计算性能，会保证数组的大小始终是2的整数次方。例如设置concurrentyLevel=9，在构造函数里面会找到比9大且距9最近的2的整数次方，也就是ssize=16。对应segmentShift、segmentMask两个变量，是为了方便计算hash使用的。
初始的时候，如果不指定任何参数，就会使用默认值，代码如下所示。可以看到，默认的Segment数组大小是16。

```java
    //初始的容量
    static final int DEFAULT_INITIAL_CAPACITY = 16;
    //初始的加载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    //初始的并发等级（下面会叙述作用）
    static final int DEFAULT_CONCURRENCY_LEVEL = 16;
    //最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
    //最小的segment数量
    static final int MIN_SEGMENT_TABLE_CAPACITY = 2;
    //最大的segment数量
    static final int MAX_SEGMENTS = 1 << 16; 
    //
    static final int RETRIES_BEFORE_LOCK = 2;
```

第1个参数，initialCapacity是整个ConcurrentHashMap的初始大小。用initialCapacity除以ssize，是每个Segment的初始大小。这里也会保证Segment里面HashEntry[]数组的大小是2的整数次方。
第2个参数，loadFactor即负载因子，传给了Segment内部。当每个Segment的元素个数达到一定阈值，进行rehash。Segment的个数不能扩容，但每个Segment的内部可以扩容。

###### 5.put（..）函数分析

```java
    public V put(K key, V value) {
        Segment<K,V> s;
        //ConcurrentHashMap的key和value都不能为null
        if (value == null)
            throw new NullPointerException();

        //这里对key求hash值，并确定应该放到segment数组的索引位置
        int hash = hash(key);//key映射到32位整数
        //把该整数映射到第j个Segement
        int j = (hash >>> segmentShift) & segmentMask;
        if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
             (segments, (j << SSHIFT) + SBASE)) == null) //Segment为空就初始化
            s = ensureSegment(j);
        //这里很关键，找到了对应的Segment，则把元素放到Segment中去
        return s.put(key, hash, value, false);
    }
```

由于Segment的个数是2的整数次方，假设其为16，则segmentShift=32-ssshift=32-4=28，segmentMask=ssize-1=16-1=15（参见上面的构造函数）。
hash值是一个32位的整数，int j=（hash＞＞＞segmentShift）&segmentMask 这行代码就是把hash值右移28位，再和15进行与操作，表达的意思是：以hash值的最高4位作为对应的Segment数组下标j。

在上面的代码中没有加锁操作，因为锁是加在s.put内部的，也就是分段加锁。另外，多个线程可能同时调用ensureSegment对Segment[j]进行初始化，在这个函数的内部要避免重复初始化，下面详细分析。

```java
private Segment<K,V> ensureSegment(int k) {
    final Segment<K,V>[] ss = this.segments;
    long u = (k << SSHIFT) + SBASE; // 下标k对应内存地址偏移量为u
    Segment<K,V> seg;
    if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
        // 这里看到为什么之前要初始化 segment[0] 了，
        // 使用当前 segment[0] 处的数组长度和负载因子来初始化 segment[k]
        // 为什么要用“当前”，因为 segment[0] 可能早就扩容过了
        Segment<K,V> proto = ss[0];
        int cap = proto.table.length;
        float lf = proto.loadFactor;
        int threshold = (int)(cap * lf);

        // 初始化 segment[k] 内部的数组
        HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];
        if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
            == null) { // 再次检查一遍该槽是否被其他线程初始化了。

            Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
            // 使用 while 循环，内部用 CAS，当前线程成功设值或其他线程成功设值后，退出,如果其他线程成功设置后，这里获取到直接返回
            while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
                   == null) {
                if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
                    break;
            }
        }
    }
    return seg;
}
```

上面这个函数的目的是当segments[k]=null时对其进行初始化。由于多个线程可能同时调用该函数，UNSAFE.getObjectVolatile（ss，u）== null 出现了3次，在3个不同的时间点重复检查segments[k]==null。但即使如此，也不能完全保证避免重复初始化，所以最后需要执行一个CAS操作UNSAFE.compareAndSwapObject（ss，u，null，seg=s）保证只被初始化一次。保险起见，把这次CAS操作放在一个while循环里，保证出来的时候segments[k]一定不为空。segments[j]被成功地初始化了，下面进入内部，了解如何把元素放进去。

```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
            //这里是并发的关键，每一个Segment进行put时，都会加锁
            HashEntry<K,V> node = tryLock() ? null :
                scanAndLockForPut(key, hash, value);
            V oldValue;
            try {
                //tab是当前segment所连接的HashEntry数组
                HashEntry<K,V>[] tab = table;
                //确定key的hash值所在HashEntry数组的索引位置
                int index = (tab.length - 1) & hash;
                //取得要放入的HashEntry链的链头
                HashEntry<K,V> first = entryAt(tab, index);
                //遍历当前HashEntry链
                for (HashEntry<K,V> e = first;;) {
                    //如果链头不为null
                    if (e != null) {
                        K k;
                        //如果在该链中找到相同的key，则用新值替换旧值，并退出循环
                        if ((k = e.key) == key ||
                            (e.hash == hash && key.equals(k))) {
                            oldValue = e.value;
                            if (!onlyIfAbsent) {
                                e.value = value;
                                ++modCount;
                            }
                            break;
                        }
                        //如果没有和key相同的，一直遍历到链尾，链尾的next为null，进入到else
                        e = e.next;
                    }
                    else {//如果没有找到key相同的，则把当前Entry插入到链头

                        if (node != null)
                            node.setNext(first);
                        else
                            node = new HashEntry<K,V>(hash, key, value, first);
                        //此时数量+1
                        int c = count + 1;
                        if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                            //如果超出了限制，要进行扩容
                            rehash(node);
                        else
                            setEntryAt(tab, index, node);
                        ++modCount;
                        count = c;
                        oldValue = null;
                        break;
                    }
                }
            } finally {
                //最后释放锁
                unlock();
            }
            return oldValue;
        }
```

关于上面的代码，有几点说明：

（1）在Segment里面有2个count变量，count与modCount，前者表示元素的个数，后者表示修改的次数。当待put的元素、key值或者hash值和链表中某个节点相等时，不会重复插入新节点。此时再进一步判断，如果onlyIfAbsent=true，则什么都不做；如果onlyIfAbsent=false，则修改该节点的value，同时modCount累加一次。

（2）进入上面的else分支，说明已经遍历到了链表尾部，并且没有发现与key值或者hash值相等的节点，此时在链表头部插入1个新节点，并把table[index]赋值为该节点。因为first就是链表头部，所以直接把node的next指针指向first就可以了。
（3）在函数的开始，加锁的时候，进行了一次优化。如果tryLock（）成功，即拿到锁，则进入下面的代码；如果tryL
ock（）不成功，也不能闲着，那进入scanAndLockForPut（key，hash，value）做什么呢？

```java
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null;
    int retries = -1; // negative while locating node

    // 循环获取锁
    while (!tryLock()) {
        HashEntry<K,V> f; // to recheck first below
        if (retries < 0) {
            if (e == null) {
                if (node == null) // speculatively create node 创建一个新节点
                    // 进到这里说明数组该位置的链表是空的，没有任何元素
                    // 当然，进到这里的另一个原因是 tryLock() 失败，所以该槽存在并发，不一定是该位置
                    node = new HashEntry<K,V>(hash, key, value, null);
                retries = 0;
            }
            else if (key.equals(e.key))
                retries = 0;
            else
                // 顺着链表往下走
                e = e.next;
        }
        // 重试次数如果超过 MAX_SCAN_RETRIES（单核1多核64），那么不抢了，进入到阻塞队列等待锁
        //    lock() 是阻塞方法，直到获取锁后返回
        else if (++retries > MAX_SCAN_RETRIES) {
            lock();
            break;
        }
        else if ((retries & 1) == 0 &&
                 // 这个时候是有大问题了，那就是有新的元素进到了链表，成为了新的表头
                 //     所以这边的策略是，相当于重新走一遍这个 scanAndLockForPut 方法
                 (f = entryForHash(this, hash)) != first) {
            e = first = f; // re-traverse if entry changed
            retries = -1;
        }
    }
    return node;
}
```

上面的函数看似复杂，实则只是做了两件事：一是拿不到锁，不立即阻塞，而是先自旋，若自旋到一定次数仍未拿到锁，再调用lock（）阻塞；二是在自旋的过程中遍历了链表，若发现没有重复的节点，则提前新建一个节点，为后面再插入节省时间。

我们来重新理一理思路：
　　1. 首先对key进行第1次hash，通过hash值确定segment的位置
　　2. 然后在segment内进行操作，获取锁
　　3. 接着获取当前segment的HashEntry数组，然后对key进行第2次hash，通过hash值确定在HashEntry数组的索引位置
　　4. 然后对当前索引的HashEntry链进行遍历，如果有重复的key，则替换；如果没有重复的，则插入到链头
　　5. 关闭锁

**整个put过程中，进行了2次hash操作，才最终确定key的位置**

###### 6.扩容

在上面的代码中提到了，超过一定的阈值后，Segment内部会进行扩容，代码如下。

```java
// 方法参数上的 node 是这次扩容后，需要添加到新的数组中的数据。
private void rehash(HashEntry<K,V> node) {
    HashEntry<K,V>[] oldTable = table;
    int oldCapacity = oldTable.length;
    // 2 倍
    int newCapacity = oldCapacity << 1;
    threshold = (int)(newCapacity * loadFactor);
    // 创建新数组
    HashEntry<K,V>[] newTable =
        (HashEntry<K,V>[]) new HashEntry[newCapacity];
    // 新的掩码，如从 16 扩容到 32，那么 sizeMask 为 31，对应二进制 ‘000...00011111’
    int sizeMask = newCapacity - 1;

    // 遍历原数组，老套路，将原数组位置 i 处的链表拆分到 新数组位置 i 和 i+oldCap 两个位置
    for (int i = 0; i < oldCapacity ; i++) {
        // e 是链表的第一个元素
        HashEntry<K,V> e = oldTable[i];
        if (e != null) {
            HashEntry<K,V> next = e.next;
            // 计算应该放置在新数组中的位置，
            // 假设原数组长度为 16，e 在 oldTable[3] 处，那么 idx 只可能是 3 或者是 3 + 16 = 19
            int idx = e.hash & sizeMask;
            if (next == null)   // 该位置处只有一个元素，那比较好办
                newTable[idx] = e;
            else { // Reuse consecutive sequence at same slot
                // e 是链表表头
                HashEntry<K,V> lastRun = e;
                // idx 是当前链表的头结点 e 的新位置
                int lastIdx = idx;

                // 下面这个 for 循环会找到一个 lastRun 节点，这个节点之后的所有元素是将要放到一起的
                for (HashEntry<K,V> last = next;
                     last != null;
                     last = last.next) {
                    int k = last.hash & sizeMask;
                    if (k != lastIdx) {
                        lastIdx = k;//寻找链表中最后一个hash值不等于lastindx的元素
                        lastRun = last;
                    }
                }
                // 将 lastRun 及其之后的所有节点组成的这个链表放到 lastIdx 这个位置
                newTable[lastIdx] = lastRun;
                // 下面的操作是处理 lastRun 之前的节点，
                //    这些节点可能分配在另一个链表中，也可能分配到上面的那个链表中
                for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                    V v = p.value;
                    int h = p.hash;
                    int k = h & sizeMask;
                    HashEntry<K,V> n = newTable[k];
                    newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                }
            }
        }
    }
    // 将新来的 node 放到新数组中刚刚的 两个链表之一 的 头部
    int nodeIndex = node.hash & sizeMask; // add the new node
    node.setNext(newTable[nodeIndex]);
    newTable[nodeIndex] = node;
    table = newTable;
}
```

关于该扩容函数，有几点需要说明：
（1）函数的参数，也就是将要加入的最新节点。在扩容完成之后，把该节点加入新的Hash表。
（2）整个数组的长度是2的整数次方，每次按二倍扩容，而hash函数就是对数组长度取模，即node.hash&sizeMask。因此，如果元素之前处于第i个位置，当再次hash时，必然处于第i个或者第i+oldCapacity个位置。
（3）上面的扩容进行了一次优化，并没有对元素依次拷贝，而是先找到lastRun位置，也就是for循环。lastRun到链表末尾的所有元素，其hash值没有改变，所以不需要依次重新拷贝，只需把这部分链表链接到新链表所对应的位置就可以，也就是 newTable[lastIdx]=lastRun。lastRun之前的元素则需要依次拷贝。因此，即使把一段for循环去掉，整个程序的逻辑仍然是正确的。

###### 7.get实现分析

```java
public V get(Object key) {
    Segment<K,V> s; // manually integrate access methods to reduce overhead
    HashEntry<K,V>[] tab;
    // 1. hash 值
    int h = hash(key);
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
    // 2. 根据 hash 找到对应的 segment 第一次hash
    if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
        (tab = s.table) != null) {
        // 3. 找到segment 内部数组相应位置的链表，遍历 第二次hash
        for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
             (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
             e != null; e = e.next) {
            K k;
            if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                return e.value;
        }
    }
    return null;
}
```

整个get过程也就是两次hash：
第一次hash，函数为（h＞＞＞segmentShift）&segmentMask，计算出所在的Segment；
第二次hash，函数为h&（tab.length-1），即h对数组长度取模，找到Segment里面对应的HashEntry数组下标，然后遍历该位置的链表。

整个读的过程完全没有加锁，而是使用了UNSAFE.getObjectVolatile函数。

##### JDK1.8实现

synchronized+CAS+HashEntry+红黑树

数组+ 链表+红黑树的实现方式来设计，内部大量采用 CAS 操作

JDK8 中彻底放弃了 Segment 转而采用的是 Node，其设计思想也不再是JDK1.7 中的分段锁思想。

Node：保存key，value及key的hash值的数据结构。其中value和next都用volatile修饰，保证并发的可见性。

```java
 static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;
```

###### 1.对比

1. 数据结构：取消了 Segment 分段锁的数据结构，取而代之的是数组+链表+ 红黑树的结构。

2. 保证线程安全机制：JDK1.7 采用 segment 的分段锁机制实现线程安全，其中segment 继承自 ReentrantLock。JDK1.8 采用 CAS+Synchronized 保证线程安全。

3. 锁的粒度：原来是对需要进行数据操作的 Segment 加锁，现调整为对每个数组元素加锁（Node）

4. 链表转化为红黑树:定位结点的 hash 算法简化会带来弊端,Hash 冲突加剧,因此在链表节点数量大于 8 时，会将链表转化为红黑树进行存储。

5. 查询时间复杂度：从原来的遍历链表 O(n)，变成遍历红黑树 O(logN)。

   

在JDK 7中的分段锁，有三个好处：
（1）减少Hash冲突，避免一个槽里有太多元素。
（2）提高读和写的并发度。段与段之间相互独立。
（3）提供扩容的并发度。扩容的时候，不是整个 ConcurrentHashMap 一起扩容，而是每个Segment独立扩容。

针对这三个好处，我们来看一下在JDK 8中相应的处理方式：
（1）使用红黑树，当一个槽里有很多元素时，其查询和更新速度会比链表快很多，Hash冲突的问题由此得到较好的解决。
（2）加锁的粒度，并非整个ConcurrentHashMap，而是对每个头节点分别加锁，即并发度，就是Node数组的长度，初始长度为16，和在JDK 7中初始Segment的个数相同。
（3）并发扩容，这是难度最大的。在JDK 7中，一旦Segment的个数在初始化的时候确立，不能再更改，并发度被固定。之后只是在每个Segment内部扩容，这意味着每个Segment独立扩容，互不影响，不存在并发扩容的问题。但在JDK 8中，相当于只有1个Segment，当一个线程要扩容Node数组的时候，其他线程还要读写，因此处理过程很复杂，后面会详细分析。

由上述对比可以总结出来：JDK 8的实现一方面降低了Hash冲突，另一方面也提升了并发度。

###### 2.构造函数分析

```java
public ConcurrentHashMap(int initialCapacity) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
               MAXIMUM_CAPACITY :
               tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
    this.sizeCtl = cap;
}
```

在上面的代码中，变量cap就是Node数组的长度，保持为2的整数次方。tableSizeFor（..）函数是根据传入的初始容量，计算出一个合适的数组长度。具体而言：1.5倍的初始容量+1，再往上取最接近的2的整数次方，作为数组长度cap的初始值。这里的 sizeCtl，其含义是用于控制在初始化或者并发扩容时候的线程数，只不过其初始值设置成cap。

###### 3.初始化

在上面的构造函数里只计算了数组的初始大小，并没有对数组进行初始化。当多个线程都往里面放入元素的时候，再进行初始化。这就存在一个问题：多个线程重复初始化。下面看一下是如何处理的。

```java
private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin 自旋等待
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {//关键:把sizectl设为-1
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];//初始化
                        table = tab = nt;
                        sc = n - (n >>> 2);//sizectl并非表示数组长度,init成功就不等于数组长度,而是下一次扩容阈值
                    }
                } finally {
                    sizeCtl = sc;//把sizectl设回去
                }
                break;
            }
        }
        return tab;
    }
```

通过上面的代码可以看到，多个线程的竞争是通过对sizeCtl进行
CAS操作实现的。如果某个线程成功地把 sizeCtl 设置为-1，它就拥
有了初始化的权利，进入初始化的代码模块，等到初始化完成，再把s
izeCtl设置回去；其他线程则一直执行while循环，自旋等待，直到数
组不为null，即当初始化结束时，退出整个函数。
因为初始化的工作量很小，所以此处选择的策略是让其他线程一
直等待，而没有帮助其初始化。

###### 4.put（..）实现分析

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();//分支1:整个数组初始化
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {//分支2:第i个元素初始化
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);//分支3:扩容
        else {//分支4:放入元素
            V oldVal = null;
            synchronized (f) {//关键:加锁
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {//链表
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {//红黑树
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                              value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {//如果是链表,则上面bitCount会从1累加
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);//超出阈值转为红黑树
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);//总元素累加1
    return null;
}
```

上面的for循环有4个大的分支：
第1个分支，是整个数组的初始化，前面已讲；
第2个分支，是所在的槽为空，说明该元素是该槽的第一个元素，直接新建一个头节点，然后返回；
第3个分支，说明该槽正在进行扩容，帮助其扩容；
第4个分支，就是把元素放入槽内。槽内可能是一个链表，也可能是一棵红黑树，通过头节点的类型可以判断是哪一种。第4个分支是包裹在synchronized （f）里面的，f对应的数组下标位置的头节点，意味着每个数组元素有一把锁，并发度等于数组的长度。

上面的binCount表示链表的元素个数，当这个数目超过TREEIFY_THRESHOLD=8时，把链表转换成红黑树，也就是 treeifyBin（tab，i）函数。但在这个函数内部，不一定需要进行红黑树转换，可能只做扩容操作，所以接下来从扩容讲起。

###### 5.扩容

扩容的实现是最复杂的，下面从treeifyBin（tab，i）讲起。

```java
private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n, sc;
    if (tab != null) {
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            tryPresize(n << 1);//数组长度小于阈值64,不做红黑树转换,直接扩容
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            synchronized (b) {//链表转红黑树
                if (tabAt(tab, index) == b) {
                    TreeNode<K,V> hd = null, tl = null;
                    for (Node<K,V> e = b; e != null; e = e.next) {
                        TreeNode<K,V> p =//遍历链表构建红黑树
                            new TreeNode<K,V>(e.hash, e.key, e.val,
                                              null, null);
                        if ((p.prev = tl) == null)
                            hd = p;
                        else
                            tl.next = p;
                        tl = p;
                    }
                    setTabAt(tab, index, new TreeBin<K,V>(hd));
                }
            }
        }
    }
}
```

在上面的代码中，MIN_TREEIFY_CAPACITY=64，意味着当数组的长度没有超过64的时候，数组的每个节点里都是链表，只会扩容，不会转换成红黑树。只有当数组长度大于或等于64时，才考虑把链表转换成红黑树。

在 tryPresize（int size）内部调用了一个核心函数 transfer（Node＜K，V＞[] tab，Node＜K，V＞[] nextTab），先从这个函数的分析说起。

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // 计算步长
        if (nextTab == null) {            // init 新hashmap
            try {
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];//扩容2倍
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n;//初始的transferIndex为旧的hashmap长度
        }
        int nextn = nextTab.length;
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        boolean advance = true;
        boolean finishing = false; 
        //i为遍历下表,bound为遍历边界,成功拿到i=nextIndex-1
        //bound=nextIndex-stride 如果拿不到任务,则i=0,bound=0
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            //advance表示从i=transFerIndex-1遍历到bound位置的过程中,是否一直继续
            //三个分支都是advance=false,三分支都不执行,才能一直循环,目的在于,对transferIndex执行cas操作不成功,进行自旋,以拿到一个stride的迁移任务
            while (advance) {
                int nextIndex, nextBound;
                if (--i >= bound || finishing)//对数组遍历,通过--i进行,执行了--i就不用继续循环,每次advance只能步进一步
                    advance = false;
                else if ((nextIndex = transferIndex) <= 0) {//transferIndex<=0,整个hashmap完成
                    i = -1;
                    advance = false;
                }
                else if (U.compareAndSwapInt//对transferIndex进行cas操作,也就是为当前线程分配一个stride,cas成功,线程拿到一个stride迁移任务,cas操作不成功会自旋
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            //i越界,整个hashmap遍历完成
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                if (finishing) {//表示扩容完成
                    nextTable = null;
                    table = nextTab;//把nextTab赋值给当前table
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            else if ((f = tabAt(tab, i)) == null)//tab[i]迁移完毕,赋值一个ForwardingNode
                advance = casTabAt(tab, i, null, fwd);
            else if ((fh = f.hash) == MOVED)//tab[i]在迁移过程中
                advance = true; // already processed
            else {//tab[i]进行迁移操作,可以是链表或红黑树
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        if (fh >= 0) {//链表
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;//lastrun之后元素,hash值一样,记录这个位置
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;//类似jdk7链表迁移优化
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                        else if (f instanceof TreeBin) {//红黑树迁移,和链表类似
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }
```

该函数非常复杂，下面一步步分析：
（1）扩容的基本原理如图5-11所示，首先建一个新的HashMap，其数组长度是旧数组长度的2倍，然后把旧的元素逐个迁移过来。所以，上面的函数参数有2个，第1个参数tab是扩容之前的HashMap，第2个参数nextTab是扩容之后的HashMap。当nextTab=null的时候，函数最初会对nextTab进行初始化。这里有一个关键点要说明：该函数会被多个线程调用，所以每个线程只是扩容旧的HashMap部分，这就涉及如何划分任务的问题。

![image-20210811142130841](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210811142130841.png)

（2）图5-12所示为多个线程并行扩容-任务划分示意图。旧数组的长度是 N，每个线程扩容一段，一段的长度用变量stride（步长）来表示，transferIndex表示了整个数组扩容的进度。stride的计算公式如上面的代码所示，即：在单核模式下直接等于 n，因为在单核模式下没有办法多个线程并行扩容，只需要1个线程来扩容整个数组；

在多核模式下为 （n＞＞＞3）/NCPU，并且保证步长的最小值是 16。显然，需要的线程个数约为 n/stride。

```java
if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
    stride = MIN_TRANSFER_STRIDE; // subdivide range
```

transferIndex是ConcurrentHashMap的一个成员变量，记录了扩容的进度。初始值为n，从大到小扩容，每次减stride个位置，最终减至n＜=0，表示整个扩容完成。因此，从[0，transferIndex-1]的位置表示还没有分配到线程扩容的部分，从[transfexIndex，n-1]的位置表示已经分配给某个线程进行扩容，当前正在扩容中，或者已经扩容成功。

因为transferIndex会被多个线程并发修改，每次减stride，所以需要通过CAS进行操作，如下面的代码所示。

```java
else if (U.compareAndSwapInt
         (this, TRANSFERINDEX, nextIndex,
          nextBound = (nextIndex > stride ?
                       nextIndex - stride : 0))) {
```

![image-20210811142604057](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210811142604057.png)

（3）在扩容未完成之前，有的数组下标对应的槽已经迁移到了新的HashMap里面，有的还在旧的 HashMap 里面。这个时候，所有调用get（k，v）的线程还是会访问旧 HashMap，怎么处理呢？图5-13所示为扩容过程中的转发示意图：当Node[0]已经迁移成功，而其他Node还在迁移过程中时，如果有线程要读取Node[0]的数据，就会访问失败。
为此，新建一个ForwardingNode，即转发节点，在这个节点里面记录的是新的 ConcurrentHashMap 的引用。这样，当线程访问到ForwardingNode之后，会去查询新的ConcurrentHashMap。

（4）因为数组的长度 tab.length 是2的整数次方，每次扩容又是2倍。而 Hash 函数是hashCode%tab.length，等价于hashCode&（tab.length-1）。这意味着：处于第i个位置的元素，在新的Hash表的数组中一定处于第i个或者第i+n个位置，如图5-11所示。举个简单的例子：假设数组长度是8，扩容之后是16：

若hashCode=5，5%8=0，扩容后，5%16=0，位置保持不变；
若hashCode=24，24%8=0，扩容后，24%16=8，后移8个位置；
若hashCode=25，25%8=1，扩容后，25%16=9，后移8个位置；
若hashCode=39，39%8=7，扩容后，39%8=7，位置保持不变；

![image-20210811143357891](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210811143357891.png)

正因为有这样的规律，所以如下有代码：

```java
setTabAt(nextTab, i, ln);
setTabAt(nextTab, i + n, hn);
setTabAt(tab, i, fwd);
```

也就是把tab[i]位置的链表或红黑树重新组装成两部分，一部分链接到nextTab[i]的位置，一部分链接到nextTab[i+n]的位置，如图5-11所示。然后把tab[i]的位置指向一个ForwardingNode节点。同时，当tab[i]后面是链表时，使用类似于JDK 7中在扩容时的优化方法，从lastRun往后的所有节点，不需依次拷贝，而是直接链接到新的链表头部。从lastRun往前的所有节点，需要依次拷贝。

了解了核心的迁移函数transfer（tab，nextTab），再回头看tryPresize（int size）函数。这个函数的输入是整个Hash表的元素个数，在函数里面，根据需要对整个Hash表进行扩容。想要看明白这个函数，需要透彻地理解sizeCtl变量，下面这段注释摘自源码。

```java
/**
     * Table initialization and resizing control.  When negative, the
     * table is being initialized or resized: -1 for initialization,
     * else -(1 + the number of active resizing threads).  Otherwise,
     * when table is null, holds the initial table size to use upon
     * creation, or 0 for default. After initialization, holds the
     * next element count value upon which to resize the table.
     */
private transient volatile int sizeCtl;
```

当sizeCtl=-1时，表示整个HashMap正在初始化；
当sizeCtl=某个其他负数时，表示多个线程在对HashMap做并发扩容；
当sizeCtl=cap时，tab=null，表示未初始之前的初始容量（如上面的构造函数所示）；
扩容成功之后，sizeCtl存储的是下一次要扩容的阈值，即上面初始化代码中的n-（n＞＞＞2）=0.75n。

所以，sizeCtl变量在Hash表处于不同状态时，表达不同的含义。明白了这个道理，再来看上面的tryPresize（int size）函数。

```java
private final void tryPresize(int size) {
    int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
        tableSizeFor(size + (size >>> 1) + 1);//根据元素个数计算数组大小
    int sc;
    while ((sc = sizeCtl) >= 0) {
        Node<K,V>[] tab = table; int n;
        if (tab == null || (n = tab.length) == 0) {//hash表初始化,和上面初始化时一样
            n = (sc > c) ? sc : c;
            if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if (table == tab) {
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = nt;
                        sc = n - (n >>> 2);//即0.75n,下次扩容阈值
                    }
                } finally {
                    sizeCtl = sc;
                }
            }
        }
        else if (c <= sc || n >= MAXIMUM_CAPACITY)
            break;
        else if (tab == table) {
            int rs = resizeStamp(n);
            if (sc < 0) {//多线程并发扩容
                Node<K,V>[] nt;
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)//扩容结束
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);//帮着扩容
            }
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);//第一次扩容
        }
    }
}
```

tryPresize（int size）是根据期望的元素个数对整个Hash表进行扩容，核心是调用transfer函数。在第一次扩容的时候，sizeCtl会被设置成一个很大的负数U.compareAndSwapInt（this，SIZECTL，sc，（rs ＜＜ RESIZE_STAMP_SHIFT）+2）；之后每一个线程扩容的时候，sizeCtl 就加 1，U.compareAndSwapInt（this，SIZECTL，sc，sc+1），待扩容完成之后，sizeCtl减1。

























## JUC





**如何减少上下文切换**

减少上下文切换的方法有无锁并发编程、CAS算法、使用最少线程和使用协程。 

1. 无锁并发编程。多线程竞争锁时，会引起上下文切换，所以多线程处理数据时，可以用一 些办法来避免使用锁，如将数据的ID按照Hash算法取模分段，不同的线程处理不同段的数据。
2. CAS算法。Java的Atomic包使用CAS算法来更新数据，而不需要加锁。
3. 使用最少线程。避免创建不需要的线程，比如任务很少，但是创建了很多线程来处理，这 样会造成大量线程都处于等待状态。
4. 协程：在单线程里实现多任务的调度，并在单线程里维持多个任务间的切换。



**避免死锁的几个常见方法。**

* 避免一个线程同时获取多个锁。
* 避免一个线程在锁内同时占用多个资源，尽量保证每个锁只占用一个资源。
* 尝试使用定时锁，使用lock.tryLock（timeout）来替代使用内部锁机制。
* 对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败的情况。



**并发编程速度影响**

* CPU上下文切换

* 资源限制

  ​	硬件资源限制(带宽/硬盘/CPU) --解决 集群

  ​	软件资源限制有数据库的连接数和socket连接数等  -- 解决 资源池





### 1.juc基础

进程：指在系统中正在运行的一个应用程序；程序一旦运行就是进程；进程——资源分配的最小单位
线程：系统分配处理器时间资源的基本单元，或者说进程之内独立执行的一个单元执行流。线程——程序执行的最小单位

#### 1.0 Thread

##### 线程状态

```java
public enum State {
    BLOCKED,
    NEW,
    RUNNABLE,
    TERMINATED,
    TIMED_WAITING,
    WAITING
```

wait/sleep的区别
（1）sleep是Thread的静态方法，wait是Object的方法，任何对象实例都能调用。
（2）sleep不会释放锁，它也不需要占用锁。wait会释放锁，但调用它的前提是当前线程占有锁(即代码要在synchronized中)。
（3）它们都可以被interrupted方法中断。

##### 管程

管程(monitor)是保证了同一时刻只有一个进程在管程内活动,即管程内定义的操作在同一时刻只被一个进程调用(由编译器实现).但是这样并不能保证进程以设计的顺序执行
JVM中同步是基于进入和退出管程(monitor)对象实现的，每个对象都会有一个管程(monitor)对象，管程(monitor)会随着java对象一同创建和销毁
执行线程首先要持有管程对象，然后才能执行方法，当方法完成之后会释放管程，方法在执行时候会持有管程，其他线程无法再获取同一个管程

#### 1.1 线程的优雅关闭

##### 1.1.1 stop()与destory()函数

线程是“一段运行中的代码”，或者说是一个运行中的函数。既然是在运行中，就存在一个最基本的问题：运行到一半的线程能否强制杀死？
答案肯定是不能。在Java中，有stop（）、destory（）之类的函数，但这些函数都是官方明确不建议使用的。原因很简单，如果强制杀死线程，则线程中所使用的资源，例如文件描述符、网络连接等不能正常关闭。
因此，一个线程一旦运行起来，就不要去强行打断它，合理的关闭办法是让其运行完（也就是函数执行完毕），干净地释放掉所有资源，然后退出。如果是一个不断循环运行的线程，就需要用到线程间的通信机制，让主线程通知其退出。

##### 1.1.2 守护线程

```java
public static void main(String[] args) {
    final Thread thread = new Thread(() -> {
        while (true) {
            System.out.println("thread");
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }, "thread1");
    thread.setDaemon(true);//主线程退出 thread1 也退出 守护线程退出
    thread.start();// 不加的话 主线程退出 thread1不退出 守护线程不退出
}
```

当在一个JVM进程里面开多个线程时，这些线程被分成两类：守护线程和非守护线程。默认开的都是非守护线程。在Java中有一个规定：当所有的非守护线程退出后，整个JVM进程就会退出。意思就是守护线程“不算作数”，守护线程不影响整个 JVM 进程的退出。例如，垃圾回收线程就是守护线程，它们在后台默默工作，当开发者的所有前台线程（非守护线程）都退出之后，整个JVM进程就退出了。

##### 1.1.3 设置关闭的标志位

在上面的代码中，线程是一个死循环。但在实际工作中，开发人员通常不会这样编写，而是通过一个标志位来实现，如下面的代码所示。

![image-20210805092731563](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805092731563.png)

但上面的代码有一个问题：如果MyThread t在while循环中阻塞在某个地方，例如里面调用了 object.wait（）函数，那它可能永远没有机会再执行 while（！stopped）代码，也就一直无法退出循环。此时，就要用到下面所讲的InterruptedException（）与interrupt（）函数。

#### 1.2 InterruptedException()&interrupt()

##### 1.2.1 抛出Interrupted异常

Interrupt这个词很容易让人产生误解。从字面意思来看，好像是说一个线程运行到一半，把它中断了，然后抛出了InterruptedException异常，其实并不是。仍以上面的代码为例，假设while循环中没有调用任何的阻塞函数，就是通常的算术运算，或者打印一行日志

这个时候，在主线程中调用一句t.interrupt（）线程不会抛出异常

只有那些声明了会抛出 InterruptedException 的函数才会抛出异常，也就是下面这些常用的函数：

```java
sleep()
join()
wait()
```

##### 1.2.2 轻量级阻塞与重量级阻塞

能够被中断的阻塞称为轻量级阻塞，对应的线程状态是WAITING或者TIMED_WAITING；而像 synchronized 这种不能被中断的阻塞称为重量级阻塞，对应的状态是 BLOCKED。

![image-20210805093559496](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805093559496.png)



初始线程处于NEW状态，调用start（）之后开始执行，进入RUNNING或者READY状态。如果没有调用任何的阻塞函数，线程只会在RUNNING和READY之间切换，也就是系统的时间片调度。这两种状态的切换是操作系统完成的，开发者基本没有机会介入，除了可以调用yield（）函数，放弃对CPU的占用。
一旦调用了图中的任何阻塞函数，线程就会进入WAITING或者TIMED_WAITING状态，两者的区别只是前者为无限期阻塞，后者则传入了一个时间参数，阻塞一个有限的时间。如果使用了synchronized关键字
或者synchronized块，则会进入BLOCKED状态。

除了常用的阻塞/唤醒函数，还有一对不太常见的阻塞/唤醒函数，LockSupport.park（）/unpark（）。这对函数非常关键，Concurrent包中Lock的实现即依赖这一对操作原语。
故而t.interrupted（）的精确含义是“唤醒轻量级阻塞”，而不是字面意思“中断一个线程”。

##### 1.2.3 t.isInterrupted（）与Thread.interrupted（）的区别

因为 t.interrupted（）相当于给线程发送了一个唤醒的信号，所以如果线程此时恰好处于WAITING或者TIMED_WAITING状态，就会抛出一个InterruptedException，并且线程被唤醒。而如果线程此时并没有被阻塞，则线程什么都不会做。但在后续，线程可以判断自己是否收到过其他线程发来的中断信号，然后做一些对应的处理，这也是本节要讲的两个函数。
这两个函数都是线程用来判断自己是否收到过中断信号的，前者是非静态函数，后者是静态函数。二者的区别在于，前者只是读取中断状态，不修改状态；后者不仅读取中断状态，还会重置中断标志位。

#### 1.3 synchronized关键字

##### 1.3.1 锁的对象是什么

普通方法,锁是当前实例对象

静态方法,锁是当前类的class对象

同步方法块,锁是Synchronized括号里的对象

==对于非静态成员函数，锁其实是加在对象a上面的；对于静态成员函数，锁是加在A.class上面的。当然，class本身也是对象。==
这间接回答了关于 synchronized 的常见问题：一个静态成员函数和一个非静态成员函数，都加了synchronized关键字，分别被两个线程调用，它们是否互斥？很显然，因为是两把不同的锁，所以不会互斥。

##### 1.3.2 锁的本质是什么

从程序角度来看，锁其实就是一个“对象”，这个对象要完成以
下几件事情：

1. 这个对象内部得有一个标志位（state变量），记录自己有没有被某个线程占用（也就是记录当前有没有游客已经进入了房子）。最简单的情况是这个state有0、1两个取值，0表示没有线程占用这个锁，1表示有某个线程占用了这个锁。
2. 如果这个对象被某个线程占用，它得记录这个线程的thread ID，知道自己是被哪个线程占用了（也就是记录现在是谁在房子里面）。
3. 这个对象还得维护一个thread id list，记录其他所有阻塞的、等待拿这个锁的线程（也就是记录所有在外边等待的游客）。在当前线程释放锁之后（也就是把state从1改回0），从这个thread id list里面取一个线程唤醒。

既然锁是一个“对象”，要访问的共享资源本身也是一个对象，例如前面的对象 a，这两个对象可以合成一个对象。代码就变成synchronized（this）{…}，我们要访问的共享资源是对象a，锁也是加在对象a上面的。当然，也可以另外新建一个对象，代码变成synchronized（obj1）{…}。这个时候，访问的共享资源是对象a，而锁是加在新建的对象obj1上面的。
资源和锁合二为一，使得在Java里面，synchronized关键字可以加在任何对象的成员上面。这意味着，这个对象既是共享资源，同时也具备“锁”的功能！
下面来看 Java 是如何做到让任何一个对象都具备“锁”的功能的，这也就是 synchronized的实现原理

##### 1.3.3 synchronized实现原理

答案在Java的对象头里。在对象头里，有一块数据叫Mark Word。在64位机器上，Mark Word是8字节（64位）的，这64位中有2个重要字段：锁标志位和占用该锁的thread ID。因为不同版本的JVM实现，对象头的数据结构会有各种差异，此处不再进一步讨论。
此处主要是想说明锁实现的思路，因为后面讲ReentrantLock的详细实现时，也基于类似的思路。在这个基本的思路之上，synchronized还会有偏向、自旋等优化策略，ReentrantLock同样会用到这些优化策略，到时会结合代码详细展开。





monitor

偏向锁、轻量级锁、重量级锁适用于不同的并发场景：

- 偏向锁：无实际竞争，且将来只有第一个申请锁的线程会使用锁。
- 轻量级锁：无实际竞争，多个线程交替使用锁；允许短时间的锁竞争。
- 重量级锁：有实际竞争，且锁竞争时间长。

![image-20210525175843106](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210525175843106.png)

###### 重量级锁

JDK 1.6之前 监视器锁可以认为直接对应底层操作系统中的互斥量（mutex）。这种同步方式的成本非常高，包括系统调用引起的内核态与用户态切换、线程阻塞造成的线程切换等。因此，后来称这种锁为“**重量级锁**”。

###### 自旋锁

**通过自旋锁，可以减少线程阻塞造成的线程切换**

适用于

1. **锁的持有时间比较短**

2. **锁持有时间长，但竞争不激烈**

**缺点**

1. 线程多处理器少，自旋会造成浪费
2. 占用CPU
3. 锁持有时间长，且竞争激烈的场景中, 自旋通常不能获得锁, 此时应主动禁用自旋锁

> 使用-XX:-UseSpinning参数关闭自旋锁优化；-XX:PreBlockSpin参数修改默认的自旋次数。



###### 自适应自旋锁

自适应意味着自旋的时间不再固定了，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定：

- 如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也很有可能再次成功，进而它将允许自旋等待持续相对更长的时间，比如100个循环。
- 相反的，如果对于某个锁，自旋很少成功获得过，那在以后要获取这个锁时将可能减少自旋时间甚至省略自旋过程，以避免浪费处理器资源。

**自适应自旋解决的是“锁竞争时间不确定”的问题**。JVM很难感知到确切的锁竞争时间，而交给用户分析就违反了JVM的设计初衷。*自适应自旋假定不同线程持有同一个锁对象的时间基本相当，竞争程度趋于稳定，因此，可以根据上一次自旋的时间与结果调整下一次自旋的时间*。

**缺点**

然而，自适应自旋也没能彻底解决该问题，*如果默认的自旋次数设置不合理（过高或过低），那么自适应的过程将很难收敛到合适的值*。



###### 轻量级锁

**轻量级锁的目标是，减少无实际竞争情况下，使用重量级锁产生的性能消耗**



使用轻量级锁时，不需要申请互斥量，仅仅*将Mark Word中的部分字节CAS更新指向线程栈中的Lock Record，如果更新成功，则轻量级锁获取成功*，记录锁状态为轻量级锁；*否则，说明已经有线程获得了轻量级锁，目前发生了锁竞争（不适合继续使用轻量级锁），接下来膨胀为重量级锁*。



由于轻量级锁天然瞄准不存在锁竞争的场景，如果存在锁竞争但不激烈，仍然可以用自旋锁优化，*自旋失败后再膨胀为重量级锁*。

**缺点**

同自旋锁相似：

- 如果*锁竞争激烈*，那么轻量级将很快膨胀为重量级锁，那么维持轻量级锁的过程就成了浪费。



###### 偏向锁

**偏向锁的目标是，减少无竞争且只有一个线程使用锁的情况下，使用轻量级锁产生的性能消耗**

轻量级锁每次申请、释放锁都至少需要一次CAS，但偏向锁只有初始化时需要一次CAS。



*偏向锁假定将来只有第一个申请锁的线程会使用锁*（不会有任何线程再来申请锁），因此，*只需要在Mark Word中CAS记录owner（本质上也是更新，但初始值为空），如果记录成功，则偏向锁获取成功*，记录锁状态为偏向锁，*以后当前线程等于owner就可以零成本的直接获得锁；否则，说明有其他线程竞争，膨胀为轻量级锁*。



**缺点**

同样的，如果明显存在其他线程申请锁，那么偏向锁将很快膨胀为轻量级锁。

> 参数-XX:-UseBiasedLocking禁止偏向锁优化（默认打开）



###### markword

java对象数据结构中的一部分  和java各种类型的锁密切相关



markword数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit和64bit，它的**最后2bit是锁状态标志位**，用来标记当前对象的状态，对象的所处的状态，决定了markword存储的内容

| 状态             | 标志位 | 存储内容                             |
| ---------------- | ------ | ------------------------------------ |
| 未锁定           | 01     | 对象哈希码、对象分代年龄             |
| 轻量级锁定       | 00     | 指向锁记录的指针                     |
| 膨胀(重量级锁定) | 10     | 执行重量级锁定的指针                 |
| GC标记           | 11     | 空(不需要记录信息)                   |
| 可偏向           | 01     | 偏向线程ID、偏向时间戳、对象分代年龄 |

###### 锁分配和膨胀过程

<img src="https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/java-Synchroized.png" alt="java-Synchroized"  />

**内置锁只能沿着偏向锁、轻量级锁、重量级锁的顺序逐渐膨胀**

**只有第一个申请偏向锁的线程能够返回成功，后续线程都必然失败**

![image-20210525171745115](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210525171745115.png)

简化版中指出了`重偏向`过程。这一过程对于性能优化和膨胀过程都非常重要













#### 1.4 wait（）与notify（）

##### 1.4.1 生产者-消费者模型

![image-20210805110742066](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805110742066.png)



1. 内存队列本身要加锁，才能实现线程安全。

2. 阻塞。当内存队列满了，生产者放不进去时，会被阻塞；当内存队列是空的时候，消费者无事可做，会被阻塞。

3. 双向通知。消费者被阻塞之后，生产者放入新数据，要notify（）消费者；反之，生产者被阻塞之后，消费者消费了数据，要notify（）生产者。

第（1）件事情必须要做，第（2）件和第（3）件事情不一定要做。例如，可以采取一个简单的办法，生产者放不进去之后，睡眠几百毫秒再重试，消费者取不到数据之后，睡眠几百毫秒再重试。但这个办法效率低下，也不实时。所以，我们只讨论如何阻塞、如何通知的问题。

**1.如何阻塞？**

办法1：线程自己阻塞自己，也就是生产者、消费者线程各自调用wait（）和notify（）。
办法2：用一个阻塞队列，当取不到或者放不进去数据的时候，入队/出队函数本身就是阻塞的。这也就是BlockingQueue的实现，后面会详细讲述。

**2.如何双向通知？**
办法1：wait（）与notify（）机制。
办法2：Condition机制。
此处，先讲wait（）与notify（）机制，后面会专门讲Condition机制与BlockingQueue机制

##### 1.4.2 为什么必须和synchronized一起使用

![image-20210805111721590](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805111721590.png)

两个线程，线程A调用f1（），线程B调用f2（）。答案已经很明显：两个线程之间要通信，对于同一个对象来说，一个线程调用该对象的wait（），另一个线程调用该对象的notify（），该对象本身就需要同步！所以，在调用wait（）、notify（）之前，要先通过synchronized关键字同步给对象，也就是给该对象加锁。

前面已经讲了，synchronized关键字可以加在任何对象的成员函数上面，任何对象都可能成为锁。那么，wait（）和notify（）要同样如此普及，也只能放在Object里面了。

##### 1.4.3 为什么wait（）的时候必须释放锁

避免死锁问题

```java
wait(){
//释放锁
//阻塞,等待被其他线程notify
//重新拿锁
}
```

##### 1.4.4 wait（）与notify（）的问题

wait()用while循环判断

![image-20210805112159176](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805112159176.png)

生产者本来只想通知消费者，但它把其他的生产者也通知了；消费者本来只想通知生产者，但它被其他的消费者通知了。原因就是wait（）和notify（）所作用的对象和synchronized所作用的对象是同一个，只能有一个对象，无法区分队列空和列队满两个条件。这正是Condition要解决的问题。

#### 1.5 volatile

##### 1.5.1 64位写入的原子性(Half Write)

举一个简单的例子，对于一个long型变量的赋值和取值操作而言，在多线程场景下，线程A调用set（100），线程B调用get（），在某些场景下，返回值可能不是100。

这有点反直觉，如此简单的一个赋值和取值操作，在多线程下面为什么会不对呢？这是因为JVM的规范并没有要求64位的long或者double的写入是原子的。在32位的机器上，一个64位变量的写入可能被拆分成两个32位的写操作来执行。这样一来，读取的线程就可能读到“一半的值”。解决办法也很简单，在long前面加上volatile关键字。

##### 1.5.2 内存可见性

##### 1.5.3 重排序：DCL问题

单例模式的线程安全的写法不止一种，常用写法为DCL（Double Checking Locking），如下所示。

![image-20210805113132682](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805113132682.png)

上述的instance=new Instance（）代码有问题：其底层会分为三个操作：

1. 分配一块内存。

2. 在内存上初始化成员变量。

3. 把instance引用指向内存。

在这三个操作中，操作（2）和操作（3）可能重排序，即先把instance指向内存，再初始化成员变量，因为二者并没有先后的依赖关系。此时，另外一个线程可能拿到一个未完全初始化的对象。这时，直接访问里面的成员变量，就可能出错。这就是典型的“构造函数溢出”问题。解决办法也很简单，就是为instance变量加上volatile修饰。

通过上面的例子，可以总结出volatile的三重功效：

==64位写入的原子性、内存可见性和禁止重排序。==

接下来，我们进入volatile原理的探究。

#### 1.6 JMM与happen-before

##### 1.6.1 为什么会存在“内存可见性”问题

要解释清楚这个问题，就涉及现代CPU的架构。图1-4所示为x86架构下CPU缓存的布局，即在一个CPU 4核下，L1、L2、L3三级缓存与主内存的布局。每个核上面有L1、L2缓存，L3缓存为所有核共用。

![image-20210805113433165](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805113433165.png)

因为存在CPU缓存一致性协议，例如MESI，多个CPU之间的缓存不会出现不同步的问题，不会有“内存可见性”问题。但是，缓存一致性协议对性能有很大损耗，为了解决这个问题，CPU 的设计者们在这个基础上又进行了各种优化。例如，在计算单元和L1之间加了Store Buffer、Load Buffer（还有其他各种Buffer），如图1-5所示。
L1、L2、L3和主内存之间是同步的，有缓存一致性协议的保证，但是Store Buffer、Load Buffer和L1之间却是异步的。也就是说，往内存中写入一个变量，这个变量会保存在Store Buffer里面，稍后才异步地写入L1中，同时同步写入主内存中。

注意，这里只是简要画了x86的CPU缓存体系，还没有进一步讨论SMP架构和NUMA的区别，还有其他CPU架构体系，例如PowerPC、MIPS、ARM等，不同CPU的缓存体系会有各种差异。

![image-20210805113721306](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805113721306.png)

但站在操作系统内核的角度，可以统一看待这件事情，也就是图1-6所示的操作系统内核视角下的CPU缓存模型。

![image-20210805113753556](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805113753556.png)



多CPU，每个CPU多核，每个核上面可能还有多个硬件线程，对于操作系统来讲，就相当于一个个的逻辑CPU。每个逻辑CPU都有自己的缓存，这些缓存和主内存之间不是完全同步的。
对应到Java里，就是JVM抽象内存模型，如图1-7所示。
到此为止，介绍了不同CPU架构下复杂的缓存体系，也就回答了为什么会出现“内存可见性”问题。

![image-20210805113845882](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805113845882.png)

##### 1.6.2 重排序与内存可见性的关系

Store Buffer的延迟写入是重排序的一种，称为内存重排序（Memory Ordering）。除此之外，还有编译器和CPU的指令重排序。下面对重排序做一个分类：

1. 编译器重排序。对于没有先后依赖关系的语句，编译器可以重新调整语句的执行顺序。

2. CPU指令重排序。在指令级别，让没有依赖关系的多条指令并行。

3. CPU内存重排序。CPU有自己的缓存，指令的执行顺序和写入主内存的顺序不完全一致。

在三种重排序中，第三类就是造成“内存可见性”问题的主因，下面再举一个例子来进一步说明这个问题。如下所示。

线程1：
X=1
a=Y
线程2：
Y=1
b=X

假设X、Y是两个全局变量，初始的时候，X=0，Y=0。请问，这两个线程执行完毕之后，a、b的正确结果应该是什么？
很显然，线程1和线程2的执行先后顺序是不确定的，可能顺序执行，也可能交叉执行，最终正确的结果可能是：
(1)a=0,b=1
(2)a=1,b=0
(3)a=1,b=1
也就是不管谁先谁后，执行结果应该是这三种场景中的一种。但实际可能是a=0，b=0。
两个线程的指令都没有重排序，执行顺序就是代码的顺序，但仍然可能出现a=0，b=0。原因是线程1先执行X=1，后执行a=Y，但此时X=1还在自己的Store Buffer里面，没有及时写入主内存中。所以，线程2看到的X还是0。线程2的道理与此相同。
这就是一个有意思的地方，虽然线程1觉得自己是按代码顺序正常执行的，但在线程2看来，a=Y和X=1顺序却是颠倒的。指令没有重排序，是写入内存的操作被延迟了，也就是内存被重排序了，这就造成内存可见性问题。

##### 1.6.3 as-if-serial语义

###### 1.单线程程序的重排序规则

无论什么语言，站在编译器和CPU的角度来说，不管怎么重排序，单线程程序的执行结果不能改变，这就是单线程程序的重排序规则。换句话说，只要操作之间没有数据依赖性，如上例所示，编译器和CPU都可以任意重排序，因为执行结果不会改变，代码看起来就像是完全串行地一行行从头执行到尾，这也就是as-if-serial语义。对于单线程程序来说，编译器和CPU可能做了重排序，但开发者感知不到，也不存在内存可见性问题。

###### 2.多线程程序的重排序规则

编译器和CPU的这一行为对于单线程程序没有影响，但对多线程程序却有影响。对于多线程程序来说，线程之间的数据依赖性太复杂，编译器和CPU没有办法完全理解这种依赖性并据此做出最合理的优化。所以，编译器和CPU只能保证每个线程的as-if-serial语义。线程之间的数据依赖和相互影响，需要编译器和CPU的上层来确定。上层要告知编译器和CPU在多线程场景下什么时候可以重排序，什么时候不能重排序。

##### 1.6.4 happen-before是什么

为了明确定义在多线程场景下，什么时候可以重排序，什么时候不能重排序，Java 引入了JMM（Java Memory Model），也就是Java内存模型（单线程场景不用说明，有as-if-serial语义保证）。这个模型就是一套规范，对上，是JVM和开发者之间的协定；对下，是JVM和编译器、CPU之间的协定。

定义这套规范，其实是要在开发者写程序的方便性和系统运行的效率之间找到一个平衡点。一方面，要让编译器和CPU可以灵活地重排序；另一方面，要对开发者做一些承诺，明确告知开发者不需要感知什么样的重排序，需要感知什么样的重排序。然后，根据需要决定这种重排序对程序是否有影响。如果有影响，就需要开发者显示地通过volatile、synchronized等线程同步机制来禁止重排序

为了描述这个规范，JMM引入了happen-before，使用happen-before描述两个操作之间的内存可见性。那么，happen-before是什么呢？

==如果A happen-before B，意味着A的执行结果必须对B可见，也就是保证跨线程的内存可见性==。A happen before B不代表A一定在B之前执行。因为，对于多线程程序而言，两个操作的执行顺序是不确定的。happen-before只确保如果A在B之前执行，则A的执行结果必须对B可见。定义了内存可见性的约束，也就定义了一系列重排序的约束

##### 1.6.5 happen-before的传递性

除了这些基本的happen-before规则，happen-before还具有传递性，即若A happen-before B，B happen-before C，则A happen-before C。
如果一个变量不是volatile变量，当一个线程读取、一个线程写入时可能有问题。那岂不是说，在多线程程序中，我们要么加锁，要么必须把所有变量都声明为volatile变量？这显然不可能，而这就得归功于happen-before的传递性。

happen-before的传递性非常有用，后面讲到Concurrent包的很多实现的时候，还会用到这个特性。

##### 1.6.6 C++中的volatile关键字

在C++中也有volatile关键字，但其含义和Java中的有一些差别。考虑下面的代码：

![image-20210805120937658](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805120937658.png)

通过done这个标志位来标志一个任务是否完成。假设一个线程A调用process，另一个线程不断地轮询调用afterProcess，当done=true的时候，做一些额外工作。
这个代码在C++中是错误的，但在Java中却是正确的。因为Java中的volatile关键字不仅具有内存可见性，还会禁止volatile变量写入和非volatile变量写入的重排序，但C++中的volatile关键字不会禁止这种重排序。

##### 1.6.7 JSR-133对volatile语义的增强

Java的volatile比C++多出的这点特性，正是JSR-133对volatile语义的增强

也就是说，在旧的JMM模型中，volatile变量的写入会和非volatile变量的读取或写入重排序，正如C++中所做的。但新的模型不会，这也正体现了Java对happen-before规则的严格遵守。

#### 1.7 内存屏障

为了禁止编译器重排序和 CPU 重排序，在编译器和 CPU 层面都有对应的指令，也就是内存屏障（Memory Barrier）。这也正是JMM和happen-before规则的底层实现原理。

编译器的内存屏障，只是为了告诉编译器不要对指令进行重排序。当编译完成之后，这种内存屏障就消失了，CPU并不会感知到编译器中内存屏障的存在。

而CPU的内存屏障是CPU提供的指令，可以由开发者显示调用。下面主要讲CPU的内存屏障。

##### 1.7.1 Linux中的内存屏障

为了直观地说明内存屏障，下面摘录了Linux内核kfifo.c的源代码。
这是一个RingBuffer，允许一个线程写入，一个线程读取（只能一写一读），整个代码没加任何的锁，也没有CAS，但是线程是安全的，这是如何做到的呢？

![image-20210805132537967](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805132537967.png)

![image-20210805132558782](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805132558782.png)

通过上面代码可以看到，kfifo 在修改数据和更新指针（队头，队尾）之间，通过函数smp_wmb（）插入了一个Store Barrier，从而确保了：

* 更新指针的操作，不会被重排序到修改数据之前。
* 更新指针的时候，Store Cache被刷新，其他CPU可见。

这里有一个关键点要说明：在修改in、out指针之后，并没有插入内存屏障。这意味着对in、out 的修改，可能短时间内会对其他 CPU或线程不可见。也就是说，数据修改了，但是指针没变。但这不会引发问题，最多是在读的时候队列不为空，判断为空了；在写的时候，队列没满，但判断为满了。调用者的重试机制解决了这个问题。因此，这里其实是“弱一致的”，或者说是“最终一致性”的。

##### 1.7.2 JDK中的内存屏障

内存屏障是很底层的概念，对于 Java 开发者来说，一般用 volatile 关键字就足够了。但从JDK 8开始，Java在Unsafe类中提供了三个内存屏障函数，如下所示。

![image-20210805133617301](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805133617301.png)

要说明的是，这三个屏障并不是最基本的内存屏障。在理论层面，可以把基本的CPU内存屏障分成四种：

1. LoadLoad：禁止读和读的重排序。
2. StoreStore：禁止写和写的重排序。
3. LoadStore：禁止读和写的重排序。
4. StoreLoad：禁止写和读的重排序。

JDK定义的三种内存屏障和理论层面划分的四类内存屏障是什么关系呢？

1. loadFence = LoadLoad + LoadStore

   在该方法之前的所有读操作，一定在load屏障之前执行完成。

2. storeFence = StoreStore + LoadStore

   在该方法之前的所有写操作，一定在store屏障之前执行完成

3. fullFence = loadFence + storeFence + StoreLoad

   在该方法之前的所有读写操作，一定在full屏障之前执行完成，这个内存屏障相当于上面两个(load屏障和store屏障)的合体功能。

##### 1.7.3 volatile实现原理

1. 在volatile写操作的前面插入一个StoreStore屏障。保证volatile写操作不会和之前的写操作重排序。
2. 在volatile写操作的后面插入一个StoreLoad屏障。保证volatile写操作不会和之后的读操作重排序。
3. 在volatile读操作的后面插入一个LoadLoad屏障+LoadStore屏障。保证volatile读操作不会和之后的读操作、写操作重排序。

#### 1.8 final关键字

##### 1.8.1 构造函数溢出问题

![image-20210805143019658](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805143019658.png)

a，b未必一定等于1，2。和DCL的例子类似，也就是构造函数溢出问题。obj=new Example（）这行代码，分解成三个操作：

1. 分配一块内存；
2. 在内存上初始化i=1，j=2；
3.  把obj指向这块内存。

操作②和操作③可能重排序，因此线程B可能看到未正确初始化的值。对于构造函数溢出，通俗来讲，就是一个对象的构造并不是“原子的”，当一个线程正在构造对象时，另外一个线程却可以读到未构造好的“一半对象”。

##### 1.8.2 final的happen-before语义

要解决这个问题，不止有一种办法。
办法1：给i，j都加上volatile关键字。
办法2：为read/write函数都加上synchronized关键字。
如果i，j只需要初始化一次，则后续值就不会再变了，还有办法3，为其加上final关键字。
之所以能解决问题，是因为同volatile一样，final关键字也有相应的happen-before语义：
（1）对final域的写（构造函数内部），happen-before于后续对final域所在对象的读。
（2）对final域所在对象的读，happen-before于后续对final域的读。
通过这种happen-before语义的限定，保证了final域的赋值，一定在构造函数之前完成，不会出现另外一个线程读取到了对象，但对象里面的变量却还没有初始化的情形，避免出现构造函数溢出的问题。

关于final和volatile的特性与背后的原理，到此为止就讲完了，在后续Concurrent包的源码分析中会反复看到这两个关键字的身影。接下来总结常用的几个happen-before规则。

##### 1.8.3 happen-before规则总结

1. 单线程中的每个操作，happen-before于该线程中任意后续操作。

2. 对volatile变量的写，happen-before于后续对这个变量的读。

3. 对synchronized的解锁，happen-before于后续对这个锁的加锁。

4. 对final变量的写，happen-before于final域对象的读，happen-before于后续对final变量的读。

四个基本规则再加上happen-before的传递性，就构成JMM对开发者的整个承诺。在这个承诺以外的部分，程序都可能被重排序，都需要开发者小心地处理内存可见性问题。

![image-20210805143525716](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805143525716.png)



#### 1.9 综合应用：无锁编程

##### 1.9.1 一写一读的无锁队列：内存屏障

一写一读的无锁队列即Linux内核的kfifo队列，一写一读两个线程，不需要锁，只需要内存屏障。

##### 1.9.2 一写多读的无锁队列：volatile关键字

在Martin Fowler关于LMAX架构的介绍中，谈到了Disruptor。Disruptor是一个开源的并发框架，能够在无锁的情况下实现Queue并发操作。

Disruptor的RingBuffer之所以可以做到完全无锁，也是因为“单线程写”，这是“前提的前提”。离开了这个前提条件，没有任何技术可以做到完全无锁。借用Disruptor官方提到的一篇博客文章Sharing Data Among Threads Without Contention，也就是single-writerprinciple。

在这个原则下，利用 volatile 关键字可以实现一写多读的线程安全。具体来说，就是RingBuffer有一个头指针，对应一个生产者线程；多个尾指针对应多个消费者线程。每个消费者线程只会操作自己的尾指针。所有这些指针的类型都是volatile变量，通过头指针和尾指针的比较，判断队列是否为空。

##### 1.9.3 多写多读的无锁队列：CAS

同内存屏障一样，CAS（Compare And Set）也是CPU提供的一种原子指令。在第2章中会对CAS进行详细的解释。
基于CAS和链表，可以实现一个多写多读的队列。具体来说，就是链表有一个头指针head和尾指针tail。

入队列，通过对tail进行CAS操作完成；

出队列，对head进行CAS操作完成。

在第3章讲Lock的实现的时候，将反复用到这种队列，会详细展开介绍。

##### 1.9.4 无锁栈

无锁栈比无锁队列的实现更简单，只需要对 head 指针进行 CAS操纵，就能实现多线程的入栈和出栈。
在第4章讲工具类的实现的时候，会用到无锁栈。

##### 1.9.5 无锁链表

相比无锁队列与无锁栈，无锁链表要复杂得多，因为无锁链表要在中间插入和删除元素。
在5.6节，介绍ConcurrentSkipListMap实现的时候，会讲到并发的跳查表。其实现就是基于无锁链表的，到时会详细展开论述。

### 2.Atomic类

#### 2.1 AtomicInteger和AtomicLong

使用CAS

##### 2.1.1 悲观锁与乐观锁

对于悲观锁，作者认为数据发生并发冲突的概率很大，所以读操作之前就上锁。synchronized关键字，以及后面要讲的ReentrantLock都是悲观锁的典型例子。

对于乐观锁，作者认为数据发生并发冲突的概率比较小，所以读操作之前不上锁。等到写操作的时候，再判断数据在此期间是否被其他线程修改了。如果被其他线程修改了，就把数据重新读出来，重复该过程；如果没有被修改，就写回去。判断数据是否被修改，同时写回新值，这两个操作要合成一个原子操作，也就是CAS（Compare AndSet）。

AtomicInteger的实现就是典型的乐观锁，在MySQL和Redis中有类似的思路。

##### 2.1.2 Unsafe的CAS详解

上面调用的CAS函数，其实是封装的Unsafe类中的一个native函数

```java
//expect 旧值
//update 新值
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

```java
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

该函数有4个参数。在前两个参数中，第一个是对象（也就是 AtomicInteger 对象），第二个是对象的成员变量（也就是AtomictInteger里面包的int变量value），后两个参数保持不变。

要特别说明一下第二个参数，它是一个long型的整数，经常被称为xxxOffset，意思是某个成员变量在对应的类中的内存偏移量（该变量在内存中的位置），表示该成员变量本身。在Unsafe中专门有一个函数，把成员变量转化成偏移量，如下所示。

```java
 public native long objectFieldOffset(Field var1);
```

所有调用CAS的地方，都会先通过这个函数把成员变量转换成一个Offset。以AtomicInteger为例：

```java
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
```

从上面代码可以看到，无论是Unsafe还是valueOffset，都是静态的，也就是类级别的，所有对象共用的。

在转化的时候，先通过反射（getDeclaredField）获取value成员变量对应的Field对象，再通过objectFieldOffset函数转化成valueOffset。此处的valueOffset就代表了value变量本身，后面执行CAS操作的时候，不是直接操作value，而是操作valueOffset。

##### 2.1.3 自旋与阻塞

当一个线程拿不到锁的时候，有以下两种基本的等待策略。
策略1：放弃CPU，进入阻塞状态，等待后续被唤醒，再重新被操作系统调度。
策略2：不放弃CPU，空转，不断重试，也就是所谓的“自旋”。

很显然，如果是单核的CPU，只能用策略1。因为如果不放弃CPU，那么其他线程无法运行，也就无法释放锁。但对于多CPU或者多核，策略2就很有用了，因为没有线程切换的开销。

AtomicInteger的实现就用的是“自旋”策略，如果拿不到锁，就会一直重试。
有一点要说明：这两种策略并不是互斥的，可以结合使用。如果拿不到锁，先自旋几圈；如果自旋还拿不到锁，再阻塞，synchronized关键字就是这样的实现策略

除了AtomicInteger，AtomicLong也是同样的原理，此处不再赘述。

#### 2.2 AtomicBoolean和AtomicReference

##### 2.2.1 为什么需要AtomicBoolean

对于int或者long型变量，需要进行加减操作，所以要加锁；但对于一个boolean类型来说，true或false的赋值和取值操作，加上volatile关键字就够了，为什么还需要AtomicBoolean呢？
这是因为往往要实现下面这种功能：

```java
if(a==false){
    a=true;
}
```

也就是要实现 compare和set两个操作合在一起的原子性，而这也正是CAS提供的功能

同样地，AtomicReference也需要同样的功能

##### 2.2.2 如何支持boolean和double类型

在Unsafe类中，只提供了三种类型的CAS操作：int、long、Object（也就是引用类型）。如下所示。

```java
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

第一个参数是要修改的对象，第二个参数是对象的成员变量在内存中的位置（一个long型的整数），第三个参数是该变量的旧值，第四个参数是该变量的新值。

AtomicBoolean类型怎么支持呢？

对于用int型来代替的，在入参的时候，将boolean类型转换成int类型；在返回值的时候，将int类型转换成boolean类型。如下所示。

```java
public final boolean compareAndSet(boolean expect, boolean update) {
    int e = expect ? 1 : 0;
    int u = update ? 1 : 0;
    return unsafe.compareAndSwapInt(this, valueOffset, e, u);
}
```

如果是double类型，又如何支持呢？
这依赖double类型提供的一对double类型和long类型互转的函数，这点在介绍DoubleAdder的时候会提到。

```java
public static native double longBitsToDouble(long bits);
public static native long doubleToRawLongBits(double value);
```

#### 2.3 AtomicStampedReference和AtomicMarkableReference

##### 2.3.1 ABA问题与解决办法

到目前为止，CAS都是基于“值”来做比较的。但如果另外一个线程把变量的值从A改为B，再从B改回到A，那么尽管修改过两次，可是在当前线程做CAS操作的时候，却会因为值没变而认为数据没有被其他线程修改过，这就是所谓的ABA问题。

要解决 ABA 问题，不仅要比较“值”，还要比较“版本号”，而这正是 AtomicStamped-Reference做的事情，其对应的CAS函数如下：

```java
 public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp)
```

之前的 CAS只有两个参数，这里的 CAS有四个参数，后两个参数就是版本号的旧值和新值。

##### 2.3.2 为什么没有AtomicStampedInteger或AtomictStampedLong

要解决Integer或者Long型变量的ABA问题，为什么只有AtomicStampedReference，而没有AtomicStampedInteger或者AtomictStampedLong呢？

因为这里要同时比较数据的“值”和“版本号”，而Integer型或者Long型的CAS没有办法同时比较两个变量，于是只能把值和版本号封装成一个对象，也就是这里面的 Pair 内部类，然后通过对象引用的CAS来实现。代码如下所示。

```java
public class AtomicStampedReference<V> {

    private static class Pair<T> {
        final T reference;
        final int stamp;
        private Pair(T reference, int stamp) {
            this.reference = reference;
            this.stamp = stamp;
        }
        static <T> Pair<T> of(T reference, int stamp) {
            return new Pair<T>(reference, stamp);
        }
    }

    private volatile Pair<V> pair;
    public boolean compareAndSet(V   expectedReference, V   newReference,
                                 int expectedStamp, int newStamp) {
        Pair<V> current = pair;
        return
            expectedReference == current.reference &&
            expectedStamp == current.stamp &&
            ((newReference == current.reference &&
              newStamp == current.stamp) ||
             casPair(current, Pair.of(newReference, newStamp)));
    }
    private boolean casPair(Pair<V> cmp, Pair<V> val) {
        return UNSAFE.compareAndSwapObject(this, pairOffset, cmp, val);
    }
}
```

当使用的时候，在构造函数里面传入值和版本号两个参数，应用程序对版本号进行累加操作，然后调用上面的CAS。如下所示。

```java
public AtomicStampedReference(V initialRef, int initialStamp) {
    pair = Pair.of(initialRef, initialStamp);
}
```

##### 2.3.3 AtomicMarkableReference

AtomicMarkableReference与AtomicStampedReference原理类似，只是Pair里面的版本号是boolean类型的，而不是整型的累加变量，如下所示。

```java
public class AtomicMarkableReference<V> {
    private static class Pair<T> {
        final T reference;
        final boolean mark;
        private Pair(T reference, boolean mark) {
            this.reference = reference;
            this.mark = mark;
        }
        static <T> Pair<T> of(T reference, boolean mark) {
            return new Pair<T>(reference, mark);
        }
    }
    private volatile Pair<V> pair;
}
```

因为是boolean类型，只能有true、false 两个版本号，所以并不能完全避免ABA问题，只是降低了ABA发生的概率。

#### 2.4 AtomicIntegerFieldUpdater、AtomicLongFieldUpdater和AtomicReferenceFieldUpdater

##### 2.4.1 为什么需要AtomicXXXFieldUpdater

如果一个类是自己编写的，则可以在编写的时候把成员变量定义为Atomic类型。但如果是一个已经有的类，在不能更改其源代码的情况下，要想实现对其成员变量的原子操作，就需要AtomicIntegerFieldUpdater、AtomicLongFieldUpdater 和 AtomicReferenceFieldUpdater。下面以AtomicIntegerFieldUpdater为例介绍其实现原理。

首先，其构造函数是 protected，不能直接构造其对象，必须通过它提供的一个静态函数来创建，如下所示

```java
@CallerSensitive
public static <U> AtomicIntegerFieldUpdater<U> newUpdater(Class<U> tclass,
                                                          String fieldName) {
    return new AtomicIntegerFieldUpdaterImpl<U>
        (tclass, fieldName, Reflection.getCallerClass());
}
```

newUpdater（..）静态函数传入的是要修改的类（不是对象）和对应的成员变量的名字，内部通过反射拿到这个类的成员变量，然后包装成一个AtomicIntegerFieldUpdater对象。所以，这个对象表示的是类的某个成员，而不是对象的成员变量。
若要修改某个对象的成员变量的值，再传入相应的对象，如下所示。

```java
public int getAndIncrement(T obj) {
    int prev, next;
    do {
        prev = get(obj);
        next = prev + 1;
    } while (!compareAndSet(obj, prev, next));
    return prev;
}
public final boolean compareAndSet(T obj, int expect, int update) {
    accessCheck(obj);
    return U.compareAndSwapInt(obj, offset, expect, update);
}
```

accecssCheck函数的作用是检查该obj是不是tclass类型，如果不是，则拒绝修改，抛出异常。

从代码可以看到，其 CAS 原理和 AtomictInteger 是一样的，底层都调用了 Unsafe 的compareAndSwapInt（..）函数。

##### 2.4.2 限制条件

要想使用AtomicIntegerFieldUpdater修改成员变量，成员变量必须是volatile的int类型（不能是Integer包装类），该限制从其构造函数中可以看到：

```java
if (field.getType() != int.class)
    throw new IllegalArgumentException("Must be integer type");

if (!Modifier.isVolatile(modifiers))
    throw new IllegalArgumentException("Must be volatile type");
```

至于 AtomicLongFieldUpdater 、AtomicReferenceFieldUpdater，也有类似的限制条件。其底层的CAS原理，也和AtomicLong、AtomicReference一样，此处不再赘述。

#### 2.5 AtomicIntegerArray、AtomicLongArray和Atomic-ReferenceArray

Concurrent包提供了AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray三个数组元素的原子操作。注意，这里并不是说对整个数组的操作是原子的，而是针对数组中一个元素的原子操作而言。

###### 2.5.1 使用方式

相比于AtomicInteger的getAndIncrement（）函数，这里只是多了一个传入参数：数组的下标i。
其他函数也与此类似，相比于 AtomicInteger 的各种加减函数，也都是多一个下标 i

###### 2.5.2 实现原理

其底层的CAS函数用的还是compareAndSwapInt，但是把数组下标i转化成对应的内存偏移量，所用的方法和之前的AtomicInteger不太一样，如下所示。

```java
private static long byteOffset(int i) {
    return ((long) i << shift) + base;
}
```

把下标 i 转换成对应的内存地址，用到了 shift 和 base 两个变量。这两个变量都是AtomicIntegerArray的静态成员变量，用了Unsafe类的arrayBaseOffset和arrayIndexScale两个函数来获取。赋值如下：

```java
private static final int base = unsafe.arrayBaseOffset(int[].class);
private static final int shift;
private final int[] array;

static {
    int scale = unsafe.arrayIndexScale(int[].class);//scale是2的整数次方
    if ((scale & (scale - 1)) != 0)
        throw new Error("data type scale not a power of two");
    shift = 31 - Integer.numberOfLeadingZeros(scale);
}
```

其中，base表示数组的首地址的位置，scale表示一个数组元素的大小，i的偏移量则等于：i*scale+base。

但为了优化性能，使用了位移操作，shift表示scale中1的位置（scale是2的整数次方）。所以，偏移量的计算变成上面代码中的：i << shift + base，表达的意思就是：i*scale+base。
知道了偏移量的计算方式，理解CAS操作就容易了：

```java
private boolean compareAndSetRaw(long offset, int expect, int update) {
    return unsafe.compareAndSwapInt(array, offset, expect, update);
}
```

第1个参数是int[]对象，第2个参数是下标i对应的内存偏移量，第3个和第4个参数分别是旧值和新值。

#### 2.6 Striped64与LongAdder

从JDK 8开始，针对Long型的原子操作，Java又提供了LongAdder、LongAccumulator；针对Double类型，Java提供了DoubleAdder、DoubleAccumulator。Striped64相关的类的继承层次如图2-2所示。

![image-20210805155545429](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805155545429.png)



##### 2.6.1 LongAdder原理

AtomicLong内部是一个volatile long型变量，由多个线程对这个变量进行CAS操作。多个线程同时对一个变量进行CAS操作，在高并发的场景下仍不够快，如果再要提高性能，该怎么做呢？

把一个变量拆成多份，变为多个变量，有些类似于 ConcurrentHashMap 的分段锁的例子。如图2-3所示，把一个Long型拆成一个base变量外加多个Cell，每个Cell包装了一个Long型变量。当多个线程并发累加的时候，如果并发度低，就直接加到base变量上；如果并发度高，冲突大，平摊到这些Cell上。在最后取值的时候，再把base和这些Cell求sum运算。

![image-20210805155719935](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805155719935.png)

以LongAdder的sum（）函数为例，如下所示。

```java
public long sum() {
    Cell[] as = cells; Cell a;
    long sum = base;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}
```

由于无论是long，还是double，都是64位的。但因为没有double型的CAS操作，所以是通过把double型转化成long型来实现的。所以，上面的base和cell[]变量，是位于基类Striped64当中的。英文Striped意为“条带”，也就是分片

##### 2.6.2 最终一致性

在sum求和函数中，并没有对cells[]数组加锁。也就是说，一边有线程对其执行求和操作，一边还有线程修改数组里的值，也就是最终一致性，而不是强一致性。这也类似于ConcurrentHashMap 中的 clear（）函数，一边执行清空操作，一边还有线程放入数据，clear（）函数调用完毕后再读取，hash map里面可能还有元素。因此，在LongAdder开篇的注释中，把它和AtomicLong 的使用场景做了比较。它适合高并发的统计场景，而不适合要对某个 Long 型变量进行严格同步的场景。

##### 2.6.3 伪共享与缓存行填充

在Cell类的定义中，用了一个独特的注解@sun.misc.Contended，这是JDK 8之后才有的，背后涉及一个很重要的优化原理：伪共享与缓存行填充。

在讲 CPU 架构的时候提到过，每个 CPU 都有自己的缓存。缓存与主内存进行数据交换的基本单位叫Cache Line（缓存行）。在64位x86架构中，缓存行是64字节，也就是8个Long型的大小。这也意味着当缓存失效，要刷新到主内存的时候，最少要刷新64字节。

如图2-4所示，主内存中有变量X、Y、Z（假设每个变量都是一个Long型），被CPU1和CPU2分别读入自己的缓存，放在了同一行Cache Line里面。当CPU1修改了X变量，它要失效整行Cache Line，也就是往总线上发消息，通知CPU 2对应的Cache Line失效。由于Cache Line是数据交换的基本单位，无法只失效X，要失效就会失效整行的Cache Line，这会导致Y、Z变量的缓存也失效。

![image-20210805160355457](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805160355457.png)



虽然只修改了X变量，本应该只失效X变量的缓存，但Y、Z变量也随之失效。Y、Z变量的数据没有修改，本应该很好地被 CPU1 和 CPU2共享，却没做到，这就是所谓的“伪共享问题”。问题的原因是，Y、Z和X变量处在了同一行Cache Line里面。要解决这个问题，需要用到所谓的“缓存行填充”，分别在X、Y、Z后面加上7个无用的Long型，填充整个缓存行，让X、Y、Z处在三行不同的缓存行中，如图2-5所示。

![image-20210805160524108](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805160524108.png)

下面的代码来自JDK 7的Exchanger类，为了安全，它填充了15（8+7）个long型。

![image-20210805160813627](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805160813627.png)

在著名的开源无锁并发框架Disruptor中，也有类似的代码：

![image-20210805160831497](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805160831497.png)

而在JDK 8中，就不需要写这种晦涩的代码了，只需声明一个@sun.misc.Contended即可。下面的代码摘自JDK 8里面Exchanger中Node的定义，相较于JDK 7有了明显变化。

回到上面的例子，之所以这个地方要用缓存行填充，是为了不让Cell[]数组中相邻的元素落到同一个缓存行里。

##### 2.6.4 LongAdder核心实现

下面来看LongAdder最核心的累加函数add（long x），自增、自减操作都是通过调用该函数实现的。

```java
public void add(long x) {
    Cell[] as; long b, v; int m; Cell a;
    if ((as = cells) != null || !casBase(b = base, b + x)) {
        boolean uncontended = true;
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[getProbe() & m]) == null ||
            !(uncontended = a.cas(v = a.value, v + x)))
            longAccumulate(x, null, uncontended);
    }
}
```

当一个线程调用add（x）的时候，首先会尝试使用casBase把x加到base变量上。如果不成功，则再用 a.cas（..）函数尝试把 x 加到Cell 数组的某个元素上。如果还不成功，最后再调用longAccumulate（..）函数。

注意：Cell[]数组的大小始终是2的整数次方，在运行中会不断扩容，每次扩容都是增长2倍。上面代码中的as[getProbe（）&m]其实就是对数组的大小取模。因为m=as.length–1，getProbe（）为该线程生成一个随机数，用该随机数对数组的长度取模。因为数组长度是2的整数次方，所以可以用&操作来优化取模运算。

对于一个线程来说，它并不在意到底是把x累加到base上面，还是累加到Cell[]数组上面，只要累加成功就可以。因此，这里使用随机数来实现Cell的长度取模。

如果两次尝试都不成功，则调用 longAccumulate（..）函数，该函数在 Striped64 里面LongAccumulator也会用到，如下所示。

```java
final void longAccumulate(long x, LongBinaryOperator fn, boolean wasUncontended) {
    int h;
    if ((h = getProbe()) == 0) {
        ThreadLocalRandom.current(); // force initialization
        h = getProbe();
        wasUncontended = true;
    }
    boolean collide = false;                // True if last slot nonempty 写入冲突标志位
    for (;;) {
        Cell[] as; Cell a; int n; long v;
        if ((as = cells) != null && (n = as.length) > 0) {
            if ((a = as[(n - 1) & h]) == null) {
                if (cellsBusy == 0) {       // Try to attach new Cell
                    Cell r = new Cell(x);   // Optimistically create
                    if (cellsBusy == 0 && casCellsBusy()) {
                        boolean created = false;
                        try {               // Recheck under lock
                            Cell[] rs; int m, j;
                            if ((rs = cells) != null &&
                                (m = rs.length) > 0 &&
                                rs[j = (m - 1) & h] == null) {
                                rs[j] = r;
                                created = true;
                            }
                        } finally {
                            cellsBusy = 0;
                        }
                        if (created)
                            break;
                        continue;           // Slot is now non-empty
                    }
                }
                collide = false;
            }
            else if (!wasUncontended)       // CAS already known to fail
                wasUncontended = true;      // Continue after rehash
            else if (a.cas(v = a.value, ((fn == null) ? v + x :
                                         fn.applyAsLong(v, x))))
                break;
            else if (n >= NCPU || cells != as)
                collide = false;            // At max size or stale
            else if (!collide)
                collide = true;
            else if (cellsBusy == 0 && casCellsBusy()) {
                try {
                    if (cells == as) {      // Expand table unless stale 数组扩容
                        Cell[] rs = new Cell[n << 1];
                        for (int i = 0; i < n; ++i)
                            rs[i] = as[i];
                        cells = rs;
                    }
                } finally {
                    cellsBusy = 0;
                }
                collide = false;
                continue;                   // Retry with expanded table
            }
            h = advanceProbe(h);
        }
        else if (cellsBusy == 0 && cells == as && casCellsBusy()) {
            boolean init = false;
            try {                           // Initialize table  初始化数组
                if (cells == as) {
                    Cell[] rs = new Cell[2];
                    rs[h & 1] = new Cell(x);
                    cells = rs;
                    init = true;
                }
            } finally {
                cellsBusy = 0;
            }
            if (init)
                break;
        }
        else if (casBase(v = base, ((fn == null) ? v + x :
                                    fn.applyAsLong(v, x))))
            break;                          // Fall back on using base
    }
}
```

上面的函数fn就是2.6.5节LongAccumulator要用到的，但对于LongAdder而言，fn=null，就是简单的累加操作v+x。

上面的for循环被分成三个大的分支。

在第二个分支里面，进行了Cells[]数组的初始化工作，初始大小为2，然后把x累加在0下标或者1下标对应的Cell上面。

在第一个大的分支里面，完成Cells[]数组的不断扩容，每次扩容都是增长2倍

数组为空，并且有一个线程正在进行初始化工作，于是进入第三个大的分支中，尝试对base变量进行累积，如果再次失败，则会再次进入第一个大的分支

##### 2.6.5 LongAccumulator

LongAccumulator的原理和LongAdder类似，只是功能更强大，下面为两者构造函数的对比：

```java
public LongAdder() {}
public LongAccumulator(LongBinaryOperator accumulatorFunction, long identity)
```

LongAdder只能进行累加操作，并且初始值默认为0；LongAccumulator可以自己定义一个二元操作符，并且可以传入一个初始值。

```java
@FunctionalInterface
public interface LongBinaryOperator {
    long applyAsLong(long left, long right);
}
```

操作符的左值，就是base变量或者Cells[]中元素的当前值；右值，就是add（）函数传入的参数x。
下面是LongAccumulator的accumulate（x）函数，与LongAdder的add （ x ） 函数类似， 最后都是调用的Striped64 的LongAccumulate（..）函数。唯一的差别就是LongAdder的add（x）函数调用的是casBase（b，b+x），这里调用的是casBase（b，r），其中，r=function.applyAsLong（b=base，x）。

```java
public void accumulate(long x) {
    Cell[] as; long b, v, r; int m; Cell a;
    if ((as = cells) != null ||
        (r = function.applyAsLong(b = base, x)) != b && !casBase(b, r)) {
        boolean uncontended = true;
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[getProbe() & m]) == null ||
            !(uncontended =
              (r = function.applyAsLong(v = a.value, x)) == v ||
              a.cas(v, r)))
            longAccumulate(x, function, uncontended);
    }
}
```

##### 2.6.6 DoubleAdder与DoubleAccumulator

DoubleAdder 其实也是用 long 型实现的，因为没有 double 类型的 CAS 函数。DoubleAdder的add（x）函数，和LongAdder的add（x）函数基本一样，只是多了long和double类型的相互转换。

```java
public void add(double x) {
    Cell[] as; long b, v; int m; Cell a;
    if ((as = cells) != null ||
        !casBase(b = base,
                 Double.doubleToRawLongBits
                 (Double.longBitsToDouble(b) + x))) {
        boolean uncontended = true;
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[getProbe() & m]) == null ||
            !(uncontended = a.cas(v = a.value,
                                  Double.doubleToRawLongBits
                                  (Double.longBitsToDouble(v) + x))))
            doubleAccumulate(x, null, uncontended);
    }
}
```

其中的关键Double.doubleToRawLongBits（Double.longBitsToDouble（b）+x），在读出来的时候，它把 long 类型转换成 double 类型，然后进行累加，累加的结果再转换成 long 类型，通过CAS写回去。
DoubleAccumulate也是Striped64的成员函数，和longAccumulate类似，也是多了long类型和double类型的互相转换。
DoubleAccumulator和DoubleAdder的关系，与LongAccumulator和LongAdder的关系类似，只是多了一个二元操作符，此处不再赘述。到此为止，Concurrent包的所有原子类都介绍完了，接下来分析锁的实现。

### 3.Lock与Condition

#### 3.1 互斥锁

##### 3.1.1 锁的可重入性

因为在Concurrent包中的锁都是“可重入锁”，所以一般都命名为ReentrantX，因为所有的锁。“可重入锁”是指当一个线程调用 object.lock（）拿到锁，进入互斥区后，再次调用object.lock（），仍然可以拿到该锁。很显然，通常的锁都要设计成可重入的，否则就会发生死锁。

第2章讲的synchronized关键字，同样是可重入锁

##### 3.1.2 类的继承层次

在正式介绍锁的实现原理之前，先看一下 Concurrent 包中的与互斥锁（ReentrantLock）相关类之间的继承层次，如图3-1所示。在图3-1中，I表示界面（Interface），A表示抽象类（AbstractClass），C表示类（Class），$表示内部类。实线表示继承关系，虚线表示引用关系。本书中所有关于类的继承层次的图片都遵循这个图例。

![image-20210805163324287](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805163324287.png)



Lock是一个接口，其定义如下：

```java
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition();
}
```

常用的方法是lock（）/unlock（）。lock（）不能被中断，对应的lockInterruptibly（）可以被中断。
ReentrantLock本身没有代码逻辑，实现都在其内部类Sync中。

##### 3.1.3 锁的公平性与非公平性

Sync是一个抽象类，它有两个子类FairSync与NonfairSync，分别对应公平锁和非公平锁。从下面的ReentrantLock构造函数可以看出，会传入一个布尔类型的变量fair指定锁是公平的还是非公平的，默认为非公平的。

```java
public ReentrantLock() {
    sync = new NonfairSync();
}
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

公平-排队  非公平- 不排队

这里默认设置的是非公平锁，其实是为了提高效率，减少线程切换。

##### 3.1.4 锁实现的基本原理

Sync的父类AbstractQueuedSynchronizer经常被称作队列同步器（AQS），这个类非常关键，下面会反复提到，该类的父类是AbstractOwnableSynchronizer。

上一章讲的Atomic类都是“自旋”性质的锁，而本章讲的锁将具备synchronized功能，也就是可以阻塞一个线程。为了实现一把具有阻塞或唤醒功能的锁，需要几个核心要素：

1. 需要一个state变量，标记该锁的状态。state变量至少有两个值：0、1。对state变量的操作，要确保线程安全，也就是会用到CAS。

2. 需要记录当前是哪个线程持有锁。

3. 需要底层支持对一个线程进行阻塞或唤醒操作。

4. 需要有一个队列维护所有阻塞的线程。这个队列也必须是线程安全的无锁队列，也需要用到CAS。

针对要素①②，在上面两个类中有对应的体现：

```java
public abstract class AbstractOwnableSynchronizer{
    private transient Thread exclusiveOwnerThread;//记录锁被哪个线程持有
}
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer{
    private volatile int state;//记录锁的状态 cas修改值
}
```

state取值不仅可以是0、1，还可以大于1，就是为了支持锁的可重入性。例如，同样一个线程，调用5次lock，state会变成5；然后调用5次unlock，state减为0。

当state=0时，没有线程持有锁，exclusiveOwnerThread=null；
当state=1时，有一个线程持有锁，exclusiveOwnerThread=该线程；
当state>1时，说明该线程重入了该锁。

针对针对要素③，在Unsafe类中，提供了阻塞或唤醒线程的一对操作原语，也就是park/unpark。

```java
public native void unpark(Object var1);
public native void park(boolean var1, long var2);
```

有一个LockSupport的工具类，对这一对原语做了简单封装：

```java
public static void park() {
    UNSAFE.park(false, 0L);
}
public static void unpark(Thread thread) {
    if (thread != null)
        UNSAFE.unpark(thread);
}
```

在当前线程中调用park（），该线程就会被阻塞；在另外一个线程中，调用unpark（Thread t），传入一个被阻塞的线程，就可以唤醒阻塞在park（）地方的线程。

尤其是 unpark（Thread t），它实现了一个线程对另外一个线程的“精准唤醒”。前面讲到的wait（）/notify（），notify也只是唤醒某一个线程，但无法指定具体唤醒哪个线程。



针对要素④，在AQS中利用双向链表和CAS实现了一个阻塞队列。如下所示。

```java
public abstract class AbstractQueuedSynchronizer{
    static final class Node {
        volatile Node prev;
        volatile Node next;
        volatile Thread thread;
    }
    private transient volatile Node head;
    private transient volatile Node tail;
}
```

阻塞队列是整个AQS核心中的核心，下面做进一步的阐述。如图3-2所示，head指向双向链表头部，tail指向双向链表尾部。入队就是把新的Node加到tail后面，然后对tail进行CAS操作；出队就是对head进行CAS操作，把head向后移一个位置

![image-20210805165618177](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805165618177.png)

初始的时候，head=tail=NULL；然后，在往队列中加入阻塞的线程时，会新建一个空的Node，让head和tail都指向这个空Node；之后，在后面加入被阻塞的线程对象。所以，当head=tail的时候，说明队列为空

##### 3.1.5 公平与非公平的lock（）的实现差异

下面分析基于AQS，ReentrantLock在公平性和非公平性上的实现差异。
![image-20210805165729326](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210805165729326.png)

acquire（）是AQS的一个模板方法，如下所示。

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

tryAcquire（..）是一个虚函数，也就是再次尝试拿锁，被NonfairSync与FairSync分别实现。acquireQueued（..）函数的目的是把线程放入阻塞队列，然后阻塞该线程。

下面再分别来看FairSync与NonfairSync的tryAcquire（1）有什么区别。

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {//无人持锁, 开抢
        if (compareAndSetState(0, acquires)) {//拿锁成功,设置当前线程持有
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {// 拿锁重入, state++
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
//
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        acquire(1);
    }

    /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }

```

这两段代码非常相似，唯一的区别是第二段代码多了一个if （!hasQueuedPredecessors（））。什么意思呢？就是只有当c==0（没有线程持有锁），并且排在队列的第1个时（即当队列中没有其他线程的时候），才去抢锁，否则继续排队，这才叫“公平”！

##### 3.1.6 阻塞队列与唤醒机制

下面进入锁的最为关键的部分，即acquireQueued（..）函数内部一探究竟。

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

先说addWaiter（..） 函数，就是为当前线程生成一个Node，然后把Node放入双向链表的尾部。要注意的是，这只是把Thread对象放入了一个队列中而已，线程本身并未阻塞。

```java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {//尝试加到队列尾部 不成功执行enq(node)
            pred.next = node;
            return node;
        }
    }
    enq(node);//进行队列init. 新建一个node.然后自选,直到把node加到队列尾部
    return node;
}
```

在addWaiter（..）函数把Thread对象加入阻塞队列之后的工作就要靠acquireQueued（..）函数完成。线程一旦进入acquireQueued（..）就会被无限期阻塞，即使有其他线程调用interrupt（）函数也不能将其唤醒，除非有其他线程释放了锁，并且该线程拿到了锁，才会从accquireQueued（..）返回。

注意：进入acquireQueued（..），该线程被阻塞。在该函数返回的一刻，就是拿到锁的那一刻，也就是被唤醒的那一刻，此时会删除队列的第一个元素（head指针前移1个节点）。

```java
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {//被唤醒,如果在队头(自己前一个节点是head指向的空节点),则尝试拿锁
                    setHead(node);//拿锁成功,出队(head指针前移一个)同时把node的thread变量置空,所以head指向空节点
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())//自己调park()阻塞自己
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

首先，acquireQueued（..）函数有一个返回值，表示什么意思呢？虽然该函数不会中断响应，但它会记录被阻塞期间有没有其他线程向它发送过中断信号。如果有，则该函数会返回true；否则，返回false。

基于这个返回值，才有了下面的代码：

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
static void selfInterrupt() {
    Thread.currentThread().interrupt();
}
```

当 acquireQueued（..） 返回 true 时，会调用 selfInterrupt（），自己给自己发送中断信号，也就是自己把自己的中断标志位设为true。之所以要这么做，是因为自己在阻塞期间，收到其他线程中断信号没有及时响应，现在要进行补偿。这样一来，如果该线程在lock代码块内部有调用sleep（）之类的阻塞方法，就可以抛出异常，响应该中断信号。

阻塞就发生在下面这个函数中：

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

线程调用 park（）函数，自己把自己阻塞起来，直到被其他线程唤醒，该函数返回。park（）函数返回有两种情况。
情况1：其他线程调用了unpark（Thread t）。
情况2：其他线程调用了t.interrupt（）。

这里要注意的是，lock（）不能响应中断，但LockSupport.park（）会响应中断。也正因为LockSupport.park（）可能被中断唤醒，acquireQueued（..）函数才写了一个for死循环。唤醒之后，如果发现自己排在队列头部，就去拿锁；如果拿不到锁，则再次自己阻塞自己。不断重复此过程，直到拿到锁。

被唤醒之后，通过Thread.interrupted（）来判断是否被中断唤醒。如果是情况1，会返回false；如果是情况2，则返回true。

##### 3.1.7 unlock（）实现分析

说完了lock，下面分析unlock的实现。unlock不区分公平还是非公平。

```java
public void unlock() {
    sync.release(1);
}

public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != owner)//锁拥有者才能unlock 否则抛异常
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) { //调一次state-1 直到为0 锁释放成功
        free = true;
        owner = null;
    }
    setState(c); //没用cas 直接set 因为排他锁只有一个线程能减state
    return free;
}

private void unparkSuccessor(Node node) {

    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

release（）里面做了两件事：tryRelease（..）函数释放锁；unparkSuccessor（..）函数唤醒队列中的后继者。
在上面的代码中有一个关键点要说明：因为是排他锁，只有已经持有锁的线程才有资格调用 release（..），这意味着没有其他线程与它争抢。所以，在上面的 tryRelease（..）函数中，对 state值的修改，不需要CAS操作，直接减1即可。

但对于读写锁中的读锁，也就是releaseShared（..），就不一样了，见后续分析。

##### 3.1.8 lockInterruptibly（）实现分析

上面的 lock 不能被中断，这里的 lockInterruptibly（）可以被中断，下面看一下两者在实现上有什么差别。

```java
public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);
}
public final void acquireInterruptibly(int arg)
    throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (!tryAcquire(arg))
        doAcquireInterruptibly(arg);
}
```

这里的 acquireInterruptibly（..）也是 AQS 的模板方法，里面的 tryAcquire（..）分别被 FairSync和NonfairSync实现，此处不再重复叙述。这里主要讲doAcquireInterruptibly（..）函数。

```java
private void doAcquireInterruptibly(int arg)
    throws InterruptedException {
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException();//收到信号, 不在阻塞, 直接抛异常
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

明白了accquireQueued（..） 原理，此处就很简单了。当parkAndCheckInterrupt（）返回true的时候，说明有其他线程发送中断信号，直接抛出InterruptedException，跳出for循环，整个函数返回。

##### 3.1.9 tryLock（）实现分析

```java
public boolean tryLock() {
    return sync.nonfairTryAcquire(1);
}
```

tryLock（）实现基于调用非公平锁的tryAcquire（..），对state进行CAS操作，如果操作成功就拿到锁；如果操作不成功则直接返回false，也不阻塞。

#### 3.2 读写锁

##### 3.2.1 类继承层次

如图3-3所示，ReadWriteLock是一个接口，内部由两个Lock接口组成。

```java
public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}
```

![image-20210806092038583](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210806092038583.png)

ReentrantReadWriteLock实现了该接口，使用方式如下：

![image-20210806092115841](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210806092115841.png)

也就是说，当使用 ReadWriteLock 的时候，并不是直接使用，而是获得其内部的读锁和写锁，然后分别调用lock/unlock。

##### 3.2.2 读写锁实现的基本原理

从表面来看，ReadLock和WriteLock是两把锁，实际上它只是同一把锁的两个视图而已。什么叫两个视图呢？可以理解为是一把锁，线程分成两类：读线程和写线程。读线程和读线程之间不互斥（可以同时拿到这把锁），读线程和写线程互斥，写线程和写线程也互斥。

从下面的构造函数也可以看出，readerLock和writerLock实际共用同一个sync对象。sync对象同互斥锁一样，分为非公平和公平两种策略，并继承自AQS。

```java
 public ReentrantReadWriteLock() {
        this(false);
    }
 public ReentrantReadWriteLock(boolean fair) {
     sync = fair ? new FairSync() : new NonfairSync();
     readerLock = new ReadLock(this);
     writerLock = new WriteLock(this);
 }
```

同互斥锁一样，读写锁也是用state变量来表示锁状态的。只是state变量在这里的含义和互斥锁完全不同。在内部类Sync中，对state变量进行了重新定义，如下所示。

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
    static final int SHARED_SHIFT   = 16;
    static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
    static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
    static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
    //持有读锁的线程的重入次数
    static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
    //持有写锁的线程的重入次数
    static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
}
```

也就是把 state 变量拆成两半，低16位，用来记录写锁。但同一时间既然只能有一个线程写，为什么还需要16位呢？这是因为一个写线程可能多次重入。例如，低16位的值等于5，表示一个写线程重入了5次。

高16位，用来“读”锁。例如，高16位的值等于5，可以表示5个读线程都拿到了该锁；也可以表示一个读线程重入了5次。

这个地方的设计很巧妙，为什么要把一个int类型变量拆成两半，而不是用两个int型变量分别表示读锁和写锁的状态呢？这是因为无法用一次CAS 同时操作两个int变量，所以用了一个int型的高16位和低16位分别表示读锁和写锁的状态。

当state=0时，说明既没有线程持有读锁，也没有线程持有写锁；当state！=0时，要么有线程持有读锁，要么有线程持有写锁，两者不能同时成立，因为读和写互斥。这时再进一步通过sharedCount（state）和exclusiveCount（state）判断到底是读线程还是写线程持有了该锁。

##### 3.2.3 AQS的两对模板方法

下面介绍在ReentrantReadWriteLock的两个内部类ReadLock和WriteLock中，是如何使用state变量的。

```java
public static class ReadLock implements Lock, java.io.Serializable {
    public void lock() {
        sync.acquireShared(1);
    }
    public void unlock() {
        sync.releaseShared(1);
    }
}
public static class WriteLock implements Lock, java.io.Serializable {
    public void lock() {
        sync.acquire(1);
    }
    public void unlock() {
        sync.release(1);
    }
}
```

acquire/release、acquireShared/releaseShared 是AQS里面的两对模板方法。互斥锁和读写锁的写锁都是基于acquire/release模板方法来实现的。读写锁的读锁是基于acquireShared/releaseShared这对模板方法来实现的。这两对模板方法的代码如下：

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer
    implements java.io.Serializable {

    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&         //tryAcquire被各种Sync子类实现
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
    public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)   //tryAcquireShared被各种Sync子类实现
            doAcquireShared(arg);
    }
    public final boolean release(int arg) {
        if (tryRelease(arg)) {           //tryRelease被各种Sync子类实现
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
    public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {       //tryReleaseShared被各种Sync子类实现
            doReleaseShared();
            return true;
        }
        return false;
    }
}
```

将读/写、公平/非公平进行排列组合，就有4种组合。如图3-4所示，上面的两个函数都是在Sync中实现的。Sync中的两个函数又是模板方法，在NonfairSync和FairSync中分别有实现。最终的对应关系如下：

1. 读锁的公平实现：Sync.tryAccquireShared（）+FairSync中的两个覆写的子函数。
2. 读锁的非公平实现：Sync.tryAccquireShared（）+NonfairSync中的两个覆写的子函数。
3. 写锁的公平实现：Sync.tryAccquire（）+FairSync中的两个覆写的子函数。
4. 写锁的非公平实现：Sync.tryAccquire（）+NonfairSync中的两个覆写的子函数。.

![image-20210806101125478](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210806101125478.png)

```java
static final class NonfairSync extends Sync {
    final boolean writerShouldBlock() {//写线程抢锁应该阻塞吗
        return false; // writers can always barge 不阻塞, 非公平
    }
    final boolean readerShouldBlock() {//读阻塞吗?
        return apparentlyFirstQueuedIsExclusive();//如果队列中第一个元素是写线程,阻塞
    }
}
static final class FairSync extends Sync {
    final boolean writerShouldBlock() {
        return hasQueuedPredecessors();//写线程抢锁前,如果队列中有其他线程排队,阻塞,公平
    }
    final boolean readerShouldBlock() {
        return hasQueuedPredecessors();//读线程抢锁前,如果队列中有其他线程排队,阻塞,公平
    }
}
```

上面的代码介绍了ReentrantReadWriteLock里面的NonfairSync和FairSync的实现过程，对应了上面的四种实现策略，下面分别解释。

对于公平，比较容易理解，不论是读锁，还是写锁，只要队列中有其他线程在排队（排队等读锁，或者排队等写锁），就不能直接去抢锁，要排在队列尾部。

对于非公平，读锁和写锁的实现策略略有差异。先说写锁，写线程能抢锁，前提是state=0，只有在没有其他线程持有读锁或写锁的情况下，它才有机会去抢锁。或者state！=0，但那个持有写锁的线程是它自己，再次重入。写线程是非公平的，就是不管三七二十一就去抢，即一直返回false。

但对于读线程，能否也不管三七二十一，上来就去抢呢？不行！
因为读线程和读线程是不互斥的，假设当前线程被读线程持有，然后其他读线程还非公平地一直去抢，可能导致写线程永远拿不到锁，所以对于读线程的非公平，要做一些“约束”。当发现队列的第1个元素是写线程的时候，读线程也要阻塞一下，不能“肆无忌惮”地直接去抢。

明白策略后，下面具体介绍四种实现方面的差异。

##### 3.2.4 WriteLock公平与非公平实现

写锁是排他锁，实现策略类似于互斥锁，重写了tryAcquire/tryRelease方法。

###### 1.tryAcquire()实现分析

```java
protected final boolean tryAcquire(int acquires) {
            /*
             * Walkthrough:
             * 1. If read count nonzero or write count nonzero
             *    and owner is a different thread, fail.
             * 2. If count would saturate, fail. (This can only
             *    happen if count is already nonzero.)
             * 3. Otherwise, this thread is eligible for lock if
             *    it is either a reentrant acquire or
             *    queue policy allows it. If so, update state
             *    and set owner.
             */
            Thread current = Thread.currentThread();
            int c = getState();
            int w = exclusiveCount(c);//写线程只能有一个,写线程可多次重入
            if (c != 0) {             //c!=0说明读或写线程持有锁
                // (Note: if c != 0 and w == 0 then shared count != 0)
                if (w == 0 || current != getExclusiveOwnerThread())//w==0,说明读持有锁,返回,持有不是自己,返回
                    return false;
                if (w + exclusiveCount(acquires) > MAX_COUNT)//16位用满了,超过最大重入次数
                    throw new Error("Maximum lock count exceeded");
                // Reentrant acquire
                setState(c + acquires);
                return true;
            }
            if (writerShouldBlock() ||     //writerShouldBlock是四种不同的实现策略
                !compareAndSetState(c, c + acquires))
                return false;
            setExclusiveOwnerThread(current);
            return true;
        }
```

把上面的代码拆开进行分析，如下：

1. if （c！=0） and w==0，说明当前一定是读线程拿着锁，写锁一定拿不到，返回false。
2. if （c！=0） and w！=0，说明当前一定是写线程拿着锁，执行current！=getExclusive-OwnerThread（）的判断，发现ownerThread不是自己，返回false。
3.  c !=0, w ! =0, 且current=getExclusiveOwnerThread()，才会走到if (w+exclusive-Count(acquires))>MAX_COUN
   T）。判断重入次数，重入次数超过最大值，抛出异常。
   因为是用state的低16位保存写锁重入次数的，所以MAX_COUNT是2的16次方。如果超出这个值，会写到读锁的高16位上。为了避免这种情形，这里做了一个检测。当然，一般不可能重入这么多次。
4. if（c=0），说明当前既没有读线程，也没有写线程持有该锁。可以通过CAS操作开抢了。

```java
if (writerShouldBlock() || !compareAndSetState(c, c + acquires))
```

抢成功后，调用setExclusiveOwnerThread（current），把ownerThread设成自己。
公平实现和非公平实现几乎一模一样，只是 writerShouldBlock（）分别被 FairSync 和NonfairSync实现，在上一节已讲。

###### 2.tryRelease()实现分析

```java
  protected final boolean tryRelease(int releases) {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            int nextc = getState() - releases;
            boolean free = exclusiveCount(nextc) == 0;
            if (free)
                setExclusiveOwnerThread(null);
            setState(nextc);//写锁排他,当前线程持有写锁,state直接减一
            return free;
        }
```

##### 3.2.5 ReadLock公平与非公平实现

读锁是共享锁，重写了 tryAcquireShared/tryReleaseShared 方法，其实现策略和排他锁有很大的差异。

###### 1.tryAcquireShared()实现分析

```java
 protected final int tryAcquireShared(int unused) {
            /*
             * Walkthrough:
             * 1. If write lock held by another thread, fail.
             * 2. Otherwise, this thread is eligible for
             *    lock wrt state, so ask if it should block
             *    because of queue policy. If not, try
             *    to grant by CASing state and updating count.
             *    Note that step does not check for reentrant
             *    acquires, which is postponed to full version
             *    to avoid having to check hold count in
             *    the more typical non-reentrant case.
             * 3. If step 2 fails either because thread
             *    apparently not eligible or CAS fails or count
             *    saturated, chain to version with full retry loop.
             */
            Thread current = Thread.currentThread();
            int c = getState();
            if (exclusiveCount(c) != 0 &&  //写锁被持有, 还不是自己,读锁拿不到,直接返回
                getExclusiveOwnerThread() != current)
                return -1;
            int r = sharedCount(c);
            if (!readerShouldBlock() &&  //公平非公平差异在这个函数
                r < MAX_COUNT &&
                compareAndSetState(c, c + SHARED_UNIT)) { //cas拿锁 ,高16位加一
                if (r == 0) {   //r之前等于0,说明第一个拿到读锁的线程
                    firstReader = current;
                    firstReaderHoldCount = 1;
                } else if (firstReader == current) {  //不是第一个
                    firstReaderHoldCount++;
                } else {
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;
                }
                return 1;
            }
            return fullTryAcquireShared(current);//上面拿读锁失败,这个函数自旋拿锁
        }
```

下面是关于此代码的解释：
1. 

```java
 if (exclusiveCount(c) != 0 &&  //写锁被持有, 还不是自己,读锁拿不到,直接返回
                getExclusiveOwnerThread() != current)
                return -1;
```

低16位不等于0，说明有写线程持有锁，并且只有当ownerThread！=自己时，才返回-1。这里面有一个潜台词：如果current=ownerThread，则这段代码不会返回。==这是因为一个写线程可以再次去拿读锁！也就是说，一个线程在持有了WriteLock后，再去调用ReadLock.lock也是可以的。==

（2）上面的compareAndSetState（c，c+SHARED_UNIT），其实是把state的高16位加1（读锁的状态），但因为是在高16位，必须把1左移16位再加1。
（3）firstReader，cachedHoldConunter 之类的变量，只是一些统计变量，在 ReentrantRead-WriteLock对外的一些查询函数中会用到，例如，查询持有读锁的线程列表，但对整个读写互斥机制没有影响，此处不再展开解释。

###### 2.tryReleaseShared()实现分析

```java
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    if (firstReader == current) {
        // assert firstReaderHoldCount > 0;
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    for (;;) {
        int c = getState();
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            // Releasing the read lock has no effect on readers,
            // but it may allow waiting writers to proceed if
            // both read and write locks are now free.
            return nextc == 0;
    }
}
```

因为读锁是共享锁，多个线程会同时持有读锁，所以对读锁的释放不能直接减1，而是需要通过一个for循环+CAS操作不断重试。这是tryReleaseShared和tryRelease的根本差异所在。

#### 3.3 Condition

##### 3.3.1 Condition与Lock的关系

Condition本身也是一个接口，其功能和wait/notify类似，如下所示。

```java
public interface Condition {
    void await() throws InterruptedException;
    void awaitUninterruptibly();
    void signal();
    void signalAll();
}
```

在讲多线程基础的时候，强调wait（）/notify（）必须和synchronized一起使用，Condition也是如此，必须和Lock一起使用。因此，在Lock的接口中，有一个与Condition相关的接口：

```java
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition(); //所有的Condition都是从Lock中构造出来的
}
```

##### 3.3.2 Condition的使用场景

在讲Condition的实现原理之前，先以ArrayBlockingQueue的实现为例，介绍Condition的使用场景。如下所示为一个用数组实现的阻塞队列，执行put（..）操作的时候，队列满了，生成者线程被阻塞；执行take（）操作的时候，队列为空，消费者线程被阻塞。









```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
    implements BlockingQueue<E>, java.io.Serializable {

    /** The queued items */
    final Object[] items;
    /** items index for next take, poll, peek or remove */
    int takeIndex;
    /** items index for next put, offer, or add */
    int putIndex;
    /** Number of elements in the queue */
    int count;

    //核心是一把锁+两个条件
    /** Main lock guarding all access */
    final ReentrantLock lock;
    /** Condition for waiting takes */
    private final Condition notEmpty;
    /** Condition for waiting puts */
    private final Condition notFull;

    public ArrayBlockingQueue(int capacity, boolean fair) {
        if (capacity <= 0)
            throw new IllegalArgumentException();
        this.items = new Object[capacity];
        lock = new ReentrantLock(fair);  //构造函数,一把锁两个Condition
        notEmpty = lock.newCondition();
        notFull =  lock.newCondition();
    }

    public void put(E e) throws InterruptedException {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length)
                notFull.await();//put的时候,队列满了,阻塞于非满条件
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }
    private void enqueue(E x) {
        // assert lock.getHoldCount() == 1;
        // assert items[putIndex] == null;
        final Object[] items = this.items;
        items[putIndex] = x;
        if (++putIndex == items.length)
            putIndex = 0;
        count++;
        notEmpty.signal(); //put进去后,通知非空条件
    }
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();//take的时候,队列为空,阻塞在非空条件
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
    private E dequeue() {
        // assert lock.getHoldCount() == 1;
        // assert items[takeIndex] != null;
        final Object[] items = this.items;
        @SuppressWarnings("unchecked")
        E x = (E) items[takeIndex];
        items[takeIndex] = null;
        if (++takeIndex == items.length)
            takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeued();
        notFull.signal();//操作完成,通知非满条件
        return x;
    }
```

##### 3.3.3 Condition实现原理

可以发现Condition的使用很简洁，避免了 wait/notify 的生成者通知生成者、消费者通知消费者的问题，这是如何做到的呢？下面进入Condition内部一探究竟。
因为Condition必须和Lock一起使用，所以Condition的实现也是Lock的一部分。下面先分别看一下互斥锁和读写锁中Condition的构造。

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    public Condition newCondition() {
        return sync.newCondition();
    }
}
public class ReentrantReadWriteLock
    implements ReadWriteLock, java.io.Serializable {
    public static class ReadLock implements Lock, java.io.Serializable {
        public Condition newCondition() {
            throw new UnsupportedOperationException();//读锁不支持Condition
        }
    }
}
public static class WriteLock implements Lock, java.io.Serializable {
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```

首先，==读写锁中的 ReadLock 是不支持 Condition 的，读写锁的写锁和互斥锁都支持Condition。==虽然它们各自调用的是自己的内部类Sync，但内部类Sync都继承自AQS。因此，上面的代码sync.newCondition最终都调用了AQS中的newCondition。

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
      public class ConditionObject implements Condition, java.io.Serializable {
          //Condition的所有实现,都在ConditionObject里
      }
}
```

每一个Condition对象上面，都阻塞了多个线程。因此，在ConditionObject内部也有一个双向链表组成的队列，如下所示。

```java
  public class ConditionObject implements Condition, java.io.Serializable {
        /** First node of condition queue. */
        private transient Node firstWaiter;
        /** Last node of condition queue. */
        private transient Node lastWaiter;
  }
```

下面来看一下在await（）/notify（）函数中，是如何使用这个队列的。

##### 3.3.4 await（）实现分析

```java
public final void await() throws InterruptedException {
    if (Thread.interrupted())//收到中断信号抛异常
        throw new InterruptedException();
    Node node = addConditionWaiter();//加入Condition的等待队列
    int savedState = fullyRelease(node);//关键: 阻塞在Condition前释放锁, 否则死锁
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);//自己阻塞自己
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)//重新拿锁
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);//被中断唤醒,向外抛出中断异常
}
```

关于await，有几个关键点要说明：
（1）线程调用 await（）的时候，肯定已经先拿到了锁。所以，在 addConditionWaiter（）内部，对这个双向链表的操作不需要执行CAS操作，线程天生是安全的，代码如下。

```java
private Node addConditionWaiter() {
    Node t = lastWaiter;
    // If lastWaiter is cancelled, clean out.
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters();
        t = lastWaiter;
    }
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;
    return node;
}
```

（2）在线程执行wait操作之前，必须先释放锁。也就是fullyRelease（node），否则会发生死锁。这个和wait/notify与synchronized的配合机制一样。
（3）线程从wait中被唤醒后，必须用acquireQueued（node，savedState）函数重新拿锁。
（4）checkInterruptWhileWaiting（node）代码在park（this）代码之后，是为了检测在park期间是否收到过中断信号。当线程从park中醒来时，有两种可能：一种是其他线程调用了unpark，另一种是收到中断信号。这里的await（）函数是可以响应中断的，所以当发现自己是被中断唤醒的，而不是被unpark唤醒的时，会直接退出while循环，await（）函数也会返回。
（5）isOnSyncQueue（node）用于判断该Node是否在AQS的同步队列里面。初始的时候，Node只在Condition的队列里，而不在AQS的队列里。但执行notity操作的时候，会放进AQS的同步队列。

##### 3.3.5 awaitUninterruptibly（）实现分析

与await（）不同，awaitUninterruptibly（）不会响应中断，其函数的定义中不会有中断异常抛出，下面分析其实现和await（）的区别。

```java
public final void awaitUninterruptibly() {
    Node node = addConditionWaiter();
    int savedState = fullyRelease(node);
    boolean interrupted = false;
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        if (Thread.interrupted())//从park中醒来,收到中断,不退出,继续执行while
            interrupted = true;
    }
    if (acquireQueued(node, savedState) || interrupted)
        selfInterrupt();
}
```

可以看出，整体代码和 await（）类似，区别在于收到异常后，不会抛出异常，而是继续执行while循环。

##### 3.3.6 notify（）实现分析

```java
public final void signal() {
    if (!isHeldExclusively()) //只有有锁,才能调用signal()
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}
private void doSignal(Node first) { //唤醒队列中第一个线程
    do {
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
        first.nextWaiter = null;
    } while (!transferForSignal(first) &&
             (first = firstWaiter) != null);
}
final boolean transferForSignal(Node node) {
        /*
         * If cannot change waitStatus, the node has been cancelled.
         */
        if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;

        /*
         * Splice onto queue and try to set waitStatus of predecessor to
         * indicate that thread is (probably) waiting. If cancelled or
         * attempt to set waitStatus fails, wake up to resync (in which
         * case the waitStatus can be transiently and harmlessly wrong).
         */
        Node p = enq(node);//关键:把node放入互斥锁的同步队列,在调用unpark
        int ws = p.waitStatus;
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
            LockSupport.unpark(node.thread);
        return true;
    }
```

同 await（）一样，在调用 notify（）的时候，必须先拿到锁（否则就会抛出上面的异常），是因为前面执行await（）的时候，把锁释放了。然后，从队列中取出firstWait，唤醒它。在通过调用unpark唤醒它之前，先用enq（node）函数把这个Node放入AQS的锁对应的阻塞队列中。也正因为如此，才有了await（）函数里面的判断条件while（！isOnSyncQueue（node）），这个判断条件被满足，说明await线程不是被中断，而是被unpark唤醒的。
知道了notify（）实现原理，notifyAll（）与此类似，此处不再赘述。

#### 3.4 StampedLock

##### 3.4.1 为什么引入StampedLock

在JDK 8中新增了StampedLock，有了读写锁，为什么还要引入StampedLock呢？来看一下表3-1的对比。

![image-20210806141246505](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210806141246505.png)

可以看到，从ReentrantLock到StampedLock，并发度依次提高。StampedLock是如何做到“读”与“写”也不互斥、并发地访问的呢？

在《软件架构设计：大型网站技术架构与业务架构融合之道》中，谈到 MySQL 高并发的核心机制 MVCC，也就是一份数据多个版本，此处的StampedLock有异曲同工之妙。

另一方面，因为ReentrantLock采用的是“悲观读”的策略，当第一个读线程拿到锁之后，第二个、第三个读线程还可以拿到锁，使得写线程一直拿不到锁，可能导致写线程“饿死”。虽然在其公平或非公平的实现中，都尽量避免这种情形，但还有可能发生。

StampedLock引入了“乐观读”策略，读的时候不加读锁，读出来发现数据被修改了，再升级为“悲观读”，相当于降低了“读”的地位，把抢锁的天平往“写”的一方倾斜了一下，避免写线程被饿死。

##### 3.4.2 使用场景

在剖析其原理之前，下面先以官方的一个例子来看一下StampedLock如何使用。

```java
public class StampedLockTest {
    private double x, y;
    private final StampedLock stampedLock = new StampedLock();
    void move(double x, double y) {
        long stamp = stampedLock.writeLock();
        try {
            this.x += x;
            this.y += y;
        } finally {
            stampedLock.unlockWrite(stamp);
        }
    }
    double distanceFromOrigin() {//多线程调用 求距离
        long l = stampedLock.tryOptimisticRead();//乐观读
        double currentX = x, currentY = y;//将共享变量拷贝到线程栈
        if (!stampedLock.validate(l)) {//读期间有其他线程修改数据
            l = stampedLock.readLock();//脏读, 升级为悲观锁
            try {
                currentX += x;
                currentY += y;
            } finally {
                stampedLock.unlockRead(l);
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }
}
```

如上面代码所示，有一个Point类，多个线程调用move（）函数，修改坐标；还有多个线程调用 distanceFromOrigin（）函数，求距离。

首先，执行move操作的时候，要加写锁。这个用法和ReadWriteLock的用法没有区别，写操作和写操作也是互斥的。关键在于读的时候，用了一个“乐观读”sl.tryOptimisticRead（），相当于在读之前给数据的状态做了一个“快照”。然后，把数据拷贝到内存里面，在用之前，再比对一次版本号。如果版本号变了，则说明在读的期间有其他线程修改了数据。读出来的数据废弃，重新获取读锁。关键代码就是下面这三行：

```java
long l = stampedLock.tryOptimisticRead();//乐观读 读之前获取版本号
double currentX = x, currentY = y;//将共享变量拷贝到线程栈
if (!stampedLock.validate(l)) {//读之后, 对比两次版本号, 判断有没有修改
```

要说明的是，这三行关键代码对顺序非常敏感，不能有重排序。因为 state 变量已经是volatile，所以可以禁止重排序，但l并不是volatile的。为此，在validate（l）函数里面插入内存屏障。

##### 3.4.3 “乐观读”的实现原理

首先，StampedLock是一个读写锁，因此也会像读写锁那样，把一个state变量分成两半，分别表示读锁和写锁的状态。同时，它还需要一个数据的version。但正如前面所说，一次CAS没有办法操作两个变量，所以这个state变量本身同时也表示了数据的version。下面先分析state变量。

```java
public class StampedLock implements java.io.Serializable {
    private static final int LG_READERS = 7;
    // Values for lock state and stamp operations
    private static final long RUNIT = 1L;
    private static final long WBIT  = 1L << LG_READERS;//第8位表示写锁
    private static final long RBITS = WBIT - 1L;//最低七位表示读锁
    private static final long RFULL = RBITS - 1L;//读锁数
    private static final long ABITS = RBITS | WBIT;//读写状态加到一起
    private static final long SBITS = ~RBITS; // note overlap with ABITS

    private static final long ORIGIN = WBIT << 1;
    private transient volatile long state;//state初始值
}
```

结合代码和图3-5：用最低的8位表示读和写的状态，其中第8位表示写锁的状态，最低的7位表示读锁的状态。因为写锁只有一个bit位，所以写锁是不可重入的。

![image-20210806144129498](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210806144129498.png)

初始值不为0，而是把WBIT 向左移动了一位，也就是上面的ORIGIN 常量，构造函数如下所示。

```java
   public StampedLock() {
        state = ORIGIN;
    }
```

为什么state的初始值不设为0呢？这就要从乐观锁的实现说起。

```java
public long tryOptimisticRead() {
    long s;
    return (((s = state) & WBIT) == 0L) ? (s & SBITS) : 0L;
}
public boolean validate(long stamp) {
    U.loadFence();
    return (stamp & SBITS) == (state & SBITS);//当stamp=0,validate返回false
}
```

上面两个函数必须结合起来看：当state&WBIT！=0的时候，说明有线程持有写锁，上面的tryOptimisticRead会永远返回0。这样，再调用validate（stamp），也就是validate（0）也会永远返回false。

这正是我们想要的逻辑：当有线程持有写锁的时候，validate永远返回false，无论写线程是否释放了写锁。因为无论是否释放了（state回到初始值）写锁，state值都不为0，所以validate（0）永远为false。

为什么上面的validate（..）函数不直接比较stamp=state，而要比较state&SBITS=state&SBITS 呢？因为读锁和读锁是不互斥的！所以，即使在“乐观读”的时候，state 值被修改了，但如果它改的是第7位，validate（..）还是会返回true。

另外要说明的一点是，上面使用了内存屏障 U.loadFence（），是因为在这行代码的下一行里面的stamp、SBITS变量不是volatile的，由此可以禁止其和前面的currentX=X，currentY=Y进行重排序。通过上面的分析，可以发现state的设计非常巧妙。只通过一个变量，既实现了读锁、写锁的状态记录，还实现了数据的版本号的记录。

##### 3.4.4 悲观读/写：“阻塞”与“自旋”策略实现差异

同ReadWriteLock一样，StampedLock也要进行悲观的读锁和写锁操作。不过，它不是基于AQS实现的，而是内部重新实现了一个阻塞队列。如下所示。

```java
public class StampedLock implements java.io.Serializable { 
    /** Wait nodes */
    static final class WNode {
        volatile WNode prev;
        volatile WNode next;
        volatile WNode cowait;    // list of linked readers
        volatile Thread thread;   // non-null while possibly parked
        volatile int status;      // 取值 0, WAITING, or CANCELLED
        final int mode;           // 取值 RMODE or WMODE
        WNode(int m, WNode p) { mode = m; prev = p; }
    }
    /** Head of CLH queue */
    private transient volatile WNode whead;
    /** Tail (last) of CLH queue */
    private transient volatile WNode wtail;
}
```

这个阻塞队列和 AQS 里面的很像。刚开始的时候，whead=wtail=NULL，然后初始化，建一个空节点，whead和wtail都指向这个空节点，之后往里面加入一个个读线程或写线程节点。但基于这个阻塞队列实现的锁的调度策略和AQS很不一样，也就是“自旋”。在AQS里面，当一个线程CAS state失败之后，会立即加入阻塞队列，并且进入阻塞状态。但在StampedLock中，CAS state失败之后，会不断自旋，自旋足够多的次数之后，如果还拿不到锁，才进入阻塞状态。为此，根据CPU的核数，定义了自旋次数的常量值。如果是单核的CPU，肯定不能自旋，在多核情况下，才采用自旋策略。

```java
private static final int NCPU = Runtime.getRuntime().availableProcessors();
/** Maximum number of retries before enqueuing on acquisition */
private static final int SPINS = (NCPU > 1) ? 1 << 6 : 0;
```

下面以写锁的加锁，也就是StampedLock的writeLock（）函数为例，来看一下自旋的实现。

```java
public long writeLock() {
    long s, next;  // bypass acquireWrite in fully unlocked case only
    return ((((s = state) & ABITS) == 0L &&
             U.compareAndSwapLong(this, STATE, s, next = s + WBIT)) ?
            next : acquireWrite(false, 0L));
}
```

如上面代码所示，当state&ABITS==0的时候，说明既没有线程持有读锁，也没有线程持有写锁，此时当前线程才有资格通过CAS操作state。若操作不成功，则调用acquireWrite（）函数进入阻塞队列，并进行自旋，这个函数是整个加锁操作的核心，代码如下。

```java
 private long acquireWrite(boolean interruptible, long deadline) {
        WNode node = null, p;
        for (int spins = -1;;) { // spin while enqueuing 入队时自旋
            long m, s, ns;
            if ((m = (s = state) & ABITS) == 0L) {
                if (U.compareAndSwapLong(this, STATE, s, ns = s + WBIT))
                    return ns;  //自旋拿到锁,函数返回
            }
            else if (spins < 0)
                spins = (m == WBIT && wtail == whead) ? SPINS : 0;
            else if (spins > 0) {
                if (LockSupport.nextSecondarySeed() >= 0)
                    --spins;//不断自旋,一一定概率把spins值累减
            }
            else if ((p = wtail) == null) { // initialize queue 初始化队列
                WNode hd = new WNode(WMODE, null);
                if (U.compareAndSwapObject(this, WHEAD, null, hd))
                    wtail = hd;
            }
            else if (node == null)
                node = new WNode(WMODE, p);
            else if (node.prev != p)
                node.prev = p;
            else if (U.compareAndSwapObject(this, WTAIL, p, node)) {
                p.next = node;
                break;//cas tail成功 退出循环
            }
        }

        for (int spins = -1;;) {
            WNode h, np, pp; int ps;
            if ((h = whead) == p) {
                if (spins < 0)
                    spins = HEAD_SPINS;
                else if (spins < MAX_HEAD_SPINS)
                    spins <<= 1;
                for (int k = spins;;) { // spin at head
                    long s, ns;
                    if (((s = state) & ABITS) == 0L) {//再次尝试拿锁
                        if (U.compareAndSwapLong(this, STATE, s,
                                                 ns = s + WBIT)) {
                            whead = node;
                            node.prev = null;
                            return ns;
                        }
                    }
                    else if (LockSupport.nextSecondarySeed() >= 0 &&
                             --k <= 0)//不断自旋
                        break;
                }
            }
            else if (h != null) { // help release stale waiters
                WNode c; Thread w;
                while ((c = h.cowait) != null) {//自己从阻塞中唤醒,然后唤醒cowait中所有reader线程
                    if (U.compareAndSwapObject(h, WCOWAIT, c, c.cowait) &&
                        (w = c.thread) != null)
                        U.unpark(w);
                }
            }
            if (whead == h) {
                if ((np = node.prev) != p) {
                    if (np != null)
                        (p = np).next = node;   // stale
                }
                else if ((ps = p.status) == 0)
                    U.compareAndSwapInt(p, WSTATUS, 0, WAITING);
                else if (ps == CANCELLED) {
                    if ((pp = p.prev) != null) {
                        node.prev = pp;
                        pp.next = node;
                    }
                }
                else {
                    long time; // 0 argument to park means no timeout
                    if (deadline == 0L)
                        time = 0L;
                    else if ((time = deadline - System.nanoTime()) <= 0L)
                        return cancelWaiter(node, node, false);
                    Thread wt = Thread.currentThread();
                    U.putObject(wt, PARKBLOCKER, this);
                    node.thread = wt;
                    if (p.status < 0 && (p != h || (state & ABITS) != 0L) &&
                        whead == h && node.prev == p)
                        U.park(false, time);  // emulate LockSupport.park 进入阻塞,之后被另一个线程release唤醒,接着往下执行
                    node.thread = null;
                    U.putObject(wt, PARKBLOCKER, null);
                    if (interruptible && Thread.interrupted())
                        return cancelWaiter(node, node, true);
                }
            }
        }
    }
```

整个acquireWrite（..）函数是两个大的for循环，内部实现了非常复杂的自旋策略。在第一个大的for循环里面，目的就是把该Node加入队列的尾部，一边加入，一边通过CAS操作尝试获得锁。如果获得了，整个函数就会返回；如果不能获得锁，会一直自旋，直到加入队列尾部。

在第二个大的for循环里，也就是该Node已经在队列尾部了。这个时候，如果发现自己刚好也在队列头部，说明队列中除了空的Head节点，就是当前线程了。此时，再进行新一轮的自旋，直到达到MAX_HEAD_SPINS次数，然后进入阻塞。这里有一个关键点要说明：当release（..）函数被调用之后，会唤醒队列头部的第1个元素，此时会执行第二个大的for循环里面的逻辑，也就是接着for循环里面park（）函数后面的代码往下执行。

另外一个不同于AQS的阻塞队列的地方是，在每个WNode里面有一个cowait指针，用于串联起所有的读线程。例如，队列尾部阻塞的是一个读线程 1，现在又来了读线程 2、3，那么会通过cowait指针，把1、2、3串联起来。1被唤醒之后，2、3也随之一起被唤醒，因为读和读之间不互斥。

明白加锁的自旋策略后，下面来看锁的释放操作。和读写锁的实现类似，也是做了两件事情：一是把state变量置回原位，二是唤醒阻塞队列中的第一个节点。节点被唤醒之后，会继续执行上面的第二个大的for循环，自旋拿锁。如果成功拿到，则出队列；如果拿不到，则再次进入阻塞，等待下一次被唤醒。

```java
public void unlockWrite(long stamp) {
    WNode h;
    if (state != stamp || (stamp & WBIT) == 0L)
        throw new IllegalMonitorStateException();
    state = (stamp += WBIT) == 0L ? ORIGIN : stamp;
    if ((h = whead) != null && h.status != 0)
        release(h);
}
private void release(WNode h) {
    if (h != null) {
        WNode q; Thread w;
        U.compareAndSwapInt(h, WSTATUS, WAITING, 0);
        if ((q = h.next) == null || q.status == CANCELLED) {
            for (WNode t = wtail; t != null && t != h; t = t.prev)
                if (t.status <= 0)
                    q = t;
        }
        if (q != null && (w = q.thread) != null)
            U.unpark(w);
    }
}
```

#### 3.5锁优化

除了偏向锁，JVM实现锁的方式都用了循环 CAS，即当一个线程想进入同步块的时候使用循环CAS的方式来获取锁，当它退出同步块的时 候使用循环CAS释放锁

##### 减少锁的时间

不需要同步执行的代码，能不放在同步快里面执行就不要放在同步快内，可以让锁尽快释放；

##### 减少锁的粒度

**它的思想是将物理上的一个锁，拆成逻辑上的多个锁，增加并行度，从而降低锁竞争**。它的思想也是用空间来换时间；

java中很多数据结构都是采用这种方法提高并发操作的效率：

##### ConcurrentHashMap

java中的ConcurrentHashMap在jdk1.8之前的版本，使用一个Segment 数组

>  Segment< K,V >[] segments

Segment继承自ReenTrantLock，所以每个Segment就是个可重入锁，每个Segment 有一个HashEntry< K,V >数组用来存放数据，put操作时，先确定往哪个Segment放数据，只需要锁定这个Segment，执行put，其它的Segment不会被锁定；所以数组中有多少个Segment就允许同一时刻多少个线程存放数据，这样增加了并发能力。

##### LongAdder

LongAdder 实现思路也类似ConcurrentHashMap，LongAdder有一个根据当前并发状况动态改变的Cell数组，Cell对象里面有一个long类型的value用来存储值;
开始没有并发争用的时候或者是cells数组正在初始化的时候，会使用cas来将值累加到成员变量的base上，在并发争用的情况下，LongAdder会初始化cells数组，在Cell数组中选定一个Cell加锁，数组有多少个cell，就允许同时有多少线程进行修改，最后将数组中每个Cell中的value相加，在加上base的值，就是最终的值；cell数组还能根据当前线程争用情况进行扩容，初始长度为2，每次扩容会增长一倍，直到扩容到大于等于cpu数量就不再扩容，这也就是为什么LongAdder比cas和AtomicInteger效率要高的原因，后面两者都是volatile+cas实现的，他们的竞争维度是1，LongAdder的竞争维度为“Cell个数+1”为什么要+1？因为它还有一个base，如果竞争不到锁还会尝试将数值加到base上；

##### LinkedBlockingQueue

LinkedBlockingQueue也体现了这样的思想，在队列头入队，在队列尾出队，入队和出队使用不同的锁，相对于LinkedBlockingArray只有一个锁效率要高；

**拆锁的粒度不能无限拆，最多可以将一个锁拆为当前cup数量个锁即可**

##### 锁粗化

大部分情况下我们是要让锁的粒度最小化，锁的粗化则是要增大锁的粒度;
在以下场景下需要粗化锁的粒度：
假如有一个循环，循环内的操作需要加锁，我们应该把锁放到循环外面，否则每次进出循环，都进出一次临界区，效率是非常差的；

##### 使用读写锁

ReentrantReadWriteLock 读操作加读锁，可以并发读，写操作使用写锁，只能单线程写；

##### 读写分离

CopyOnWriteArrayList 、CopyOnWriteArraySet
写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。
　CopyOnWrite并发容器用于读多写少的并发场景，因为，读的时候没有锁，但是对其进行更改的时候是会加锁的，否则会导致多个线程同时复制出多个副本，各自修改各自的；

##### 使用cas

如果需要同步的操作执行速度非常快，并且线程竞争并不激烈，这时候使用cas效率会更高，因为加锁会导致线程的上下文切换，如果上下文切换的耗时比同步操作本身更耗时，且线程对资源的竞争不激烈，使用volatiled+cas操作会是非常高效的选择；

##### 消除缓存行的伪共享

1. 在jdk1.7之前会 将需要独占缓存行的变量前后添加一组long类型的变量，依靠这些无意义的数组的填充做到一个变量自己独占一个缓存行；
2. 在jdk1.7因为jvm会将这些没有用到的变量优化掉，所以采用继承一个声明了好多long变量的类的方式来实现；
3. 在jdk1.8中通过添加sun.misc.Contended注解来解决这个问题，若要使该注解有效必须在jvm中添加以下参数：
   -XX:-RestrictContended
   sun.misc.Contended注解会在变量**前面**添加**128字节**的padding将当前变量与其他变量进行隔离



### 4.同步工具类

除了锁与 Condition，Concurrent 包还提供了一系列同步工具类。这些同步工具类的原理，有些也是基于AQS的，有些则需要特殊的实现机制，这一章将对所有同步工具类的实现原理进行剖析。

#### 4.1 Semaphore

Semaphore也就是信号量，提供了资源数量的并发访问控制，其使用代码很简单，如下所示。

```java
Semaphore semaphore = new Semaphore(10,true);//初始10个共享资源,第二个是公平非公平选项
semaphore.acquire();//每次取一个, 取不到就阻塞
semaphore.release();//用完释放
```

假设有n个线程来获取Semaphore里面的资源（n>10），n个线程中只有10个线程能获取到，其他线程都会阻塞。直到有线程释放了资源，其他线程才能获取到。

当初始的资源个数为1的时候，Semaphore退化为排他锁。正因为如此，Semaphone的实现原理和锁十分类似，是基于AQS，有公平和非公平之分。Semaphore相关类的继承体系如图4-2所示。

![image-20210806171629704](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210806171629704.png)

```java
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
public void release() {
    sync.releaseShared(1);   
}
```

由于Semaphore和锁的实现原理基本相同，上面的代码不再展开解释。资源总数即state的初始值，在acquire里对state变量进行CAS减操作，减到0之后，线程阻塞；在release里对state变量进行CAS加操作。

#### 4.2 CountDownLatch

##### 4.2.1 CountDownLatch使用场景

考虑一个场景：一个主线程要等待10个 Worker 线程工作完毕才退出，就能使用CountDownLatch来实现。

```java
CountDownLatch countDownLatch = new CountDownLatch(10);//初始为10
countDownLatch.await();//主线程调用阻塞在这
countDownLatch.countDown();//10个线程工作完成后调用countDown减一,减到0主线程唤醒
```

图4-3所示为CountDownLatch相关类的继承层次，CountDownLatch原理和Semaphore原理类似，同样是基于AQS，不过没有公平和非公平之分。

![image-20210806172216901](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210806172216901.png)

##### 4.2.2 await（）实现分析

如下所示，await（）调用的是 AQS 的模板方法，这个方法在前面已经介绍过。CountDownLatch.Sync重新实现了tryAccuqireShared方法。

```java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);//AQS的模板方法
}
public final void acquireSharedInterruptibly(int arg)
    throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)//被CountDownLatch.Sync重新实现
        doAcquireSharedInterruptibly(arg);
}
protected int tryAcquireShared(int acquires) {
    return (getState() == 0) ? 1 : -1;
}
```

从tryAcquireShared（..）方法的实现来看，只要state！=0，调用await（）方法的线程便会被放入AQS的阻塞队列，进入阻塞状态。

```java
public void countDown() {
    sync.releaseShared(1);
}
//AQS模板方法
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {//CountDownLatch.Sync实现
        doReleaseShared();
        return true;
    }
    return false;
}
protected boolean tryReleaseShared(int releases) {
    // Decrement count; signal when transition to zero
    for (;;) {
        int c = getState();
        if (c == 0)
            return false;
        int nextc = c-1;
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
```

countDown（）调用的 AQS 的模板方法 releaseShared（），里面的 tryReleaseShared（..）被CountDownLatch.Sync重新实现。从上面的代码可以看出，只有state=0，tryReleaseShared（..）才会返回true，然后执行doReleaseShared（..），一次性唤醒队列中所有阻塞的线程。

#### 4.3 CyclicBarrier

##### 4.3.1 CyclicBarrier使用场景

CyclicBarrier使用代码也很简单，如下所示

```java
CyclicBarrier cyclicBarrier = new CyclicBarrier(10);
cyclicBarrier.await();
```

考虑这样一个场景：10个工程师一起来公司应聘，招聘方式分为笔试和面试。首先，要等人到齐后，开始笔试；笔试结束之后，再一起参加面试。把10个人看作10个线程，10个线程之间的同步过程如图4-5所示。

![image-20210806173745070](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210806173745070.png)

在整个过程中，有2个同步点：第1个同步点，要等所有应聘者都到达公司，再一起开始笔试；第2个同步点，要等所有应聘者都结束笔试，之后一起进入面试环节。具体到每个线程的run（）方法中，就是下面的伪代码：

![image-20210806173813840](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210806173813840.png)

##### 4.3.2 CyclicBarrier实现原理

CyclicBarrier基于ReentrantLock+Condition实现

```java
public class CyclicBarrier {
    private final ReentrantLock lock = new ReentrantLock();
    /** Condition to wait on until tripped */
    private final Condition trip = lock.newCondition();//用于线程间相互唤醒
    private final int parties;//总线程数
    private Generation generation = new Generation();
    private int count;
```

下面详细介绍 CyclicBarrier 的实现原理。先从构造函数说起，可以看到，不仅可以传入参与方的总数量，还可以传入一个回调函数。当所有的线程被唤醒时，barrierAction被执行。

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}
```

接下来看一下await（）函数的实现过程。

```java
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException,TimeoutException {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        final Generation g = generation;

        if (g.broken)
            throw new BrokenBarrierException();

        if (Thread.interrupted()) {//响应中断
            breakBarrier();//唤醒所有阻塞的线程
            throw new InterruptedException();
        }

        int index = --count;//每次线程调用一次await,count--,count减到0此线程唤醒其他所有线程
        if (index == 0) {  // tripped
            boolean ranAction = false;
            try {
                final Runnable command = barrierCommand;
                if (command != null)//一起唤醒后,还可以执行一个回调
                    command.run();
                ranAction = true;
                nextGeneration();//唤醒其他线程,把count值复原,用于下一次CyclicBarrier
                return 0;
            } finally {
                if (!ranAction)
                    breakBarrier();
            }
        }

        // loop until tripped, broken, interrupted, or timed out
        for (;;) {//count>0,说明人没齐,阻塞自己
            try {
                if (!timed)
                    trip.await();//关键: await阻塞自己会释放锁,这样别的线程拿到锁执行上面count--
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    Thread.currentThread().interrupt();
                }
            }

            if (g.broken)
                throw new BrokenBarrierException();

            if (g != generation)//阻塞唤醒,返回
                return index;

            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        lock.unlock();
    }
}
```

关于上面的函数，有几点说明：

1. CyclicBarrier是可以被重用的。以上一节的应聘场景为例，来了10个线程，这10个线程互相等待，到齐后一起被唤醒，各自执行接下来的逻辑；然后，这10个线程继续互相等待，到齐后再一起被唤醒。每一轮被称为一个Generation，就是一次同步点。
2. CyclicBarrier 会响应中断。10 个线程没有到齐，如果有线程收到了中断信号，所有阻塞的线程也会被唤醒，就是上面的breakBarrier（）函数。然后count被重置为初始值（parties），重新开始。
3. 上面的回调函数，barrierAction只会被第10个线程执行1次（在唤醒其他9个线程之前），而不是10个线程每个都执行1次。. 

#### 4.4 Exchanger

##### 4.4.1 使用场景

Exchanger用于线程之间交换数据，其使用代码很简单，是一个exchange（..）函数

它提供一个同步点，在这个同步点两个线程可以交换彼此的数据。这两个线程通过exchange方法交换数据， 如果第一个线程先执行exchange方法，它会一直等待第二个线程也执行exchange，当两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方。因此使用Exchanger的重点是成对的线程使用exchange()方法，当有一对线程达到了同步点，就会进行交换数据。因此该工具类的线程对象是成对的。        

Exchanger类提供了两个方法，String exchange(V x):用于交换，启动交换并等待另一个线程调用exchange；String exchange(V x,long timeout,TimeUnit unit)：用于交换，启动交换并等待另一个线程调用exchange，并且设置最大等待时间，当等待时间超过timeout便停止等待。

```java
public class ExChangerTest {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newCachedThreadPool();
        final Exchanger exchanger = new Exchanger();

        executor.execute(new Runnable() {
            String data1 = "A";

            @Override
            public void run() {
                nbaTrade(data1, exchanger);
            }
        });
        executor.execute(new Runnable() {
            String data1 = "B";

            @Override
            public void run() {
                nbaTrade(data1, exchanger);
            }
        });
        executor.execute(new Runnable() {
            String data1 = "C";

            @Override
            public void run() {
                nbaTrade(data1, exchanger);
            }
        });
        executor.execute(new Runnable() {
            String data1 = "D";

            @Override
            public void run() {
                nbaTrade(data1, exchanger);
            }
        });
        executor.shutdown();
    }

    private static void nbaTrade(String data1, Exchanger exchanger) {
        try {
            System.out.println(Thread.currentThread().getName() + "在交易截止之前把 " + data1 + " 交易出去");
            Thread.sleep((long) (Math.random() * 1000));
            String data2 = (String) exchanger.exchange(data1);
            System.out.println(Thread.currentThread().getName() + "交易得到" + data2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}
```

在上面的例子中，4个线程并发地调用exchange（..），会两两交互数据，如A/B、C/D、A/C、B/D、A/D或B/C。

##### 4.4.2 实现原理

Exchanger的核心机制和Lock一样，也是CAS+park/unpark。在实现上面，JDK7和JDK8有一定差异，这里以JDK7的实现为例进行分析。首先，在Exchanger内部，有两个内部类：Slot和Node，代码如下：

![image-20210809093154128](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210809093154128.png)

每个线程在调用exchange（..）函数交换数据的时候，会先创建一个Node对象，这个Node对象就是对该线程的包装，里面包含了2个字段：1个是该线程要交互的数据，另1个是该线程自身。这里有个关键点：Node本身是继承自AtomicReference的，所以除了这2个字段，Node还有第3个字段，记录的是对方所要交换的数据，初始为NULL。

Slot其实是一个AtomicReference，里面的q0，q1，…，qd变量都是多余的。为什么要添加这些多余的变量呢？是为了优化性能而做的缓存行填充。

Slot的AtomicReference就是指向的一个Node，通过Slot和Node相结合，实现了2个线程之间的数据交换，如图4-6所示。线程 1 持有数据 item1，线程2持有数据 item2，各自调用exchange（..），会各自生成一个Node。而Slot只会指向2个Node中的1个：如果是线程1先调用的exchange（..），那么Slot就指向Node1，线程1阻塞，等待线程2来交换；反之，如果是线程2先调用的exchange（..），那么Slot就指向Node2，线程2阻塞，等待线程1来交换数据。

![image-20210809093430366](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210809093430366.png)

一个Slot只能支持2个线程之间交换数据，要实现多个线程并行地交换数据，需要多个Slot，因此在Exchanger里面定义了Slot数组：

##### 4.4.3 exchange（V x）实现分析

明白了大致思路，下面来看exchange（V x）函数的详细实现：

```java
// 用于左移Node数组下标，以便得出数组在内存中偏移量来获取数据，避免伪共享 
private static final int ASHIFT = 7; 
// Node数组最大下标 
private static final int MMASK = 0xff; 
// 用于递增bound，每次递增一个SEQ 
private static final int SEQ = MMASK + 1; 
// CPU核心数 
private static final int NCPU = Runtime.getRuntime().availableProcessors(); 
// 当前数组最大下标 
static final int FULL = (NCPU >= (MMASK << 1)) ? MMASK : NCPU >>> 1; 
// 自旋次数，CPU核心为1时自旋会被禁用 
private static final int SPINS = 1 << 10; 
// 用于exchange方法中参数为null时传递给其它线程的对象 
private static final Object NULL_ITEM = new Object(); 
// 用于超时传递的对象 
private static final Object TIMED_OUT = new Object(); 
// 节点用于保持需要交换的数据 
@sun.misc.Contended static final class Node { 
    int index;              // arana数组下标，多槽位时使用 
    int bound;              // 上一次记录的bound  
    int collides;           // CAS失败次数  
    int hash;               // 用于自旋的伪随机数  
    Object item;            // 当前线程需要交换的数据  
    volatile Object match;  // 匹配线程交换的数据  
    volatile Thread parked; // 记录当前挂起的线程 
} 
// 用户记录线程状态的内部类
static final class Participant extends ThreadLocal {    
    public Node initialValue() { 
        return new Node();                   
    } 
} 
// 记录线程状态 
private final Participant participant; 
// 多槽位数据交换使用 
private volatile Node[] arena; 
// 用于交换数据的槽位 
private volatile Node slot; 
/** * The index of the largest valid arena position, OR'ed with SEQ * number in high bits, incremented on each update.  The initial * update from 0 to SEQ is used to ensure that the arena array is * constructed only once. */ 
private volatile int bound;
```

```java
public V exchange(V x) throws InterruptedException {
    Object v;
    Object item = (x == null) ? NULL_ITEM : x; // translate null args
    if ((arena != null ||
         (v = slotExchange(item, false, 0L)) == null) &&
        ((Thread.interrupted() || // disambiguates null return
          (v = arenaExchange(item, false, 0L)) == null)))
        throw new InterruptedException();
    return (v == NULL_ITEM) ? null : (V)v;
}
```

```java
private final Object slotExchange(Object item, boolean timed, long ns) {
    Node p = participant.get();//获取当前节点对象
    Thread t = Thread.currentThread();//当前线程
    //线程中断,直接返回null
    if (t.isInterrupted()) // preserve interrupt status so caller can recheck
        return null;
    //自旋
    for (Node q;;) {
        //slot不为空,说明已存在线程等待交换数据
        if ((q = slot) != null) {
            //cas置空槽位slot
            if (U.compareAndSwapObject(this, SLOT, q, null)) {
                Object v = q.item;//获取槽位中需要交换的对象
                q.match = item;//将当前需要交换的数据置入match
                Thread w = q.parked;//获取被挂起线程
                //存在挂起线程,唤醒
                if (w != null)
                    U.unpark(w);
                return v;//返回交换后的数据
            }
            //存在竞争,其他线程抢先,使用多槽位交换
            // CPU为多核心 且 bound等于0(arana数组未初始化)，则CAS操作将bound增加SEQ
            // create arena on contention, but continue until slot null
            if (NCPU > 1 && bound == 0 &&
                U.compareAndSwapInt(this, BOUND, 0, SEQ))
                arena = new Node[(FULL + 2) << ASHIFT];
        }
        else if (arena != null)//多槽位不为空,执行多槽位交换
            return null; // caller must reroute to arenaExchange
        else {
            //表示当前线程是第一个线程进来交换数据 或 之前交换已完成,可重新认为是第一个线程
            //将需要交换的数据存到slot的item属性
            p.item = item;
            //cas设置槽位为p
            if (U.compareAndSwapObject(this, SLOT, null, p))
                break;//cas成功,结束自旋
            p.item = null;//cas失败,置空item,继续自选操作
        }
    }

    //当前线程已占slot,等待其他线程交换数据
    int h = p.hash;
    long end = timed ? System.nanoTime() + ns : 0L;
    int spins = (NCPU > 1) ? SPINS : 1;//自旋次数
    Object v;
    //其他线程成功交换槽位中的数据
    while ((v = p.match) == null) {
        if (spins > 0) {//自旋
            h ^= h << 1; h ^= h >>> 3; h ^= h << 10;
            if (h == 0)
                h = SPINS | (int)t.getId();
            else if (h < 0 && (--spins & ((SPINS >>> 1) - 1)) == 0)
                Thread.yield();//线程让步,提供cpu利用率
        }
        //存在线程交换数据,已修改slot,未修改match,则等待
        else if (slot != p)
            spins = SPINS;
        //线程未中断 且 不是多槽位交换 且 没设置超时或超时时间未到
        else if (!t.isInterrupted() && arena == null &&
                 (!timed || (ns = end - System.nanoTime()) > 0L)) {
            U.putObject(t, BLOCKER, this);//设置线程t被当前阻塞
            p.parked = t;//设置节点挂起线程属性parked
            //如果slot不为空,表明没有线程与之交换数据,挂起当前线程
            if (slot == p)
                U.park(false, ns);
            p.parked = null;//线程被唤醒,将节点挂起线程属性设置为null
            U.putObject(t, BLOCKER, null);//设置线程t没有被任何对象阻塞
        }
        //不满足上述添加,交换失败,重置slot
        else if (U.compareAndSwapObject(this, SLOT, p, null)) {
            v = timed && ns <= 0L && !t.isInterrupted() ? TIMED_OUT : null;
            break;
        }
    }
    U.putOrderedObject(p, MATCH, null);//置空match
    p.item = null;//置空item
    p.hash = h;
    return v;//返回交换数据
}
```

关于上面的代码，有几个点要说明：

1. 当一个线程调用exchange准备和其他线程交换数据的时候，无外乎两种情况。一种是没有其他线程要交换数据，自己只能自旋或者阻塞，等待；另一种是恰好有其他线程在 Slot 里面等着，那么和对方交换。
2. 由于Slot不止1个，而是多个。如果运气好，根据自己的thread id 找到对应的Slot，里面恰好有别的线程在等待，就和对方交换。交换办法是：取出Slot指向的Node，也就是对方的Node，然后把这个Node（本身也是一个AtomicReference）指向自己的item，唤醒对方，同时返回对方的item。（再次强调一次：1个Node实际上有2个item字段，1个记录的是自己的item，1个记录的是对方的item，因此实现2个线程的数据交换）。
3. 如果运气不好，Slot是空的，如何处理呢？当前Slot为空，不代表其他Slot没有线程在等待。因此，如果当前Slot的index=0，自己就阻塞在那；如果index！=0，则需要遍历所有的 Slot，看其他的Slot 里面是否有线程在等待。最好是遍历一圈发现没有其他线程，自己再在index=0的位置等待。

#### 4.5 Phaser

Phaser的同步模型于CountDownLatch、CyclicBarrier类似，但提供了更加灵活的同步支持，另外由于实现采用了无锁算法，整个同步操作的实现变得更加复杂。考虑到高并发条件下的CAS竞争，也提供了不同机制去优化性能（堆叠，自旋等）

##### 4.5.1 用Phaser替代CyclicBarrier和CountDownLatch

从JDK7开始，新增了一个同步工具类Phaser，其功能比CyclicBarrier和CountDownLatch更加强大。

Phaser一般是定义一个parties数（parties一般代指需要进行同步的线程），当这些parties到达某个执行点，就会调用await方法，表示到达某个阶段（phase），然后就会阻塞直到有足够的parites数（也就是线程数）都调用过await方法后，这些线程才会被逐个唤醒，另外，在唤醒之前，可以选择性地执行某个定制的任务。Phaser对比起CyclicBarrier，不仅它是可以重复同步，并且parties数是可以动态注册的，另外还提供了非阻塞的arrive方法表示先到达阶段等，大大提高了同步模型的灵活性，当然了，实现也会相对复杂。

###### 1.用Phaser替代CountDownLatch

考虑讲CountDownLatch时的例子，1个主线程要等10个worker线程完成之后，才能做接下来的事情，也可以用Phaser来实现此功能。在CountDownLatch中，主要是2个函数：await（）和countDown（），在Phaser中，与之相对应的函数是awaitAdance（int n）和arrive（）。

```java
final Phaser phaser = new Phaser(10);//初始10
phaser.awaitAdvance(phaser.getPhase());//主线程调用阻塞在这awaitAdvance表示等待当前phase(当前同步点)完成
phaser.arrive();//10个worker线程,完成工作调用arrive,表示到达同步点
```

###### 2.用Phaser替代CyclicBarrier

考虑前面讲CyclicBarrier时，10个工程师去公司应聘的例子，也可以用Phaser实现，代码基本类似。

![image-20210809103920664](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210809103920664.png)

arriveAndAwaitAdance（）就是 arrive（）与 awaitAdvance（int）的组合，表示“我自己已到达这个同步点，同时要等待所有人都到达这个同步点，然后再一起前行”。

##### 4.5.2 Phaser新特性

###### 特性1：动态调整线程个数

注册（Registration）

CyclicBarrier 所要同步的线程个数是在构造函数中指定的，之后不能更改，而 Phaser 可以在运行期间动态地调整要同步的线程个数。Phaser 提供了下面这些函数来增加、减少所要同步的线程个数。

```java
phaser.register();//注册一个
phaser.bulkRegister(10);//注册多个
phaser.arriveAndDeregister();//解注册
```

和其它栅栏不一样，在phaser上进行同步的注册parites数可能会随着时间改变而不同。可以在任何时间注册任务（调用register，bulkRegister方法，或者可以初始化parties数的构造函数形式），另外还可以在任何到达栅栏的时候反注册（arriveAndDeregister方法）。作为大部分同步结构的基础，注册和反注册只会影响内部计数；它们没有做任何额外的内部记录，所以任务不能够查询它们是否被注册。（不过你可以继承这个类以引入这样的记录）

###### 终结（Termination）

phaser可能进入一个终结状态，可以通过isTerminated来检查。当终结的时候，所有的同步方法都不会在等待下一个阶段而直接返回，返回一个负值来表示该状态。类似地，在终结的时候尝试注册没有任何效果。当onAdvance调用返回true的时候就会触发终结。onAdvance默认实现为当一个反注册导致注册parties数降为0的时候返回true。当phser要控制操作在一个固定得迭代次数时，就可以很方便地重写这个方法，当当前阶段号到达阀值得时候就返回true导致终结。forceTermination方法也时另一个可以突然释放等待线程并且允许它们终结。

###### 特性2：层次Phaser

多个Phaser可以组成如图4-7所示的树状结构，可以通过在构造函数中传入父Phaser来实现。

```java
 public Phaser(Phaser parent, int parties) {
```

![image-20210809104343819](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210809104343819.png)

先简单看一下Phaser内部关于树状结构的存储，如下面代码所示。

```java
 private final Phaser parent;
```

可以发现，在Phaser的内部结构中，每个Phaser记录了自己的父节点，但并没有记录自己的子节点列表。所以，每个 Phaser 知道自己的父节点是谁，但父节点并不知道自己有多少个子节点，对父节点的操作，是通过子节点来实现的。
树状的Phaser怎么使用呢？考虑如下代码，会组成如图4-8所示的树状Phaser。

```java
Phaser root = new Phaser(2);
Phaser c1 = new Phaser(root,3);
Phaser c2 = new Phaser(root,2);
Phaser c3 = new Phaser(c1,0);
```

![image-20210809104714234](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210809104714234.png)

本来root有两个参与者，然后为其加入了两个子Phaser（c1，c2），每个子Phaser会算作1个参与者，root的参与者就变成2+2=4个。c1本来有3个参与者，为其加入了一个子Phaser c3，参与者数量变成3+1=4个。c3的参与者初始为0，后续可以通过调用register（）函数加入。

对于树状Phaser上的每个节点来说，可以当作一个独立的Phaser来看待，其运作机制和一个单独的Phaser是一样的。具体来讲：父Phaser并不用感知子Phaser的存在，当子Phaser中注册的参与者数量大于0时，会把自己向父节点注册；当子Phaser中注册的参与者数量等于0时，会自动向父节点解注册。父Phaser把子Phaser当作一个正常参与的线程就可以了。



**堆叠（Tiering）**

Phaser可以被堆叠在一起（也就是说，以树形结构构造）来降低竞争。Phaser的parties数很大的时候，以一组子phasers共享一个公共父亲能够减轻严重的同步竞争的成本。这样做可以大大提供吞吐量，但同时也会导致每个操作的更高的成本。

在一棵堆叠的phaser树中，子phaser在父亲上的注册和反注册都会被自动管理。当子phaser的注册parties树为非0的时候，子phaser就会注册到父亲上。当由于arriveAndDeregister的调用使注册的parties数变为0时，子phaser就会从父亲中反注册。这样就算父phaser的所有parties都到达了阶段，也必须等待子phaser的所有parties都到达了阶段并显式调用父phaser的awaitAdvance才算到达新的阶段。反之亦然。这样父phaser或者子phaser里注册过的所有parties就可以一起互相等待到新的阶段。另外，在这个堆叠结构的实现里，可以确保root结点必然是最先更新阶段号，然后才到其子结点，逐渐传递下去。

###### 监控（Monitoring）

同步方法只能被注册的parties调用时，phaser的当前状态可以被任何调用者监控。在任何时刻，有getRegisteredParties总共的parties，其中，有getArrivedParties个parites到达getPhase的当前阶段。当剩下getUnarrivedParties个parties到达，phase增加。这些方法的返回值可能反映短暂的状态，因此一般在同步控制中不太有用。toString方法以一种可以方便信息监控的格式返回这些状态的快照。

##### 4.5.3 state变量解析

大致了解了Phaser的用法和新特性之后，下面仔细剖析其实现原理。Phaser没有基于AQS来实现，但具备AQS的核心特性：state变量、CAS操作、阻塞队列。先从state变量说起。

这个64位的state变量被拆成4部分，如图4-9所示为state变量各部分表示的意思。

![image-20210809104958694](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210809104958694.png)

最高位0表示未同步完成，1表示同步完成，初始最高位为0。
Phaser提供了一系列的成员函数来从state中获取图4-9中的几个数字，如下所示。

```java
//获取当前的轮数.当前轮数已经同步完成,返回值是一个负数(高位为1)
public final int getPhase() {
    return (int)(root.state >>> PHASE_SHIFT);//当前phase未完成,返回值是个负数
}
private static final int  PHASE_SHIFT  = 32;
//当前轮数同步完成,最高位为1
public boolean isTerminated() {
    return root.state < 0L;
}
//获取总注册线程数
public int getRegisteredParties() {
    return partiesOf(state);
}
private static int partiesOf(long s) {
    return (int)s >>> PARTIES_SHIFT;//把state转成32位,再右移16位
}
private static final int  PARTIES_SHIFT   = 16;
//获取未达到的线程数
public int getUnarrivedParties() {
    return unarrivedOf(reconcileState());
}
private static int unarrivedOf(long s) {
    int counts = (int)s;
    return (counts == EMPTY) ? 0 : (counts & UNARRIVED_MASK);//截取低16位
}
private static final int  UNARRIVED_MASK  = 0xffff;      // to mask ints
```

下面再看一下state变量在构造函数中是如何被赋值的：

```java
public Phaser(Phaser parent, int parties) {
    if (parties >>> PARTIES_SHIFT != 0)//parties超出最大2的16次方,直接抛异常
        throw new IllegalArgumentException("Illegal number of parties");
    int phase = 0;//初始轮数为0
    this.parent = parent;
    if (parent != null) {
        final Phaser root = parent.root;
        this.root = root;
        this.evenQ = root.evenQ;
        this.oddQ = root.oddQ;
        if (parties != 0)
            phase = parent.doRegister(1);
    }
    else {
        this.root = this;
        this.evenQ = new AtomicReference<QNode>();
        this.oddQ = new AtomicReference<QNode>();
    }
    this.state = (parties == 0) ? (long)EMPTY :
    ((long)phase << PHASE_SHIFT) |  //或操作,赋给state.最高位为0,表示同步未完成
        ((long)parties << PARTIES_SHIFT) |
        ((long)parties);
}
private static final int  PARTIES_SHIFT   = 16;
private static final int  PHASE_SHIFT     = 32;
private static final int  EMPTY           = 1;
```

当parties=0时，state被赋予一个EMPTY常量，常量为1；
当parties！=0时，把phase值左移32位；把parties左移16位；然后parties也作为最低的16位，3个值做或操作，赋值给state。这个赋值操作也反映了图4-9的意思。

##### 4.5.4 阻塞与唤醒（Treiber Stack）

基于上述的state变量，对其执行CAS操作，并进行相应的阻塞与唤醒。如图4-10所示，右边的主线程会调用awaitAdvance（）进行阻塞；左边的arrive（）会对state进行CAS的累减操作，当未到达的线程数减到0时，唤醒右边阻塞的主线程。

![image-20210809112256219](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210809112256219.png)

在这里，阻塞使用的是一个称为Treiber Stack的数据结构，而不是AQS的双向链表。Treiber Stack是一个无锁的栈，由R.Kent Treiber在其于1986年发表的论文Systems Programming：Coping with Parallelism 中首次提出。它是一个单向链表，出栈、入栈都在链表头部，所以只需要一个head指针，而不需要tail指针。实现代码如下所示。

```java
public class Phaser {
    static final class QNode implements ForkJoinPool.ManagedBlocker {
        volatile Thread thread; //和aqs一样,每个node对应一个thread对象
        QNode next;//单向链表
    }
    private final AtomicReference<QNode> evenQ;//链表头指针
    private final AtomicReference<QNode> oddQ;
}
```

首先可以注意到QNode类是实现了ForkJoinPool.ManagedBlocker接口，这个接口可以确保如果使用ForkJoinWorkerThread的时候就可以保持并发执行任务。



为了减少并发冲突，这里定义了2个链表，也就是2个Treiber Stack。当phase为奇数轮的时候，阻塞线程放在oddQ里面；当phase为偶数轮的时候，阻塞线程放在evenQ里面。代码如下所示。

```java
private AtomicReference<QNode> queueFor(int phase) {
    return ((phase & 1) == 0) ? evenQ : oddQ;
}
```

##### 4.5.5 arrive（）函数分析

Phaser提供了单独表示到达阶段的非阻塞函数，即

```java
public int arrive() {//一个party到达
    return doArrive(ONE_ARRIVAL);
}
public int arriveAndDeregister() {//一个party到达并且反注册这个party
    return doArrive(ONE_DEREGISTER);
}
```

arrive（）和 arriveAndDeregister（）内部调用的都是 doArrive（boolean）函数。区别在于前者只是把“未达到线程数”减1；后者则把“未到达线程数”和“下一轮的总线程数”都减1。下面看一下doArrive（boolean）函数的实现。

```java
  private int doArrive(int adjust) {
        final Phaser root = this.root;
        for (;;) {
            long s = (root == this) ? state : reconcileState();
            int phase = (int)(s >>> PHASE_SHIFT);
            if (phase < 0)
                return phase;
            int counts = (int)s;
            int unarrived = (counts == EMPTY) ? 0 : (counts & UNARRIVED_MASK);
            if (unarrived <= 0)
                throw new IllegalStateException(badArrive(s));
            if (UNSAFE.compareAndSwapLong(this, stateOffset, s, s-=adjust)) {
                if (unarrived == 1) {
                    long n = s & PARTIES_MASK;  // base of next state
                    int nextUnarrived = (int)n >>> PARTIES_SHIFT;
                    if (root == this) {
                        if (onAdvance(phase, nextUnarrived))
                            n |= TERMINATION_BIT;
                        else if (nextUnarrived == 0)
                            n |= EMPTY;
                        else
                            n |= nextUnarrived;
                        int nextPhase = (phase + 1) & MAX_PHASE;
                        n |= (long)nextPhase << PHASE_SHIFT;
                        UNSAFE.compareAndSwapLong(this, stateOffset, s, n);
                        releaseWaiters(phase);
                    }
                    else if (nextUnarrived == 0) { // propagate deregistration
                        phase = parent.doArrive(ONE_DEREGISTER);
                        UNSAFE.compareAndSwapLong(this, stateOffset,
                                                  s, s | EMPTY);
                    }
                    else
                        phase = parent.doArrive(ONE_ARRIVAL);
                }
                return phase;
            }
        }
    }
```

 doArrive看上去很复杂，但其实逻辑并不算太复杂。

1. 首先依然是进入一个自旋结构，然后根据是否有堆叠（root == this）来获取正确状态值，接着计算阶段号phase，未到达parties数unarrived，此时如果unarrived少于等于0，则必须抛出异常，表示这次到达是非法的（因为所有的注册parties数已经到达）；
2. 接着就是利用CAS把状态值进行更改，这里是减去参数adjust值，arrive的传入参数是ONE_ARRIVE，也就是1，arriveAndDeregister是ONE_ARRIVAL|ONE_PARTY，减去之后刚好把未到达数和注册数都减去一。
3. 如果CAS成功，如果CAS之前的unarrived刚好为1，则表示此次到达是最后一个未到达party，然后重新开始计算下一个阶段值n，接着需要根据是否堆叠进行判断：
   1. 如果没有堆叠（root == this）则按照定义我们调用onAdvance，传入相对参数，此时如果onAdvance返回true，我们给n添加终结标识，如果onAdvance返回false，但下阶段的未到达parties数（同时也是当前注册的parties数）为0（可能由于反注册造成），因此要给n添加EMPTY值，否则就给n添加新的未到达parties数，接着就调用CAS把当前状态值更改为n，然后调用releaseWaiters释放上一阶段号的等待队列。注意这里第二个CAS的返回值可以忽略，因为这里与doRegister的冲突已经由doRegister的unarrived判断解决。
   2. 如果（1）判断失败，则出现了堆叠，另外如果此时新的未到达数为0（所有之前的注册parties数都被反注册），根据堆叠的结构，我们必须向parent表示已经到达一个party并且反注册自己，并且同时把当前状态CAS为EMPTY，同样，这里的CAS可以忽略返回值。
   3. 如果（1）（2）判断都失败，则只需要简单地把向parent调用doArrive(ONE_ARRIVE)表示自己当前所有已经注册的parties数都到达了，然后parent就会减去一个代表这个子phaser的到达parties数。

##### 4.5.6 awaitAdvance（）函数分析

```java
public int awaitAdvance(int phase) {
    final Phaser root = this.root;
    long s = (root == this) ? state : reconcileState();//当前只有一个Phaser,没有树状结构时,root=this
    int p = (int)(s >>> PHASE_SHIFT);
    if (phase < 0)//phase已经结束,不用阻塞,直接返回
        return phase;
    if (p == phase)
        return root.internalAwaitAdvance(phase, null);//阻塞再phase这一轮上面
    return p;
}
```

下面的while循环中有4个分支：初始的时候，node==null，进入第1个分支进行自旋，自旋次数满足之后，会新建一个QNode节点；之后执行第3、第4个分支，分别把该节点入栈并阻塞。

```java
private int internalAwaitAdvance(int phase, QNode node) {
    // assert root == this;
    releaseWaiters(phase-1);          // ensure old queue clean
    boolean queued = false;           // true when node is enqueued
    int lastUnarrived = 0;            // to increase spins upon change
    int spins = SPINS_PER_ARRIVAL;
    long s;
    int p;
    while ((p = (int)((s = state) >>> PHASE_SHIFT)) == phase) {
        if (node == null) {           // spinning in noninterruptible mode 自旋
            int unarrived = (int)s & UNARRIVED_MASK;
            if (unarrived != lastUnarrived &&
                (lastUnarrived = unarrived) < NCPU)
                spins += SPINS_PER_ARRIVAL;
            boolean interrupted = Thread.interrupted();
            if (interrupted || --spins < 0) { // need node to record intr自旋结束,建一个节点,进入阻塞
                node = new QNode(this, phase, false, false, 0L);
                node.wasInterrupted = interrupted;
            }
        }
        else if (node.isReleasable()) // done or aborted 阻塞唤醒,退出循环
            break;
        else if (!queued) {           // push onto queue
            AtomicReference<QNode> head = (phase & 1) == 0 ? evenQ : oddQ;
            QNode q = node.next = head.get();
            if ((q == null || q.phase == phase) &&
                (int)(state >>> PHASE_SHIFT) == phase) // avoid stale enq
                queued = head.compareAndSet(q, node);//节点入栈
        }
        else {
            try {
                ForkJoinPool.managedBlock(node);//调用node.block()阻塞
            } catch (InterruptedException ie) {
                node.wasInterrupted = true;
            }
        }
    }

    if (node != null) {
        if (node.thread != null)
            node.thread = null;       // avoid need for unpark()
        if (node.wasInterrupted && !node.interruptible)
            Thread.currentThread().interrupt();
        if (p == phase && (p = (int)(state >>> PHASE_SHIFT)) == phase)
            return abortWait(phase); // possibly clean up on abort
    }
    releaseWaiters(phase);
    return p;
}
```

这里调用了ForkJoinPool.managedBlock（ManagedBlocker blocker）函数，目的是把node对应的线程阻塞。ManagerdBlocker是ForkJoinPool里面的一个接口，定义如下：

```java
public static interface ManagedBlocker {
    boolean block() throws InterruptedException;
    boolean isReleasable();
}
```

QNode实现了该接口，实现原理还是park（），如下所示。之所以没有直接使用park（）/unpark（）来实现阻塞、唤醒，而是封装了ManagedBlocker这一层，主要是出于使用上的方便考虑。一方面是park（）可能被中断唤醒，另一方面是带超时时间的park（），把这二者都封装在一起。

```java
static final class QNode implements ForkJoinPool.ManagedBlocker {
    public boolean isReleasable() {
        if (thread == null)
            return true;
        if (phaser.getPhase() != phase) {
            thread = null;
            return true;
        }
        if (Thread.interrupted())
            wasInterrupted = true;
        if (wasInterrupted && interruptible) {
            thread = null;
            return true;
        }
        if (timed) {
            if (nanos > 0L) {
                nanos = deadline - System.nanoTime();
            }
            if (nanos <= 0L) {
                thread = null;
                return true;
            }
        }
        return false;
    }

    public boolean block() {//block封装pack和parkNanos,使用更方便
        if (isReleasable())
            return true;
        else if (!timed)
            LockSupport.park(this);
        else if (nanos > 0L)
            LockSupport.parkNanos(this, nanos);
        return isReleasable();
    }
```

arriveAndAwaitAdvance对比起先调用arrive和awaitAdvance，更加方便并且由于减少了一些多余的变量读取和逻辑，速度更加快。

### 5.ConcurrentCollection

#### 5.1 BlockingQueue

在所有的并发容器中，BlockingQueue是最常见的一种。BlockingQueue是一个带阻塞功能的队列，当入队列时，若队列已满，则阻塞调用者；当出队列时，若队列为空，则阻塞调用者。
在Concurrent包中，BlockingQueue是一个接口，有许多个不同的实现类，如图5-1所示。

![image-20210809182731877](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210809182731877.png)

该接口的定义如下：

```java
public interface BlockingQueue<E> extends Queue<E> {
    boolean add(E e);
    boolean offer(E e);
    void put(E e) throws InterruptedException;
    E take() throws InterruptedException;
    ...
}
```

可以看到，该接口和JDK集合包中的Queue接口是兼容的，同时在其基础上增加了阻塞功能。

在这里，入队提供了add（..）、offer（..）、put （..）3个函数，有什么区别呢？

从上面的定义可以看到，add（..）和offer（..）的返回值是布尔类型，而put无返回值，还会抛出中断异常，所以add（..）和offer（..）是无阻塞的，也是Queue本身定义的接口，而put（..）是阻塞式的。add（..）和offer（..）的区别不大，当队列为满的时候，前者会抛出异常，后者则直接返回false。drainTo()一次取多个

![image-20210816144449529](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210816144449529.png)

出队列与之类似，提供了remove（）、peek（）、take（）等函数，remove（）和peek（）是非阻塞式的，take（）是阻塞式的。下面分别介绍BlockingQueue的各种不同实现。

##### 5.1.1 ArrayBlockingQueue

ArrayBlockingQueue 是一个用数组实现的环形队列，在构造函数中，会要求传入数组的容量。

```java
public ArrayBlockingQueue(int capacity, boolean fair) {}
```

其核心数据结构如下：

```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {
    final Object[] items;//数组及队头队尾
    int takeIndex;
    int putIndex;
    int count;
    //核心是一把锁两个条件
    final ReentrantLock lock;
    private final Condition notEmpty;
    private final Condition notFull;
```

其put/take函数也很简单，如下所示。

```java
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();//使用可中断lock
    try {
        while (count == items.length)//满即阻塞
            notFull.await();
        enqueue(e);
    } finally {
        lock.unlock();
    }
}
    private void enqueue(E x) {
        // assert lock.getHoldCount() == 1;
        // assert items[putIndex] == null;
        final Object[] items = this.items;
        items[putIndex] = x;
        if (++putIndex == items.length)
            putIndex = 0;
        count++;
        notEmpty.signal();//put之后通知非空条件
    }
  public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)//空即阻塞
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
    private E dequeue() {
        // assert lock.getHoldCount() == 1;
        // assert items[takeIndex] != null;
        final Object[] items = this.items;
        @SuppressWarnings("unchecked")
        E x = (E) items[takeIndex];
        items[takeIndex] = null;
        if (++takeIndex == items.length)
            takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeued();
        notFull.signal();//take结束通知非满条件
        return x;
    }
```

##### 5.1.2 LinkedBlockingQueue

LinkedBlockingQueue是一种基于单向链表的阻塞队列。因为队头和队尾是2个指针分开操作的，所以用了2把锁+2个条件，同时有1个AtomicInteger的原子变量记录count数

```java
public class LinkedBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {
    private final int capacity;
    private final AtomicInteger count = new AtomicInteger();//原子变量
    transient Node<E> head;//单向链表头尾
    private transient Node<E> last;
    //2锁2条件
   private final ReentrantLock takeLock = new ReentrantLock();
    private final Condition notEmpty = takeLock.newCondition();
    private final ReentrantLock putLock = new ReentrantLock();
    private final Condition notFull = putLock.newCondition();
```

在其构造函数中，也可以指定队列的总容量。如果不指定，默认为Integer.MAX_VALUE。

```java
public LinkedBlockingQueue() {
    this(Integer.MAX_VALUE);
}
public LinkedBlockingQueue(int capacity) {
    if (capacity <= 0) throw new IllegalArgumentException();
    this.capacity = capacity;
    last = head = new Node<E>(null);
}
```

下面看一下其put/take实现。

```java
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    // Note: convention in all put/take/etc is to preset local var
    // holding count negative to indicate failure unless set.
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly();
    try {
        /*
         * Note that count is used in wait guard even though it is
         * not protected by lock. This works because count can
         * only decrease at this point (all other puts are shut
         * out by lock), and we (or some other waiting put) are
         * signalled if it ever changes from capacity. Similarly
         * for all other uses of count in other wait guards.
         */
        while (count.get() == capacity) {
            notFull.await();//通知其他put线程
        }
        enqueue(node);
        c = count.getAndIncrement();
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();//加takelock
}
public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();
    try {
        while (count.get() == 0) {
            notEmpty.await();
        }
        x = dequeue();
        c = count.getAndDecrement();
        if (c > 1)
            notEmpty.signal();//通知其他take线程
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();//加putlock
    return x;
}
```

LinkedBlockingQueue和ArrayBlockingQueue的实现有一些差异，有几点要特别说明：

1. 为了提高并发度，用2把锁，分别控制队头、队尾的操作。意味着在put（..）和put（..）之间、take（）与take（）之间是互斥的，put（..）和take（）之间并不互斥。但对于count变量，双方都需要操作，所以必须是原子类型。
2. 因为各自拿了一把锁，所以当需要调用对方的condition的signal时，还必须再加上对方的锁，就是signalNotEmpty（）和signalNotFull（）函数。示例如下所示。

```java
private void signalNotEmpty() {
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock();//先拿takelock,才能调用 notEmpty.signal();
    try {
        notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
}
private void signalNotFull() {
    final ReentrantLock putLock = this.putLock;
    putLock.lock();//先拿putlock
    try {
        notFull.signal();
    } finally {
        putLock.unlock();
    }
}
```

3. 不仅put会通知 take，take 也会通知 put。当put 发现非满的时候，也会通知其他 put线程；当take发现非空的时候，也会通知其他take线程。

##### 5.1.3 PriorityBlockingQueue

队列通常是先进先出的，而PriorityQueue是按照元素的优先级从小到大出队列的。正因为如此，PriorityQueue中的2个元素之间需要可以比较大小，并实现Comparable接口。
其核心数据结构如下：

```java
public class PriorityBlockingQueue<E> extends AbstractQueue<E>
    implements BlockingQueue<E>, java.io.Serializable {
    private transient Object[] queue;//数组实现的二叉小根堆
    private transient int size;
    private transient Comparator<? super E> comparator;
    //一锁一条件
    private final ReentrantLock lock;
    private final Condition notEmpty;
```

其构造函数如下所示，如果不指定初始大小，内部会设定一个默认值11，当元素个数超过这个大小之后，会自动扩容。

```java
public PriorityBlockingQueue() {
    this(DEFAULT_INITIAL_CAPACITY, null);
}
private static final int DEFAULT_INITIAL_CAPACITY = 11;
public PriorityBlockingQueue(int initialCapacity,
                             Comparator<? super E> comparator) {
    if (initialCapacity < 1)
        throw new IllegalArgumentException();
    this.lock = new ReentrantLock();
    this.notEmpty = lock.newCondition();
    this.comparator = comparator;
    this.queue = new Object[initialCapacity];
}
```

下面是对应的put/take函数的实现

```java
public void put(E e) {
    offer(e); // never need to block
}
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    final ReentrantLock lock = this.lock;
    lock.lock();
    int n, cap;
    Object[] array;
    while ((n = size) >= (cap = (array = queue).length))
        tryGrow(array, cap);//超长,扩容
    try {
        Comparator<? super E> cmp = comparator;
        if (cmp == null)//没有定义比较,使用默认
            siftUpComparable(n, e, array);//元素入堆,即执行siftup操作
        else
            siftUpUsingComparator(n, e, array, cmp);
        size = n + 1;
        notEmpty.signal();
    } finally {
        lock.unlock();
    }
    return true;
}
  public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        E result;
        try {
            while ( (result = dequeue()) == null)//出队
                notEmpty.await();
        } finally {
            lock.unlock();
        }
        return result;
    }
private E dequeue() {
        int n = size - 1;
        if (n < 0)
            return null;
        else {
            Object[] array = queue;
            E result = (E) array[0];//最小二叉堆,堆顶即是要出队元素
            E x = (E) array[n];
            array[n] = null;
            Comparator<? super E> cmp = comparator;
            if (cmp == null)
                siftDownComparable(0, x, array, n);//调整堆,执行siftdown操作
            else
                siftDownUsingComparator(0, x, array, n, cmp);
            size = n;
            return result;
        }
    }
```

从上面可以看到，在阻塞的实现方面，和ArrayBlockingQueue的机制相似，主要区别是用数组实现了一个二叉堆，从而实现按优先级从小到大出队列。另一个区别是没有notFull条件，当元素个数超出数组长度时，执行扩容操作。

##### 5.1.4 DelayQueue

DelayQueue即延迟队列，也就是一个按延迟时间从小到大出队的PriorityQueue。所谓延迟时间，就是“未来将要执行的时间”-“当前时间”。为此，放入DelayQueue中的元素，必须实现Delayed接口，如下所示。

```java
public interface Delayed extends Comparable<Delayed> {
    long getDelay(TimeUnit unit);
}
```

关于该接口，有两点说明：

1. 如果getDelay的返回值小于或等于0，则说明该元素到期，需要从队列中拿出来执行。
2. 该接口首先继承了 Comparable 接口，所以要实现该接口，必须实现 Comparable 接口。具体来说，就是基于getDelay（）的返回值比较两个元素的大小。

下面看一下DelayQueue的核心数据结构。

```java
public class DelayQueue<E extends Delayed> extends AbstractQueue<E>
    implements BlockingQueue<E> {

    //一锁一非空条件
    private final transient ReentrantLock lock = new ReentrantLock();
    private final PriorityQueue<E> q = new PriorityQueue<E>();//优先级队列
    private final Condition available = lock.newCondition();
```

下面介绍put/take的实现，先从take说起，因为这样更能看出DelayQueue的特性。

```java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            E first = q.peek();//取二叉堆堆顶元素,也就是延迟时间最小的
            if (first == null)
                available.await();//队空,take阻塞
            else {
                long delay = first.getDelay(NANOSECONDS);
                if (delay <= 0)//堆顶元素延迟时间小于或等于0,返回
                    return q.poll();
                first = null; // don't retain ref while waiting
                if (leader != null)//有其他线程也在等待这个元素,则无线阻塞
                    available.await();
                else {
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        available.awaitNanos(delay);//否则阻塞有限的时间
                    } finally {
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        if (leader == null && q.peek() != null)
            available.signal();//自己是leader,已经获取堆顶元素,唤醒其他线程
        lock.unlock();
    }
}
```

关于take（）函数，有两点需要说明：

1. 不同于一般的阻塞队列，只在队列为空的时候，才阻塞。如果堆顶元素的延迟时间没到，也会阻塞。

2. 在上面的代码中使用了一个优化技术，用一个Thread leader变量记录了等待堆顶元素的第1个线程。为什么这样做呢？通过 getDelay（..）可以知道堆顶元素何时到期，不必无限期等待，可以使用condition.awaitNanos（）等待一个有限的时间；只有当发现还有其他线程也在等待堆顶元素（leader！=NULL）时，才需要无限期等待。

下面看一下put的实现。

```java
public void put(E e) {
    offer(e);
}
public boolean offer(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        q.offer(e);//元素放入二叉堆
        if (q.peek() == e) {//放入元素如果在堆顶,说明放入元素延迟时间是最小的,通知等待线程
            				//否则放入元素不在堆顶没有必要通知等待线程
            leader = null;
            available.signal();
        }
        return true;
    } finally {
        lock.unlock();
    }
}
```

关于上面的实现，有一点要说明：不是每放入一个元素，都需要通知等待的线程。放入的元素，如果其延迟时间大于当前堆顶的元素延迟时间，就没必要通知等待的线程；只有当延迟时间是最小的，在堆顶时，才有必要通知等待的线程，也就是上面代码中的if（q.peek（）==e）段落。

##### 5.1.5 SynchronousQueue

SynchronousQueue是一种特殊的BlockingQueue，它本身没有容量。先调put（..），线程会阻塞；直到另外一个线程调用了take（），两个线程才同时解锁，反之亦然。对于多个线程而言，例如3个线程，调用3次put（..），3个线程都会阻塞；直到另外的线程调用3次take（），6个线程才同时解锁，反之亦然。

在讲解线程池中的CachedThreadPool实现的时候会使用到SynchronousQueue的这种特性。

接下来就看一下，SynchronousQueue是如何实现的。先从构造函数看起。

```java
public SynchronousQueue(boolean fair) {
    transferer = fair ? new TransferQueue<E>() : new TransferStack<E>();
}
```

和锁一样，也有公平和非公平模式。如果是公平模式，则用TransferQueue实现；如果是非公平模式，则用TransferStack实现。这两个类分别是什么呢？先看一下put/take的实现。

```java
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    if (transferer.transfer(e, false, 0) == null) {
        Thread.interrupted();
        throw new InterruptedException();
    }
}
public E take() throws InterruptedException {
    E e = transferer.transfer(null, false, 0);
    if (e != null)
        return e;
    Thread.interrupted();
    throw new InterruptedException();
}
```

可以看到，put/take都调用了transfer（..）接口。而TransferQueue和TransferStack分别实现了这个接口。该接口在SynchronousQueue内部，如下所示。如果是put（..），则第1个参数就是对应的元素；如果是take（），则第1个参数为null。后2个参数分别为是否设置超时和对应的超时时间。

```java
abstract static class Transferer<E> {
    abstract E transfer(E e, boolean timed, long nanos);
}
```

接下来看一下什么是公平模式和非公平模式，如图5-2所示。假设3个线程分别调用了put（..），3个线程会进入阻塞状态，直到其他线程调用3次take（），和3个put（..）一一配对。

如果是公平模式（队列模式），则第1个调用put（..）的线程1会在队列头部，第1个到来的take（）线程和它进行配对，遵循先到先配对的原则，所以是公平的；如果是非公平模式（栈模式），则第3个调用put（..）的线程3会在栈顶，第1个到来的take（）线程和它进行配对，遵循的是后到先配对的原则，所以是非公平的。

![image-20210810110518477](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210810110518477.png)

下面分别看一下TransferQueue和TransferStack的实现。

###### 1.TransferQueue

```java
public class SynchronousQueue<E> extends AbstractQueue<E>
    implements BlockingQueue<E>, java.io.Serializable { 
    static final class TransferQueue<E> extends Transferer<E> {
        static final class QNode {
            volatile QNode next;          // 单向链表
            volatile Object item;         // put item!=null/ take item = null
            volatile Thread waiter;       // put/take对应的阻塞线程和上面item对应
            final boolean isData;		  //put,isdata = true, 否则为false
            transient volatile QNode head;//单向链表头尾
            transient volatile QNode tail;
```

从上面的代码可以看出，TransferQueue是一个基于单向链表而实现的队列，通过head和tail 2个指针记录头部和尾部。初始的时候，head和tail会指向一个空节点，构造函数如下所示。

```java
TransferQueue() {
    QNode h = new QNode(null, false); // initialize to dummy node.空节点
    head = h;
    tail = h;
}
```

图5-3所示为TransferQueue的工作原理。
阶段（a）：队列中是一个空的节点，head/tail都指向这个空节点。

阶段（b）：3个线程分别调用put，生成3个QNode，进入队列。

阶段（c）：来了一个线程调用take，会和队列头部的第1个QNode进行配对。

阶段（d）：第1个QNode出队列。

![image-20210810111510733](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210810111510733.png)

这里有一个关键点：put节点和take节点一旦相遇，就会配对出队列，所以在队列中不可能同时存在put节点和take节点，要么所有节点都是put节点，要么所有节点都是take节点。

接下来看一下TransferQueue的代码实现。

```java
E transfer(E e, boolean timed, long nanos) {
    QNode s = null; // constructed/reused as needed
    boolean isData = (e != null);

    for (;;) {
        QNode t = tail;
        QNode h = head;
        if (t == null || h == null)         // saw uninitialized value 未初始化,自旋等待
            continue;                       // spin

        if (h == t || t.isData == isData) { // empty or same-mode 队为空 或 为同一种元素
            QNode tn = t.next;
            if (t != tail)                  // inconsistent read 不一致读,重新执行for
                continue;
            if (tn != null) {               // lagging tail
                advanceTail(t, tn);
                continue;
            }
            if (timed && nanos <= 0)        // can't wait
                return null;
            if (s == null)
                s = new QNode(e, isData);   //新建一个节点
            if (!t.casNext(null, s))        // failed to link in 加入尾部
                continue;

            advanceTail(t, s);              // swing tail and wait 后移tail指针
            Object x = awaitFulfill(s, e, timed, nanos);//进入阻塞
            if (x == s) {                   // wait was cancelled
                clean(t, s);
                return null;
            }

            if (!s.isOffList()) {           // not already unlinked 唤醒,队列第一个元素
                advanceHead(t, s);          // unlink if head
                if (x != null)              // and forget fields
                    s.item = s;
                s.waiter = null;
            }
            return (x != null) ? (E)x : e;

        } else {                            // complementary-mode 当前线程与队列第一个元素配对
            QNode m = h.next;               // node to fulfill 取第一个元素
            if (t != tail || m == null || h != head)//不一致读 重新for
                continue;                   // inconsistent read

            Object x = m.item;
            if (isData == (x != null) ||    // m already fulfilled 已经配对过
                x == m ||                   // m cancelled
                !m.casItem(x, e)) {         // lost CAS   尝试配对
                advanceHead(h, m);          // dequeue and retry 已配对 出队列
                continue;
            }

            advanceHead(h, m);              // successfully fulfilled 配对成功,出队列
            LockSupport.unpark(m.waiter);	//唤醒队列中与第一个元素对应的线程
            return (x != null) ? (E)x : e;
        }
    }
}
```

整个 for 循环有两个大的 if-else 分支，如果当前线程和队列中的元素是同一种模式（都是put节点或者take节点），则与当前线程对应的节点被加入队列尾部并且阻塞；如果不是同一种模式，则选取队列头部的第1个元素进行配对。

这里的配对就是m.casItem（x，e），把自己的item x换成对方的item e，如果CAS操作成功，则配对成功。如果是put节点，则isData=true，item！=null；如果是take节点，则isData=false，item=null。如果CAS操作不成功，则isData和item之间将不一致，也就是isData！=（x！=null），通过这个条件可以判断节点是否已经被匹配过了。

###### 2.TransferStack

TransferStack的定义如下所示，首先，它也是一个单向链表。不同于队列，只需要head指针就能实现入栈和出栈操作。
链表中的节点有三种状态，REQUEST对应take节点，DATA对应put节点，二者配对之后，会生成一个FULFILLING节点，入栈，然后FULLING节点和被配对的节点一起出栈。

```java
static final class TransferStack<E> extends Transferer<E> {
    static final int REQUEST    = 0;
    static final int DATA       = 1;
    static final int FULFILLING = 2;
    static final class SNode {
        volatile SNode next; //单向链表
        volatile SNode match;  //匹配的节点
        volatile Thread waiter; 
        Object item;  
        int mode;//三种模式
    }
    volatile SNode head;
}
```

如图5-4所示为TransferStack的工作原理。
阶段（a）：head指向NULL。不同于TransferQueue，这里没有空的头节点。
阶段（b）：3个线程调用3次put，依次入栈。
阶段（c）：线程4调用take，和栈顶的第1个元素配对，生成FULLFILLING节点，入栈。
阶段（d）：栈顶的2个元素同时入栈。

![image-20210810112726579](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210810112726579.png)

下面看一下具体的代码实现。

```java
E transfer(E e, boolean timed, long nanos) {
    SNode s = null; // constructed/reused as needed
    int mode = (e == null) ? REQUEST : DATA;

    for (;;) {
        SNode h = head;
        if (h == null || h.mode == mode) {  // empty or same-mode
            if (timed && nanos <= 0) {      // can't wait
                if (h != null && h.isCancelled())
                    casHead(h, h.next);     // pop cancelled node
                else
                    return null;
            } else if (casHead(h, s = snode(s, e, h, mode))) {//入栈
                SNode m = awaitFulfill(s, timed, nanos);//阻塞
                if (m == s) {               // wait was cancelled
                    clean(s);
                    return null;
                }
                if ((h = head) != null && h.next == s)
                    casHead(h, s.next);     // help s's fulfiller
                return (E) ((mode == REQUEST) ? m.item : s.item);
            }
        } else if (!isFulfilling(h.mode)) { // try to fulfill 不同种模式,待匹配
            if (h.isCancelled())            // already cancelled
                casHead(h, h.next);         // pop and retry
            else if (casHead(h, s=snode(s, e, h, FULFILLING|mode))) {//生成一个FULFILLING节点入栈
                for (;;) { // loop until matched or waiters disappear
                    SNode m = s.next;       // m is s's match
                    if (m == null) {        // all waiters are gone
                        casHead(s, null);   // pop fulfill node
                        s = null;           // use new node next time
                        break;              // restart main loop
                    }
                    SNode mn = m.next;
                    if (m.tryMatch(s)) {
                        casHead(s, mn);     // pop both s and m 两节点一起出栈
                        return (E) ((mode == REQUEST) ? m.item : s.item);
                    } else                  // lost match
                        s.casNext(m, mn);   // help unlink
                }
            }
        } else {                            // help a fulfiller 匹配过 出栈
            SNode m = h.next;               // m is h's match
            if (m == null)                  // waiter is gone
                casHead(h, null);           // pop fulfilling node
            else {
                SNode mn = m.next;
                if (m.tryMatch(h))          // help match
                    casHead(h, mn);         // pop both h and m配对 出栈
                else                        // lost match
                    h.casNext(m, mn);       // help unlink
            }
        }
    }
}
```

#### 5.2 BlockingDeque

BlockingDeque定义了一个阻塞的双端队列接口，如下所示。

```java
public interface BlockingDeque<E> extends BlockingQueue<E>, Deque<E> {
    void putFirst(E e) throws InterruptedException;
    void putLast(E e) throws InterruptedException;
    E takeFirst() throws InterruptedException;
    E takeLast() throws InterruptedException;
```

可以看到，该接口在继承了BlockingQueue接口的同时，增加了对应的双端队列操作接口。该接口只有一个实现，就是LinkedBlockingDeque。
其核心数据结构如下所示，是一个双向链表。

```java
public class LinkedBlockingDeque<E> extends AbstractQueue<E>
    implements BlockingDeque<E>, java.io.Serializable {
    static final class Node<E> {
        E item;
        Node<E> prev;//双向链表node
        Node<E> next;
        Node(E x) {
            item = x;
        }
    }
    transient Node<E> first;//队列头尾
    transient Node<E> last;
    private transient int count;//元素个数
    private final int capacity;//容量
    //一锁2条件
    final ReentrantLock lock = new ReentrantLock();
    private final Condition notEmpty = lock.newCondition();
    private final Condition notFull = lock.newCondition();
```

对应的实现原理，和LinkedBlockingQueue基本一样，只是LinkedBlockingQueue是单向链表，而LinkedBlockingDeque是双向链表。

```java
public E takeFirst() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        E x;
        while ( (x = unlinkFirst()) == null)
            notEmpty.await();
        return x;
    } finally {
        lock.unlock();
    }
}

public E takeLast() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        E x;
        while ( (x = unlinkLast()) == null)
            notEmpty.await();
        return x;
    } finally {
        lock.unlock();
    }
}
public void putFirst(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    Node<E> node = new Node<E>(e);
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        while (!linkFirst(node))
            notFull.await();
    } finally {
        lock.unlock();
    }
}
public void putLast(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    Node<E> node = new Node<E>(e);
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        while (!linkLast(node))
            notFull.await();
    } finally {
        lock.unlock();
    }
}
```

#### 5.3 CopyOnWrite

CopyOnWrite指在“写”的时候，不是直接“写”源数据，而是把数据拷贝一份进行修改，再通过悲观锁或者乐观锁的方式写回。那为什么不直接修改，而是要拷贝一份修改呢？这是为了在“读”的时候不加锁。下面通过几个案例来展现CopyOnWrite的应用。

###### 5.3.1 CopyOnWriteArrayList

和ArrayList一样，CopyOnWriteArrayList的核心数据结构也是一个数组，代码如下。

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    private transient volatile Object[] array;
```

下面是CopyOnArrayList的几个“读”函数：

```java
public E get(int index) {
    return get(getArray(), index);//未加锁
}
```

既然这些“读”函数都没有加锁，那么是如何保证“线程安全”呢？答案在“写”函数里面。

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();//加悲观锁
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        //copyonwrite,写之前备份
        newElements[len] = e;
        setArray(newElements);//新数组赋值给老数组
        return true;
    } finally {
        lock.unlock();
    }
}
```

其他“写”函数，例如remove和add类似，此处不再详述。

###### 5.3.2 CopyOnWriteArraySet

CopyOnWriteArraySet 就是用 Array 实现的一个 Set，保证所有元素都不重复。其内部是封装的一个CopyOnWriteArrayList。

```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E>
    implements java.io.Serializable {
    private final CopyOnWriteArrayList<E> al;//封装copyonwritearraylist
    public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }
    public boolean add(E e) {
        return al.addIfAbsent(e);//不重复,add
    }
```

#### 5.4 ConcurrentLinkedQueue/Deque

前面详细分析了AQS内部的阻塞队列实现原理：基于双向链表，通过对head/tail进行CAS操作，实现入队和出队。

ConcurrentLinkedQueue 的实现原理和AQS 内部的阻塞队列类似：同样是基于 CAS，同样是通过head/tail指针记录队列头部和尾部，但还是有稍许差别。

首先，它是一个单向链表，定义如下。

```java
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E>
    implements Queue<E>, java.io.Serializable {
    private static class Node<E> {
        volatile E item;
        volatile Node<E> next;
    }
    private transient volatile Node<E> head;
    private transient volatile Node<E> tail;
```

其次，在AQS的阻塞队列中，每次入队后，tail一定后移一个位置；每次出队，head一定前移一个位置，以保证head指向队列头部，tail指向链表尾部。

但在ConcurrentLinkedQueue中，head/tail的更新可能落后于节点的入队和出队，因为它不是直接对 head/tail指针进行 CAS操作的，而是对 Node中的 item进行操作。下面进行详细分析：

1. 初始化
   初始的时候，如图5-5所示，head和tail都执行一个NULL节点。对应的代码如下。

   ```java
   public ConcurrentLinkedQueue() {
       head = tail = new Node<E>(null);
   }
   ```

2. 入队列
   代码如下所示。

   ```java
   public boolean offer(E e) {
       checkNotNull(e);
       final Node<E> newNode = new Node<E>(e);
       for (Node<E> t = tail, p = t;;) {
           Node<E> q = p.next;
           if (q == null) {
               if (p.casNext(null, newNode)) {//关键:对tail的next指针cas,而不是tail
                   if (p != t) 
                       casTail(t, newNode);  //每入两个节点,后移一次tail指针,失败也没关系
                   return true;
               }
           }
           else if (p == q)
               p = (t != (t = tail)) ? t : head;//到达队尾
           else
               // Check for tail updates after two hops.
               p = (p != t && t != (t = tail)) ? t : q;//后移p指针
       }
   }
   ```

上面的入队其实是每次在队尾追加2个节点时，才移动一次tail节点，具体过程如图5-6所示。
初始的时候，队列中有1个节点item1，tail指向该节点，假设线程1要入队item2节点：
step1:p=tail,q=p.next=NULL.
step2：对p的next执行CAS操作，追加item2，成功之后，p=tail。所以上面的casTail函数不会执行，直接返回。此时tail指针没有变化。

![image-20210810151226046](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210810151226046.png)

之后，假设线程2要入队item3节点，如图5-7所示。
step3:p=tail,q=p.next.
step4：q！=NULL，因此不会入队新节点。p，q都后移1位。
step5：q=NULL，对p的next执行CAS操作，入队item3节点。
step6：p！=t，满足条件，执行上面的casTail操作，tail后移2个位置，到达队列尾部。

![image-20210811093129327](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210811093129327.png)

最后总结一下入队列的两个关键点：
（1）即使tail指针没有移动，只要对p的next指针成功进行CAS操作，就算成功入队列。
（2）只有当 p！=tail的时候，才会后移tail指针。也就是说，每连续追加2个节点，才后移1次tail指针。即使CAS失败也没关系，可以由下1个线程来移动tail指针。

3. 出队列
   上面说了入队列之后，tail指针不变化，那是否会出现入队列之后，要出队列却没有元素可出的情况呢？

   ```java
   public E poll() {
       restartFromHead:
       for (;;) {
           for (Node<E> h = head, p = h, q;;) {
               E item = p.item;
               if (item != null && p.casItem(item, null)) {//关键:出队列,没有移动head,而是item置位null
                   if (p != h) // hop two nodes at a time 产生2个null节点,才把head指针后移2位
                       updateHead(h, ((q = p.next) != null) ? q : p);
                   return item;
               }
               else if ((q = p.next) == null) {
                   updateHead(h, p);
                   return null;
               }
               else if (p == q)
                   continue restartFromHead;
               else
                   p = q;
           }
       }
   }
   ```

出队列的代码和入队列类似，也有p、q2个指针，整个变化过程如图5-8所示。假设初始的时候head指向空节点，队列中有item1、item2、item3 三个节点。
step1:p=head,q=p.next.p!=q.
step2：后移p指针，使得p=q。
step3：出队列。关键点：此处并没有直接删除item1节点，只是把该节点的item通过CAS操作置为了NULL。
step4：p！=head，此时队列中有了2个 NULL 节点，再前移1次head指针，对其执行updateHead操作。

![image-20210811093720611](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210811093720611.png)

最后总结一下出队列的关键点：
（1）出队列的判断并非观察 tail 指针的位置，而是依赖于 head 指针后续的节点是否为NULL这一条件。
（2）只要对节点的item 执行CAS操作，置为NULL成功，则出队列成功。即使head指针没有成功移动，也可以由下1个线程继续完成。

4. 队列判空
   因为head/tail 并不是精确地指向队列头部和尾部，所以不能简单地通过比较 head/tail 指针来判断队列是否为空，而是需要从head指针开始遍历，找第1个不为NULL的节点。如果找到，则队列不为空；如果找不到，则队列为空。代码如下所示。

   ```java
   public boolean isEmpty() {
       return first() == null;//找第一个不为空节点
   }
   Node<E> first() {//从head开始遍历
       restartFromHead:
       for (;;) {
           for (Node<E> h = head, p = h, q;;) {
               boolean hasItem = (p.item != null);
               if (hasItem || (q = p.next) == null) {
                   updateHead(h, p);
                   return hasItem ? p : null;
               }
               else if (p == q)
                   continue restartFromHead;
               else
                   p = q;
           }
       }
   }
   ```

#### 5.5 ConcurrentHashMap 

#### 5.6 ConcurrentSkipListMap/Set

ConcurrentHashMap 是一种 key 无序的 HashMap，ConcurrentSkipListMap 则是key 有序的，实现了NavigableMap接口，此接口又继承了SortedMap接口。

##### 5.6.1 ConcurrentSkipListMap

###### 1.为什么要使用SkipList实现Map？

在Java的util包中，有一个非线程安全的HashMap，也就是TreeMap，是key有序的，基于红黑树实现。

而在Concurrent包中，提供的key有序的HashMap，也就是ConcurrentSkipListMap，是基于SkipList（跳查表）来实现的。这里为什么不用红黑树，而用跳查表来实现呢？

借用Doug Lea的原话：

也就是目前计算机领域还未找到一种高效的、作用在树上的、无锁的、增加和删除节点的办法。

那为什么SkipList可以无锁地实现节点的增加、删除呢？这要从无锁链表的实现说起。

###### 2.无锁链表

Doug Lea在注释中引用了一篇无锁链表的论文：A pragmatic implementation of non-blocking linked lists。
表面上看，无锁链表是很简单的，根本不需要写一篇论文来专门论述。在上文中讲解AQS时，曾反复用到无锁队列，其实现也是链表。

究竟二者的区别在哪呢？

上文所讲的无锁队列、栈，都是只在队头、队尾进行CAS操作，通常不会有问题。如果在链表的中间进行插入或删除操作，按照通常的CAS做法，就会出现问题！

关于这个问题，Doug Lea的论文中有清晰的论述，此处引用如下：

操作1：在节点10后面插入节点20。如图5-14所示，首先把节点20的next指针指向节点30，然后对节点10的next指针执行CAS操作，使其指向节点20即可。

![image-20210811165912402](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210811165912402.png)

操作2：删除节点10。如图5-15所示，只需把头节点的next指针，进行CAS操作到节点30即可。

![image-20210811165948588](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210811165948588.png)

但是，如果两个线程同时操作，一个删除节点10，一个要在节点10后面插入节点20。并且这两个操作都各自是CAS的，此时就会出现问题。如图5-16所示，删除节点10，会同时把新插入的节点20也删除掉！这个问题超出了CAS的解决范围。

![image-20210811170100896](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210811170100896.png)



为什么会出现这个问题呢？

究其原因：在删除节点10的时候，实际受到操作的是节点10的前驱，也就是头节点。节点10本身没有任何变化。故而，再往节点10后插入节点20的线程，并不知道节点10已经被删除了！

针对这个问题，在论文中提出了如下的解决办法，如图5-17所示，把节点 10 的删除分为两2步：

第一步，把节点10的next指针，mark成删除，即软删除；
第二步，找机会，物理删除。

做标记之后，当线程再往节点10后面插入节点20的时候，便可以先进行判断，节点10是否已经被删除，从而避免在一个删除的节点10后面插入节点20。==这个解决方法有一个关键点：“把节点10的next指针指向节点20（插入操作）”和“判断节点10本身是否已经删除（判断操作）”，必须是原子的，必须在1个CAS操作里面完成！==

![image-20210811170241944](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210811170241944.png)

具体的实现有两个办法：

办法一：AtomicMarkableReference
保证每个 next 是 AtomicMarkableReference 类型。但这个办法不够高效，Doug Lea 在ConcurrentSkipListMap的实现中用了另一种办法。

办法2：Mark节点
我们的目的是标记节点10已经删除，也就是标记它的next字段。那么可以新造一个marker节点，使节点10的next指针指向该Marker节点。这样，当向节点10的后面插入节点20的时候，就可以在插入的同时判断节点10的next指针是否执行了一个Marker节点，这两个操作可以在一个CAS操作里面完成。

###### 3.跳表

解决了无锁链表的插入或删除问题，也就解决了跳查表的一个关键问题。因为跳查表就是多层链表叠起来的。
下面先看一下跳查表的数据结构（下面所用代码都引用自JDK 7，JDK 8中的代码略有差异，但不影响下面的原理分析）。

```java
//node节点所有的kv对,都是这个单向链表串起来的
static final class Node<K,V> {
    //键值对的key和value
    final K key;
    volatile Object value;
    //链表中的下一个节点
    volatile Node<K,V> next;

    Node(K key, Object value, Node<K,V> next) {
        this.key = key;
        this.value = value;
        this.next = next;
    }

    Node(Node<K,V> next) {
        this.key = null;
        this.value = this;
        this.next = next;
    }

    //cas修改value
    boolean casValue(Object cmp, Object val) {
        return UNSAFE.compareAndSwapObject(this, valueOffset, cmp, val);
    }

    //cas修改next
    boolean casNext(Node<K,V> cmp, Node<K,V> val) {
        return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
    }

    boolean isMarker() {
        return value == this;
    }

    boolean isBaseHeader() {
        return value == BASE_HEADER;
    }

    //插入一个value指向自己的节点，next为f的节点
    //删除某个节点时会将其value置为null，然后调用此方法在该节点后插入一个marker，将该节点从next链表中移除
    //marker在这里是打一个标识同时保存下一个节点的引用，保证将其从next链表中移除失败，即cas修改前一个节点的next属性失败的情形下该节点可以被正常移除
    boolean appendMarker(Node<K,V> f) {
        return casNext(f, new Node<K,V>(f));
    }

    //在查询或者修改节点时，发现某个节点的value已经是null了，会调用此方法
    //b是前一个节点，f是next节点
    void helpDelete(Node<K,V> b, Node<K,V> f) {
        if (f == next && this == b.next) {
            //再次检查链表关系
            if (f == null || f.value != f) 
                //该节点未插入marker节点，此处重新插入，下次调用时进入else分支
                casNext(f, new Node<K,V>(f));
            else
                //f是一个marker节点
                //将this和f都从链表中移除
                b.casNext(this, f.next);
        }
    }


    V getValidValue() {
        Object v = value;
        if (v == this || v == BASE_HEADER) //value是无效
            return null;
        @SuppressWarnings("unchecked") V vv = (V)v;
        return vv;
    }


    AbstractMap.SimpleImmutableEntry<K,V> createSnapshot() {
        Object v = value;
        if (v == null || v == this || v == BASE_HEADER) //value是无效
            return null;
        @SuppressWarnings("unchecked") V vv = (V)v;
        return new AbstractMap.SimpleImmutableEntry<K,V>(key, vv);
    }

    //获取属性偏移量
    private static final sun.misc.Unsafe UNSAFE;
    private static final long valueOffset;
    private static final long nextOffset;

    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> k = Node.class;
            valueOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("value"));
            nextOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("next"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
//上层index层节点
static class Index<K,V> {
    final Node<K,V> node;//不存实际数据,指向node
    final Index<K,V> down;//关键:每个index必有一个指针,指向下一个level对应的节点
    volatile Index<K,V> right;//自己也组成单向链表
}
//ConcurrentSkipListMap只记录顶层head节点
public class ConcurrentSkipListMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentNavigableMap<K,V>, Cloneable, Serializable {
    //头索引节点
    private transient volatile HeadIndex<K,V> head;

    //比较大小的比较器
    final Comparator<? super K> comparator;

    //key视图，下面这些视图都是在请求相关方法时才会创建
    private transient KeySet<K> keySet;
    //键值对视图
    private transient EntrySet<K,V> entrySet;
    //value视图
    private transient Values<V> values;
    //倒序遍历的视图
    private transient ConcurrentNavigableMap<K,V> descendingMap;
}
```

![image-20210811172022748](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210811172022748.png)

下面详细分析如何从跳查表上查找、插入和删除元素。

Index是HeadIndex的父类，都表示SkipList中的索引节点，其实现如下：

```java
static class Index<K,V> {
    //关联的node节点
    final Node<K,V> node; 
    //down指向的节点的node和当前节点的node是同一个，但是down节点位于下一个层级，层级越往下包含的节点数越多，最底层包含了所有的node节点
    final Index<K,V> down; 
    //right指向的节点的key大于当前节点，跟当前节点位于同一个层级，沿着right属性构成的链表遍历，node的key值越来越大
    volatile Index<K,V> right; ，

        Index(Node<K,V> node, Index<K,V> down, Index<K,V> right) {
        this.node = node;
        this.down = down;
        this.right = right;
    }

    //cas修改right属性
    final boolean casRight(Index<K,V> cmp, Index<K,V> val) {
        return UNSAFE.compareAndSwapObject(this, rightOffset, cmp, val);
    }

    final boolean indexesDeletedNode() {
        return node.value == null;
    }

    //将新节点插入到当前节点的rigth链表中，原right节点的前面  
    final boolean link(Index<K,V> succ, Index<K,V> newSucc) {
        Node<K,V> n = node;
        newSucc.right = succ;
        //node的vlue不为空时，将right由succ修改成newSucc
        return n.value != null && casRight(succ, newSucc);
    }

    //将succ从right链表中移除
    final boolean unlink(Index<K,V> succ) {
        //node的vlue不为空时，将right由succ修改成succ.right
        return node.value != null && casRight(succ, succ.right);
    }

    //获取right属性偏移量
    private static final sun.misc.Unsafe UNSAFE;
    private static final long rightOffset;
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> k = Index.class;
            rightOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("right"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}


static final class HeadIndex<K,V> extends Index<K,V> {
    //level记录层级
    final int level;
    HeadIndex(Node<K,V> node, Index<K,V> down, Index<K,V> right, int level) {
        super(node, down, right);
        this.level = level;
    }
}
```

构造方法

```java
public ConcurrentSkipListMap() {
    this.comparator = null;
    initialize();
}

public ConcurrentSkipListMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
    initialize();
}

public ConcurrentSkipListMap(Map<? extends K, ? extends V> m) {
    this.comparator = null;
    initialize();
    //使用父类AbstractMap的实现，遍历m中的键值对，通过put方法添加到当前map中
    putAll(m);
}

public ConcurrentSkipListMap(SortedMap<K, ? extends V> m) {
    this.comparator = m.comparator();
    initialize();
    buildFromSorted(m);
}

private void initialize() {
    keySet = null;
    entrySet = null;
    values = null;
    descendingMap = null;
    head = new HeadIndex<K,V>(new Node<K,V>(null, BASE_HEADER, null),
                              null, null, 1);
}

//该方法是在构造方法中调用的，不会被并行访问，所以构建跳跃表的过程中没有使用CAS
private void buildFromSorted(SortedMap<K, ? extends V> map) {
    if (map == null)
        throw new NullPointerException();

    HeadIndex<K,V> h = head;
    Node<K,V> basepred = h.node;

    ArrayList<Index<K,V>> preds = new ArrayList<Index<K,V>>();
    //初始化数组元素的
    for (int i = 0; i <= h.level; ++i)
        preds.add(null);

    Index<K,V> q = h;
    //将每一层的head节点保存到preds中，下面的while循环过程中会把同一层级的最右侧的节点保存到preds中
    for (int i = h.level; i > 0; --i) {
        preds.set(i, q);
        q = q.down;
    }

    Iterator<? extends Map.Entry<? extends K, ? extends V>> it =
        map.entrySet().iterator();
    //因为SortedMap中的节点是已经排序好了，所以这里直接构建跳跃表即可，不需要为新节点查找插入位置
    while (it.hasNext()) {
        Map.Entry<? extends K, ? extends V> e = it.next();
        //生成一个随机数
        int rnd = ThreadLocalRandom.current().nextInt();
        int j = 0;
        //计算新节点的层级
        if ((rnd & 0x80000001) == 0) { //如果rnd是一个偶数
            do {
                ++j;
            } while (((rnd >>>= 1) & 1) != 0); //将rnd右移1位，即除以2，如果结果是偶数就终止循环，否则继续
            //如果大于head的level
            if (j > h.level) j = h.level + 1;
        }
        K k = e.getKey();
        V v = e.getValue();
        if (k == null || v == null)
            throw new NullPointerException();
        Node<K,V> z = new Node<K,V>(k, v, null);
        basepred.next = z;
        basepred = z;
        if (j > 0) {
            Index<K,V> idx = null;
            //从层级1开始往上逐一处理
            for (int i = 1; i <= j; ++i) {
                idx = new Index<K,V>(z, idx, null);
                if (i > h.level)
                    //创建一个新的head节点
                    h = new HeadIndex<K,V>(h.node, h, idx, i);

                if (i < preds.size()) {
                    preds.get(i).right = idx; //index作为上一个节点的right节点
                    preds.set(i, idx); //更新i，即当前节点是这一层级中的最右边的节点
                } else
                    //i>h.level的情形
                    preds.add(idx);
            }
        }
    }
    //更新head
    head = h;
}
```

1. put实现分析。

```java
public V put(K key, V value) {
    if (value == null)
        throw new NullPointerException();
    return doPut(key, value, false);
}

public V putIfAbsent(K key, V value) {
    if (value == null)
        throw new NullPointerException();
    return doPut(key, value, true);
}

private V doPut(K key, V value, boolean onlyIfAbsent) {
    Node<K,V> z;             // added node
    if (key == null)
        throw new NullPointerException();
    Comparator<? super K> cmp = comparator;
    //在next链表中找到合适的位置，将新节点插入进去
    outer: for (;;) {
        //findPredecessor方法返回当前Map中小于key的最大的一个索引节点所指向的节点，且该索引节点位于最底层了，然后遍历通过next属性构成的链表
        //直到找到一个大于key的节点，即下面c小于0的情形，将新节点插入到该节点的前面
        for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
            if (n != null) {
                Object v; int c;
                Node<K,V> f = n.next;
                if (n != b.next) //next节点变了，终止内层for循环，通过外层的for循环重新获取b和n
                    break;
                if ((v = n.value) == null) {//n被删除了
                    n.helpDelete(b, f);
                    break;
                }
                if (b.value == null || v == n) //b被删除了，下一次findPredecessor会将b从链表中移除
                    break;
                if ((c = cpr(cmp, key, n.key)) > 0) { //如果key大于n.key,继续遍历通过next属性构成的链表中的下一个节点
                    b = n;
                    n = f;
                    continue;
                }
                if (c == 0) {
                    //如果找到key相等的节点，
                    if (onlyIfAbsent || n.casValue(v, value)) {
                        //onlyIfAbsent为true表示不需要修改，为false则修改value，返回原来的value
                        @SuppressWarnings("unchecked") V vv = (V)v;
                        return vv;
                    }
                    break; //如果cas修改失败，则继续for循环尝试修改，直到修改成功为止
                }
                //c小于0会进入下面的new Node
            }
            //如果n为null或者c小于0，c小于0时，b.key < key <n.key
            //创建一个新节点，插入到b和n的中间
            z = new Node<K,V>(key, value, n);
            if (!b.casNext(n, z))
                break;         //如果修改失败则重试
            break outer;  //终止外层的for循环
        } //内层for循环
    }//外层for循环

    int rnd = ThreadLocalRandom.nextSecondarySeed();
    if ((rnd & 0x80000001) == 0) { //如果是偶数
        //level表示新节点的层级，从1开始增加，0表示最底层的一个层架
        int level = 1, max;
        while (((rnd >>>= 1) & 1) != 0) //将rnd除以2以后，如果是奇数level加1
            ++level;
        //idx表示当前节点通过down属性构成的一个链表，所有节点都指向新节点z    
        Index<K,V> idx = null;
        HeadIndex<K,V> h = head;
        if (level <= (max = h.level)) { //如果level小于head的level，则创建一个通过down属性构成的Index链表
            for (int i = 1; i <= level; ++i)
                //idx是作为新节点的down节点的
                idx = new Index<K,V>(z, idx, null);
        }
        else { //如果level大于head的level，需要增加head节点的层级
            level = max + 1; //比head大的话，只能在原来的基础上加1
            //创建一个Index数组
            @SuppressWarnings("unchecked")Index<K,V>[] idxs =
                (Index<K,V>[])new Index<?,?>[level+1]; //加1是为了加上底层0对应的Index
            //初始化数组元素
            for (int i = 1; i <= level; ++i)
                idxs[i] = idx = new Index<K,V>(z, idx, null);
            for (;;) {
                h = head;
                int oldLevel = h.level;
                if (level <= oldLevel) //初始状态下level肯定大于oldLevel
                    break;
                HeadIndex<K,V> newh = h;
                Node<K,V> oldbase = h.node;
                //创建新层级对应的head节点，down节点指向原来的head，right节点指向新节点z对应的索引节点
                //通常oldLevel就比level小1，即只创建一个HeadIndex节课
                for (int j = oldLevel+1; j <= level; ++j)
                    newh = new HeadIndex<K,V>(oldbase, newh, idxs[j], j);
                if (casHead(h, newh)) { //cas修改head，如果修改失败说明其他线程在修改head，则重新for循环读取head
                    h = newh; //使用最新的head
                    idx = idxs[level = oldLevel]; //此处level等于oldLevel
                    break;
                }
            }
        }
        //如果新节点z对应的level不是0，则创建对应层级的down链表，如果层级大于head的还需要调整head的层级，创建新的head节点，新层级的head节点的right节点
        //就指向新节点z
        // find insertion points and splice in
        splice: for (int insertionLevel = level;;) {
            int j = h.level; 
            for (Index<K,V> q = h, r = q.right, t = idx;;) {
                if (q == null || t == null)
                    break splice;
                if (r != null) {
                    Node<K,V> n = r.node;
                    // compare before deletion check avoids needing recheck
                    int c = cpr(cmp, key, n.key);
                    if (n.value == null) { //r节点是无效的
                        if (!q.unlink(r)) //将r从q的right链表中移除，移除失败重新读取q
                            break;
                        r = q.right;//获取新的right节点
                        continue;
                    }
                    if (c > 0) {//key大于n.key，则继续遍历下一个right
                        q = r;
                        r = r.right;
                        continue;
                    }
                }
                //如果c小于等于0，即key小于等于n.key,或者r等于null
                //初始状态j肯定是大于insertionLevel，后面没for循环一次，j减1，j就会等于insertionLevel
                if (j == insertionLevel) {
                    if (!q.link(r, t))  //将t插入到q的right链表中，插入到r的前面
                        break; //link失败，重新读取h
                    //link成功    
                    if (t.node.value == null) { //正常不可能为null，极端并发下可能为null，因为此时已经将新节点插入到链表中了，有可能被其他某个线程给删除了
                        findNode(key); //通过findNode将该节点移除，然后终止外层for循环
                        break splice;
                    }
                    //如果相等了，j和insertionLevel都会减1，后面的for循环中两个都会相等，如果不等则只是j减1
                    if (--insertionLevel == 0) //到底层节点了，终止外层for循环
                        break splice;
                }


                if (--j >= insertionLevel //j肯定大于等于insertionLevel
                    && j < level) //如果j大于level，说明未遍历到新节点所在的最高层级，小于level了说明上一个层级的right节点处理好了，需要遍历下一个层级
                    t = t.down;
                //遍历下一个层级的节点    
                q = q.down;
                r = q.right;
            } //内层for循环
        }//外层for循环
    }//if结束
    return null;
}

//从head节点开始遍历，找到小于key的最大的一个索引节点指向的节点
private Node<K,V> findPredecessor(Object key, Comparator<? super K> cmp) {
    if (key == null)
        throw new NullPointerException(); // don't postpone errors
    for (;;) {
        //从head开始往后遍历
        for (Index<K,V> q = head, r = q.right, d;;) {
            if (r != null) {
                Node<K,V> n = r.node;
                K k = n.key;
                if (n.value == null) { //r节点即将被删除
                    //将r从q的right链表中移除，如果q的node的value为null或者cas失败都会则返回false
                    if (!q.unlink(r)) 
                        break;      //终止内层for循环，通过外层的for循环重新开始，重新读取head
                    r = q.right;         //如果unlink成功，重新读取r
                    continue;
                }
                //n.value不为null
                if (cpr(cmp, key, k) > 0) {
                    //如果key大于k，继续遍历下一个right
                    q = r;
                    r = r.right;
                    continue;
                }
            }
            //如果key小于k 或者r等于null
            //如果down节点为null，说明已经到最下面的一层节点了，如果不为null，则继续遍历下一层节点
            //最简单的一个场景，head只有一个层级，即down为null，right不为null，会沿着right属性构成的链表一直遍历直到遇到一个大于key的节点
            //此时q就是小于key的最大的一个节点
            //如果down不为null，则遍历down节点的right，注意down节点和q节点指向的node是同一个，同样的沿着right属性构成的链表一直遍历直到遇到
            //一个大于key的节点,q就是上一个小于key的最大的节点
            if ((d = q.down) == null) 
                return q.node;
            //如果down不为null，则继续用down的right节点跟key比    
            q = d;
            r = d.right;
        }
    }
}
//c不为空null，通过c比较大小，否则强转成Comparable比较
static final int cpr(Comparator c, Object x, Object y) {
    return (c != null) ? c.compare(x, y) : ((Comparable)x).compareTo(y);
}

private Node<K,V> findNode(Object key) {
    if (key == null)
        throw new NullPointerException(); // don't postpone errors
    Comparator<? super K> cmp = comparator;
    outer: for (;;) {
        for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
            Object v; int c;
            if (n == null) //没有匹配的节点，终止外层循环，返回null
                break outer;
            Node<K,V> f = n.next;
            if (n != b.next)  //重新读取b
                break;
            if ((v = n.value) == null) {//n被删除了
                n.helpDelete(b, f);
                break;
            }
            if (b.value == null || v == n)  //b呗删除了
                break;
            if ((c = cpr(cmp, key, n.key)) == 0)//找到目标节点
                return n;
            if (c < 0) //没有匹配节点，终止外层循环，返回null
                break outer;
            //c大于0，继续遍历下一个节点    
            b = n;
            n = f;
        }
    }
    return null;
}

private boolean casHead(HeadIndex<K,V> cmp, HeadIndex<K,V> val) {
    return UNSAFE.compareAndSwapObject(this, headOffset, cmp, val);
}

```

如图5-19所示为跳查表查找示意图。

在底层，节点按照从小到大的顺序排列，上面的index层间隔地串在一起，因为从小到大排列。查找的时候，从顶层index开始，自左往右、自上往下，形成图示的遍历曲线。假设要查找的元素是32，遍历过程如下：

先遍历第2层Index，发现在21的后面；
从21下降到第1层Index，从21往后遍历，发现在21和35之间；
从21下降到底层，从21往后遍历，最终发现在29和35之间。

在整个的查找过程中，范围不断缩小，最终定位到底层的两个元素之间。

![image-20210811173223498](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210811173223498.png)

关于上面的put（..）函数，有一个关键点需要说明：在通过findPredecessor找到了待插入的元素在[b，n]之间之后，并不能马上插入。因为其他线程也在操作这个链表，b、n都有可能被删除，所以在插入之前执行了一系列的检查逻辑，而这也正是无锁链表的复杂之处。

2. remove（..）分析

  remove方法的核心实现是doRemove，先按照同样的方式遍历找到目标节点，然后将该节点的value置为null，紧接着在该节点后插入一个空的marker节点，如果成功了则将该节点和之前插入对的marker节点一起从链表中移除，插入marker节点的原因就是为了保证后面的从链表移除的动作可以顺利完成，最后再移除该节点对应的索引节点，将其从同一层级的前一个节点的right链表中移除。 最后两步的移除动作，也可以并行的被其他线程执行，他们判断所引用的Node节点的value为null，会自动执行对应的删除动作，所以删除动作都是CAS实现的。

```java
public V remove(Object key) {
    return doRemove(key, null);
}

public boolean remove(Object key, Object value) {
    if (key == null)
        throw new NullPointerException();
    return value != null && doRemove(key, value) != null;
}

//将等于key的节点的value替换成指定值，返回替换前的值，相当于put方法
public V replace(K key, V value) {
    if (key == null || value == null)
        throw new NullPointerException();
    for (;;) {
        Node<K,V> n; Object v;
        if ((n = findNode(key)) == null)
            return null; //如果没有找到目标节点
        //校验n.value不为空是为了避免该节点被删除了
        //然后cas修改value    
        if ((v = n.value) != null && n.casValue(v, value)) {
            @SuppressWarnings("unchecked") V vv = (V)v;
            return vv;//返回以前的值
        }
    }
}

//要求key和value都相等才替换，返回替换是否成功
public boolean replace(K key, V oldValue, V newValue) {
    if (key == null || oldValue == null || newValue == null)
        throw new NullPointerException();
    for (;;) {
        Node<K,V> n; Object v;
        if ((n = findNode(key)) == null)
            return false;
        if ((v = n.value) != null) {
            if (!oldValue.equals(v)) //如果当前值不是oldValue返回false
                return false;
            if (n.casValue(v, newValue)) //如果cas修改成功，返回true
                return true;
        }
    }
}

public void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
    if (function == null) throw new NullPointerException();
    V v;
    //获取第一个有效节点后，通过next遍历所有的节点
    for (Node<K,V> n = findFirst(); n != null; n = n.next) {
        while ((v = n.getValidValue()) != null) { //获取有效value
            V r = function.apply(n.key, v);
            //返回值为空，抛出异常
            if (r == null) throw new NullPointerException();
            if (n.casValue(v, r)) //cas修改value成功则终止内层while循环，处理下一个节点
                break;
        }
    }
}

//获取第一个有效节点，其实就是head.node.next
final Node<K,V> findFirst() {
    for (Node<K,V> b, n;;) {
        if ((n = (b = head.node).next) == null)
            return null; //如果next为null，返回null
        if (n.value != null)
            return n;  
        //如果n.value为null，则删除该节点    
        n.helpDelete(b, n.next);
    }
}

final V doRemove(Object key, Object value) {
    if (key == null)
        throw new NullPointerException();
    Comparator<? super K> cmp = comparator;
    outer: for (;;) {
        for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
            Object v; int c;
            if (n == null)  //没有匹配的节点，终止外层for循环，返回null
                break outer;
            Node<K,V> f = n.next;
            if (n != b.next)  //next节点被修改了，重新读取b
                break;
            if ((v = n.value) == null) { //n被删除了
                n.helpDelete(b, f); //删除节点n
                break;
            }
            if (b.value == null || v == n) // b被删除了，重新读取b
                break;
            if ((c = cpr(cmp, key, n.key)) < 0) //key小于n.key，不会有匹配的节点了
                break outer;
            if (c > 0) { //key大于n.key，继续遍历下一个next节点
                b = n;
                n = f;
                continue;
            }
            //c等于0
            if (value != null && !value.equals(v)) //value不为空跟当前节点的value不一致，终止外层for循环，返回null 
                break outer;
            if (!n.casValue(v, null)) //cas修改失败，内层for循环重试
                break;
            //casValue成功，在n和f之间插入一个空节点，如果成功则将b的next节点修改成f
            if (!n.appendMarker(f) || !b.casNext(n, f))
                //value已经置为null，其他线程不可能修改n的next属性，此时只有修改b的next属性会失败
                findNode(key);   //上述操作有一个失败则执行findNode，借此清除节点和关联的索引节点
            else {
                //修改成功，通过findPredecessor清理关联的Index节点
                findPredecessor(key, cmp);      // clean index
                if (head.right == null) //如果head的right节点为null，则尝试减少层级
                    tryReduceLevel();
            }
            @SuppressWarnings("unchecked") V vv = (V)v;
            return vv;
        }
    }
    return null;
}

public void clear() {
    for (;;) {
        Node<K,V> b, n;
        HeadIndex<K,V> h = head, d = (HeadIndex<K,V>)h.down;
        if (d != null)
            casHead(h, d);  //修改head，原head的right链表就都被移除了
        //d为null，遍历到最底层的节点了     
        else if ((b = h.node) != null && (n = b.next) != null) {
            Node<K,V> f = n.next;     // remove values
            if (n == b.next) { //b未改变
                Object v = n.value;
                if (v == null) //n被删除了
                    n.helpDelete(b, f);
                //v不为null，将n的value置为null并插入marker，将b的next由n修改成f    
                else if (n.casValue(v, null) && n.appendMarker(f))
                    b.casNext(n, f); //如果cas成功，则n从链表中移除了，如果失败了，假如此时其他线程在并行的往b后面插入一个节点，则下次遍历到n时通过helpDelete将其从链表中移除
            }
        }
        else
            break;
    }
}

private void tryReduceLevel() {
    HeadIndex<K,V> h = head;
    HeadIndex<K,V> d;
    HeadIndex<K,V> e;
    if (h.level > 3 &&
        (d = (HeadIndex<K,V>)h.down) != null &&
        (e = (HeadIndex<K,V>)d.down) != null &&
        e.right == null &&
        d.right == null &&
        h.right == null && //从head往下连续三个层级的right都为null
        casHead(h, d) && //将head从h修改成d
        h.right != null) // 最后一次检查，如果不为null则恢复
        casHead(d, h);   // try to backout
}
```

3. get

```java
public V get(Object key) {
    return doGet(key);
}

public V getOrDefault(Object key, V defaultValue) {
    V v;
    return (v = doGet(key)) == null ? defaultValue : v;
}

//doGet方法的实现和findNode方法是基本一致的
private V doGet(Object key) {
    if (key == null)
        throw new NullPointerException();
    Comparator<? super K> cmp = comparator;
    outer: for (;;) {
        for (Node<K,V> b = findPredecessor(key, cmp), n = b.next;;) {
            Object v; int c;
            if (n == null) //没有匹配节点
                break outer;
            Node<K,V> f = n.next;
            if (n != b.next)  //重新读取b
                break;
            if ((v = n.value) == null) {    //n被删除了
                n.helpDelete(b, f);
                break;
            }
            if (b.value == null || v == n)  // b被删除了
                break;
            if ((c = cpr(cmp, key, n.key)) == 0) { //找到目标节点
                @SuppressWarnings("unchecked") V vv = (V)v;
                return vv;
            }
            if (c < 0)
                break outer; 没有匹配节点
                    //c大于0，则遍历下一个next节点    
                    b = n;
            n = f;
        }
    }
    return null;
}
```

##### 5.6.2 ConcurrentSkipListSet

如下面代码所示，ConcurrentSkipListSet只是对ConcurrentSkipListMap的简单封装，此处不再进一步展开叙述。

```java
public class ConcurrentSkipListSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable {
    //封装了一个ConcurrentSkipListMap
    private final ConcurrentNavigableMap<E,Object> m;
    public ConcurrentSkipListSet() {
        m = new ConcurrentSkipListMap<E,Object>();
    }
      public boolean add(E e) {
        return m.putIfAbsent(e, Boolean.TRUE) == null;
    }
}
```

### 6.线程池与Future

#### 6.1 线程池的实现原理

图6-1所示为线程池的实现原理：调用方不断地向线程池中提交任务；线程池中有一组线程，不断地从队列中取任务，这是一个典型的生产者—消费者模型

![image-20210811181247859](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210811181247859.png)

原要实现这样一个线程池，有几个问题需要考虑：
（1）队列设置多长？如果是无界的，调用方不断地往队列中放任务，可能导致内存耗尽。如果是有界的，当队列满了之后，调用方如何处理？
（2）线程池中的线程个数是固定的，还是动态变化的？
（3）每次提交新任务，是放入队列？还是开新线程？
（4）当没有任务的时候，线程是睡眠一小段时间？还是进入阻塞？如果进入阻塞，如何唤醒？

针对问题（4），有3种做法：

做法（1）：不使用阻塞队列，只使用一般的线程安全的队列，也无阻塞—唤醒机制。当队列为空时，线程池中的线程只能睡眠一会儿，然后醒来去看队列中有没有新任务到来，如此不断轮询。

做法（2）：不使用阻塞队列，但在队列外部、线程池内部实现了阻塞—唤醒机制。

做法（3）：使用阻塞队列。

很显然，做法（3）最完善，既避免了线程池内部自己实现阻塞—唤醒机制的麻烦，也避免了做法（1）的睡眠—轮询带来的资源消耗和延迟。正因为如此，接下来要讲的ThreadPoolExector/ScheduledThreadPoolExecutor都是基于阻塞队列来实现的，而不是一般的队列，至此，各式各样的阻塞队列就要派上用场了。

#### 6.2 线程池的类继承体系

在正式了解线程池的实现原理之前，先对线程池的类继承体系进行一个宏观介绍，如图6-2所示。

![image-20210811181521246](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210811181521246.png)

在这里，有两个核心的类：ThreadPoolExector和ScheduledThreadPoolExecutor，后者不仅可以执行某个任务，还可以周期性地执行任务。

向线程池中提交的每个任务，都必须实现Runnable接口，通过最上面的Executor接口中的execute（Runnable command）向线程池提交任务。

然后，在 ExecutorService 中，定义了线程池的关闭接口 shutdown（），还定义了可以有返回值的任务，也就是Callable，接下来的章节会详细介绍。

#### 6.3 ThreadPoolExector

##### 6.3.1 核心数据结构

基于线程池的实现原理，下面看一下ThreadPoolExector的核心数据结构。

```JAVA
public class ThreadPoolExecutor extends AbstractExecutorService {
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));//状态变量
    private final BlockingQueue<Runnable> workQueue;//阻塞队列存放任务
    private final ReentrantLock mainLock = new ReentrantLock();//对线程池内各种变量互斥访问控制
    private final HashSet<Worker> workers = new HashSet<Worker>();//线程集合
}
```

每一个线程是一个Worker对象。Worker是ThreadPoolExector的内部类，核心数据结构如下：

```java
private final class Worker extends AbstractQueuedSynchronizer implements Runnable{
    final Thread thread;//Worker封装的线程
    Runnable firstTask;//woker接收到的第一个任务
    volatile long completedTasks;//worker执行完毕任务数
}
```

由定义会发现，Worker继承于AQS，也就是说Worker本身就是一把锁。这把锁有什么用处呢？在接下来分析线程池的关闭、线程执行任务的过程时会了解到。

##### 6.3.2 核心配置参数解释

针对本章最开始提出的线程池实现的几个问题，ThreadPoolExecutor在其构造函数中提供了几个核心配置参数，来配置不同策略的线程池。了解了清楚每个参数的含义，也就明白了线程池的各种不同策略。

```java
public ThreadPoolExecutor(int corePoolSize,//固定线程数
                          int maximumPoolSize,//最大线程数
                          long keepAliveTime,//超出的存活时间
                          TimeUnit unit,//时间单位
                          BlockingQueue<Runnable> workQueue,//任务阻塞队列
                          ThreadFactory threadFactory,//线程工厂
                          RejectedExecutionHandler handler) {//拒绝策略
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

上面的各个参数，解释如下：
（1）corePoolSize：在线程池中始终维护的线程个数。
（2）maxPoolSize：在corePooSize已满、队列也满的情况下，扩充线程至此值。
（3）keepAliveTime/TimeUnit：maxPoolSize 中的空闲线程，销毁所需要的时间，总线程数收缩回corePoolSize。
（4）blockingQueue：线程池所用的队列类型。
（5）threadFactory：线程创建工厂，可以自定义，也有一个默认的。
（6）RejectedExecutionHandler：corePoolSize 已满，队列已满，maxPoolSize 已满，最后的拒绝策略。
下面来看这6个配置参数在任务的提交过程中是怎么运作的。在每次往线程池中提交任务的时候，有如下的处理流程：

step1：判断当前线程数是否大于或等于corePoolSize。如果小于，则新建线程执行；如果大于，则进入step2。
step2：判断队列是否已满。如未满，则放入；如已满，则进入step3。
step3：判断当前线程数是否大于或等于maxPoolSize。如果小于，则新建线程执行；如果大于，则进入step4。
step4：根据拒绝策略，拒绝任务。

总结一下：首先判断corePoolSize，其次判断blockingQueue是否已满，接着判断maxPoolSize，最后使用拒绝策略。
很显然，基于这种流程，如果队列是无界的，将永远没有机会走到step 3，也即maxPoolSize没有使用，也一定不会走到step 4。

##### 6.3.3 线程池的优雅关闭

在1.1章节中，讲了线程的优雅关闭，是一个很需要注意的地方。而线程池的关闭，较之线程的关闭更加复杂。当关闭一个线程池的时候，有的线程还正在执行某个任务，有的调用者正在向线程池提交任务，并且队列中可能还有未执行的任务。因此，关闭过程不可能是瞬时的，而是需要一个平滑的过渡，这就涉及线程池的完整生命周期管理。

###### 1.线程池的生命周期

在JDK 7中，把线程数量（workerCount）和线程池状态（runState）这两个变量打包存储在一个字段里面，即ctl变量。如图6-3所示，最高的3位存储线程池状态，其余29位存储线程个数。而在JDK 6中，这两个变量是分开存储的。

![image-20210811183216315](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210811183216315.png)

```java
//初始,线程池状态为running,线程数为0
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;//最高三位表示线程池状态
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// 线程池5种状态
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

// Packing and unpacking ctl ctl分解出runstate和workercount
private static int runStateOf(int c)     { return c & ~CAPACITY; }
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }//rs和wc打包成ctl
```

由上面的代码可以看到，ctl变量被拆成两半，最高的3位用来表示线程池的状态，低的29位表示线程的个数。线程池的状态有五种，分别是RUNNING、SHUTDOWN、STOP、TIDYING和TERMINATED。

下面分析状态之间的迁移过程，如图6-4所示。

![image-20210811184440122](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210811184440122.png)

线程池有两个关闭函数，shutdown（）和shutdownNow（），这两个函数会让线程池切换到不同的状态。在队列为空，线程池也为空之后，进入 TIDYING 状态；最后执行一个钩子函数terminated（），进入TERMINATED状态，线程池才“寿终正寝”。

这里的状态迁移有一个非常关键的特征：从小到大迁移，-1，0，1，2，3，只会从小的状态值往大的状态值迁移，不会逆向迁移。例如，当线程池的状态在TIDYING=2时，接下来只可能迁移到TERMINATED=3，不可能迁移回STOP=1或者其他状态。

除 terminated（）之外，线程池还提供了其他几个钩子函数，这些函数的实现都是空的。如果想实现自己的线程池，可以重写这几个	函数。

```java
protected void beforeExecute(Thread t, Runnable r) { }
protected void afterExecute(Runnable r, Throwable t) { }
protected void terminated() { }
```

###### 2.正确关闭线程池的步骤

通过上面的分析，我们知道了线程池的关闭需要一个过程，在调用 shutDown（）或者shutdownNow（）之后，线程池并不会立即关闭，接下来需要调用 awaitTermination 来等待线程池关闭。关闭线程池的正确步骤如下：

```java
threadPoolExecutor.shutdown();

threadPoolExecutor.shutdownNow();
try {
    boolean loop = true;
    do {
        loop = !threadPoolExecutor.awaitTermination(10,TimeUnit.SECONDS);
    }while (loop);
}catch (Exception e){
    
}
```

awaitTermination（..）函数的内部实现很简单，如下所示。不断循环判断线程池是否到达了最终状态TERMINATED，如果是，就返回；如果不是，则通过termination条件变量阻塞一段时间，“苏醒”之后，继续判断。

```java
public boolean awaitTermination(long timeout, TimeUnit unit)
    throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (;;) {
            if (runStateAtLeast(ctl.get(), TERMINATED))//判断状态是否是TERMINATED
                return true;
            if (nanos <= 0)
                return false;
            nanos = termination.awaitNanos(nanos);
        }
    } finally {
        mainLock.unlock();
    }
}
```

###### 3.shutdown（）与shutdownNow（）的区别

下面的代码展示了shutdown（）和shutdownNow（）的区别：
（1）前者不会清空任务队列，会等所有任务执行完成，后者再清空任务队列。
（2）前者只会中断空闲的线程，后者会中断所有线程。

```java
public void shutdown() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();//检测是否有关闭权限
        advanceRunState(SHUTDOWN);//状态设置到SHUTDOWN
        interruptIdleWorkers();//中断空闲线程
        onShutdown(); // hook for ScheduledThreadPoolExecutor 钩子函数 空的
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
}
public List<Runnable> shutdownNow() {
    List<Runnable> tasks;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        advanceRunState(STOP);//状态设为STOP
        interruptWorkers();//中断所有线程
        tasks = drainQueue();//清空队列
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
    return tasks;
}
```

下面看一下在上面的代码里中断空闲线程和中断所有线程的区别。

```java
private void interruptIdleWorkers() {
    interruptIdleWorkers(false);
}
private void interruptIdleWorkers(boolean onlyOne) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (Worker w : workers) {
            Thread t = w.thread;
            if (!t.isInterrupted() && w.tryLock()) {//关键:trylock成功,说明线程是空闲状态,不成功,说明线程持有锁,正在执行任务
                try {
                    t.interrupt();
                } catch (SecurityException ignore) {
                } finally {
                    w.unlock();
                }
            }
            if (onlyOne)
                break;
        }
    } finally {
        mainLock.unlock();
    }
}
private void interruptWorkers() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (Worker w : workers)
            w.interruptIfStarted();//不管线程状态,一律发送中断信号
    } finally {
        mainLock.unlock();
    }
}
```

关键区别点在tryLock（）：一个线程在执行一个任务之前，会先加锁（在6.3.4节会详细讲），这意味着通过是否持有锁，可以判断出线程是否处于空闲状态。tryLock（）如果调用成功，说明线程处于空闲状态，向其发送中断信号；否则不发送。

在上面的代码中，shutdown（） 和shutdownNow（）都调用了tryTerminate（）函数，如下所示。

```java
final void tryTerminate() {
    for (;;) {
        int c = ctl.get();
        if (isRunning(c) ||
            runStateAtLeast(c, TIDYING) ||
            (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
            return;
        if (workerCountOf(c) != 0) { // Eligible to terminate
            interruptIdleWorkers(ONLY_ONE);
            return;
        }
//workqueue为空,workcount为0,进入
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {//状态切换为TIDYING
                try {
                    terminated();//调钩子函数
                } finally {
                    ctl.set(ctlOf(TERMINATED, 0));//TIDYING改为TERMINATED
                    termination.signalAll();//通知awaitTermination()
                }
                return;
            }
        } finally {
            mainLock.unlock();
        }
        // else retry on failed CAS
    }
}
```

tryTerminate（）不会强行终止线程池，只是做了一下检测：当workerCount为0，workerQueue为空时，先把状态切换到TIDYING，然后调用钩子函数terminated（）。当钩子函数执行完成时，把状态从 TIDYING 改为 TERMINATED，接着调用 termination.sinaglAll（），通知前面阻塞在awaitTermination的所有调用者线程。

所以，TIDYING和TREMINATED的区别是在二者之间执行了一个钩子函数terminated（），目前是一个空实现。

##### 6.3.4 任务的提交过程分析

提交任务的函数如下：

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {//当前线程小于corePoolSize,开新线程
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    //如果当前线程数>=corePoolSize,调用workQueue.offer放入队列
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }//放队列失败,开新线程
    else if (!addWorker(command, false))
        reject(command);//线程数大于maxPoolSize,调用拒绝策略
}
//此函数用于开一个新线程
//第二个参数为true,corePoolSize作为上界
//第二个参数为false,maxPoolSize作为上界
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.状态>=SHUTDOWN,说明线程池进入关闭过程
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;//线程数超过上界,直接返回false
            if (compareAndIncrementWorkerCount(c))//workCount成功加一,跳出for循环
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)//runstate发生变化,重新开始for循环
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }
	//workcount成功加一,开始加线程
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);//创建一个线程
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w);//线程加入集合
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                t.start();//加入成功,启动该线程
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)//加入失败
            addWorkerFailed(w);//workcount-1
    }
    return workerStarted;
}
```

##### 6.3.5 任务的执行过程分析

在上面的任务提交过程中，可能会开启一个新的Worker，并把任务本身作为firstTask赋给该Worker。但对于一个Worker来说，不是只执行一个任务，而是源源不断地从队列中取任务执行，这是一个不断循环的过程。

下面来看Woker的run（）方法的实现过程。

```java
private final class Worker extends AbstractQueuedSynchronizer implements Runnable{
    Worker(Runnable firstTask) {
        setState(-1); // inhibit interrupts until runWorker 初始状态-1
        this.firstTask = firstTask;
        this.thread = getThreadFactory().newThread(this);
    }
    public void run() {
        runWorker(this);
    }
    //核心函数
    final void runWorker(Worker w) {
        Thread wt = Thread.currentThread();
        Runnable task = w.firstTask;
        w.firstTask = null;
        w.unlock(); // allow interrupts
        boolean completedAbruptly = true;
        try {
            while (task != null || (task = getTask()) != null) {//不断从队列取任务执行
                w.lock();//关键:执行任务前加锁,对应前面shutdown()的trylock
                if ((runStateAtLeast(ctl.get(), STOP) ||
                     (Thread.interrupted() &&
                      runStateAtLeast(ctl.get(), STOP))) &&
                    !wt.isInterrupted())
                    wt.interrupt();
                	//拿到任务,执行前重新检测线程池的状态,如果已开始关闭,就自己给自己发中断信号
                try {
                    beforeExecute(wt, task);//任务之前的钩子函数,空实现
                    Throwable thrown = null;
                    try {
                        task.run();//执行任务代码
                    } catch (RuntimeException x) {
                        thrown = x; throw x;
                    } catch (Error x) {
                        thrown = x; throw x;
                    } catch (Throwable x) {
                        thrown = x; throw new Error(x);
                    } finally {
                        afterExecute(task, thrown);//任务之后钩子函数,空实现
                    }
                } finally {
                    task = null;
                    w.completedTasks++;//完成任务,completedTasks累加
                    w.unlock();//释放锁
                }
            }
            completedAbruptly = false;//判断worker是正常退出,还是中断退出,还是异常退出
        } finally {
            processWorkerExit(w, completedAbruptly);//worker退出
        }
    }
}
```

###### 1.shutdown（）与任务执行过程综合分析

把任务的执行过程和上面的线程池的关闭过程结合起来进行分析，当调用 shutdown（）的时候，可能出现以下几种场景：

场景1：当调用shutdown（）的时候，所有线程都处于空闲状态。这意味着任务队列一定是空的。此时，所有线程都会阻塞在 getTask（）函数的地方。然后，所有线程都会收到interruptIdleWorkers（）发来的中断信号，getTask（）返回null，所有Worker都会退出while循环，之后执行processWorkerExit。

场景2：当调用shutdown（）的时候，所有线程都处于忙碌状态。此时，队列可能是空的，也可能是非空的。interruptIdleWorkers（）内部的tryLock调用失败，什么都不会做，所有线程会继续执行自己当前的任务。之后所有线程会执行完队列中的任务，直到队列为空，getTask（）才会返回null。之后，就和场景1一样了，退出while循环。

场景3：当调用shutdown（）的时候，部分线程忙碌，部分线程空闲。有部分线程空闲，说明队列一定是空的，这些线程肯定阻塞在 getTask（）函数的地方。空闲的这些线程会和场景1一样处理，不空闲的线程会和场景2一样处理。

下面看一下getTask（）函数的内部细节。

```java
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        //关键:rs>=stop,即调用了shutdownnow()此处返回null
        //rs>=shutdown,即调用了shutdown(),队列为空,此处返回null
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;//返回null,worker退出while循环,死亡
        }

        int wc = workerCountOf(c);

        // Are workers subject to culling?
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {//关键:队列空,阻塞poll/take,线程闲置,前置带超时,后置不带
            //收到中断信号,抛异常,对应上文场景一
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```

###### 2.shutdownNow（） 与任务执行过程综合分析

和上面的 shutdown（）类似，只是多了一个环节，即清空任务队列。在第1章中已经讲到，如果一个线程正在执行某个业务代码，即使向它发送中断信号，也没有用，只能等它把代码执行完成。从这个意义上讲，中断空闲线程和中断所有线程的区别并不是很大，除非线程当前刚好阻塞在某个地方。

有兴趣的读者也可以分析在不同场景下调用shutdownNow（）所遇到的情况，此处不再展开。
当一个Worker最终退出的时候，会执行清理工作，代码如下所示。

```java
private void processWorkerExit(Worker w, boolean completedAbruptly) {
    //worker正常退出,在上面getTask里workerCount-1,此方法都是非正常退出,在此workerCount-1
    if (completedAbruptly) // If abrupt, then workerCount wasn't adjusted
        decrementWorkerCount();

    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        completedTaskCount += w.completedTasks;
        workers.remove(w);//从workers集合移除
    } finally {
        mainLock.unlock();
    }
	//和shutdown/shutdownNow一样,每个线程结束时,都调用这个函数,尝试终止整个线程池
    tryTerminate();

    //最后是一个保险:自己退出,发现线程池状态小于stop,且队列不为空,且当前没有工作线程数了,就addworker新建线程,执行完队列任务
    int c = ctl.get();
    if (runStateLessThan(c, STOP)) {
        if (!completedAbruptly) {
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
            if (min == 0 && ! workQueue.isEmpty())
                min = 1;
            if (workerCountOf(c) >= min)
                return; // replacement not needed
        }
        addWorker(null, false);
    }
}
```

##### 6.3.6 线程池的4种拒绝策略

在execute（Runnable command）的最后，调用了reject（command）执行拒绝策略，代码如下所示。

```java
final void reject(Runnable command) {
    handler.rejectedExecution(command, this);
}
private volatile RejectedExecutionHandler handler;
```

RejectedExecutionHandler 是一个接口，定义了四种实现，分别对应四种不同的拒绝策略，默认是AbortPolicy。四种策略的实现代码如下：

```java
public static class CallerRunsPolicy implements RejectedExecutionHandler {//调用者自己处理
    public CallerRunsPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            r.run();
        }
    }
}
public static class AbortPolicy implements RejectedExecutionHandler {//直接抛异常
    public AbortPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        throw new RejectedExecutionException("Task " + r.toString() +
                                             " rejected from " +
                                             e.toString());
    }
}
public static class DiscardPolicy implements RejectedExecutionHandler {//线程池直接丢掉任务
    public DiscardPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    }
}
//把队列里最老的任务替换
public static class DiscardOldestPolicy implements RejectedExecutionHandler {
    public DiscardOldestPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            e.getQueue().poll();
            e.execute(r);
        }
    }
}
```

#### 6.4 Callable与Future

execute（Runnable command）接口是无返回值的，与之相对应的是一个有返回值的接口Future submit（Callable task），这点在6.2节中已经提到。

Callable也就是一个有返回值的Runnable

```java
final ExecutorService executor = Executors.newSingleThreadExecutor();
final Future<String> callable = executor.submit(() -> {
    System.out.println("Callable");
    return "123";
});
System.out.println(callable.get());
executor.shutdown();
//Future接口,主要取返回值
public interface Future<V> {
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}

final FutureTask<String> target = new FutureTask<>(() -> "123");
new Thread(target).start();
System.out.println(target.get());
```

submit（Callable task）并不是在 ThreadPoolExecutor 里面直接实现的，而是实现在其父类AbstractExecutorService中，源码如下：

```java
public abstract class AbstractExecutorService implements ExecutorService {  
    public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task);//callable转runnable
        execute(ftask);//调用ThreadPoolExecutor的execute接口
        return ftask;
    }
}
```

从这段代码中可以看出，Callable其实是用Runnable实现的。在submit内部，把Callable通过FutureTask这个Adapter转化成Runnable，然后通过execute执行。如图6-5所示为Callable被转换成Runnable示意图。

![image-20210812102443773](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210812102443773.png)

FutureTask是一个Adapter对象。一方面，它实现了Runnable接口，也实现了Future接口；另一方面，它的内部包含了一个Callable对象，从而实现了把Callable转换成Runnable。

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
//实现了runnable & future
public class FutureTask<V> implements RunnableFuture<V> {
    private volatile int state;//cas state变量 + LockSupport.park/unpark
    private Callable<V> callable;//内部封装的callable对象
    private Object outcome; //callable执行结果
    public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }
}
public void run() {
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {//关键:callable的call()转成runable的run()
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);//返回值存入outcome变量
        }
    } finally {
        runner = null;
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);//内部调用了LockSupport.park()
    return report(s);
}
```

如图6-6所示，一方面，线程池内部的线程在执行RunTask的run（）方法；另一方面，外部多个线程又在调用get（）方法，等着返回结果，因此这个地方需要一个阻塞—通知机制。

![image-20210812103350010](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210812103350010.png)

在JDK 6中借用AQS的功能来实现阻塞—唤醒。但自JDK 7开始，既没有借用AQS的功能，也没有使用Condition的await（）/notify（）机制，而是直接基于CAS state变量+park/unpark（）来实现阻塞—唤醒机制。由于这个原理在上文讲AQS和Condition的时候已反复提及，此处就不再对awaitDone（）进一步展开分析了。

#### 6.5 ScheduledThreadPoolExecutor

ScheduledThreadPoolExecutor实现了按时间调度来执行任务，具体而言有两个方面：
（1）延迟执行任务

```java
public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);
public <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit);
```

（2）周期执行任务

```java
   public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,long initialDelay,
                                                  long period, TimeUnit unit)
 public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,long initialDelay,
                                                  long delay, TimeUnit unit)
```

这两个函数的区别如下：

AtFixedRate：按固定频率执行，与任务本身执行时间无关。但有个前提条件，任务执行时间必须小于间隔时间，例如间隔时间是5s，每5s执行一次任务，任务的执行时间必须小于5s。

WithFixedDelay：按固定间隔执行，与任务本身执行时间有关。例如，任务本身执行时间是10s，间隔2s，则下一次开始执行的时间就是12s。

##### 6.5.1 延迟执行和周期性执行的原理

从6.2节的类继承体系可以看到，ScheduledThreadPoolExecutor继承了ThreadPoolExecutor，这意味着其内部的数据结构和ThreadPoolExecutor是基本一样的，那它是如何实现延迟执行任务和周期性执行任务的呢？

延迟执行任务依靠的是DelayQueue。在5.1 节中已经提到，DelayQueue是 BlockingQueue的一种，其实现原理是二叉堆。而周期性执行任务是执行完一个任务之后，再把该任务扔回到任务队列中，如此就可以对一个任务反复执行。

不过这里并没有使用5.1节中的DelayQueue，而是在ScheduledThreadPoolExecutor内部又实现了一个特定的DelayQueue，如下所示。

```java
static class DelayedWorkQueue extends AbstractQueue<Runnable>
    implements BlockingQueue<Runnable> {
```

其原理和DelayQueue一样，但针对任务的取消进行了优化，此处不再进一步展开。下面主要看一下延迟执行和周期性执行的实现过程。

##### 6.5.2 延迟执行

```java
public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit) {
    if (command == null || unit == null)
        throw new NullPointerException();
    RunnableScheduledFuture<?> t = decorateTask(command,
        new ScheduledFutureTask<Void>(command, null,
                                      triggerTime(delay, unit)));
    delayedExecute(t);
    return t;
}
```

传进去的是一个Runnable，外加延迟时间delay。在内部通过decorateTask （ .. ） 函数把Runnable 包装成一个ScheduleFutureTask 对象，而DelayedWorkerQueue中存放的正是这种类型的对象，这种类型的对象一定实现了Delayed接口，如5.1.4节所述。

```java
private void delayedExecute(RunnableScheduledFuture<?> task) {
    if (isShutdown())
        reject(task);
    else {
        super.getQueue().add(task);//把ScheduleFutureTask放入延迟队列
        if (isShutdown() &&
            !canRunInCurrentRunState(task.isPeriodic()) &&
            remove(task))
            task.cancel(false);
        else
            ensurePrestart();
    }
}
//wc < corePoolSize会新开线程,否则什么都不做
void ensurePrestart() {
    int wc = workerCountOf(ctl.get());
    if (wc < corePoolSize)
        addWorker(null, true);
    else if (wc == 0)
        addWorker(null, false);
}
```

从上面的代码中可以看出，schedule（）函数本身很简单，就是把提交的 Runnable 任务加上delay时间，转换成ScheduledFutureTask对象，放入DelayedWorkerQueue中。任务的执行过程还是复用的ThreadPoolExecutor，延迟的控制是在DelayedWorkerQueue内部完成的。

##### 6.5.3 周期性执行



```java
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay,
                                                 long delay,TimeUnit unit) {
    //command - 要执行的任务
    //initialdelay - 首次执行的延迟时间
    //delay - 一次执行终止和下一次执行开始之间的延迟
    //unit - initialdelay 和 delay 参数的时间单位
    if (command == null || unit == null)
        throw new NullPointerException();
    if (delay <= 0)
        throw new IllegalArgumentException();
    ScheduledFutureTask<Void> sft =
        new ScheduledFutureTask<Void>(command,
                                      null,
                                      triggerTime(initialDelay, unit),
                                      unit.toNanos(-delay));//负数
    RunnableScheduledFuture<Void> t = decorateTask(command, sft);
    sft.outerTask = t;
    delayedExecute(t);
    return t;
}
//每隔多少时间，固定执行任务。
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay,
                                              long period, TimeUnit unit) {
    if (command == null || unit == null)
        throw new NullPointerException();
    if (period <= 0)
        throw new IllegalArgumentException();
    ScheduledFutureTask<Void> sft =
        new ScheduledFutureTask<Void>(command,
                                      null,
                                      triggerTime(initialDelay, unit),
                                      unit.toNanos(period));//正数
    RunnableScheduledFuture<Void> t = decorateTask(command, sft);
    sft.outerTask = t;
    delayedExecute(t);
    return t;
}
```

和schedule（..）函数的框架基本一样，也是包装一个ScheduledFutureTask对象，只是在延迟时间参数之外多了一个周期参数，然后放入DelayedWorkerQueue就结束了。

两个函数的区别在于一个传入的周期是一个负数，另一个传入的周期是一个正数，为什么要这样做呢？下面进入ScheduledFutureTask的内部一探究竟。

```java
private class ScheduledFutureTask<V>
    extends FutureTask<V> implements RunnableScheduledFuture<V> {

    private final long sequenceNumber;
    private long time;
    private final long period;

    RunnableScheduledFuture<V> outerTask = this;
    int heapIndex;

    ScheduledFutureTask(Runnable r, V result, long ns, long period) {
        super(r, result);
        this.time = ns;//延迟时间
        this.period = period;//周期
        this.sequenceNumber = sequencer.getAndIncrement();//自增序列号
    }
    //实现Delayed接口
    public long getDelay(TimeUnit unit) {
        return unit.convert(time - now(), NANOSECONDS);
    }

    //实现Comparable
    public int compareTo(Delayed other) {
        if (other == this) // compare zero if same object
            return 0;
        if (other instanceof ScheduledFutureTask) {
            ScheduledFutureTask<?> x = (ScheduledFutureTask<?>)other;
            long diff = time - x.time;
            if (diff < 0)
                return -1;
            else if (diff > 0)
                return 1;
            else if (sequenceNumber < x.sequenceNumber)//两个延迟时间相等,在比序列号
                return -1;
            else
                return 1;
        }
        long diff = getDelay(NANOSECONDS) - other.getDelay(NANOSECONDS);
        return (diff < 0) ? -1 : (diff > 0) ? 1 : 0;
    }
    public boolean isPeriodic() {
        return period != 0;
    }
    //设置下一次执行时间
    private void setNextRunTime() {
        long p = period;
        if (p > 0)
            time += p;
        else
            time = triggerTime(-p);
    }
    long triggerTime(long delay) {
        return now() +
            ((delay < (Long.MAX_VALUE >> 1)) ? delay : overflowFree(delay));
    }
    public boolean cancel(boolean mayInterruptIfRunning) {
        boolean cancelled = super.cancel(mayInterruptIfRunning);
        if (cancelled && removeOnCancel && heapIndex >= 0)
            remove(this);
        return cancelled;
    }
    //实现runnable接口
    public void run() {
        boolean periodic = isPeriodic();
        if (!canRunInCurrentRunState(periodic))
            cancel(false);
        else if (!periodic)//非周期的,执行一次就结束
            ScheduledFutureTask.super.run();
        else if (ScheduledFutureTask.super.runAndReset()) {//周期的,执行完,放回队列
            setNextRunTime();
            reExecutePeriodic(outerTask);
        }
    }
    //回到队列,等下一次执行
    void reExecutePeriodic(RunnableScheduledFuture<?> task) {
        if (canRunInCurrentRunState(true)) {
            super.getQueue().add(task);
            if (!canRunInCurrentRunState(true) && remove(task))
                task.cancel(false);
            else
                ensurePrestart();
        }
    }
}
```

withFixedDelay和atFixedRate的区别就体现在setNextRunTime里面。

如果是atFixedRate，period＞0，下一次开始执行时间等于上一次开始执行时间+period；
如果是withFixedDelay，period ＜ 0，下一次开始执行时间等于triggerTime（-p），为now+（-period），now即上一次执行的结束时间。

##### 6.6 Executors工具类

```java
public static ExecutorService newSingleThreadExecutor() {//单线程线程池
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
public static ExecutorService newFixedThreadPool(int nThreads) {//固定线程数目线程池
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
public static ExecutorService newCachedThreadPool() {//来一个任务,建一个线程
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
//单线程周期调度线程池
public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
    return new DelegatedScheduledExecutorService (new ScheduledThreadPoolExecutor(1));
}
//多线程周期调度线程池
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
```

从上面的代码中可以看出，这些不同类型的线程池，其实都是由前面的几个关键配置参数配置而成的。

在《阿里巴巴Java开发手册》中，明确禁止使用Executors创建线程池，并要求开发者直接使用ThreadPoolExector或ScheduledThreadPoolExecutor进行创建。这样做是为了强制开发者明确线程池的运行策略，使其对线程池的每个配置参数皆做到心中有数，以规避因使用不当而造成资源耗尽的风险。

### 7.ForkJoinPool

#### 7.1 ForkJoinPool用法

分治算法。其基本思路是：将一个大的任务分为若干个子任务，这些子任务分别计算，然后合并出最终结果，在这个过程中通常会用到递归。
ForkJoinPool就是JDK7提供的一种“分治算法”的多线程并行计算框架。Fork意为分叉，Join意为合并，一分一合，相互配合，形成"分治算法"。

相比于ThreadPoolExecutor，ForkJoinPool可以更好地实现计算的负载均衡，提高资源利用率。假设有5个任务，在ThreadPoolExecutor中有5个线程并行执行，其中一个任务的计算量很大，其余4个任务的计算量很小，这会导致1个线程很忙，其他4个线程则处于空闲状态。而利用ForkJoinPool，可以把大的任务拆分成很多小任务，然后这些小任务被所有的线程执行，从而实现任务计算的负载均衡



##### demo

下面通过两个简单例子看一下ForkJoinPool的用法：

###### 例子1：快排

快排有2个步骤：
第1步，利用数组的第1个元素把数组划分成两半，左边数组里面的元素小于或等于该元素，右边数组里面的元素比该元素大；
第2步，对左右的2个子数组分别排序。
可以看出，这里左右2个子数组是可以相互独立、并行计算的。因此可以利用ForkJoinPool，代码如下所示。

```java
//定义一个Task, 继承自RecursiveAc巨on, 实现其compute方法
public class SortTask extends RecursiveAction {

    final long[] array;
    final int lo;
    final int hi;
    private int THRESHOLD = 0; //For demo only

    public SortTask(long[] array) {
        this.array = array;
        this.lo = 0;
        this.hi = array.length - 1;
    }
    public SortTask(long[] array, int lo, int hi) {
        this.array = array;
        this.lo = lo;
        this.hi = hi;
    }

    @Override
    protected void compute() {

        if (lo < hi) {
            int pivot = partition(array, lo, hi); //划分
            SortTask left = new SortTask(array, lo, pivot - 1);
            SortTask right = new SortTask(array, pivot + 1, hi);
            left.fork();
            right.fork();
            left.join();
            right.join();
        }
    }

    private int partition(long[] array, int lo, int hi) {
        long x = array[hi];
        int i = lo - 1;
        for (int j = lo; j < hi; j++) {
            if (array[j] <= x) {
                i++;
                swap(array, i, j);
            }
        }
        swap(array, i + 1, hi);
        return i + 1;
    }

    private void swap(long[] array, int i, int j) {
        if (i != j) {
            long temp = array[i];
            array[i] = array[j];
            array[j] = temp;
        }
    }

    public void test() {
        final SortTask sortTask = new SortTask(array);//一个任务
        final ForkJoinPool forkJoinPool = new ForkJoinPool();//一个ForkJoinPool
        forkJoinPool.submit(sortTask);//提交任务
        forkJoinPool.shutdown();//结束 开启多线程执行子任务
        forkJoinPool.awaitQuiescence(30, TimeUnit.SECONDS);
    }
    
    public static void main(String[] args) {
        long[] a = new long[]{1, 2, 3, 6, 4, 2, 5, 3, 7, 8};
        final SortTask sortTask = new SortTask(a);
        sortTask.test();
        System.out.println(Arrays.toString(a));
    }
}
```

###### 例子2: 1到n之和

```java
public class SumTask extends RecursiveTask<Long> {
    public static final int THRESHOLD = 100;
    private long start;
    private long end;

    public SumTask(long n) {
        this(1, n);
    }

    public SumTask() {
    }

    public SumTask(long start, long end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        long sum = 0;
        if ((end - start) <= THRESHOLD) {
            for (long l = start; l <= end; l++) {
                sum += l;
            }
        } else {
            long mid = (start + end) >>> 1;
            SumTask left = new SumTask(start, mid);
            SumTask right = new SumTask(mid + 1, end);

            left.fork();
            right.fork();
            sum = left.join() + right.join();
        }
        return sum;
    }
    public void testSum(long l) throws ExecutionException, InterruptedException {
        final SumTask sumTask = new SumTask(l);
        final ForkJoinPool forkJoinPool = new ForkJoinPool();
        final ForkJoinTask<Long> submit = forkJoinPool.submit(sumTask);
        System.out.println(submit.get());//获取返回值
        forkJoinPool.shutdown();
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        new SumTask().testSum(1000);
    }
}
```

上面的代码用到了 RecursiveAction 和 RecursiveTask 两个类，它们都继承自抽象类ForkJoinTask，用到了其中关键的接口 fork（）、join（）。二者的区别是一个有返回值，一个没有返回值。AC表示Abstract Class。



![image-20210804155845766](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210804155845766.png)



```java
public <T> ForkJoinTask<T> submit(ForkJoinTask<T> task) {
```

#### 7.2核心数据结构

不同于ThreadPoolExector，除一个全局的任务队列之外，每个线程还有一个自己的局部队列

![image-20210804160123385](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210804160123385.png)

```java
   //最高的16位表示获取线程数，第二个16位表示总的线程数
   //如果有空闲线程，最低的16位中保存空闲线程关联的WorkQueue在WorkQueue数组中的索引 
   volatile long ctl;                   // main pool control
    
    //描述线程池的状态
    volatile int runState;               // lockable status
    
    //高16位保存线程池的队列模式，FIFO或者LIFO
    //低16位保存线程池的parallelism属性
    final int config;                    // parallelism, mode
    
    //累加SEED_INCREMENT生成的一个随机数，决定新增的Worker对应的WorkQueue在数组中的索引
    int indexSeed;                       // to generate worker index
 
    //WorkQueue数组
    volatile WorkQueue[] workQueues;     // main registry
    
    //生成Worker线程的工厂类
    final ForkJoinWorkerThreadFactory factory;
    
    //异常处理器
    final UncaughtExceptionHandler ueh;  // per-worker UEH
    
    //生成的Worker线程的线程名前缀
    final String workerNamePrefix;       // to create worker name string
    
    //累计的从其他WorkQueue中偷过来的待执行的任务
    volatile AtomicLong stealCounter;    // also used as sync monitor
```

 包含的静态属性如下

```javascript
    //创建ForkJoinWorkerThread的工厂类
    public static final ForkJoinWorkerThreadFactory
        defaultForkJoinWorkerThreadFactory;
 
    //线程池关闭时检查当前线程是否有此权限
    private static final RuntimePermission modifyThreadPermission;
 
    //common线程池
    static final ForkJoinPool common;
 
    //common线程池的parallelism属性
    static final int commonParallelism;
 
    //限制最大的线程数，默认值为常量DEFAULT_COMMON_MAX_SPARES，256
    private static int commonMaxSpares;
   
    //生成poolId使用
    private static int poolNumberSequence;
 
    // Unsafe mechanics
    private static final sun.misc.Unsafe U;
    private static final int  ABASE;
    private static final int  ASHIFT;
    private static final long CTL;
    private static final long RUNSTATE;
    private static final long STEALCOUNTER;
    private static final long PARKBLOCKER;
    private static final long QTOP;
    private static final long QLOCK;
    private static final long QSCANSTATE;
    private static final long QPARKER;
    private static final long QCURRENTSTEAL;
    private static final long QCURRENTJOIN;
 
    static {
        //获取属性的偏移量
        try {
            U = sun.misc.Unsafe.getUnsafe();
            Class<?> k = ForkJoinPool.class;
            CTL = U.objectFieldOffset
                (k.getDeclaredField("ctl"));
            RUNSTATE = U.objectFieldOffset
                (k.getDeclaredField("runState"));
            STEALCOUNTER = U.objectFieldOffset
                (k.getDeclaredField("stealCounter"));
            Class<?> tk = Thread.class;
            PARKBLOCKER = U.objectFieldOffset
                (tk.getDeclaredField("parkBlocker"));
            Class<?> wk = WorkQueue.class;
            QTOP = U.objectFieldOffset
                (wk.getDeclaredField("top"));
            QLOCK = U.objectFieldOffset
                (wk.getDeclaredField("qlock"));
            QSCANSTATE = U.objectFieldOffset
                (wk.getDeclaredField("scanState"));
            QPARKER = U.objectFieldOffset
                (wk.getDeclaredField("parker"));
            QCURRENTSTEAL = U.objectFieldOffset
                (wk.getDeclaredField("currentSteal"));
            QCURRENTJOIN = U.objectFieldOffset
                (wk.getDeclaredField("currentJoin"));
            Class<?> ak = ForkJoinTask[].class;
            //用于获取ForkJoinTask数组指定索引元素的偏移量
            ABASE = U.arrayBaseOffset(ak);
            int scale = U.arrayIndexScale(ak);
            if ((scale & (scale - 1)) != 0)
                throw new Error("data type scale not a power of two");
            ASHIFT = 31 - Integer.numberOfLeadingZeros(scale);
        } catch (Exception e) {
            throw new Error(e);
        }
        
        //DEFAULT_COMMON_MAX_SPARES的值是256
        commonMaxSpares = DEFAULT_COMMON_MAX_SPARES;
        defaultForkJoinWorkerThreadFactory =
            new DefaultForkJoinWorkerThreadFactory();
        modifyThreadPermission = new RuntimePermission("modifyThread");
 
        common = java.security.AccessController.doPrivileged
            (new java.security.PrivilegedAction<ForkJoinPool>() {
                public ForkJoinPool run() { return makeCommonPool(); }});
        //获取common线程池的parallelism属性        
        int par = common.config & SMASK; // report 1 even if threads disabled
        commonParallelism = par > 0 ? par : 1;
    }
 
    //创建common线程池
    private static ForkJoinPool makeCommonPool() {
        int parallelism = -1;
        ForkJoinWorkerThreadFactory factory = null;
        UncaughtExceptionHandler handler = null;
        try {  //读取三个属性值
            String pp = System.getProperty
                ("java.util.concurrent.ForkJoinPool.common.parallelism");
            String fp = System.getProperty
                ("java.util.concurrent.ForkJoinPool.common.threadFactory");
            String hp = System.getProperty
                ("java.util.concurrent.ForkJoinPool.common.exceptionHandler");
            //根据属性配置初始化变量   
            if (pp != null)
                parallelism = Integer.parseInt(pp);
            if (fp != null)
                factory = ((ForkJoinWorkerThreadFactory)ClassLoader.
                           getSystemClassLoader().loadClass(fp).newInstance());
            if (hp != null)
                handler = ((UncaughtExceptionHandler)ClassLoader.
                           getSystemClassLoader().loadClass(hp).newInstance());
        } catch (Exception ignore) {
        }
        if (factory == null) {
            if (System.getSecurityManager() == null)
                factory = defaultForkJoinWorkerThreadFactory;
            else // use security-managed default
                factory = new InnocuousForkJoinWorkerThreadFactory();
        }
        if (parallelism < 0 && //如果没有设置属性parallelism，则默认使用CPU核数减1，如果是单核的则为1
            (parallelism = Runtime.getRuntime().availableProcessors() - 1) <= 0)
            parallelism = 1;
        if (parallelism > MAX_CAP) //如果配置的parallelism大于MAX_CAP，则为MAX_CAP，0x7fff，32767
            parallelism = MAX_CAP;
         //注意common线程池的模式是LIFO，后进先出    
        return new ForkJoinPool(parallelism, factory, handler, LIFO_QUEUE,
                                "ForkJoinPool.commonPool-worker-");
    }
```

 包含的静态常量如下：

```java
    //用于获取保存在config属性低16位中的parallelism属性
    static final int SMASK        = 0xffff;        // short bits == max index
    //parallelism属性的最大值
    static final int MAX_CAP      = 0x7fff;        // max #workers - 1
    //跟SMASK相比，最后一位是0而不是1，用来计算遍历workQueues的步长
    static final int EVENMASK     = 0xfffe;        // even short bits
    //取值是126，二进制表示是1111110,WorkQueue数组元素个数的最大值
    static final int SQMASK       = 0x007e;        // max 64 (even) slots
 
    //WorkQueue.scanState属性中表示状态的位
    //最低位为1表示关联的Worker线程正在扫描获取待执行的任务，为0表示正在执行任务
    static final int SCANNING     = 1;             // false when running tasks
    //最高位为1表示当前WorkQueue为非激活状态，刚创建时状态就是INACTIVE，可通过signalWork方法将其变成激活状态
    //最高位为0表示当前WorkQueue为激活状态
    static final int INACTIVE     = 1 << 31;       // must be negative
    //一个自增值，ctl属性的低32位加上SS_SEQ，计算一个新的scanState
    static final int SS_SEQ       = 1 << 16;       // version count
 
    // ForkJoinPool.config 和 WorkQueue.config 属性中，高16位表示队列的模式
    static final int MODE_MASK    = 0xffff << 16;  // top half of int
    static final int LIFO_QUEUE   = 0; //后进先出模式  
    static final int FIFO_QUEUE   = 1 << 16;  //先进先出模式
    static final int SHARED_QUEUE = 1 << 31;       //共享模式
 
    //Worker线程的空闲时间，单位是纳秒
    private static final long IDLE_TIMEOUT = 2000L * 1000L * 1000L; // 2sec
 
    //允许的阻塞时间误差
    private static final long TIMEOUT_SLOP = 20L * 1000L * 1000L;  // 20ms
 
    //commonMaxSpares的初始值，线程池中线程数的最大值
    private static final int DEFAULT_COMMON_MAX_SPARES = 256;
 
    //自旋等待的次数，设置成0可减少对CPU的占用，且没有对应的方法可以修改SPINS属性
    private static final int SPINS  = 0;
 
    //生成随机数的种子
    private static final int SEED_INCREMENT = 0x9e3779b9;
 
    
    //获取低32位的值
    private static final long SP_MASK    = 0xffffffffL;
    //获取高32位的值
    private static final long UC_MASK    = ~SP_MASK;
 
    //活跃线程数，保存在ctl中的最高16位
    private static final int  AC_SHIFT   = 48;
    private static final long AC_UNIT    = 0x0001L << AC_SHIFT;
    private static final long AC_MASK    = 0xffffL << AC_SHIFT;
 
    //总的线程数，保存在ctl中的第二个16位
    private static final int  TC_SHIFT   = 32;
    private static final long TC_UNIT    = 0x0001L << TC_SHIFT;
    private static final long TC_MASK    = 0xffffL << TC_SHIFT;
    //如果第48位为1，表示需要新增一个Worker线程
    //第48位对应于TC的最高位，如果为1，说明总的线程数小于parallelism属性，可以新增线程
    private static final long ADD_WORKER = 0x0001L << (TC_SHIFT + 15); // sign
 
    //runState属性不同的位表示的含义，描述线程池的状态
    //如果runState小于0，即最高位为1，说明线程池正在关闭的过程中
    //如果runState大于0，说明线程池是正常运行的
    private static final int  RSLOCK     = 1; //表示已加锁
    private static final int  RSIGNAL    = 1 << 1; //表示有线程等待获取锁
    private static final int  STARTED    = 1 << 2; //表示线程已启动
    //表示线程池的状态是STOP，不接受新的任务且会丢弃掉任务队列中未执行的任务
    private static final int  STOP       = 1 << 29;
    //表示线程池已终止
    private static final int  TERMINATED = 1 << 30;
    //表示线程池的状态是SHUTDOWN，不接受新的任务
    private static final int  SHUTDOWN   = 1 << 31; 
```

##### 构造方法

```java
public ForkJoinPool() {
    this(Math.min(MAX_CAP, Runtime.getRuntime().availableProcessors()),
         defaultForkJoinWorkerThreadFactory, null, false);
}

public ForkJoinPool(int parallelism) {
    this(parallelism, defaultForkJoinWorkerThreadFactory, null, false);
}

public ForkJoinPool(int parallelism,
                    ForkJoinWorkerThreadFactory factory,
                    UncaughtExceptionHandler handler,
                    boolean asyncMode) {
    this(checkParallelism(parallelism),
         checkFactory(factory),
         handler,
         asyncMode ? FIFO_QUEUE : LIFO_QUEUE,
         "ForkJoinPool-" + nextPoolId() + "-worker-");
    checkPermission();
}

private static void checkPermission() {
    SecurityManager security = System.getSecurityManager();
    if (security != null)
        security.checkPermission(modifyThreadPermission);
}

private static int checkParallelism(int parallelism) {
    if (parallelism <= 0 || parallelism > MAX_CAP)
        throw new IllegalArgumentException();
    return parallelism;
}

private static ForkJoinWorkerThreadFactory checkFactory
    (ForkJoinWorkerThreadFactory factory) {
    if (factory == null)
        throw new NullPointerException();
    return factory;
}

private static final synchronized int nextPoolId() {
    return ++poolNumberSequence;
}

private ForkJoinPool(int parallelism,
                     ForkJoinWorkerThreadFactory factory,
                     UncaughtExceptionHandler handler,
                     int mode,
                     String workerNamePrefix) {
    this.workerNamePrefix = workerNamePrefix;
    this.factory = factory;
    this.ueh = handler;
    //parallelism不能超过MAX_CAP，跟SMASK求且的结果不变
    this.config = (parallelism & SMASK) | mode;
    long np = (long)(-parallelism); // offset ctl counts
    //左移48位后取高16位 和 左移32位后取第二个16位
    //以parallelism是3为例，此时ctl是1111111111111101111111111111110100000000000000000000000000000000
    this.ctl = ((np << AC_SHIFT) & AC_MASK) | ((np << TC_SHIFT) & TC_MASK);
}
```

#### 7.3 工作窃取队列

关于上面的全局队列，有一个关键点需要说明：它并非使用BlockingQueue，而是基于一个普通的数组得以实现。

这个队列又名工作窃取队列，为 ForkJoinPool 的工作窃取算法提供服务。在 ForkJoinPool开篇的注释中，Doug Lea 特别提到了工作窃取队列的实现，其陈述来自如下两篇论文：＂Dynamic CircularWork-Stealing Deque＂ by Chase and Lev，SPAA 2005与＂Idempotent work stealing＂ by Michael，Saraswat，and Vechev，PPoPP 2009。读者可以在网上查阅相应论文。

所谓工作窃取算法，是指一个Worker线程在执行完毕自己队列中的任务之后，可以窃取其他线程队列中的任务来执行，从而实现负载均衡，以防有的线程很空闲，有的线程很忙。这个过程要用到工作窃取队列，图7-3所示为工作窃取队列示意图。

![image-20210812140657668](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210812140657668.png)

这个队列只有三个操作：

（1）Worker线程自己，在队列头部，通过对queueTop指针执行加、减操作，实现入队或出队，这是单线程的。
（2）其他Worker线程，在队列尾部，通过对queueBase进行累加，实现出队操作，也就是窃取，这是多线程的，需要通过CAS操作。正因为如此，在上面的数据结构定义中，queueTop 不是 volatile 的，queueBase 是 volatile类型。

这个队列，在Dynamic Ci rcular Work-Stealing Deque这篇论文中被称为dynamic-cyclic-array。之所以这样命名，是因为有两个关键点：

（1）整个队列是环形的，也就是一个数组实现的RingBuffer。并且queueBase会一直累加，不会减小；queueTop会累加、减小。最后，queueBase、queueTop的值都会大于整个数组的长度，只是计算数组下标的时候，会取queueTop&（queue.length-1），queueBase&（queue.length-1）。因为queue.length是2的整数次方，这里也就是对queue.length进行取模操作。当queueTop-queueBase=queue.length-1 的时候，队列为满，此时需要扩容；当queueTop=queueBase的时候，队列为空，Worker线程即将进入阻塞状态。

（2）当队列满了之后会扩容，所以被称为是动态的。但这就涉及一个棘手的问题：多个线程同时在读写这个队列，如何实现在不加锁的情况下一边读写、一边扩容呢？

通过分析工作窃取队列的特性，我们会发现：在 queueBase 一端，是多线程访问的，但它们只会使queueBase变大，也就是使队列中的元素变少。所以队列为满，一定发生在queueTop一端，对queueTop进行累加的时候，这一端却是单线程的！队列的扩容恰好利用了这个单线程的特性！即在扩容过程中，不可能有其他线程对queueTop进行修改，只有线程对queueBase进行修改！

图7-4所示为工作窃取队列扩容示意图。扩容之后，数组长度变成之前的二倍，但queueTop、queueBase的值是不变的！通过queueTop、queueBase对新的数组长度取模，仍然可以定位到元素在新数组中的位置。

![image-20210812141334760](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210812141334760.png)

下面结合ForkJoinWorkerThread扩容的代码进一步分析。

```java
final ForkJoinTask<?>[] growArray() {
    ForkJoinTask<?>[] oldA = array;
    int size = oldA != null ? oldA.length << 1 : INITIAL_QUEUE_CAPACITY;
    //扩大两倍
    if (size > MAXIMUM_QUEUE_CAPACITY)
        throw new RejectedExecutionException("Queue capacity exceeded");
    int oldMask, t, b;
    ForkJoinTask<?>[] a = array = new ForkJoinTask<?>[size];
    if (oldA != null && (oldMask = oldA.length - 1) >= 0 &&
        (t = top) - (b = base) > 0) {
        int mask = size - 1;
        do { // emulate poll from old array, push to new array
            ForkJoinTask<?> x;
            int oldj = ((b & oldMask) << ASHIFT) + ABASE;
            int j    = ((b &    mask) << ASHIFT) + ABASE;
            x = (ForkJoinTask<?>)U.getObjectVolatile(oldA, oldj);//取旧数组元素
            if (x != null &&
                U.compareAndSwapObject(oldA, oldj, x, null))//旧数组元素设置为空
                U.putObjectVolatile(a, j, x);//赋值新数组
        } while (++b != t);
    }
    return a;
}
```

在上面的代码中有两个关键点：

（1）扩容之后的新数组还是空的时候，就已经赋给了成员变量 queue。而 queueTop、queueBase的值是不变的，这意味着，其他窃取线程若此时来窃取任务，取到的将全是null，即取不到任务。不过，虽然此时窃取不到，可以阻塞一会儿，待扩容完成就可以窃取到了，不会影响整个算法的正确性。

（2）在把旧数组的元素复制过来之前，先通过CAS操作把旧数组中的该元素置为null。只有CAS成功置为null了，才能赋值到新数组。这样可以避免同1个元素在旧数组、新数组中各有1份。1个窃取线程还在读旧数组，另1个窃取线程读取新数组，导致同1个元素被2个线程重复窃取

#### 7.4 ForkJoinPool状态控制

##### 7.4.1 状态变量ctl解析

类似于ThreadPoolExecutor，在ForkJoinPool中也有一个ctl变量负责表达ForkJoinPool的整个生命周期和相关的各种状态。不过ctl变量更加复杂，是一个long型变量，代码如下所示。

```java
//最高的16位表示获取线程数，第二个16位表示总的线程数
//如果有空闲线程，最低的16位中保存空闲线程关联的WorkQueue在WorkQueue数组中的索引 
volatile long ctl;                   // main pool control

//用于获取保存在config属性低16位中的parallelism属性
static final int SMASK        = 0xffff;        // short bits == max index
//parallelism属性的最大值
static final int MAX_CAP      = 0x7fff;        // max #workers - 1
//跟SMASK相比，最后一位是0而不是1，用来计算遍历workQueues的步长
static final int EVENMASK     = 0xfffe;        // even short bits
//取值是126，二进制表示是1111110,WorkQueue数组元素个数的最大值
static final int SQMASK       = 0x007e;        // max 64 (even) slots

//WorkQueue.scanState属性中表示状态的位
//最低位为1表示关联的Worker线程正在扫描获取待执行的任务，为0表示正在执行任务
static final int SCANNING     = 1;             // false when running tasks
//最高位为1表示当前WorkQueue为非激活状态，刚创建时状态就是INACTIVE，可通过signalWork方法将其变成激活状态
//最高位为0表示当前WorkQueue为激活状态
static final int INACTIVE     = 1 << 31;       // must be negative
//一个自增值，ctl属性的低32位加上SS_SEQ，计算一个新的scanState
static final int SS_SEQ       = 1 << 16;       // version count

// ForkJoinPool.config 和 WorkQueue.config 属性中，高16位表示队列的模式
static final int MODE_MASK    = 0xffff << 16;  // top half of int
static final int LIFO_QUEUE   = 0; //后进先出模式  
static final int FIFO_QUEUE   = 1 << 16;  //先进先出模式
static final int SHARED_QUEUE = 1 << 31;       //共享模式

//Worker线程的空闲时间，单位是纳秒
private static final long IDLE_TIMEOUT = 2000L * 1000L * 1000L; // 2sec

//允许的阻塞时间误差
private static final long TIMEOUT_SLOP = 20L * 1000L * 1000L;  // 20ms

//commonMaxSpares的初始值，线程池中线程数的最大值
private static final int DEFAULT_COMMON_MAX_SPARES = 256;

//自旋等待的次数，设置成0可减少对CPU的占用，且没有对应的方法可以修改SPINS属性
private static final int SPINS  = 0;

//生成随机数的种子
private static final int SEED_INCREMENT = 0x9e3779b9;


//获取低32位的值
private static final long SP_MASK    = 0xffffffffL;
//获取高32位的值
private static final long UC_MASK    = ~SP_MASK;

//活跃线程数，保存在ctl中的最高16位
private static final int  AC_SHIFT   = 48;
private static final long AC_UNIT    = 0x0001L << AC_SHIFT;
private static final long AC_MASK    = 0xffffL << AC_SHIFT;

//总的线程数，保存在ctl中的第二个16位
private static final int  TC_SHIFT   = 32;
private static final long TC_UNIT    = 0x0001L << TC_SHIFT;
private static final long TC_MASK    = 0xffffL << TC_SHIFT;
//如果第48位为1，表示需要新增一个Worker线程
//第48位对应于TC的最高位，如果为1，说明总的线程数小于parallelism属性，可以新增线程
private static final long ADD_WORKER = 0x0001L << (TC_SHIFT + 15); // sign

//runState属性不同的位表示的含义，描述线程池的状态
//如果runState小于0，即最高位为1，说明线程池正在关闭的过程中
//如果runState大于0，说明线程池是正常运行的
private static final int  RSLOCK     = 1; //表示已加锁
private static final int  RSIGNAL    = 1 << 1; //表示有线程等待获取锁
private static final int  STARTED    = 1 << 2; //表示线程已启动
//表示线程池的状态是STOP，不接受新的任务且会丢弃掉任务队列中未执行的任务
private static final int  STOP       = 1 << 29;
//表示线程池已终止
private static final int  TERMINATED = 1 << 30;
//表示线程池的状态是SHUTDOWN，不接受新的任务
private static final int  SHUTDOWN   = 1 << 31; 
```

图7-5所示为ctl变量的64个比特位含义示意图。这64个比特位被分成五部分：

AC：最高的16个比特位，表示Active线程数-parallelism，parallelism是上面的构造函数传进去的参数；
TC：次高的16个比特位，表示Total线程数-parallelism；
ST：1个比特位，如果是1，表示整个ForkJoinPool正在关闭；
EC：15个比特位，表示阻塞栈的栈顶线程的wait count（关于什么是wait count，接下来的7.4.2章节会解释）；
ID：16个比特位，表示阻塞栈的栈顶线程对应的poolIndex。

什么叫阻塞栈呢？下面来详细解释：

![image-20210812142806430](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210812142806430.png)

##### 7.4.2 阻塞栈–Treiber Stack

要实现多个线程的阻塞、唤醒，除了park/unpark这一对操作原语，还需要一个无锁链表实现的阻塞队列，把所有阻塞的线程串在一起。

在ForkJoinPool中，没有使用阻塞队列，而是使用了阻塞栈。把所有空闲的Worker线程放在一个栈里面，这个栈同样通过链表来实现，名为Treiber Stack。在4.5节讲解Phaser的实现原理的时候，也用过这个数据结构。
图7-6所示为所有阻塞的Worker线程组成的Treiber Stack。

![image-20210812143005235](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210812143005235.png)

首先，ForkJoinWorkerThread有一个poolIndex变量，记录了自己在ForkJoinWorkerThread[]数组中的下标位置，poolIndex变量就相当于每个ForkJoinPoolWorkerThread对象的地址；其次，ForkJoinWorkerThread还有一个nextWait变量，记录了前一个阻塞线程的poolIndex，这个nextWait变量就相当于链表的next指针，把所有的阻塞线程串联在一起，组成一个Treiber Stack。

最后，ctl变量的最低16位，记录了栈的栈顶线程的poolIndex；中间的15位，记录了栈顶线程被阻塞的次数，也称为wait count。

##### 7.4.3 ctl变量的初始值

在7.4.1节的构造函数中，有如下的代码：

```java
 long np = (long)(-parallelism); // offset ctl counts
    //左移48位后取高16位 和 左移32位后取第二个16位
    //以parallelism是3为例，此时ctl是1111111111111101111111111111110100000000000000000000000000000000
    this.ctl = ((np << AC_SHIFT) & AC_MASK) | ((np << TC_SHIFT) & TC_MASK);
```

因为在初始的时候，ForkJoinPool 中的线程个数为 0，所以 AC=0-parallelism，TC=0-parallelism。这意味着只有高32位的AC、TC两个部分填充了值，低32位都是0填充。

##### 7.4.4 ForkJoinWorkThread状态与个数分析

在ThreadPoolExecutor 中， 有corePoolSize 和maxmiumPoolSize两个参数联合控制总的线程数，而在ForkJoinPool中只传入了一个parallelism参数，且这个参数并不是实际的线程数。那么，ForkJoinPool在实际的运行过程中，线程数究竟是由哪些因素决定的呢？

要回答这个问题，先得明白ForkJoinPool中的线程都可能有哪几种状态？可能的状态有三种：
（1）空闲状态（放在Treiber Stack里面）。
（2）活跃状态（正在执行某个ForkJoinTask，未阻塞）。
（3）阻塞状态（正在执行某个ForkJoinTask，但阻塞了，于是调用join，等待另外一个任务的结果返回）。

ctl变量很好地反映出了三种状态：
高32位：u=（int）（ctl＞＞＞32），然后u又拆分成tc、ac 两个16位；
低32位：e=（int）ctl。

（1）e＞0，说明Treiber Stack不为空，有空闲线程；e=0，说明没有空闲线程；
（2）ac＞0，说明有活跃线程；ac＜=0，说明没有空闲线程，并且还未超出parallelism；
（3）tc＞0，说明总线程数 ＞parallelism。

tc与 ac的差值，也就是总线程数与活跃线程数的差异，在ForkJoinPool中有另外一个变量blockedCount记录，如下：
所以，通过crl和blockedCount这两个变量，可以知道在整个ForkJoinPool中，所有空闲线程、活跃线程以及阻塞线程的数量。
当一个新任务到来时，发现既没有空闲线程，也没有活跃线程，所有线程都阻塞着，在等待任务返回，此时便会开新线程来执行任务。

接下来，我们将结合代码详细了解任务是如何提交的，在执行过程中，线程是如何开启，以及如何执行任务的。在此之前，先了解一下线程的阻塞和唤醒机制。

#### 7.5 Worker线程的阻塞—唤醒机制

ForkerJoinPool 没有使用 BlockingQueue，也就不曾利用其阻塞—唤醒机制，而是利用了park/unpark原语，并自行实现了Treiber Stack。下面进行详细分析ForkerJoinPool，在阻塞和唤醒的时候，分别是如何入栈的。

##### 7.5.1 阻塞–入栈

当一个线程窃取不到任何任务，也就是处于空闲状态时就会阻塞，入栈。其核心逻辑在tryAwaitWork函数里。

```java
private boolean awaitWork(WorkQueue w, int r) {
    if (w == null || w.qlock < 0)                 // w is terminating
        return false;
    for (int pred = w.stackPred, spins = SPINS, ss;;) {
        if ((ss = w.scanState) >= 0)
            break;
        else if (spins > 0) {
            r ^= r << 6; r ^= r >>> 21; r ^= r << 7;
            if (r >= 0 && --spins == 0) {         // randomize spins
                WorkQueue v; WorkQueue[] ws; int s, j; AtomicLong sc;
                if (pred != 0 && (ws = workQueues) != null &&
                    (j = pred & SMASK) < ws.length &&
                    (v = ws[j]) != null &&        // see if pred parking
                    (v.parker == null || v.scanState >= 0))
                    spins = SPINS;                // continue spinning
            }
        }
        else if (w.qlock < 0)                     // recheck after spins
            return false;
        else if (!Thread.interrupted()) {
            long c, prevctl, parkTime, deadline;
            int ac = (int)((c = ctl) >> AC_SHIFT) + (config & SMASK);
            if ((ac <= 0 && tryTerminate(false, false)) ||
                (runState & STOP) != 0)           // pool terminating
                return false;
            if (ac <= 0 && ss == (int)c) {        // is last waiter
                prevctl = (UC_MASK & (c + AC_UNIT)) | (SP_MASK & pred);
                int t = (short)(c >>> TC_SHIFT);  // shrink excess spares
                if (t > 2 && U.compareAndSwapLong(this, CTL, c, prevctl))
                    return false;                 // else use timed wait
                parkTime = IDLE_TIMEOUT * ((t >= 0) ? 1 : 1 - t);
                deadline = System.nanoTime() + parkTime - TIMEOUT_SLOP;
            }
            else
                prevctl = parkTime = deadline = 0L;
            Thread wt = Thread.currentThread();
            U.putObject(wt, PARKBLOCKER, this);   // emulate LockSupport
            w.parker = wt;
            if (w.scanState < 0 && ctl == c)      // recheck before park
                U.park(false, parkTime);
            U.putOrderedObject(w, QPARKER, null);
            U.putObject(wt, PARKBLOCKER, null);
            if (w.scanState >= 0)
                break;
            if (parkTime != 0L && ctl == c &&
                deadline - System.nanoTime() <= 0L &&
                U.compareAndSwapLong(this, CTL, c, prevctl))
                return false;                     // shrink pool
        }
    }
    return true;
}
```

##### 7.5.2 唤醒–出栈

在新的任务到来之后，空闲的线程被唤醒，其核心逻辑在signalWork函数里面。

```java
final void signalWork(WorkQueue[] ws, WorkQueue q) {
    long c; int sp, i; WorkQueue v; Thread p;
    while ((c = ctl) < 0L) {                       // too few active
        if ((sp = (int)c) == 0) {                  // no idle workers
            if ((c & ADD_WORKER) != 0L)            // too few workers
                tryAddWorker(c);
            break;
        }
        if (ws == null)                            // unstarted/terminated
            break;
        if (ws.length <= (i = sp & SMASK))         // terminated
            break;
        if ((v = ws[i]) == null)                   // terminating
            break;
        int vs = (sp + SS_SEQ) & ~INACTIVE;        // next scanState
        int d = sp - v.scanState;                  // screen CAS
        long nc = (UC_MASK & (c + AC_UNIT)) | (SP_MASK & v.stackPred);
        if (d == 0 && U.compareAndSwapLong(this, CTL, c, nc)) {
            v.scanState = vs;                      // activate v
            if ((p = v.parker) != null)
                U.unpark(p);
            break;
        }
        if (q != null && q.base == q.top)          // no more work
            break;
    }
}
```

#### 7.6 任务的提交过程分析

在明白了工作窃取队列、ctl变量的各种状态、Worker的各种状态，以及线程阻塞—唤醒机制之后，接下来综合这些知识，详细分析任务的提交和执行过程。关于任务的提交，ForkJoinPool最外层的接口如下所示。

```java
public <T> ForkJoinTask<T> submit(ForkJoinTask<T> task) {
    if (task == null)
        throw new NullPointerException();
    externalPush(task);
    return task;
}

// 1. 如果workqueues未初始化，初始化WorkQueue[]数组
// 2. 根据当前线程的探针值获取在workequeue[]数组中的位置，如果对应的WorkQueue是空，则new一个新的WorkQueue，然后将task放置在WorkQueue中ForkJoinTask[]数组中
// 3. signalWork开启启动相关线程开启任务的处理
//Tips: Doug Lea大神大部分都是通过Unsafe的相关方法来进行赋值操作，这种方式直接操作内存，效率很高，但是很容易出错，不建议大家平时使用
private void externalSubmit(ForkJoinTask<?> task) {
    int r;                                    // initialize caller's probe
    if ((r = ThreadLocalRandom.getProbe()) == 0) {
        ThreadLocalRandom.localInit();
        r = ThreadLocalRandom.getProbe();
    }
    for (;;) {
        WorkQueue[] ws; WorkQueue q; int rs, m, k;
        boolean move = false;
        if ((rs = runState) < 0) {
            tryTerminate(false, false);     // help terminate
            throw new RejectedExecutionException();
        }
        else if ((rs & STARTED) == 0 ||     // initialize
                 ((ws = workQueues) == null || (m = ws.length - 1) < 0)) {
            int ns = 0;
            rs = lockRunState();
            try {
                if ((rs & STARTED) == 0) {
                    U.compareAndSwapObject(this, STEALCOUNTER, null,
                                           new AtomicLong());
                    // create workQueues array with size a power of two
                    int p = config & SMASK; // ensure at least 2 slots
                    int n = (p > 1) ? p - 1 : 1;
                    n |= n >>> 1; n |= n >>> 2;  n |= n >>> 4;
                    n |= n >>> 8; n |= n >>> 16; n = (n + 1) << 1;
                    workQueues = new WorkQueue[n];
                    ns = STARTED;
                }
            } finally {
                unlockRunState(rs, (rs & ~RSLOCK) | ns);
            }
        }
        else if ((q = ws[k = r & m & SQMASK]) != null) {
            if (q.qlock == 0 && U.compareAndSwapInt(q, QLOCK, 0, 1)) {
                ForkJoinTask<?>[] a = q.array;
                int s = q.top;
                boolean submitted = false; // initial submission or resizing
                try {                      // locked version of push
                    if ((a != null && a.length > s + 1 - q.base) ||
                        (a = q.growArray()) != null) {
                        int j = (((a.length - 1) & s) << ASHIFT) + ABASE;
                        U.putOrderedObject(a, j, task);
                        U.putOrderedInt(q, QTOP, s + 1);
                        submitted = true;
                    }
                } finally {
                    U.compareAndSwapInt(q, QLOCK, 1, 0);
                }
                if (submitted) {
                    signalWork(ws, q);
                    return;
                }
            }
            move = true;                   // move on failure
        }
        else if (((rs = runState) & RSLOCK) == 0) { // create new queue
            q = new WorkQueue(this, null);
            q.hint = r;
            q.config = k | SHARED_QUEUE;
            q.scanState = INACTIVE;
            rs = lockRunState();           // publish index
            if (rs > 0 &&  (ws = workQueues) != null &&
                k < ws.length && ws[k] == null)
                ws[k] = q;                 // else terminated
            unlockRunState(rs, rs & ~RSLOCK);
        }
        else
            move = true;                   // move if busy
        if (move)
            r = ThreadLocalRandom.advanceProbe(r);
    }
}
```

#### 7.7 工作窃取算法：任务的执行过程分析

全局队列有任务，局部队列也有任务，每一个Worker线程都会不间断地扫描这些队列，窃取任务来执行。下面从Worker线程的run函数开始分析：

```java
public void run() {
    if (workQueue.array == null) { // only run once
        Throwable exception = null;
        try {
            onStart();
            pool.runWorker(workQueue);
        } catch (Throwable ex) {
            exception = ex;
        } finally {
            try {
                onTermination(exception);
            } catch (Throwable ex) {
                if (exception == null)
                    exception = ex;
            } finally {
                pool.deregisterWorker(this, exception);
            }
        }
    }
}
```

run（）函数调用的是所在ForkJoinPool的runWorker函数，如下所示。

```java
final void runWorker(WorkQueue w) {
    w.growArray();                   // allocate queue
    int seed = w.hint;               // initially holds randomization hint
    int r = (seed == 0) ? 1 : seed;  // avoid 0 for xorShift
    for (ForkJoinTask<?> t;;) {
        if ((t = scan(w, r)) != null)
            w.runTask(t);
        else if (!awaitWork(w, r))
            break;
        r ^= r << 13; r ^= r >>> 17; r ^= r << 5; // xorshift
    }
}
```

循环里面，不断扫描任务。如扫描到了，则执行任务；如扫描不到，则调用awaitWork进入阻塞状态，阻塞被唤醒之后，继续扫描。

下面详细看扫描过程scan

```java
private ForkJoinTask<?> scan(WorkQueue w, int r) {
    WorkQueue[] ws; int m;
    if ((ws = workQueues) != null && (m = ws.length - 1) > 0 && w != null) {
        int ss = w.scanState;                     // initially non-negative
        for (int origin = r & m, k = origin, oldSum = 0, checkSum = 0;;) {
            WorkQueue q; ForkJoinTask<?>[] a; ForkJoinTask<?> t;
            int b, n; long c;
            if ((q = ws[k]) != null) {
                if ((n = (b = q.base) - q.top) < 0 &&
                    (a = q.array) != null) {      // non-empty
                    long i = (((a.length - 1) & b) << ASHIFT) + ABASE;
                    if ((t = ((ForkJoinTask<?>)
                              U.getObjectVolatile(a, i))) != null &&
                        q.base == b) {
                        if (ss >= 0) {
                            if (U.compareAndSwapObject(a, i, t, null)) {
                                q.base = b + 1;
                                if (n < -1)       // signal others
                                    signalWork(ws, q);
                                return t;
                            }
                        }
                        else if (oldSum == 0 &&   // try to activate
                                 w.scanState < 0)
                            tryRelease(c = ctl, ws[m & (int)c], AC_UNIT);
                    }
                    if (ss < 0)                   // refresh
                        ss = w.scanState;
                    r ^= r << 1; r ^= r >>> 3; r ^= r << 10;
                    origin = k = r & m;           // move and rescan
                    oldSum = checkSum = 0;
                    continue;
                }
                checkSum += b;
            }
            if ((k = (k + 1) & m) == origin) {    // continue until stable
                if ((ss >= 0 || (ss == (ss = w.scanState))) &&
                    oldSum == (oldSum = checkSum)) {
                    if (ss < 0 || w.qlock < 0)    // already inactive
                        break;
                    int ns = ss | INACTIVE;       // try to inactivate
                    long nc = ((SP_MASK & ns) |
                               (UC_MASK & ((c = ctl) - AC_UNIT)));
                    w.stackPred = (int)c;         // hold prev stack top
                    U.putInt(w, QSCANSTATE, ns);
                    if (U.compareAndSwapLong(this, CTL, c, nc))
                        ss = ns;
                    else
                        w.scanState = ss;         // back out
                }
                checkSum = 0;
            }
        }
    }
    return null;
}
```

##### 7.7.1 顺序锁eqLock

顺序锁有点类似于 JDK 8 引入的 StampedLock，也是乐观读的思想。通过一个 sequence number来控制对共享数据的访问，具体来说，就是：

（1）读线程在读取共享数据之前先读取sequence number，在读取数据之后再读一次sequence number，如果两次的值不同，说明在此期间有其他线程修改了数据，此次读取数据无效，重新读取；
（2）写线程，在写入数据之前，累加一次sequence number，在写入数据之后，再累加一次sequence number。

​	最初，sequence number=0，读线程不会修改sequence number，而一个写线程会累加两次sequence number，所以sequence number始终是偶数。如果sequence number是奇数，说明当前某个写线程正在修改数据，其他写线程被互斥了。

所以，对于写线程而言，发现sequence number是奇数，就不能修改共享数据了。对于读线程而言，发现sequence number是奇数，也不能再读取数据；如果发现sequence number是偶数，那么在读取数据前后分别读取一次sequence number，如果两次的值相同，则读取成功，否则重新读取。

##### 7.7.2 scanGuard解析

scanGuard 变量充当了顺序锁的 sequence number 的功能，共享数据就是 Worker 线程数组ws。所以，在上面的scan函数中，在函数开始的时候，读取了一次scanGuard，扫描完ws对应的队列，又读取了一次scanGuard，发现两次的值不同。说明在这期间ws的数组发生了变化，可能是新加了线程，ws数组扩容了，于是返回false，重新进入scan函数。

当然，scanGuard变量不仅承担了顺序锁的功能，还表示距离数组中元素最大下标最近的2的整数次方-1。具体来说，就是：
（1）其低16位，m=scanGuard&SMASK，其值等于距离数组最大下标最近的2的整数次方-1。这句话有些拗口，举个例子来说明：假设数组长度是1024（数组长度始终是2的整数次方），里面只装了10个线程，最大下标是12（意味着在0～12里面，有3个格子是null的，这3个线程已经死亡，剩下10个线程，12以后的所有格子也都是null）。那么m=16-1=15。15是离12最近的，2的4次方-1。之所以这样记，是为了避免不必要的数组遍历，在这里只需要从0遍历到15就可以了，而不需要从0遍历到1024；之所以是2的整数次方-1，是为了取模优化，这个技巧在整个JUC的代码中反复用到，也就是上面代码中ws[k&m]，等价于ws[k%（m+1）]。很显然，当数组中装满线程的时候，数组长度等于线程个数，此时m=数组长度-1。

（2）第17位，也就是SG_UNIT位置，标识当前是否有线程在修改ws数组。这里使用的其实是SeqLock的变体，用第17位来控制多个写线程的互斥，用低16位来检测在读的过程中是否有写线程修改ws数组。

```java
volatile int scanState;    // versioned, <0: inactive; odd:scanning
static final int SMASK        = 0xffff;        // short bits == max index
static final int SS_SEQ       = 1 << 16;       // version count
```

ws数组的修改主要发生在把一个Worker线程加入ws数组的时候，下面看一下在这个过程中，scanGuard变量是如何起作用的。每1个Worker线程在创建的时候，都会调用下面的函数，把自己注册到ForkJoinPool的ws数组里面，下面来看具体过程。

```java
final WorkQueue registerWorker(ForkJoinWorkerThread wt) {
    UncaughtExceptionHandler handler;
    wt.setDaemon(true);                           // configure thread
    if ((handler = ueh) != null)
        wt.setUncaughtExceptionHandler(handler);
    WorkQueue w = new WorkQueue(this, wt);
    int i = 0;                                    // assign a pool index
    int mode = config & MODE_MASK;
    int rs = lockRunState();
    try {
        WorkQueue[] ws; int n;                    // skip if no array
        if ((ws = workQueues) != null && (n = ws.length) > 0) {
            int s = indexSeed += SEED_INCREMENT;  // unlikely to collide
            int m = n - 1;
            i = ((s << 1) | 1) & m;               // odd-numbered indices
            if (ws[i] != null) {                  // collision
                int probes = 0;                   // step by approx half n
                int step = (n <= 4) ? 2 : ((n >>> 1) & EVENMASK) + 2;
                while (ws[i = (i + step) & m] != null) {
                    if (++probes >= n) {
                        workQueues = ws = Arrays.copyOf(ws, n <<= 1);
                        m = n - 1;
                        probes = 0;
                    }
                }
            }
            w.hint = s;                           // use as random seed
            w.config = i | mode;
            w.scanState = i;                      // publication fence
            ws[i] = w;
        }
    } finally {
        unlockRunState(rs, rs & ~RSLOCK);
    }
    wt.setName(workerNamePrefix.concat(Integer.toString(i >>> 1)));
    return w;
}
```

#### 7.8 ForkJoinTask的fork/join

如果局部队列、全局中的任务全部是相互独立的，就很简单了。但问题是，对于分治算法来说，分解出来的一个个任务并不是独立的，而是相互依赖，一个任务的完成要依赖另一个前置任务的完成。这种依赖关系是通过ForkJoinTask中的join（）来体现的。回到7.1节中使用的案例，有这样的代码。

```java
SumTask left = new SumTask(start, mid);
SumTask right = new SumTask(mid + 1, end);

left.fork();
right.fork();
sum = left.join() + right.join();
```

线程在执行当前ForkJoinTask的时候，产生了left、right 两个子Task。所谓fork，是指把这两个子Task放入队列里面，join（）则是要等待2个子Task完成。而子Task在执行过程中，会再次产生两个子Task。如此层层嵌套，类似于递归调用，直到最底层的Task计算完成，再一级级返回。

##### 7.8.1 fork

fork（）的代码很简单，就是把自己放入当前线程所在的局部队列中。

```java
public final ForkJoinTask<V> fork() {
    Thread t;
    if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread)
        ((ForkJoinWorkerThread)t).workQueue.push(this);
    else
        ForkJoinPool.common.externalPush(this);
    return this;
}
```

##### 7.8.2 join的层层嵌套

###### 1.join的层层嵌套阻塞原理

join会导致线程的层层嵌套阻塞，如图7-7所示。

![image-20210812174324058](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210812174324058.png)

线程1 在执行 ForkJoinTask1，在执行过程中调用了 forkJoinTask2.join（），所以要等ForkJoinTask2完成，线程1才能返回；
线程2在执行ForkJoinTask2，但由于调用了forkJoinTask3.join（），只有等ForkJoinTask3完成后，线程2才能返回；
线程3在执行ForkJoinTask3。

结果是：线程3首先执行完，然后线程2才能执行完，最后线程1再执行完。所有的任务其实组成一个有向无环图DAG。如果线程3调用了forkJoinTask1.join（），那么会形成环，造成死锁

那么，这种层次依赖、层次通知的 DAG，在 ForkJoinTask 内部是如何实现的呢？站在ForkJoinTask的角度来看，每1个ForkJoinTask，都可能有多个线程在等待它完成，有1个线程在执行它。所以每个ForkJoinTask就是一个同步对象，线程在调用join（）的时候，阻塞在这个同步对象上面，执行完成之后，再通过这个同步对象通知所有等待的线程。

如图7-8所示，利用synchronized关键字和Java原生的wait（）/notify机制，实现了线程的等待-唤醒机制。调用join（）的这些线程，内部其实是调用ForkJoinTask这个对象的wait（）；执行该任务的Worker线程，在任务执行完毕之后，顺便调用notifyAll（）。

![image-20210812174619005](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210812174619005.png)

###### 2.ForkJoinTask的状态解析

要实现fork（）/join（）的这种线程间的同步，对应的ForkJoinTask一定是有各种状态的，这个状态变量是实现fork/join的基础。

```java
volatile int status; // accessed directly by pool and workers
static final int DONE_MASK   = 0xf0000000;  // mask out non-completion bits
static final int NORMAL      = 0xf0000000;  // must be negative
static final int CANCELLED   = 0xc0000000;  // must be < NORMAL
static final int EXCEPTIONAL = 0x80000000;  // must be < CANCELLED
static final int SIGNAL      = 0x00010000;  // must be >= 1 << 16
static final int SMASK       = 0x0000ffff;  // short bits for tags
```

初始时，status=0。共有五种状态，可以分为两大类：
（1）未完成：0，SIGNAL=1。即status＞=0。0为初始未完成状态，1表示有线程阻塞在任务上，等待任务完成。
（2）已完成：NORMAL=-1，CANCELLED=-2，EXCEPTIONAL=-3。即status＜0。
NORMAL：正常完成；
CANCELLED：任务被取消，完成。
EXCEPTIONAL：任务在执行过程中发生异常，退出也是完成。

所以，通过判断是status＞=0，还是status＜0，就可知道任务是否完成，进而决定调用join（）的线程是否需要被阻塞。

###### 3.join的详细实现

下面看一下代码的详细实现

```java
public final V join() {
    int s;
    if ((s = doJoin() & DONE_MASK) != NORMAL)
        reportException(s);//非正常结束,抛异常
    return getRawResult();//正常结束
}

private int doJoin() {
    int s; Thread t; ForkJoinWorkerThread wt; ForkJoinPool.WorkQueue w;
    return (s = status) < 0 ? s :
    ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) ?
        (w = (wt = (ForkJoinWorkerThread)t).workQueue).
        tryUnpush(this) && (s = doExec()) < 0 ? s :
    wt.pool.awaitJoin(w, this, 0L) :
    externalAwaitDone();
}

final int awaitJoin(WorkQueue w, ForkJoinTask<?> task, long deadline) {
    int s = 0;
    if (task != null && w != null) {
        ForkJoinTask<?> prevJoin = w.currentJoin;
        U.putOrderedObject(w, QCURRENTJOIN, task);
        CountedCompleter<?> cc = (task instanceof CountedCompleter) ?
            (CountedCompleter<?>)task : null;
        for (;;) {
            if ((s = task.status) < 0)
                break;
            if (cc != null)
                helpComplete(w, cc, 0);
            //此处是join的核心
            else if (w.base == w.top || w.tryRemoveAndExec(task))
                helpStealer(w, task);
            if ((s = task.status) < 0)
                break;
            long ms, ns;
            if (deadline == 0L)
                ms = 0L;
            else if ((ns = deadline - System.nanoTime()) <= 0L)
                break;
            else if ((ms = TimeUnit.NANOSECONDS.toMillis(ns)) <= 0L)
                ms = 1L;
            if (tryCompensate(w)) {
                task.internalWait(ms);
                U.getAndAddLong(this, CTL, AC_UNIT);
            }
        }
        U.putOrderedObject(w, QCURRENTJOIN, prevJoin);
    }
    return s;
}

//join会发生什么呢，它会去遍历workqueue中的ForkJoinTask数组的列表，找到在队列中之前通过fork方法放入的ForkJoinTask，用EmptyTask替换它，然后在当前线程中直接通过doExec方法直接执行，我想此处是work-steal的精髓所在

final boolean tryRemoveAndExec(ForkJoinTask<?> task) {
    ForkJoinTask<?>[] a; int m, s, b, n;
    if ((a = array) != null && (m = a.length - 1) >= 0 &&
        task != null) {
        while ((n = (s = top) - (b = base)) > 0) {
            for (ForkJoinTask<?> t;;) {      // traverse from s to b
                long j = ((--s & m) << ASHIFT) + ABASE;
                if ((t = (ForkJoinTask<?>)U.getObject(a, j)) == null)
                    return s + 1 == top;     // shorter than expected
                else if (t == task) {
                    boolean removed = false;
                    if (s + 1 == top) {      // pop
                        if (U.compareAndSwapObject(a, j, task, null)) {
                            U.putOrderedInt(this, QTOP, s);
                            removed = true;
                        }
                    }
                    else if (base == b)      // replace with proxy
                        removed = U.compareAndSwapObject(
                        a, j, task, new EmptyTask());
                    if (removed)
                        //直接执行
                        task.doExec();
                    break;
                }
                else if (t.status < 0 && s + 1 == top) {
                    if (U.compareAndSwapObject(a, j, t, null))
                        U.putOrderedInt(this, QTOP, s);
                    break;                  // was cancelled
                }
                if (--n == 0)
                    return false;
            }
            if (task.status < 0)
                return false;
        }
    }
    return true;
}
```

getRawResult（）是ForkJoinTask中的一个模板方法，分别被RecusiveAction和RecursiveTask实现，前者没有返回值，所以返回null，后者返回一个类型为V的result变量。

```java
public abstract class RecursiveAction extends ForkJoinTask<Void> {
    public final Void getRawResult() { return null; }
}
public abstract class RecursiveTask<V> extends ForkJoinTask<V> {
    public final V getRawResult() {
        return result;
    }
}
```

reportResult（）只是对 getRawResult（）的一个包装，里面多了对 CANCELLED 和EXCEPTIONAL这两种异常完成状态的处理。

```java
private void reportException(int s) {
    if (s == CANCELLED)
        throw new CancellationException();
    if (s == EXCEPTIONAL)
        rethrow(getThrowableException());
}
```

阻塞主要发生在上面的doJoin（）函数里面。在dojoin（）里调用t.join（）的线程会阻塞，然后等待任务t执行完成，再唤醒该阻塞线程，doJoin（）返回。

注意：当 doJoin（）返回的时候，就是该任务执行完成的时候，doJoin（）的返回值就是任务的完成状态，也就是上面的-1、-2、-3三种状态。

```java
private int doJoin() {
    int s; Thread t; ForkJoinWorkerThread wt; ForkJoinPool.WorkQueue w;
    return (s = status) < 0 ? s :
        ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) ?
        (w = (wt = (ForkJoinWorkerThread)t).workQueue).
        tryUnpush(this) && (s = doExec()) < 0 ? s :
        wt.pool.awaitJoin(w, this, 0L) :
        externalAwaitDone();
}
```

先看一下externalAwaitDone（），即外部线程的阻塞过程，相对简单

```java
private int externalAwaitDone() {
    int s = ((this instanceof CountedCompleter) ? // try helping
             ForkJoinPool.common.externalHelpComplete(
                 (CountedCompleter<?>)this, 0) :
             ForkJoinPool.common.tryExternalUnpush(this) ? doExec() : 0);
    if (s >= 0 && (s = status) >= 0) {
        boolean interrupted = false;
        do {
            if (U.compareAndSwapInt(this, STATUS, s, s | SIGNAL)) {
                synchronized (this) {
                    if (status >= 0) {
                        try {
                            wait(0L);
                        } catch (InterruptedException ie) {
                            interrupted = true;
                        }
                    }
                    else
                        notifyAll();
                }
            }
        } while ((s = status) >= 0);
        if (interrupted)
            Thread.currentThread().interrupt();
    }
    return s;
}
```

#### 7.9 ForkJoinPool的优雅关闭

同 ThreadPoolExecutor 一样，ForkJoinPool 的关闭也不可能是“瞬时的”，而是需要一个平滑的过渡过程。

##### 7.9.1 关键的terminate变量

对于一个Worker线程来说，它会在一个while循环里面不断轮询队列中的任务，如果有任务，那么执行，处在活跃状态；如果没有任务，则进入空闲等待状态。

```java
public boolean isTerminated() {
    return (runState & TERMINATED) != 0;
}
```

##### 7.9.2 shutdown（）与shutdownNow（）的区别

二者的代码基本相同，都是调用tryTerminate（boolean）函数，其中一个传入的是false，另一个传入的是true。tryTerminate意为试图关闭ForkJoinPool，并不保证一定可以关闭成功。

```java
public void shutdown() {
    checkPermission();
    tryTerminate(false, true);
}
public List<Runnable> shutdownNow() {
    checkPermission();
    tryTerminate(true, true);
    return Collections.emptyList();
}
```

```java
private boolean tryTerminate(boolean now, boolean enable) {
    int rs;
    if (this == common)                       // cannot shut down
        return false;
    if ((rs = runState) >= 0) {
        if (!enable)
            return false;
        rs = lockRunState();                  // enter SHUTDOWN phase
        unlockRunState(rs, (rs & ~RSLOCK) | SHUTDOWN);
    }

    if ((rs & STOP) == 0) {
        if (!now) {                           // check quiescence
            for (long oldSum = 0L;;) {        // repeat until stable
                WorkQueue[] ws; WorkQueue w; int m, b; long c;
                long checkSum = ctl;
                if ((int)(checkSum >> AC_SHIFT) + (config & SMASK) > 0)
                    return false;             // still active workers
                if ((ws = workQueues) == null || (m = ws.length - 1) <= 0)
                    break;                    // check queues
                for (int i = 0; i <= m; ++i) {
                    if ((w = ws[i]) != null) {
                        if ((b = w.base) != w.top || w.scanState >= 0 ||
                            w.currentSteal != null) {
                            tryRelease(c = ctl, ws[m & (int)c], AC_UNIT);
                            return false;     // arrange for recheck
                        }
                        checkSum += b;
                        if ((i & 1) == 0)
                            w.qlock = -1;     // try to disable external
                    }
                }
                if (oldSum == (oldSum = checkSum))
                    break;
            }
        }
        if ((runState & STOP) == 0) {
            rs = lockRunState();              // enter STOP phase
            unlockRunState(rs, (rs & ~RSLOCK) | STOP);
        }
    }

    int pass = 0;                             // 3 passes to help terminate
    for (long oldSum = 0L;;) {                // or until done or stable
        WorkQueue[] ws; WorkQueue w; ForkJoinWorkerThread wt; int m;
        long checkSum = ctl;
        if ((short)(checkSum >>> TC_SHIFT) + (config & SMASK) <= 0 ||
            (ws = workQueues) == null || (m = ws.length - 1) <= 0) {
            if ((runState & TERMINATED) == 0) {
                rs = lockRunState();          // done
                unlockRunState(rs, (rs & ~RSLOCK) | TERMINATED);
                synchronized (this) { notifyAll(); } // for awaitTermination
            }
            break;
        }
        for (int i = 0; i <= m; ++i) {
            if ((w = ws[i]) != null) {
                checkSum += w.base;
                w.qlock = -1;                 // try to disable
                if (pass > 0) {
                    w.cancelAll();            // clear queue
                    if (pass > 1 && (wt = w.owner) != null) {
                        if (!wt.isInterrupted()) {
                            try {             // unblock join
                                wt.interrupt();
                            } catch (Throwable ignore) {
                            }
                        }
                        if (w.scanState < 0)
                            U.unpark(wt);     // wake up
                    }
                }
            }
        }
        if (checkSum != oldSum) {             // unstable
            oldSum = checkSum;
            pass = 0;
        }
        else if (pass > 3 && pass > m)        // can't further help
            break;
        else if (++pass > 1) {                // try to dequeue
            long c; int j = 0, sp;            // bound attempts
            while (j++ <= m && (sp = (int)(c = ctl)) != 0)
                tryRelease(c, ws[sp & m], AC_UNIT);
        }
    }
    return true;
}
```

同ThreadPoolExecutor一样，ForkJoinPool中也有awaitTermination（…）函数，代码几乎相同，上面函数的最后一段，就是在整个线程池都已关闭，即没有任何线程存活的情况下，通知阻塞在awaitTermination（..）的外部线程。



所以，最后总结一下：shutdown（）只拒绝新提交的任务；shutdownNow（）会取消现有的全局队列和局部队列中的任务，同时唤醒所有空闲的线程，让这些线程自动退出。

### 8.CompletableFuture

#### 8.1 CompletableFuture用法

##### 8.1.1 最简单的用法

CompletableFuture实现了Future接口，所以它也具有Future的特性：调用get（）方法会阻塞在那，直到结果返回。

```java
final CompletableFuture<String> stringCompletableFuture = new CompletableFuture<>();
stringCompletableFuture.get();//调用者阻塞,等待结果返回
```

另外1个线程调用complete方法完成该Future，则所有阻塞在get（）方法的线程都将获得返回结果。

```java
stringCompletableFuture.complete("123");
```

##### 8.1.2 提交任务：runAsync与supplyAsync

上面的例子是一个空的任务，下面尝试提交一个真的任务，然后等待结果返回。
例1：runAsync（Runnable）

```java
final CompletableFuture<Void> future = completableFuture.thenRunAsync(() -> System.out.println("123"));
future.get();//主线程阻塞,等待返回

final CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "123");
System.out.println(future2.get());
```

一个有返回值,一个没有

##### 8.1.3 链式的CompletableFuture：thenRun、thenAccept和thenApply

对于 Future，在提交任务之后，只能调用 get（）等结果返回；但对于 CompletableFuture，可以在结果上面再加一个callback，当得到结果之后，再接着执行callback。

```java
final CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "123");
future2.thenRun(()-> System.out.println("thenRun"));//无参无返回
future2.thenAccept(x-> System.out.println("thenAccept" + x));//有参无返回
future2.thenApply(x-> "thenApply" + x);//有参有返回
```

##### 8.1.4 CompletableFuture的组合：thenCompose与thenCombine

###### 例1：thenCompose

在上面的例子中，thenApply接收的是一个Function，但是这个Function的返回值是一个通常的基本数据类型或一个对象，而不是另外一个 CompletableFuture。如果 Function 的返回值也是一个CompletableFuture，就会出现嵌套的CompletableFuture。

```java
CompletableFuture.supplyAsync(() -> "123").thenCompose(x -> CompletableFuture.supplyAsync(() -> x)).get()
```

###### 例2：thenCombine

```java
public <U,V> CompletableFuture<V> thenCombine( CompletionStage<? extends U> other,
    BiFunction<? super T,? super U,? extends V> fn) {
    return biApplyStage(null, other, fn);
}
```

```java
final CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "123");
final CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "456");
future1.thenCombine(future2, (x, y) -> x + y).get();
```

##### 8.1.5 任意个CompletableFuture的组合

上面的thenCompose和thenCombine只能组合2个CompletableFuture，而接下来的allOf 和anyOf 可以组合任意多个CompletableFuture。

首先，这两个函数都是静态函数，参数是变长的CompletableFuture的集合。其次，allOf和anyOf的区别，前者是“与”，后者是“或”



#### 8.2 四种任务原型

通过上面的例子可以总结出，提交给CompletableFuture执行的任务有四种类型：Runnable、Consumer、Supplier、Function。表格8-1总结了这四种任务原型的对比。

![image-20210813121308557](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210813121308557.png)

runAsync 与 supplierAsync 是 CompletableFutre 的静态方法；而 thenAccept、thenAsync、thenApply是CompletableFutre的成员方法。

因为初始的时候没有CompletableFuture对象，也没有参数可传，所以提交的只能是Runnable或者Supplier，只能是静态方法；通过静态方法生成CompletableFuture对象之后，便可以链式地提交其他任务了，这个时候就可以提交Runnable、Consumer、Function，且都是成员方法。

#### 8.3 CompletionStage接口

CompletableFuture不仅实现了Future接口，还实现了CompletableStage接口，如下所示。
CompletionStage接口定义的正是前面的各种链式方法、组合方法，如下所示。

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
}
public interface CompletionStage<T> {
    public CompletionStage<Void> thenAccept(Consumer<? super T> action);
    public CompletionStage<Void> thenRun(Runnable action);
    public <U> CompletionStage<U> thenApply(Function<? super T,? extends U> fn);
    ...
}
```

关于CompletionStage接口，有几个关键点要说明：
（1）所有方法的返回值都是CompletionStage类型，也就是它自己。正因为如此，才能实现如下的链式调用：future1.thenApply（xxx）.thenApply（xxx）.thenCompose（…）.thenRun（..）。
（2）thenApply接受的是一个有输入参数、返回值的Function。这个Function的输入参数，必须是？Super T 类型，也就是T或者T的父类型，而T必须是调用thenApplycompletableFuture对象的类型；返回值则必须是？Extends U类型，也就是U或者U的子类型，而U恰好是thenApply的返回值的CompletionStage对应的类型。

其他函数，诸如thenCompose、thenCombine也是类似的原理。

#### 8.4 CompletableFuture内部原理

##### 8.4.1 CompletableFuture的构造：ForkJoinPool

CompletableFuture中任务的执行同样依靠ForkJoinPool，代码如下所示。


```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
    private static final Executor asyncPool = useCommonPool ? ForkJoinPool.commonPool()
        : new ThreadPerTaskExecutor();

    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
        return asyncSupplyStage(asyncPool, supplier);
    }
    static <U> CompletableFuture<U> asyncSupplyStage(Executor e,
                                                     Supplier<U> f) {
        if (f == null) throw new NullPointerException();
        CompletableFuture<U> d = new CompletableFuture<U>();
        e.execute(new AsyncSupply<U>(d, f));//supplier转ForkjoinTask
        return d;
    }
```

通过上面的代码可以看到，asyncPool是一个static类型，suppli
erAsync、asyncSupplyStage也都是static函数。Static函数会返回一
个CompletableFuture类型对象，之后就可以链式调用，CompletionSt
age里面的各个方法。

##### 8.4.2 任务类型的适配

ForkJoinPool接受的任务是ForkJoinTask 类型，而我们向CompletableFuture提交的任务是Runnable/Supplier/Consumer/Function。因此，肯定需要一个适配机制，把这四种类型的任务转换成ForkJoinTask，然后提交给ForkJoinPool，如图8-1所示。

![image-20210813133922032](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210813133922032.png)

为了完成这种转换，在CompletableFuture内部定义了一系列的内部类，图8-2所示为CompletableFuture的各种内部类的继承体系。

在 supplierAsync（..）函数内部，会把一个 Supplier 转换成一个 AsyncSupply，然后提交给ForkJoinPool执行；
在runAsync（..）函数内部，会把一个Runnable转换成一个AsyncRun，然后提交给ForkJoinPool执行；
在 thenRun/thenAccept/thenApply 内部，会分别把 Runnable/Consumer/Function 转换成UniRun/UniAccept/UniApply对象，然后提交给ForkJoinPool执行；

除此之外， 还有两种 CompletableFuture 组合的情况， 分为“与”和“或”，所以有对应的Bi和Or类型的Completion类型

![image-20210813134114262](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210813134114262.png)

下面的代码分别为 UniRun、UniApply、UniAccept 的定义，可以看到，其内部分别封装了Runnable、Function、Consumer。

```java
static final class UniRun<T> extends UniCompletion<T,Void> {
    Runnable fn;
    ...
}
static final class UniAccept<T> extends UniCompletion<T,Void> {
    Consumer<? super T> fn;
    ...
}
static final class UniApply<T,V> extends UniCompletion<T,V> {
    Function<? super T,? extends V> fn;
    ...
}
```

图8-3所示为CompletableFuture的接口层面和内部实现层面对比。

![image-20210813134330428](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210813134330428.png)

##### 8.4.3 任务的链式执行过程分析

下面以CompletableFuture.supplyAsync （ … ） .thenApply（…）.thenRun（…） 链式代码为例，分析整个执行过程。
第1步：CompletableFuture future1=CompletableFuture.supplyAsync（…）

```java
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
    return asyncSupplyStage(asyncPool, supplier);
}
static <U> CompletableFuture<U> asyncSupplyStage(Executor e, Supplier<U> f) {
    if (f == null) throw new NullPointerException();
    CompletableFuture<U> d = new CompletableFuture<U>();
    e.execute(new AsyncSupply<U>(d, f));
    return d;
}
```

在上面的代码中，关键是构造了一个AsyncSupply对象，该对象有三个关键点：
（1）它继承自ForkJoinTask，所以能够提交ForkJoinPool来执行。
（2）它封装了Supplier f，即它所执行任务的具体内容。
（3）该任务的返回值，即CompletableFuture d，也被封装在里面。

图8-4所示为这几个概念之间的关系。ForkJoinPool执行一个ForkJoinTask类型的任务，即AsyncSupply。该任务的输入就是Supply，输出结果存放在CompletableFuture中。

![image-20210813134731702](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210813134731702.png)

第2步：CompletableFuture future2=future1.thenApply（…）
第1步的返回值，也就是上面代码中的 CompletableFuture d，紧接着调用其成员函数thenApply。

```java
public <U> CompletableFuture<U> thenApply(
    Function<? super T,? extends U> fn) {
    return uniApplyStage(null, fn);
}
private <V> CompletableFuture<V> uniApplyStage(
    Executor e, Function<? super T,? extends V> f) {
    if (f == null) throw new NullPointerException();
    CompletableFuture<V> d =  new CompletableFuture<V>();
    if (e != null || !d.uniApply(this, f, null)) {
        UniApply<T,V> c = new UniApply<T,V>(e, d, this, f);
        push(c);//把第二个任务压入第一个任务执行结果所在的栈
        c.tryFire(SYNC);
    }
    return d;
}
```

我们知道，必须等第1步的任务执行完毕，第2步的任务才可以执行。因此，这里提交的任务不可能立即执行，在此处构建了一个UniApply对象，也就是一个ForkJoinTask类型的任务，这个任务放入了第1个任务的栈当中。

```java
final void push(UniCompletion<?,?> c) {
    if (c != null) {
        while (result == null && !tryPushStack(c))
            lazySetNext(c, null); // clear on failure
    }
}
```

每一个CompletableFuture对象内部都有一个栈，存储着是后续依赖它的任务，如下面代码所示。这个栈也就是Treiber Stack，这里的stack存储的就是栈顶指针。

```java
volatile Completion stack;    // Top of Treiber stack of dependent actions
```

上面的UniApply对象类似于第1步里面的AsyncSupply，它的构造函数传入了4个参数：
第1个参数是执行它的ForkJoinPool；
第2个参数是输出一个CompletableFuture对象。这个参数，也是thenApply函数的返回值，用来链式执行下一个任务；
第3个参数是其依赖的前置任务，也就是第1步里面提交的任务；
第4个参数是输入（也就是一个Function对象）。

```java
UniApply(Executor executor, CompletableFuture<V> dep, CompletableFuture<T> src,
         Function<? super T,? extends V> fn) {
    super(executor, dep, src); this.fn = fn;
}
```

UniApply对象被放入了第1步的CompletableFuture的栈中，在第1步的任务执行完成之后，就会从栈中弹出并执行。下面看一下代码：

```java
static final class AsyncSupply<T> extends ForkJoinTask<Void>
        implements Runnable, AsynchronousCompletionTask {
    CompletableFuture<T> dep; Supplier<T> fn;
    AsyncSupply(CompletableFuture<T> dep, Supplier<T> fn) {
        this.dep = dep; this.fn = fn;
    }

    public final Void getRawResult() { return null; }
    public final void setRawResult(Void v) {}
    public final boolean exec() { run(); return true; }

    public void run() {
        CompletableFuture<T> d; Supplier<T> f;
        if ((d = dep) != null && (f = fn) != null) {
            dep = null; fn = null;
            if (d.result == null) {
                try {
                    d.completeValue(f.get());//suppli的get方法
                } catch (Throwable ex) {
                    d.completeThrowable(ex);
                }
            }
            d.postComplete();
        }
    }
}
```

ForkJoinPool执行上面的AsyncSupply对象的run（）方法，实质就是执行Supplier的get（）方法。执行结果被塞入了 CompletableFuture d 当中，也就是赋值给了 CompletableFuture 内部的Object result变量。

调用d.postComplete（），也正是在这个函数里面，把第2步压入的UniApply对象弹出来执行，代码如下所示。

```java
final void postComplete() {
    CompletableFuture<?> f = this; Completion h;
    while ((h = f.stack) != null ||
           (f != this && (h = (f = this).stack) != null)) {
        CompletableFuture<?> d; Completion t;
        if (f.casStack(h, t = h.next)) {//出栈
            if (t != null) {
                if (f != this) {
                    pushStack(h);
                    continue;
                }
                h.next = null;    // detach
            }
            f = (d = h.tryFire(NESTED)) == null ? this : d;//执行出栈
        }
    }
}
```

第3步：CompletableFuture future3=future2.thenRun（…）
第3步和第2步的过程类似，构建了一个 UniRun 对象，这个对象被压入第2步的CompletableFuture所在的栈中。第2步的任务，当执行完成时，从自己的栈中弹出UniRun对象并执行。

总结一下上述过程，如图8-5所示。
通过supplyAsync/thenApply/thenRun，分别提交了3个任务，每1个任务都有1个返回值对象，也就是1个CompletableFuture。这3个任务通过2个CompletableFuture完成串联。后1个任务，被放入了前1个任务的CompletableFuture里面，前1个任务在执行完成时，会从自己的栈中，弹出下1个任务执行。如此向后传递，完成任务的链式执行。

![image-20210813140306034](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210813140306034.png)

##### 8.4.4 thenApply与thenApplyAsync的区别

在上面的代码中，我们分析了thenApply，还有一个与之对应的函数是thenApplyAsync。这两个函数调用的是同一个函数，只不过传入的参数不同。

```java
public <U> CompletableFuture<U> thenApply(
    Function<? super T,? extends U> fn) {
    return uniApplyStage(null, fn);
}

public <U> CompletableFuture<U> thenApplyAsync(
    Function<? super T,? extends U> fn) {
    return uniApplyStage(asyncPool, fn);
}
private <V> CompletableFuture<V> uniApplyStage(Executor e, Function<? super T,? extends V> f) {
    if (f == null) throw new NullPointerException();
    CompletableFuture<V> d =  new CompletableFuture<V>();
    if (e != null || !d.uniApply(this, f, null)) {//thenApplyAsync的逻辑
        UniApply<T,V> c = new UniApply<T,V>(e, d, this, f);
        push(c);
        c.tryFire(SYNC);
    }
    return d;
}
```

最关键的是上面几行加粗的代码。
如果是thenApplyAsync，则e！=null，构建UniApply对象，入栈；

如果是thenApply，则会调用d.uniApply（this，f，null），该函数代码如下：

```java
final <S> boolean uniApply(CompletableFuture<S> a,
                           Function<? super S,? extends T> f,
                           UniApply<S,T> c) {
    Object r; Throwable x;
    if (a == null || (r = a.result) == null || f == null)
        return false;//前置依赖任务未完成,直接返回
    tryComplete: if (result == null) {
        if (r instanceof AltResult) {
            if ((x = ((AltResult)r).ex) != null) {
                completeThrowable(x, r);
                break tryComplete;
            }
            r = null;
        }
        try {
            if (c != null && !c.claim())
                return false;
            @SuppressWarnings("unchecked") S s = (S) r;
            completeValue(f.apply(s));//执行任务
        } catch (Throwable ex) {
            completeThrowable(ex);
        }
    }
    return true;
}
```

通过上面的代码可以看到：
（1）如果前置任务没有完成，即a.result=null，则上面的uniApply会返回false，此时thenApply也会走到thenApplyAsync的逻辑里面，生成UniApply对象入栈；
（2）只有在前置任务已经完成的情况下，thenApply才会立即执行，不会入栈，再出栈，此时thenApply和thenApplyAsync才有区别。同理，thenRun与thenRunAsync、thenAccept与thenAcceptAsync的区别与此类似。

#### 8.5 任务的网状执行：有向无环图

如果任务只是链式执行，便不需要在每个CompletableFuture里面设1个栈了，用1个指针使所有任务组成链表即可。

但实际上，任务不只是链式执行，而是网状执行，组成 1 张图。如图8-6所示，所有任务组成一个有向无环图：
任务1执行完成之后，任务2、任务3可以并行，在代码层面可以写为：future1.thenApply（任务2），future1.thenApply（任务3）；任务4在任务2执行完成时可开始执行；任务5要等待任务2、任务3都执行完成，才能开始，这里是And关系；任务6在任务3执行完成时可以开始执行；对于任务7，只要任务4、任务5、任务6中任意一个任务结束，就可以开始执行。

总而言之，任务之间是多对多的关系：1个任务有n个依赖它的后继任务；1个任务也有n个它依赖的前驱任务。

![image-20210813141248423](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210813141248423.png)

这样一个有向无环图，用什么样的数据结构表达呢？And和Or的关系又如何表达呢？
有几个关键点：
（1）在每个任务的返回值里面，存储了依赖它的接下来要执行的任务。所以在图8-6中，任务1的CompletableFuture的栈中存储了任务2、任务3；任务2的CompletableFuutre中存储了任务4、任务5；任务3的CompletableFuture中存储了任务5、任务6。也就是说，每个任务的CompletableFuture对象的栈里面，其实存储了该节点的出边对应的任务集合。

（2）任务2、任务3的CompletableFuture里面，都存储了任务5，那么任务5是不是会被触发两次，执行两次呢？任务5的确会被触发2次，但它会判断任务2、任务3的结果是不是都完成，如果只完成其中一个，它就不会执行。

（3）任务7存在于任务4、任务5、任务6的CompletableFuture的栈里面，因此会被触发三次。但它只会执行一次，只要其中1个任务执行完成，就可以执行任务7了。

（4）正因为有And和Or 两种不同的关系，因此对应BiApply和OrApply两个对象，这两个对象的构造函数几乎一样，只是在内部执行的时候，一个是And的逻辑，一个是Or的逻辑。

```java
static final class BiApply<T,U,V> extends BiCompletion<T,U,V> {
    BiFunction<? super T,? super U,? extends V> fn;
    BiApply(Executor executor, CompletableFuture<V> dep,
            CompletableFuture<T> src, CompletableFuture<U> snd,
            BiFunction<? super T,? super U,? extends V> fn) {
        super(executor, dep, src, snd); this.fn = fn;
    }
}
static final class OrApply<T,U extends T,V> extends BiCompletion<T,U,V> {
    Function<? super T,? extends V> fn;
    OrApply(Executor executor, CompletableFuture<V> dep,
            CompletableFuture<T> src,
            CompletableFuture<U> snd,
            Function<? super T,? extends V> fn) {
        super(executor, dep, src, snd); this.fn = fn;
    }
}
```

![image-20210813141533634](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210813141533634.png)

（5）BiApply和OrApply都是二元操作符，也就是说，只能传入二个被依赖的任务。但上面的任务7同时依赖于任务4、任务5、任务6，这怎么处理呢？

任何一个多元操作，都能被转换为多个二元操作的叠加。如图8-7所示，假如任务1And任务2And任务3=任务4，那么它可以被转换为右边的形式。新建了一个And任务，这个And任务和任务3再作为参数，构造任务4。Or的关系，与此类似。

明白了任务的有向无环图的存储与计算过程，也就明白了8.1.4节thenCombine的内部实现原理。thenCombine用于任务1、任务2执行完成，再执行任务3，实际场景更为简单，此处不再进一步展开源码讨论。

#### 8.6 allOf内部的计算图分析

下面以allOf函数为例，看一下有向无环计算图的内部运作过程：

```java
public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs) {
    return andTree(cfs, 0, cfs.length - 1);
}
static CompletableFuture<Void> andTree(CompletableFuture<?>[] cfs,
                                       int lo, int hi) {
    CompletableFuture<Void> d = new CompletableFuture<Void>();
    if (lo > hi) // empty
        d.result = NIL;
    else {
        CompletableFuture<?> a, b;
        int mid = (lo + hi) >>> 1;
        if ((a = (lo == mid ? cfs[lo] :
                  andTree(cfs, lo, mid))) == null ||
            (b = (lo == hi ? a : (hi == mid+1) ? cfs[hi] :
                  andTree(cfs, mid+1, hi)))  == null)
            throw new NullPointerException();
        if (!d.biRelay(a, b)) {
            BiRelay<?,?> c = new BiRelay<>(d, a, b);
            a.bipush(b, c);//c分别压入ab所在栈
            c.tryFire(SYNC);
        }
    }
    return d;
}
```

上面的函数是一个递归函数，输入是一个CompletableFuture对象的列表，输出是一个具有And关系的复合CompletableFuture对象。最关键的代码如上面的加粗代码所示，因为c要等a，b都执行完成之后才能执行，因此c会被分别压入a，b所在的栈中。

```java
final void bipush(CompletableFuture<?> b, BiCompletion<?,?,?> c) {
    if (c != null) {
        Object r;
        while ((r = result) == null && !tryPushStack(c))//c压a
            lazySetNext(c, null); // clear on failure
        if (b != null && b != this && b.result == null) {
            Completion q = (r != null) ? c : new CoCompletion(c);//c压b
            while (b.result == null && !b.tryPushStack(q))
                lazySetNext(q, null); // clear on failure
        }
    }
}
```

图8-8所示为allOf内部的运作过程：方块表示任务，椭圆表示任务的执行结果。假设allof的参数传入了future1、future2、future3、future4，则对应四个原始任务。

生成BiRelay1、BiRelay2任务，分别压入future1/future2、future3/future4的栈中。无论future1或future2完成，都会触发BiRelay1；无论future3或future4完成，都会触发BiRelay2；生成BiRelay3任务，压入future5/future6的栈中，无论future5或future6完成，都会触发BiRelay3任务。

![image-20210813142015829](https://cuichonghe.oss-cn-shenzhen.aliyuncs.com/markdown/image-20210813142015829.png)

BiRelay只是一个中转任务，它本身没有任务代码，只是参照输入的两个future是否完成。如果完成，就从自己的栈中弹出依赖它的BiRelay任务，然后执行。

# JVM

## GC

Minor GC 小GC

Major GC 大GC  等于 Full GC

### 发现垃圾

#### 1.引用计数法

循环应用问题

弱引用或使用另外的算法来排查循环引用等方法解决

#### 2.可达性算法

垃圾收集根元素( Garbage Collection Roots)

- 局部变量(Local variables)
- 活动线程(Active threads)
- 静态域(Static fields)
- JNI 引用(JNI references)
- 其他对象(稍后介绍...)

### 清理垃圾

#### 标记-清除(Mark and Sweep)

标记阶段 在安全点(safe point)STW,

会有STW

有内存碎片

#### 标记整理

无内存碎片

#### 标记复制

### 分代

#### 1.新生代

##### eden区

Eden 区被划分为多个 ==线程本地分配缓冲区==(Thread Local Allocation Buffer, 简称==TLAB==)

大部分对象直接由JVM 在对应线程的TLAB 中分配, 避免与其他线程的同步操作。



如果TLAB 中没有足够的内存空间, 就会在共享Eden 区(shared Eden space)之中分配。如果共享Eden 区也没有足
够的空间, 就会触发一次年轻代GC 来释放内存空间。如果GC 之后Eden 区依然没有足够的空闲内存区域, 则对象
就会被分配到老年代空间(Old Generation)。



标记-复制

##### from/to区

JVM 中阈值默认设置为15 个GC 周期。这也是HotSpot 中的最大值

```
‐XX:+MaxTenuringThreshold
```

#### 2.老年代

#### 3.永久代(PermGen)

Java 8之前

存储元数据(metadata)的地方,比如class 信息 内部化的字符串(internalized strings)等

```
‐XX:MaxPermSize=256m
```

#### 4.Metaspace

java8 替换永久代PermGen

位于本地内

```
‐XX:MaxMetaspaceSize=256m
```

### 查看GC信息

```

jstat ‐gc ‐t 4235 1s

‐XX:+PrintGCDetails ‐XX:+PrintGCDateStamps
‐XX:+PrintGCTimeStamps
```

### GC算法实现

#### 串行GC(Serial GC)

年轻代使用markcopy(标记-复制) 算法, 对老年代使用marksweepcompact(标记-清除-整理)算法

单线程

```
‐XX:+UseSerialGC
```



#### 并行GC(Parallel GC)

年轻代使用标记-复制(markcopy)算法, 在老年代使用标记-清除-整理(marksweepcompact)算法

```shell
‐XX:+UseParallelGC  #启用
‐XX:+UseParallelOldGC
‐XX:+UseParallelGC

‐XX:ParallelGCThreads=NNN #来指定GC 线程数。其默认值为CPU 内核数
```





#### CMS

年轻代的并行GC(Parallel New) + 老年代的CMS(Concurrent Mark and Sweep)



CMS 的设计目标是避免在老年代垃圾收集时出现长时间的卡顿。主要通过两种手段来达成此目标。

* 第一, 不对老年代进行整理, 而是使用空闲列表(freelists)来管理内存空间的回收。

* 第二, 在markandsweep(标记-清除) 阶段的大部分工作和应用线程一起并发执行

  

```
‐XX:+UseConcMarkSweepGC
```

默认情况下, CMS 使用的并发线程数等于CPU 内核数的1/4



初始标记

并发标记

并发预处理

并发可取消预处理

最终标记

并发清除

并发重置



#### G1

负责回收年轻代和老年代



垃圾最多的region被优先回收

```
‐XX:+UseG1GC
```

当堆内存的总体使用比例达到一定数值时,就会触发并发标记。默认值为45% , 但也可以通过JVM 参数
InitiatingHeapOccupancyPercent 来设置



阶段1.  Initial Mark(初始标记) 此阶段标记所有从GC root 直接可达的对象。在CMS 中需要一次STW 暂停, 但G1里面通常是在转移暂停的同时处理这些事情, 所以它的开销是很小的. 可以在Evacuation Pause 日志中的第一行看到(initialmark)暂停:

阶段2. Root Region Scan(Root 区扫描). 此阶段标记所有从"根区域" 可达的存活对象。根区域包括: 非空的区域,以及在标记过程中不得不收集的区域

阶段3. Concurrent Mark(并发标记). 此阶段非常类似于CMS: 它只是遍历对象图, 并在一个特殊的位图中标记能访
问到的对象. 为了确保标记开始时的快照准确性, 所有应用线程并发对对象图执行的引用更新,G1 要求放弃前面阶段
为了标记目的而引用的过时引用

这是通过使用Pre‐Write 屏障来实现的,(不要和之后介绍的Post‐Write 混淆, 也不要和多线程开发中的内存屏障
(memory barriers)相混淆)。PreWrite屏障的作用是: G1 在进行并发标记时, 如果程序将对象的某个属性做了变更, 就
会在log buffers 中存储之前的引用。由并发标记线程负责处理。

阶段4. Remark(再次标记). 和CMS 类似,这也是一次STW 停顿,以完成标记过程。对于G1,它短暂地停止应用线程,停止并发更新日志的写入, 处理其中的少量信息, 并标记所有在并发标记开始时未被标记的存活对象。这一阶段也执
行某些额外的清理, 如引用处理(参见Evacuation Pause log) 或者类卸载(class unloading)。

阶段5. Cleanup(清理). 最后这个小阶段为即将到来的转移阶段做准备, 统计小堆区中所有存活的对象, 并将小堆区
进行排序, 以提升GC 的效率. 此阶段也为下一次标记执行所有必需的整理工作(housekeeping activities): 维护并发
标记的内部状态。
最后要提醒的是, 所有不包含存活对象的小堆区在此阶段都被回收了。有一部分是并发的: 例如空堆区的回收,还有大
部分的存活率计算, 此阶段也需要一个短暂的STW 暂停, 以不受应用线程的影响来完成作业. 



每个小堆区都有一个remembered set, 列出了从外部指向本区的所有引用

#### Shenandoah





关于Shenandoah 的更多信息,请参考博客: https://rkennke.wordpress.com/, JEP 文档:
http://openjdk.java.net/jeps/189, 或者Google 搜索"Shenandoah GC"。



### GC 调优

Latency(延迟)
Throughput(吞吐量)
Capacity(系统容量)



### GC 调优工具

#### JMX API

从JVM 运行时获取GC 行为数据, 最简单的办法是使用标准JMX API 接口. JMX 是获取JVM 内部运行时状态信息
的标准API. 可以编写程序代码, 通过JMX API 来访问本程序所在的JVM，也可以通过JMX 客户端执行(远程)访问。

JConsole 

JVisualVM

Java Mission Control

连接远端JVM, 则目标JVM 启动时必须指定特定的环境变量,以开启远程JMX 连接/以及端口号

```
java ‐Dcom.sun.management.jmxremote.port=5432 com.yourcompany.YourApp
```

#### GCViewer

GC 日志解析为图形信息

#### 分析器(Profilers)

要检测是否因为GC 而引起延迟或吞吐量问题时, 不需要使用分析器. 前面提到的工具(jstat 或原生/可视化GC 日志)就能更好更快地检测出是否存在GC 问题. 特别是从生产环境中收集性能数据时, 最好不要使用分析器, 因为性能开销非常大。

分配分析能定位到在哪个地方创建了大量的对象. 使用分析器辅助进行GC 调优的好处是, 能确定哪种类型的对象最
占用内存, 以及哪些线程创建了最多的对象。

##### hprof

内置于JDK

```
java ‐agentlib:hprof=heap=sites com.yourcompany.YourApplication
```

##### JVisualVM

##### AProf

```
java ‐javaagent:/path‐to/aprof.jar com.yourcompany.YourApplication
```

能够快速找到最耗内存的部分。在我们看来, AProf 是最有用的分配分析器, 因为它只专注于内存分配, 所以做得最好。当然, 这款工具是开源免费的, 资源开销也最小。

### GC调优实战

#### 分配速率( Allocation rate )

表示单位时间内分配的内存量。通常使用MB/sec 作为单位, 也可以使用PB/year等。
分配速率过高就会严重影响程序的性能。在JVM 中会导致巨大的GC 开销

指定JVM 参数: ‐XX:+PrintGCDetails ‐XX:+PrintGCTimeStamps , 通过GC 日志来计算分配速率.

计算上一次垃圾收集之后,与下一次GC 开始之前的年轻代使用量, 两者的差值除以时间,就是分配速率

分配速率 影响 年轻代的minor GC , 老年代GC 的频率和持续时间不受( allocation rate )的直接影响, 而是受到( promotion rate )的影响

增加堆内存或者优化代码

#### 提升速率( promotion rate )

用于衡量单位时间内从年轻代提升到老年代的数据量。一般使用MB/sec 作为单位

GC 之前和之后的年轻代使用量以及堆内存使用量。这样就可以通过差值算出老年代的使用量

增加堆内存,或者重新指定年轻代的大小。优化数据结构, 减少内存消耗。总体目标一致: 让临时数据能够在年轻代存放得下。

#### Weak, Soft , Phantom reference 

为了防止可回收对象的残留, 虚引用对象不应该被获取: phantom reference 的get 方法返回值永远是null 。

与软引用和弱引用不同, 虚引用不会被GC 自动清除, 因为他们被存放到队列中. 通过虚引用可达的对象会继续
留在内存中, 直到调用此引用的clear 方法, 或者引用自身变为不可达。

必须手动调用clear() 来清除虚引用, 否则可能会造成OutOfMemoryError 而导致JVM 挂掉. 使用虚
引用的理由是, 对于用编程手段来跟踪某个对象何时变为不可达对象, 这是唯一的常规手段。和软引用/弱引用不同
的是, 我们不能复活虚可达(phantomreachable)
对象。

如果禁用虚引用清理, 增加JVM 启动参数( ‐Dno.ref.clearing=true ),

使用虚引用时要小心谨慎, 并及时清理虚可达对象。如果不清理, 很可能会发生OutOfMemoryError . 请相信我们的
经验教训: 处理reference queue 的线程中如果没catch 住exception , 系统很快就会被整挂了。

用JVM 参数‐XX:+PrintReferenceGC 来看看各种引用对GC 的影响

##### 解决

弱引用( Weak references ) —— 如果某个内存池的使用量增大, 造成了性能问题, 那么增加这个内存池的大小
(可能也要增加堆内存的最大容量)。
虚引用( Phantom references ) —— 请确保在程序中调用了虚引用的clear 方法。编程中很容易忽略某些虚
引用, 或者清理的速度跟不上生产的速度, 又或者清除引用队列的线程挂了, 就会对GC 造成很大压力, 最终可能
引起OutOfMemoryError 。
软引用( Soft references ) —— 如果确定问题的根源是软引用, 唯一的解决办法是修改程序源码, 改变内部
实现逻辑。

#### RMI 与 GC

如果系统提供或者消费RMI 服务, 则JVM 会定期执行full GC 来确保本地未使用的对象在另一端也不占用空间. 记
住, 即使你的代码中没有发布RMI 服务, 但第三方或者工具库也可能会打开RMI 终端. 最常见的元凶是JMX, 如果
通过JMX 连接到远端, 底层则会使用RMI 发布数据。
问题是有很多不必要的周期性full GC。查看老年代的使用情况, 一般是没有内存压力, 其中还存在大量的空闲区域,
但full GC 就是被触发了, 也就会暂停所有的应用线程。
这种周期性调用System.gc() 删除远程引用的行为, 是在sun.rmi.transport.ObjectTable 类中, 通过
sun.misc.GC.requestLatency(long gcInterval) 调用的。
对许多应用来说, 根本没必要, 甚至对性能有害。禁止这种周期性的GC 行为, 可以使用以下JVM 参数:

```shell
java ‐Dsun.rmi.dgc.server.gcInterval=9223372036854775807L
‐Dsun.rmi.dgc.client.gcInterval=9223372036854775807L
com.yourcompany.YourApplication
#这让Long.MAX_VALUE 毫秒之后, 才调用System.gc() , 实际运行的系统可能永远都不会触发
```

另一种方式是指定JVM 参数‐XX:+DisableExplicitGC , 禁止显式地调用System.gc() . 但我们强烈反对这种方
式, 因为埋有地雷。

#### JVMTI tagging

如果在程序启动时指定了Java Agent ( ‐javaagent ), agent 就可以使用JVMTI tagging 标记堆中的对象。agent 使
用tagging 的种种原因本手册不详细讲解, 但如果tagging 标记了大量的对象, 很可能会引起GC 性能问题, 导致延迟
增加, 以及吞吐量降低。
问题发生在native 代码中, JvmtiTagMap::do_weak_oops 在每次GC 时, 都会遍历所有标签(tag),并执行一些比较耗
时的操作。更坑的是, 这种操作是串行执行的。
如果存在大量的标签, 就意味着GC 时有很大一部分工作是单线程执行的, GC 暂停时间可能会增加一个数量级。
检查是否因为agent 增加了GC 暂停时间, 可以使用诊断参数–XX:+TraceJVMTIObjectTagging . 启用跟踪之后, 可以
估算出内存中tag 映射了多少native 内存, 以及遍历所消耗的时间。
如果你不是agent 的作者, 那一般是搞不定这类问题的。除了提BUG 之外你什么都做不了. 如果发生了这种情况, 请
建议厂商清理不必要的标签。

#### Humongous Allocations大对象

G1 中, 巨无霸对象是指所占空间超过一个小堆区(region) 50% 的对象

G1 垃圾收集算法, 会产生一种巨无霸对象引起的GC 性能问题

监控

```
java ‐XX:+PrintGCDetails ‐XX:+PrintGCTimeStamps
‐XX:+PrintReferenceGC ‐XX:+UseG1GC
‐XX:+PrintAdaptiveSizePolicy ‐Xmx128m
MyClass
```

第一种解决方式, 是修改region size , 以使得大多数的对象不超过50% , 也就不进行巨无霸对象区域的分配。
region 的默认大小在启动时根据堆内存的大小算出。但也可以指定参数来覆盖默认设置, ‐
XX:G1HeapRegionSize=XX 。指定的region size 必须在1~32MB 之间, 还必须是2 的幂【2^10 = 1024 = 1KB;
2^20=1MB; 所以region size 只能是: 1m , 2m , 4m , 8m , 16m , 32m 】。
这种方式也有副作用, 增加region 的大小也就变相地减少了region 的数量, 所以需要谨慎使用, 最好进行一些测试,
看看是否改善了吞吐量和延迟。
更好的方式需要一些工作量, 如果可以的话, 在程序中限制对象的大小。最好是使用分析器, 展示出巨无霸对象的信
息, 以及分配时所在的堆栈跟踪信息。









# 正则表达式

```
定义字符串的组成规则。
1. 单个字符:[]
    如： [a] [ab] [a-zA-Z0-9_]
     特殊符号代表特殊含义的单个字符:
     \d:单个数字字符 [0-9]
     \w:单个单词字符[a-zA-Z0-9_]
   1. 量词符号：
?：表示出现0次或1次
*：表示出现0次或多次
+：出现1次或多次
                        {m,n}:表示 m<= 数量 <= n
                            * m如果缺省： {,n}:最多n次
                            * n如果缺省：{m,} 最少m次
                    1. 开始结束符号
                        * ^:开始
                        * $:结束
2. 



\d      匹配一个数字字符。等价于 [0-9]。
\D     匹配一个非数字字符。等价于 [^0-9]。
\f       匹配一个换页符。等价于 \x0c 和 \cL。
\n      匹配一个换行符。等价于 \x0a 和 \cJ。
\r       匹配一个回车符。等价于 \x0d 和 \cM。
\s      匹配任何空白字符，包括空格、制表符、换页符等等。等价于 [ \f\n\r\t\v]。
\S      匹配任何非空白字符。等价于 [^ \f\n\r\t\v]。
\t       匹配一个制表符。等价于 \x09 和 \cI。
\v      匹配一个垂直制表符。等价于 \x0b 和 \cK。
\w     匹配字母、数字、下划线。等价于'[A-Za-z0-9_]'。
\W    匹配非字母、数字、下划线。等价于 '[^A-Za-z0-9_]
```

