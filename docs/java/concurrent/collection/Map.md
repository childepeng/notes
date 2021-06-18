# Map

## HashMap

非线程安全；

### 扩容

- capacity 即容量，数组长度，默认16。
- loadFactor 加载因子，默认是0.75
- threshold 阈值。当元素数量超过阈值时便会触发扩容。

#### java8

1. 空参数的构造函数：实例化的HashMap默认内部数组是null，即没有实例化。第一次调用put方法时，则会开始第一次初始化扩容，默认**capacity** 长度为16，默认**threshold = capacity * loadFactor = 12**  。

```java
	/**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
```

2. 有参构造函数：可以指定容量、加载因子。构造中首先会根据找到**不小于指定容量的2的幂数**，赋值给 **threshold**，比如：指定容量为12，阈值为2^4 = 16；指定容量为24，阈值为2^5=32。当第一次调用put方法时，会初始化数组，**capacity= threshold**  将阈值赋值给容量，然后再次修改阈值**threshold = newcapacity * loadfactor** 。

```java
	public HashMap(int initialCapacity) {        this(initialCapacity, DEFAULT_LOAD_FACTOR);    }

	public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }

	/**
     * Returns a power of two size for the given target capacity.
     * 返回不小于指定值的2的幂数
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

3. 如果不是第一次扩容，则容量变为原来的2倍，阈值也变为原来的2倍。*（容量和阈值都变为原来的2倍时，负载因子还是不变）*

```java
/**
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     */
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;								// 1
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)					// 2
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;												  // 3					
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);		// 4
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;								 	// 5
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;														// 6
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            ...略
        }
        return newTab;
    }
```

​	HashMap数据初始化和扩容都是通过 **resize()**  完成，上述步骤中，

> 1.  当 capacity > MAXIMUM_CAPACITY 时，不再扩容；如上标注 **1**
> 2.  非首次执行resize，直接进行2倍扩容；如上标注 **2**
> 3.  指定容量时，即通过有参构造创建hashmap时，threshold在构造中赋值，在resize时，newcap = threshold  
>     同时 threshold = newcap * loadfactor；如上标注 **3、5、6**
> 4.  默认情况下，初次执行resize， 使用默认参数，如上标注 **4**

#### java7

1. 空参数的构造函数：以默认容量、默认负载因子、默认阈值初始化数组。内部数组是**空数组**。
2. 有参构造函数：根据参数确定容量、负载因子、阈值等。
3. 第一次put时会初始化数组，其容量变为**不小于指定容量的2的幂数**。然后根据负载因子确定阈值。
4. 如果不是第一次扩容，则进行2倍扩容，阈值为 新容量 * 加载因子。

### 数据结构

JDK8以前，HashMap数据结构为数组链表；JDK8中，初始状态为数组链表，当Node链表长度达到 **TREEIFY_THRESHOLD（8）** 且 数组长度大于 **MIN_TREEIFY_CAPACITY（64）**  时，链表将转换为红黑树；

![](https://static.laop.cc/images/20210616153746.png)

当红黑树的节点数量小于 **UNTREEIFY_THRESHOLD（6）**时，红黑树将重新转换为链表；

### 为何要转为红黑树

链表查询时间复杂度是 **O(N)**， 红黑树是一种二叉查找树，时间复杂度是 **O(logN)**，当单个链表长度过长时，可以显著提交查询效率；

然后之所以不直接使用红黑树，因为红黑树的节点占用空间是普通节点的2倍大，同时节点数量达到一定数量，树化才有意义；JDK源码中也有解释：

```
     * Because TreeNodes are about twice the size of regular nodes, we
     * use them only when bins contain enough nodes to warrant use
     * (see TREEIFY_THRESHOLD). And when they become too small (due to
     * removal or resizing) they are converted back to plain bins.  In
     * usages with well-distributed user hashCodes, tree bins are
     * rarely used.  Ideally, under random hashCodes, the frequency of
     * nodes in bins follows a Poisson distribution
     * (http://en.wikipedia.org/wiki/Poisson_distribution) with a
     * parameter of about 0.5 on average for the default resizing
     * threshold of 0.75, although with a large variance because of
     * resizing granularity. Ignoring variance, the expected
     * occurrences of list size k are (exp(-0.5) * pow(0.5, k) /
     * factorial(k)). The first values are:
     *
     * 0:    0.60653066
     * 1:    0.30326533
     * 2:    0.07581633
     * 3:    0.01263606
     * 4:    0.00157952
     * 5:    0.00015795
     * 6:    0.00001316
     * 7:    0.00000094
     * 8:    0.00000006
     * more: less than 1 in ten million
```

### 为何链表转红黑树的阈值时 8，红黑树转链表的阈值是 6

同样，上述源码注释也有说明，在hashCode均匀分布的情况下，当阈值为8时，Hash的命中率为 0.00000006，所以树化基本不会发生；树的退化阈值为 6 ，可以避免链表和红黑树的频繁转换。

### 为何扩容都是2倍扩容

正常情况下，计算节点在table中的下标的方法是：**hash&(oldTable.length-1)**，扩容之后，table长度翻倍，计算table下标的方法是**hash&(newTable.length-1)**，也就是**hash&(oldTable.length*2-1)**，于是我们有了这样的结论：这新旧两次计算下标的结果，要不然就相同，要不然就是新下标等于旧下标加上旧数组的长度（**i + oldTable.length**）。

## LinkedHashMap

LinkedHashMap支持按插入顺序遍历；非线程安全；

HashMap的子类，同时扩展了 HashMap的内部类Node，增加 **before/after** 两个指针，分别指向前后两个节点，用于链接先后插入的节点；

链接动作是通过HashMap的预留方法完成： **afterNodeAccess**， 该方法在 HashMap 中执行插入的时候调用；以下是LinkedHashMap的实现：

```java
    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }
```

## TreeMap

红黑树，可实现二分查找树；非线程安全；

### 数据结构

```java
	static final class Entry<K,V> implements Map.Entry<K,V> {
        K key;
        V value;
        Entry<K,V> left;
        Entry<K,V> right;
        Entry<K,V> parent;
        boolean color = BLACK;
     ...
```

### 红黑树

略

## HashTable

HashMap的线程安全实现，主要方法均使用了 **Synchronized** ，并发性能低

在多线程环境下，推荐使用 **ConcurentHashMap**

## ConcurrentHashMap

功能实现与HashMap基本一致（数据结构、扩容等），只是增加了线程安全的实现。

线程安全是通过 **CAS** 和 **Synchronized** 结合实现的；

比如，put方法的实现：

```java
	/** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))			// 1
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {															// 2
                    if (tabAt(tab, i) == f) {
                        ... 略
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```

当桶为空时，通过 CAS 进行无锁插入；否则通过 Synchronized 对桶加锁，执行插入；如上步骤 **1, 2**

JDK7中采用分段锁实现线程安全，将数据分成16个Segment ，每个Segment 是 ReentrantLock 的子类，每次执行插入的时候先找到对应的segment，然后进行加锁，之后再进行插入；

JDK8 相较于JDK7 ：

1. 无序每次进行加锁，桶初始化的时候通过CAS进行插入；
2. 锁采用Synchronized实现；

## IdentityHashMap

1. 数据结构：IdentifyHashMap数据结构为一维数组，Key/Value相邻存储数组中；

2. 与HashMap的hash算法不同，HashMap允许Key自定义HashCode；IdentifyHashMap只会使用 **System.identityHashCode(Object x)** 计算hashcode，该方法只会返回对象默认的hashcode；

3. Key的比较方式不同

   HashMap:  

   ```java
   e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))
   ```

   IdentifyHashMap: （只认同一个对象）

   ```java
   item == k
   ```

## ConcurrentSkipListMap

基于跳表实现；支持高并发、可排序；

跳表是一种采用了用空间换时间思想的数据结构。它是一种多层链表，它会随机地将一些节点提升到更高的层次，以创建一种逐层的数据结构，以提高操作的速度。

![](https://static.laop.cc/images/20210617175117.png)

跳表特性：

1. 每一层的数据都是有序的，默认升序，也可以自定义排序；
2. 最底层链表包含所有数据，上层数据通过一定概率产生；

2. 如果一个元素出现在 level n，则在n的下层也会存在；
3. 每个节点有两个指针，分别指向下一层的相同节点和同一层下一个节点（右节点）；



## WeakHashMap

Entry 实现了 WeakReference，也就是说，Entry在没使用的情况下，会被虚拟机回收；

## EnumMap

Key 为 Enum 类型；内部数组用于存储Value；通过枚举类型的下标定位Value；

## Properties

k-v 配置池，继承自HashTable；

