---
title: 一文读懂 ConcurrentHashMap 原理
date: 2024-02-18 12:54:28
tags: 
 - ConcurrentHashMap
categories: 
 - java基础
---



**出现背景**

#### 1、线程不安全的HashMap

- 非原子操作：`HashMap`的操作不是原子性的，例如put()方法涉及到了多个步骤，包括计算hash值，查找或插入元素等，如果多个线程同时执行这些操作，就有可能导致数据不一致的情况。
- 容量扩容，`HashMap`在扩容时，需要重新计算元素的hash值并分配存储位置，这个过程涉及到对原数组进行复制和重新插入元素的操作。如果在扩容期间有其他线程对`HashMap`进行并发修改，就可能导致数据丢失或出现异常

#### 2、效率低下的HashTable

`HashTable`容器使用`synchronized`来保证线程安全，但在线程竞争激烈的情况下`HashTable`的效率非常低，因为当一个线程正在访问`HashTable`的同步方法时，这时，另外一个线程也来访问`HashTable`的同步方法， 可能会进入阻塞和轮询状态。如线程A使用put进行添加元素的时候，线程B不但不能使用put方法添加元素，而且不能使用get方法获取元素，所以竞争越激烈效率越低，也就是说对于`HashTable`而言，`synchronized`是针对Hash整张表的，即每次锁住整张表让线程独占，相当于所有线程进行读写的时候都去竞争一把锁。

#### ConcurrentHashMap的锁分段技术

锁分段技术就是将数据分成一段一段的储存，然后给每一段数据分配一把锁，当一个线程占用锁访问一个段数据时，其他的段数据也是可以被其他线程访问的，另外，`ConcurrentHsahMap`是可以做到读取数据不加锁

`ConcurrentHashMap`是由`Segment`数据结构和`HashEntry`数据结构组成，`Segment`是一种可重入锁`ReentrantLock`，在`ConcurrentHashMap`中扮演锁的角色，`HashEntry`则用于存储键值对数据，一个`ConcurrentHashMap`中包含一个`Segment`数组，`Segment`结构和`HashMap`类似，是一种数组和链表结构，一个`Segment`里包含一个`HashEntry`数组，每个`HashEntry`是一个链表结构的元素，每个`Segment`守护着一个`HashEntry`数组里的元素，当对`HashEntry`数组中的数据进行修改时，必须先获得它对应的`Segment`锁

##### Segment

##### 我们再来具体了解一下Segment的数据结构

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable { 
    transient volatile int count; 
    transient int modCount; 
    transient int threshold; 
    transient volatile HashEntry<K,V>[] table; 
    final float loadFactor; 
}

```

以下时是Segment里面成员变量的意义：

- count：Segment中元素的数量
- modCount：对table的大小造成影响的操作的数量（比如put或者remove操作）
- threshold：阈值，Segment里面元素的数量超过这个值依旧就会对Segment进行扩容
- table：链表数组，数组中的每一个元素代表了一个链表的头部
- loadFactor：负载因子，用于确定threshold

count用来统计该段数据的个数，它是volatile变量，它用来协调修改和读取操作，以保证读取操作能够读取到几乎最新修改的值，协调方法是这样的，每次修改操作做了结构上的变化，如增加/删除节点（修改节点的值不算结构上的变化），都要写count值，每次读取操作开始都要读取count的值。这利用了Java5中对volatile语义的增强，对同一个volatile变量的写和读存在happens-begore关系。modCount统计段数据改变的次数，主要为了检测对多个段进行遍历过程中某个段是否发生改变。threashold用来表示需要进行rehash的界限值。 table数组存储段中节点，每个数组是个hash链，用HashEntry表示。table也是volatile，这使得能够读取到最新的table值而不需要同步。loadFactor表示负载因子。

##### **HashEntry**

Segment中的元素是以HashEntry的形式存放在链表数组中的，看一下HashEntry的结构：

```java
static final class HashEntry<K,V> { 
    final K key; 
    final int hash; 
    volatile V value; 
    final HashEntry<K,V> next; 
}

