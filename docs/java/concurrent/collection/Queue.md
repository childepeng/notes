# Queue

队列，数组结构，具备先进后出的特性（FIFO）；

## BlockingQueue

阻塞队列接口，

```java
/**
* 如果可以在不违反容量限制的情况下立即将指定的元素插入到该队列中，成功时返回true，
* 如果当前没有可用空间则抛出 IllegalStateException
*/
boolean add(E e);

/**
* 如果可以在不违反容量限制的情况下立即将指定的元素插入到该队列中，成功时返回true，如果当前没有可用空间则返回false。
*/
boolean offer(E e);

/**
* 将指定的元素插入到此队列中，并在必要时等待空间可用。
* 空间不足，线程进入等待，直到被唤醒（有足够空间）；
*/
void put(E e) throws InterruptedException;

/**
* 将指定的元素插入到该队列中，如果空间不足，线程进入等待，直到等待超时或者被唤醒
*/
boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException;

/**
* 检索并删除此队列的头部，如果队列为空，则进入等待，直到被唤醒（有数据插入时）
*/
E take() throws InterruptedException;

/**
* 检索并删除此队列的头部，如果队列为空，则进入等待，直到等待超时或者被唤醒
*/
E poll(long timeout, TimeUnit unit)
        throws InterruptedException;

/**
* 返回队列剩余容量
*/
int remainingCapacity();

/**
* 移除队列指定数据
*/
boolean remove(Object o);
```



## ArrayBlockingQueue

阻塞队列；

线程安全；

数组结构，容量固定（初始化需要指定）；

队列通过 ReentrantLock 保证线程安全，读写操作共用一个锁对象；

通过 ObjectCondition（notEmpt / notFull） 实现入队出队的等待与唤醒；

## LinkedBlockingQueue

阻塞队列；

线程安全；

链表结构，长度可指定，默认 Integer.MAX_VALUE；

队列通过 ReentrantLock 保证线程安全，读写采用不同的锁对象（takeLock / putLock），支持并发读写；

通过 ObjectCondition（notEmpt / notFull） 实现入队出队的等待与唤醒；

## LinkedTransferQueue



## SynchronousQueue

## PriorityQueue

## DelayQueue

## ConcurrentLinkedQueue

## PriorityBlockingQueue

阻塞队列；

数组结构，支持自动扩容，默认容量 11；

队列通过 ReentrantLock 保证线程安全，读写操作共用一个锁对象；

通过 ObjectCondition（notEmpt ） 实现出队的等待与唤醒（只有出队操作会产生等待）；

支持通过 Comparator 比较元素的优先级，实现数组排序；（可实现优先出队）

