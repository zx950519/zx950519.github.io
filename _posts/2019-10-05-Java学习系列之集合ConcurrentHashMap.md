---
layout: post
title:  "Java学习系列之集合ConcurrentHashMap"
categories: Java
tags:  Java 集合
---

* content
{:toc}

本文系记录对Java中集合类ConcurrentHashMap的学习资料，如有异议，欢迎联系我讨论修改。PS:图侵删！如果看不清图请访问https://github.com/zx950519/zx950519.github.io/blob/master/_posts/2019-10-05-Java%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97%E4%B9%8B%E9%9B%86%E5%90%88.md






- [1 并发安全的哈希](#1-并发安全的哈希)
  - [1.1 简述对HashMap的理解](#11-简述对hashmap的理解)
  - [1.2 JDK7的ConcurrentHashMap](#12-jdk7的concurrenthashmap)
  - [1.3 JDK8的ConcurrentHashMap](#13-jdk8的concurrenthashmap)
  - [1.4 其他](#14-其他)



# 1 并发安全的哈希

## 为什么需要ConcurrentHashMap
HashMap在多线程环境下非线程安全，而HashTable通多对方法使用Synchronized加以修饰从而达到线程安全，但是HashTable实现同步的方法比较暴力，相当于所有读写线程均去读取一把锁，效率比较低下。另外一种同步Map的方法是使用Collections工具类，例如：  
```java
Map<Integer, Integer> map = Collections.synchronizedMap(new HashMap<>());
```
该种方法与HashTable实现方式类似，也是通过锁住整表来实现同步的。而ConcurrentHashMap则避免了上述两种Map同步方式锁住全表的问题。在JDK1.7中，通过引入分段锁Segment(实现了ReentrantLock)，保证了一定粒度的并行；在JDK1.8中则抛弃Segment的设计理念，将粒度完全降低到桶数量，基于CAS与Synchronized。  

## ConcurrentHashMap的基本设计思想
ConcurrentHashMap和HashMap实现上类似，最主要的差别是ConcurrentHashMap 采用了分段锁（Segment），每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是Segment的个数）。其中Segment继承自ReentrantLock。默认的并发级别为16，也就是说默认创建16个Segment。JDK1.7使用分段锁机制来实现并发更新操作，核心类为Segment，它继承自重入锁ReentrantLock，并发度与Segment数量相等。JDK1.8使用了CAS操作来支持更高的并发度，在CAS操作失败时使用内置锁synchronized。并且 JDK 1.8 的实现也在链表过长时会转换为红黑树。


## 1.2 JDK7的ConcurrentHashMap

## 基本结构
http://ww1.sinaimg.cn/large/005L0VzSly1g7ndp1emulj30md0beac3.jpg  

#### Segment
Segment是JDK1.7中ConcurrentHashMap的核心设计，通过引入分段达成提高并行处理度的效果。易于看出，Segment继承了ReentrantLock并实现了序列化接口，说明Segment的锁是可重入的。  
```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {
    transient volatile int count;   // Segment中元素的数量，由volatile修饰，支持内存可见性
    transient int modCount;         // 对table的大小造成影响的操作的数量（比如put或者remove操作）
    transient int threshold;        // 扩容阈值
    transient volatile HashEntry<K,V>[] table;    // 链表数组，数组中的每一个元素代表了一个链表的头部
    final float loadFactor;         // 负载因子
}
```

#### HashEntry
Segment中的元素是以HashEntry的形式存放在链表数组中的，其结构与普通HashMap的HashEntry基本一致，不同的是Segment的HashEntry，其value由volatile修饰，以支持内存可见性，即写操作对其他读线程即时可见。  
```java
static final class HashEntry<K,V> {
    final K key;
    final int hash;
    volatile V value;
    final HashEntry<K,V> next;    
}
```
值得注意的是，key，hash值与next域都是由final修饰的，无法修改其引用。  

## 初始化过程
下面以一个ConcurrentHashMap构造函数为例进行分析：  
```java
public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    // 根据传入的concurrencyLevel更新sshift与ssize
    // 如concurrencyLevel=5 则天花板方向上离2^3=8最近，则sshift=3，ssize=8
    int sshift = 0;
    int ssize = 1;
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    // segmentShift和segmentMask元素定位Segment下标
    segmentShift = 32 - sshift;   // 计算Segment下标时首先令hash值的位数
    segmentMask = ssize - 1;      // 计算Segment下标时随后求余数的操作数
    // 按照ssize的大小对segment数组进行初始化
    this.segments = Segment.newArray(ssize);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    int c = initialCapacity / ssize;
    // 若initialCapacity / ssize不整除，则将c=c+1
    if (c * ssize < initialCapacity)
        ++c;
    int cap = 1;
    // cap为每个segment的初始容量，其值为离c天花板方向最近的2^n
    // 例：c为5，cap则为2^3=8；c为12，cap则为2^4=16
    while (cap < c)
        cap <<= 1;
    // 创建Segment
    for (int i = 0; i < this.segments.length; ++i)
        this.segments[i] = new Segment<K,V>(cap, loadFactor);
}
```

## get方法
根据key获取value时，由于1.7中需要两次Hash过程，第一次需要定位到Segment；第二次需要定位到Segment中的桶下标。
```java
public V get(Object key) {
    // 首先计算key的hash值
    int hash = hash(key.hashCode());
    // segmentFor(hash): 定位到key在哪个segment
    // 调用segment的get(key, hash)获取到指定key的value
    return segmentFor(hash).get(key, hash);
}
final Segment<K,V> segmentFor(int hash) {
    return segments[(hash >>> segmentShift) & segmentMask];
}
V get(Object key, int hash) {
    if (count != 0) { // read-volatile
        // 取得链表的头部，就是第2次hash过程
        HashEntry<K,V> e = getFirst(hash);
        // 链表搜索，直到hash与key均相等时，返回节点的value
        while (e != null) {
            if (e.hash == hash && key.equals(e.key)) {
                V v = e.value;
                if (v != null)
                    return v;
                return readValueUnderLock(e); // recheck
            }
            e = e.next;
        }
    }
    return null;
}
HashEntry<K,V> getFirst(int hash) {
    // 获取Segment的数组结构
    HashEntry<K,V>[] tab = table;
    // 第2次hash过程，确定key位于哪一个HashEntry
    return tab[hash & (tab.length - 1)];
}
```
在第二次查找具体元素时，首先对count做了非零判断，由于count是volatile修饰的，put、remove等操作会更新count的值，所以当竞争发生的时候，volatile的语义可以保证写操作在读操作之前，也就保证了写操作对后续的读操作都是可见的，这样后面get的后续操作就可以拿到完整的元素内容。

## put方法
put操作也涉及2次hash定位过程，但是比get操作多了是否扩容、rehash等过程，2次哈希在此不做过多赘述。  
```java
V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // 先加锁
    lock();
    try {
        int c = count;
        // 对c进行+1操作，获取新的数组容量
        // 如果新的数组容量高于阈值，则先进行扩容操作
        if (c++ > threshold) // ensure capacity
            rehash();
        // 获取Segment的数组table
        HashEntry<K,V>[] tab = table;
        // 2次hash过程确定桶下标
        int index = hash & (tab.length - 1);
        // 获取index位置的头结点
        HashEntry<K,V> first = tab[index];
        HashEntry<K,V> e = first;
        // 沿链表遍历，直到找到与元素key或者hash值相同的节点
        while (e != null && (e.hash != hash || !key.equals(e.key)))
            e = e.next;
        V oldValue;
        // 若key或者hash值相同的节点存在，则进行更新操作
        if (e != null) {
            // value也是volatile修饰的，所以内存即时可见
            oldValue = e.value;
            if (!onlyIfAbsent)
                e.value = value;
        }
        // 若key或者hash值相同的节点不存在，则新建节点，追加到当前链表的头部(头插法)
        else {
            oldValue = null;
            // 更新modCount的值，记录对table的大小造成影响的操作的数量
            ++modCount;
            tab[index] = new HashEntry<K,V>(key, hash, first, value);
            // 更新count的值，内存即时可见
            count = c; // write-volatile
        }
        // 返回旧值
        return oldValue;
    } finally {
        // 必须在finally中显示释放
        unlock();
    }
}
```

## remove方法
```java
V remove(Object key, int hash, Object value) {
    // 加锁，除了读取操作，其他操作均需要加锁
    lock();
    try {
        // 计算新的Segment元素数量
        int c = count - 1;
        // 获取Segment的数组table
        HashEntry<K,V>[] tab = table;
        // 第2次hash，确定在table的哪个位置
        int index = hash & (tab.length - 1);
        HashEntry<K,V> first = tab[index];
        HashEntry<K,V> e = first;
        // 沿链表遍历，直到找到与元素key或者hash值相同的节点
        while (e != null && (e.hash != hash || !key.equals(e.key)))
            e = e.next;
        V oldValue = null;
        if (e != null) {
            V v = e.value;
            if (value == null || value.equals(v)) {
                oldValue = v;
                // 更新modCount值              
                ++modCount;
                HashEntry<K,V> newFirst = e.next;
                // 将待删除元素的前面的元素全部复制一遍，然后头插到链表上去
                for (HashEntry<K,V> p = first; p != e; p = p.next)
                    newFirst = new HashEntry<K,V>(p.key, p.hash,
                                                  newFirst, p.value);
                tab[index] = newFirst;
                // 更新新的Segment元素数量，内存即时可见
                count = c; // write-volatile
            }
        }
        // 返回旧值
        return oldValue;
    } finally {
        // 释放锁
        unlock();
    }
}
```
由于，HashEntry中的next是final的，一经赋值以后就不可修改，在定位到待删除元素的位置以后，程序就将待删除元素前面的那一些元素全部复制一遍，然后再一个一个重新接到链表上去。使用final保证其不变性从而易于并发读取，但是当修改时，其成本就显得有些高了。

## 1.3 JDK8的ConcurrentHashMap

## 基本结构
http://ww1.sinaimg.cn/large/005L0VzSly1g7nehxnf22j30hy0fyab5.jpg  
在JDK1.8的ConcurrentHashMap中，完全摒弃1.7中segment的概念，保持与1.8HashMap一致的设计，通过引入CAS与Synchronized来保证最大粒度的并行。  

#### 重要全局变量
```java
    static final int MOVED     = -1; // 表示正在转移
    static final int TREEBIN   = -2; // 表示已经转换成树
    static final int RESERVED  = -3; // hash for transient reservations
    static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash
    transient volatile Node<K,V>[] table;//默认没初始化的数组，用来保存元素
    private transient volatile Node<K,V>[] nextTable;//转移的时候用的数组
    /**
     * 用来控制表初始化和扩容的，默认值为0，当在初始化的时候指定了大小，这会将这个大小保存在sizeCtl中，大小为数组的0.75
     * 当为负的时候，说明表正在初始化或扩张，
     *     -1 表示初始化  
     *     -(1+n) n:表示活动的扩张线程
     */
    private transient volatile int sizeCtl;
```
#### Node内部类
```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;    //key的hash值
    final K key;       //key
    volatile V val;    //value
    volatile Node<K,V> next; //表示链表中的下一个节点
    Node(int hash, K key, V val, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.val = val;
        this.next = next;
    }
    public final K getKey()       { return key; }
    public final V getValue()     { return val; }
    public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
}
```

#### TreeNode内部类
```java
static final class TreeNode<K,V> extends Node<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next,
             TreeNode<K,V> parent) {
        super(hash, key, val, next);
        this.parent = parent;
    }
}
```
#### TreeBin内部类
```java
static final class TreeBin<K,V> extends Node<K,V> {
    TreeNode<K,V> root;
    volatile TreeNode<K,V> first;
    volatile Thread waiter;
    volatile int lockState;
    // values for lockState
    static final int WRITER = 1; // set while holding write lock
    static final int WAITER = 2; // set when waiting for write lock
    static final int READER = 4; // increment value for setting read lock
}
```

#### 初始化方法
初始化方法均采用延迟初始化的形式。  
```java
public ConcurrentHashMapDebug(int initialCapacity) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
               MAXIMUM_CAPACITY :
               tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
    this.sizeCtl = cap;
}
```

## put方法
- 先判断key与value是否为空。与HashMap不同，ConcurrentHashMap不允许null作为key或value，为什么这样设计? 因为ConcurrentHashmap是支持并发，这样会有一个问题，当你通过get(k)获取对应的value时，如果获取到的是null时，你无法判断，它是put（k,v）的时候value为null，还是这个key从来没有做过映射。HashMap是非并发的，可以通过contains(key)来做这个判断。而支持并发的Map在调用m.contains（key）和m.get(key)，可能已经不同了；
- 计算hash值来确定放在数组的哪个位置
- 判断当前table是否为空，空的话初始化table
- 根据重哈希算出的值通过与运算得到桶索引，利用Unsafe类直接获取内存内存中对应位置上的节点，若没有碰撞即桶中无结点CAS直接添加
- 如果取出来的节点的hash值是MOVED(-1)的话，则表示当前正在对这个数组进行扩容，复制到新的数组，则当前线程也去帮助复制
- 最后一种情况就是，如果这个节点，不为空，也不在扩容，则通过synchronized来加锁，进行添加操作
- 其他部分同HashMap中的操作

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();   //K,V都不能为空，否则的话跑出异常
    int hash = spread(key.hashCode());    //取得key的hash值
    int binCount = 0;                     //用来计算在这个节点总共有多少个元素，用来控制扩容或者转移为树
    for (Node<K,V>[] tab = table;;) {     //
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)    
            tab = initTable();    //第一次put的时候table没有初始化，则初始化table
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {    //通过哈希计算出一个表中的位置因为n是数组的长度，所以(n-1)&hash肯定不会出现数组越界
            if (casTabAt(tab, i, null,        //如果这个位置没有元素的话，则通过cas的方式尝试添加，注意这个时候是没有加锁的
                         new Node<K,V>(hash, key, value, null)))        //创建一个Node添加到数组中区，null表示的是下一个节点为空
                break;                   // no lock when adding to empty bin
        }
        /*
         * 如果检测到某个节点的hash值是MOVED，则表示正在进行数组扩张的数据复制阶段，
         * 则当前线程也会参与去复制，通过允许多线程复制的功能，一次来减少数组的复制所带来的性能损失
         */
        else if ((fh = f.hash) == MOVED)    
            tab = helpTransfer(tab, f);
        else {
            /*
             * 如果在这个位置有元素的话，就采用synchronized的方式加锁，
             *     如果是链表的话(hash大于0)，就对这个链表的所有元素进行遍历，
             *         如果找到了key和key的hash值都一样的节点，则把它的值替换到
             *         如果没找到的话，则添加在链表的最后面
             *  否则，是树的话，则调用putTreeVal方法添加到树中去
             *  
             *  在添加完之后，会对该节点上关联的的数目进行判断，
             *  如果在8个以上的话，则会调用treeifyBin方法，来尝试转化为树，或者是扩容
             */
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {        //再次取出要存储的位置的元素，跟前面取出来的比较
                    if (fh >= 0) {                //取出来的元素的hash值大于0，当转换为树之后，hash值为-2
                        binCount = 1;            
                        for (Node<K,V> e = f;; ++binCount) {    //遍历这个链表
                            K ek;
                            if (e.hash == hash &&        //要存的元素的hash，key跟要存储的位置的节点的相同的时候，替换掉该节点的value即可
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)        //当使用putIfAbsent的时候，只有在这个key没有设置值得时候才设置
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {    //如果不是同样的hash，同样的key的时候，则判断该节点的下一个节点是否为空，
                                pred.next = new Node<K,V>(hash, key,        //为空的话把这个要加入的节点设置为当前节点的下一个节点
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {    //表示已经转化成红黑树类型了
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,    //调用putTreeVal方法，将该元素添加到树中去
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)    //当在同一个节点的数目达到8个的时候，则扩张数组或将给节点的数据转为tree
                    treeifyBin(tab, i);    
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);    //计数
    return null;
}
```
关于put中对CAS和synchronized的使用：  
- CAS用于当桶为空时，使用cas尝试加入新的桶头结点
- synchronized用于桶不为空时，向链表或树中put结点的情形

## get方法
- 当key为null的时候回抛出NullPointerException的异常
- get操作通过首先计算key的hash值来确定该元素放在数组的哪个位置
- 判断table是否为空且table长度大于0且下标不为空
- 然后遍历该位置的所有节点
- 如果均无法定位到key则返回null
```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

## resize方法
当需要扩容的时候，调用的时候tryPresize方法。在tryPresize方法中，并没有加锁，允许多个线程进入，如果数组正在扩张，则当前线程也去帮助扩容。值得注意的是，复制之后的新链表不是旧链表的绝对倒序；在扩容的时候每个线程都有处理的步长，最少为16，在这个步长范围内的数组节点只有自己一个线程来处理。整个操作是在持有段锁的情况下执行。
```java
private final void tryPresize(int size) {
    int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
        tableSizeFor(size + (size >>> 1) + 1);
    int sc;
    while ((sc = sizeCtl) >= 0) {
        Node<K,V>[] tab = table; int n;
        /*
         * 如果数组table还没有被初始化，则初始化一个大小为sizeCtrl和刚刚算出来的c中较大的一个大小的数组
         * 初始化的时候，设置sizeCtrl为-1，初始化完成之后把sizeCtrl设置为数组长度的3/4
         * 为什么要在扩张的地方来初始化数组呢？这是因为如果第一次put的时候不是put单个元素，
         * 而是调用putAll方法直接put一个map的话，在putALl方法中没有调用initTable方法去初始化table，
         * 而是直接调用了tryPresize方法，所以这里需要做一个是不是需要初始化table的判断
         */
        if (tab == null || (n = tab.length) == 0) {
            n = (sc > c) ? sc : c;
            if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {    //初始化tab的时候，把sizeCtl设为-1
                try {
                    if (table == tab) {
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
            }
        }
        /*
         * 一直扩容到的c小于等于sizeCtl或者数组长度大于最大长度的时候，则退出
         * 所以在一次扩容之后，不是原来长度的两倍，而是2的n次方倍
         */
        else if (c <= sc || n >= MAXIMUM_CAPACITY) {
                break;    //退出扩张
        }
        else if (tab == table) {
            int rs = resizeStamp(n);
            /*
             * 如果正在扩容Table的话，则帮助扩容
             * 否则的话，开始新的扩容
             * 在transfer操作，将第一个参数的table中的元素，移动到第二个元素的table中去，
             * 虽然此时第二个参数设置的是null，但是，在transfer方法中，当第二个参数为null的时候，
             * 会创建一个两倍大小的table
             */
            if (sc < 0) {
                Node<K,V>[] nt;
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                /*
                 * transfer的线程数加一,该线程将进行transfer的帮忙
                 * 在transfer的时候，sc表示在transfer工作的线程数
                 */
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            /*
             * 没有在初始化或扩容，则开始扩容
             */
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2)) {
                    transfer(tab, null);
            }
        }
    }
}
```
- 复制之后的新链表不是旧链表的绝对倒序
- 在扩容的时候每个线程都有处理的步长，最少为16，在这个步长范围内的数组节点只有自己一个线程来处理

## remove方法
首先remove操作也是确定需要删除的元素的位置，不过这里删除元素的方法不是简单地把待删除元素的前面的一个元素的next指向后面一个就完事了，我们之前已经说过HashEntry中的next是final的，一经赋值以后就不可修改，在定位到待删除元素的位置以后，程序就将待删除元素前面的那一些元素全部复制一遍，然后再一个一个重新接到链表上去。中间那个for循环是做什么用的呢？（第22行）从代码来看，就是将定位之后的所有entry克隆并拼回前面去，但有必要吗？每次删除一个元素就要将那之前的元素克隆一遍？这点其实是由entry的不变性来决定的，仔细观察entry定义，发现除了value，其他所有属性都是用final来修饰的，这意味着在第一次设置了next域之后便不能再改变它，取而代之的是将它之前的节点全都克隆一次。至于entry为什么要设置为不变性，这跟不变性的访问不需要同步从而节省时间有关。  
```java
V remove(Object key, int hash, Object value) { 
    lock(); 
    try { 
        int c = count - 1; 
        HashEntry<K,V>[] tab = table; 
        int index = hash & (tab.length - 1); 
        HashEntry<K,V> first = tab[index]; 
        HashEntry<K,V> e = first; 
        while (e != null && (e.hash != hash || !key.equals(e.key))) 
            e = e.next; 
        V oldValue = null; 
        if (e != null) { 
            V v = e.value; 
            if (value == null || value.equals(v)) { 
                oldValue = v; 
                // All entries following removed node can stay 
                // in list, but all preceding ones need to be 
                // cloned. 
                ++modCount; 
                HashEntry<K,V> newFirst = e.next; 
                for (HashEntry<K,V> p = first; p != e; p = p.next) 
                    newFirst = new HashEntry<K,V>(p.key, p.hash, 
                                                  newFirst, p.value); 
                tab[index] = newFirst; 
                count = c; // write-volatile 
            } 
        } 
        return oldValue; 
    } finally { 
        unlock(); 
    } 
}
```

## size方法
计算ConcurrentHashMap的元素大小是一个有趣的问题，因为他是并发操作的，就是在你计算size的时候，他还在并发的插入数据，可能会导致你计算出来的size和你实际的size有相差（在你return size的时候，插入了多个数据），要解决这个问题，JDK1.7版本用两种方案：  
- 第一种方案使用不加锁的模式去尝试多次计算ConcurrentHashMap的size，最多三次，比较前后两次计算的结果，结果一致就认为当前没有元素加入，计算的结果是准确的
- 第二种方案是如果第一种方案不符合，就会给每个Segment加上锁，然后计算ConcurrentHashMap的size返回

## 1.4 其他

## ConcurrentHashMap的工作原理以及版本差异
ConcurrentHashMap为了提高本身的并发能力，在内部采用了一个叫做Segment的结构，一个Segment其实就是一个类HashTable的结构，Segment内部维护了一个链表数组。ConcurrentHashMap定位一个元素的过程需要进行两次Hash操作，第一次 Hash 定位到 Segment，第二次Hash定位到元素所在的链表的头部，因此，这一种结构的带来的副作用是Hash的过程要比普通的HashMap要长，但是带来的好处是写操作的时候可以只对元素所在的Segment进行操作即可，不会影响到其他的Segment，这样，在最理想的情况下，ConcurrentHashMap 可以最高同时支持 Segment 数量大小的写操作（刚好这些写操作都非常平均地分布在所有的 Segment上），所以，通过这一种结构，ConcurrentHashMap的并发能力可以大大的提高。JAVA7之前ConcurrentHashMap主要采用锁机制，在对某个Segment进行操作时，将该Segment锁定，不允许对其进行非查询操作，而在JAVA8之后采用CAS无锁算法，这种乐观操作在完成前进行判断，如果符合预期结果才给予执行，对并发操作提供良好的优化。而1.8中做了进一步的优化：  
- JDK1.8的实现降低锁的粒度，JDK1.7版本锁的粒度是基于Segment的，包含多个HashEntry，而JDK1.8锁的粒度就是HashEntry（首节点） 
- JDK1.8版本的数据结构变得更加简单，使得操作也更加清晰流畅，因为已经使用synchronized来进行同步，所以不需要分段锁的概念，也就不需要Segment这种数据结构了，由于粒度的降低，实现的复杂度也增加了
- JDK1.8使用红黑树来优化链表，基于长度很长的链表的遍历是一个很漫长的过程，而红黑树的遍历效率是很快的，代替一定阈值的链表，这样形成一个最佳拍档 
- JDK1.8为什么使用内置锁synchronized来代替重入锁ReentrantLock，我觉得有以下几点：因为粒度降低了，在相对而言的低粒度加锁方式，synchronized并不比ReentrantLock差，在粗粒度加锁中ReentrantLock可能通过Condition来控制各个低粒度的边界，更加的灵活，而在低粒度中，Condition的优势就没有了；在大量的数据操作下，对于JVM的内存压力，基于API的ReentrantLock会产生更多的内存开销。

## ConcurrentHashMap中变量使用final和volatile修饰的作用
- final可实现不变模式（immutable），是多线程安全里最简单的一种保障方式。不变模式主要通过final关键字来限定的。在JMM中final关键字还有特殊的语义。Final域使得确保初始化安全性（initialization safety）成为可能，初始化安全性让不可变形对象不需要同步就能自由地被访问和共享
- 使用volatile来保证某个变量内存的改变对其他线程即时可见，在配合CAS可以实现不加锁对并发操作的支持remove执行的开始就将table赋给一个局部变量tab，将tab依次复制出来，最后直到该删除位置，将指针指向下一个变量

