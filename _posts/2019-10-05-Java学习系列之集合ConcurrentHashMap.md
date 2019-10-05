---
layout: post
title:  "Java学习系列之集合ConcurrentHashMa"
categories: Java
tags:  Java 集合
---

* content
{:toc}

本文系记录对Java中集合类ConcurrentHashMa的学习资料，如有异议，欢迎联系我讨论修改。PS:图侵删！如果看不清图请访问https://github.com/zx950519/zx950519.github.io/blob/master/_posts/2019-10-05-Java%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97%E4%B9%8B%E9%9B%86%E5%90%88.md






- [1 并发安全的哈希](#1-并发安全的哈希)
  - [1.1 简述对HashMap的理解](#11-简述对hashmap的理解)
  - [1.2 HashMap的源码设计特性](#12-hashmap的源码设计特性)
  - [1.3 HashMap中常见方法的源码分析](#13-hashmap中常见方法的源码分析)
  - [1.4 HashMap中resize在多线程环境下的潜在危害](#14-hashmap中resize在多线程环境下的潜在危害)版本区别
  - [1.5 HashMap在不同JDK版本下的区别](#15-hashmap在不同JDK版本下的区别)
  - [1.5 HashMap存在的问题](#15-hashmap存在的问题)
  - [1.6 和其他集合类的比较](#16-和其他集合类的比较)



# 1 并发安全的哈希

## 为什么需要ConcurrentHashMap
HashMap在多线程环境下非线程安全，而HashTable通多对方法使用Synchronized加以修饰从而达到线程安全，但是HashTable实现同步的方法比较暴力，相当于所有读写线程均去读取一把锁，效率比较低下。另外一种同步Map的方法是使用Collections工具类，例如：  
```java
Map<Integer, Integer> map = Collections.synchronizedMap(new HashMap<>());
```
该种方法与HashTable实现方式类似，也是通过锁住整表来实现同步的。而ConcurrentHashMap则避免了上述两种Map同步方式锁住全表的问题。在JDK1.7中，通过引入分段锁Segment(实现了ReentrantLock)，保证了一定粒度的并行；在JDK1.8中则抛弃Segment的设计理念，将粒度完全降低到桶数量，基于CAS与Synchronized。  

## ConcurrentHashMap的基本设计思想
ConcurrentHashMap和HashMap实现上类似，最主要的差别是ConcurrentHashMap 采用了分段锁（Segment），每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是Segment的个数）。其中Segment继承自ReentrantLock。默认的并发级别为16，也就是说默认创建16个Segment。JDK1.7使用分段锁机制来实现并发更新操作，核心类为Segment，它继承自重入锁ReentrantLock，并发度与Segment数量相等。JDK1.8使用了CAS操作来支持更高的并发度，在CAS操作失败时使用内置锁synchronized。并且 JDK 1.8 的实现也在链表过长时会转换为红黑树。


## 1.2 JDK1.7的ConcurrentHashMap

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

## 简述ConcurrentHashMap的工作原理以及版本差异
ConcurrentHashMap 为了提高本身的并发能力，在内部采用了一个叫做 Segment 的结构，一个 Segment 其实就是一个类 Hash Table 的结构，Segment 内部维护了一个链表数组。ConcurrentHashMap定位一个元素的过程需要进行两次Hash操作，第一次 Hash 定位到 Segment，第二次Hash定位到元素所在的链表的头部，因此，这一种结构的带来的副作用是Hash的过程要比普通的HashMap要长，但是带来的好处是写操作的时候可以只对元素所在的Segment进行操作即可，不会影响到其他的Segment，这样，在最理想的情况下，ConcurrentHashMap 可以最高同时支持 Segment 数量大小的写操作（刚好这些写操作都非常平均地分布在所有的 Segment上），所以，通过这一种结构，ConcurrentHashMap的并发能力可以大大的提高。JAVA7之前ConcurrentHashMap主要采用锁机制，在对某个Segment进行操作时，将该Segment锁定，不允许对其进行非查询操作，而在JAVA8之后采用CAS无锁算法，这种乐观操作在完成前进行判断，如果符合预期结果才给予执行，对并发操作提供良好的优化。  

- JDK1.8的实现降低锁的粒度，JDK1.7版本锁的粒度是基于Segment的，包含多个HashEntry，而JDK1.8锁的粒度就是HashEntry（首节点） 
- JDK1.8版本的数据结构变得更加简单，使得操作也更加清晰流畅，因为已经使用synchronized来进行同步，所以不需要分段锁的概念，也就不需要Segment这种数据结构了，由于粒度的降低，实现的复杂度也增加了
- JDK1.8使用红黑树来优化链表，基于长度很长的链表的遍历是一个很漫长的过程，而红黑树的遍历效率是很快的，代替一定阈值的链表，这样形成一个最佳拍档 
- JDK1.8为什么使用内置锁synchronized来代替重入锁ReentrantLock，我觉得有以下几点：因为粒度降低了，在相对而言的低粒度加锁方式，synchronized并不比ReentrantLock差，在粗粒度加锁中ReentrantLock可能通过Condition来控制各个低粒度的边界，更加的灵活，而在低粒度中，Condition的优势就没有了；在大量的数据操作下，对于JVM的内存压力，基于API的ReentrantLock会开销更多的内存

## 简述ConcurrentHashMap中变量使用final和volatile修饰的作用

- final可实现不变模式（immutable），是多线程安全里最简单的一种保障方式。不变模式主要通过final关键字来限定的。在JMM中final关键字还有特殊的语义。Final域使得确保初始化安全性（initialization safety）成为可能，初始化安全性让不可变形对象不需要同步就能自由地被访问和共享
- 使用volatile来保证某个变量内存的改变对其他线程即时可见，在配合CAS可以实现不加锁对并发操作的支持remove执行的开始就将table赋给一个局部变量tab，将tab依次复制出来，最后直到该删除位置，将指针指向下一个变量

## 简述ConcurrentHashMap中remove
- 当要删除的结点存在时，删除的最后一步操作要将count的值减一。这必须是最后一步操作，否则读取操作可能看不到之前对段所做的结构性修改
- remove执行的开始就将table赋给一个局部变量tab，这是因为table是volatile变量，读写volatile变量的开销很大。编译器也不能对volatile变量的读写做任何优化，直接多次访问非volatile实例变量没有多大影响，编译器会做相应优化
