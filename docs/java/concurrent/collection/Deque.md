# Deque

栈，支持 FILO（先进后出） 特性

```java
package java.util;

public interface Deque<E> extends Queue<E> {
    
    /**
    * 容量允许的情况下，将指定对象插入Deque前端，否则抛出IllegalStateException
    */
    void addFirst(E e);

    /**
     * 容量允许的情况下，将指定对象插入Deque末尾，否则抛出异常
     */
    void addLast(E e);

    /**
     * 在容量允许的情况下，将指定对象插入Deque前端，并返回Ture，否则返回False
     */
    boolean offerFirst(E e);

    /**
     * 在容量允许的情况下，将指定对象插入Deque末尾，并返回Ture，否则返回False
     */
    boolean offerLast(E e);

    /**
     * 移除并返回Deque前端对象，如果不存在则抛出异常；
     */
    E removeFirst();

    /**
     * 移除并返回末尾对象，如果不存在则抛出异常
     */
    E removeLast();

    /**
     * 移除并返回前端对象，不存在则返回null
     */
    E pollFirst();

    /**
     * 移除并返回末尾对象，不存在则返回null
     */
    E pollLast();

    /**
     * 返回Deque前端对象，如果不存在则抛出异常
     */
    E getFirst();

    /**
     * 返回Deque末尾对象，如果不存在则抛出异常
     */
    E getLast();

    /**
     * 返回前端对象，不存在则返回null
     */
    E peekFirst();

    /**
     * 返回末尾对象，不存在则返回null
     */
    E peekLast();

    // *** Queue methods ***
    // 略
}

```



## ArrayDeque

内部数据结构为数组，同时提供两个指针指向栈的首尾；

支持 1 倍扩容

```java

    transient Object[] elements; 

    transient int head;

    transient int tail;

```



## LinkedList

双端队列，继承List、Deque接口；

## ConcurrentLinkedDeque

双端队列，通过CAS实现线程安全；

## LinkedBlockingDeque

双端队列，通过 ReentrantLock 实现线程安全，通过 Condition 实现阻塞与唤醒；

```java
    /** Main lock guarding all access */
    final ReentrantLock lock = new ReentrantLock();

    /** Condition for waiting takes */
    private final Condition notEmpty = lock.newCondition();

    /** Condition for waiting puts */
    private final Condition notFull = lock.newCondition();

```