```

可以看到`HashEntry`的一个特点，除了value外，其他的几个变量都是final修饰的，这意味着不能从hash链表的尾部和中部进行添加或删除节点，因为这需要修改next引用值，所有的节点的修改只能从头部开始。对于put操作，可以一律添加到hash链表的头部。但是对于remove操作，可能需要从中间删除一个节点，这时候就需要把将要删除的节点的前面的节点都复制一遍，最后一个节点指向要删除的节点的下一个节点。为了确保读操作能看到最新的值，将value设置为`volatile`，这避免了加锁。

#### ConcurrentHashMap JDK1.8与JDK1.7的区别

JDK1.8 中 `ConcurrentHashMap` 类取消了 `Segment `分段锁，采用 `CAS `+ `synchronized `来保证并发安全，数据结构跟 jdk1.8 中 `HashMap `结构类似，都是**数组 + 链表(当链表长度大于8时，链表结构转为红黑二叉树**)结构。`ConcurrentHashMap` 中 `synchronized` 只锁定当前链表或红黑二叉树的首节点，只要节点 hash 不冲突，就不会产生并发，相比 JDK1.7 的 `ConcurrentHashMap` 效率又提升了 N 倍！

#### ConcurrentHashMap的初始化

我们结合一下源码来分析一下ConcurrentHashMap的实现，先看初始化方法。

```java
public ConcurrentHashMap(int initialCapacity, 
                         float loadFactor, int concurrencyLevel) { 
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0) 
        throw new IllegalArgumentException(); 
    if (concurrencyLevel > MAX_SEGMENTS) 
        concurrencyLevel = MAX_SEGMENTS; 
    // Find power-of-two sizes best matching arguments 
    int sshift = 0; 
    int ssize = 1; 
    while (ssize < concurrencyLevel) { 
        ++sshift; 
        ssize <<= 1; 
    } 
    segmentShift = 32 - sshift; 
    segmentMask = ssize - 1; 
    this.segments = Segment.newArray(ssize); 
    if (initialCapacity > MAXIMUM_CAPACITY) 
        initialCapacity = MAXIMUM_CAPACITY; 
    int c = initialCapacity / ssize; 
    if (c * ssize < initialCapacity) 
        ++c; 
    int cap = 1; 
    while (cap < c) 
        cap <<= 1; 
    for (int i = 0; i < this.segments.length; ++i) 
        this.segments[i] = new Segment<K,V>(cap, loadFactor); 
}

```



`CurrentHashMap`的初始化一共有三个参数，一个`initialCapacity`，表示初始的容量，一个`loadFactor`，表示负载参数，最后一个是`concurrentLevel`，代表`ConcurrentHashMap`内部`Segment`的数量，`ConcurrentLevel`一经指定，不可该变，后续如果`ConcurrentHashMap`的元素数量增加导致`ConcurrentHashMap`需要扩容，`ConcurrentHashMap`不会增加`Segment`的数量，只会增加`Segment`中链表数组的容量，这样扩容的好处是不需要对`ConcurrentHashMap`中所有的元素做rehash，只需要对`Segment`中的元素做一次rehash就可以了。

默认初始表的容量，必须是2的幂（至少为1），最大为16.



#### ConcurrentHashMap的get操作

在java8中，`concurrentHashMap`的get操作是不需要加锁的。

`concurrentHashMap`的设计目标是支持高并发操作，所以在实现中，尽可能地减少锁的使用。在java8中，为了提高并发性能，`concurrentHashMap`使用node数组和链表/红黑树 ，而且get操作完全无锁。

get方法主要是通过`volatile`关键字和`Unsafe`类来保证的，在`ConcurrentHashMap`中，Node数组和Node节点的value字段都是用`volatile`关键字修饰的，这可以保证线程间的可见性， 也就是说当一个线程修改了数据，其他线程可以立即看到修改后的值，另外，Unsafe类提供了一种低级别的，直接操作内存的方式，可以用来实现无锁的并发操作。



##### volatile

`volatile`是Java中的一个关键字，它用于修饰变量，当一个变量被`volatile`修饰后，它保证了以下两个主要特征：

- 可见性：当一个线程修改了一个`volatile`变量的值，其他线程可以立即看到这个修改，这是因为volatile变量的读写都会直接操作主内存，而不是CPU缓存，从而保证了在多线程环境下的数据一致性
- 有序性：`volatile`关键字禁止指令重排序优化。也就是说，对`volatile`变量的读写操作，都会按照代码执行顺序进行。

##### unsafe

在Java中，`Unsafe`是一个类，全名为`sun.misc.Unsafe`。这个类提供了一些可以直接操作内存和线程的底层方法，因此使用它可以进行一些非常底层的操作，比如直接访问和修改对象的字段，甚至是没有被`public`修饰的字段。

以下是一些`Unsafe`类提供的方法的例子：

- `allocateMemory`、`freeMemory`和`reallocateMemory`：可以直接分配、释放和重新分配内存，就像C语言的`malloc`、`free`和`realloc`函数一样。
- `putInt`、`getInt`等：可以直接读写内存。
- `compareAndSwapInt`、`compareAndSwapLong`等：可以进行原子操作。
- `park`和`unpark`：可以挂起和恢复线程。

虽然`Unsafe`类提供了强大的功能，但是它的使用是有风险的，因为不正确的使用可能会导致程序崩溃，或者产生难以调试的问题。因此，`Unsafe`类并不是公开的API，它主要是为Java的核心类库提供服务的。在应用程序中，我们应该尽量避免使用`Unsafe`类，而是使用更高级的、更安全的API。