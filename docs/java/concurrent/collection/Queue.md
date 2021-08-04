# Queue

队列，数组结构，具备先进后出的特性（FIFO）；

```java
package java.util;

public interface Queue<E> extends Collection<E> {
    /**
     * Inserts the specified element into this queue if it is possible to do so
     * immediately without violating capacity restrictions, returning
     * {@code true} upon success and throwing an {@code IllegalStateException}
     * if no space is currently available.
     *
     * @param e the element to add
     * @return {@code true} (as specified by {@link Collection#add})
     * @throws IllegalStateException if the element cannot be added at this
     *         time due to capacity restrictions
     * @throws ClassCastException if the class of the specified element
     *         prevents it from being added to this queue
     * @throws NullPointerException if the specified element is null and
     *         this queue does not permit null elements
     * @throws IllegalArgumentException if some property of this element
     *         prevents it from being added to this queue
     */
    // 容量允许的情况下，将指定对象插入到队列，并返回true，否则抛出异常
    boolean add(E e);

    /**
     * Inserts the specified element into this queue if it is possible to do
     * so immediately without violating capacity restrictions.
     * When using a capacity-restricted queue, this method is generally
     * preferable to {@link #add}, which can fail to insert an element only
     * by throwing an exception.
     *
     * @param e the element to add
     * @return {@code true} if the element was added to this queue, else
     *         {@code false}
     * @throws ClassCastException if the class of the specified element
     *         prevents it from being added to this queue
     * @throws NullPointerException if the specified element is null and
     *         this queue does not permit null elements
     * @throws IllegalArgumentException if some property of this element
     *         prevents it from being added to this queue
     */
    // 插入成功返回 true, 插入失败返回false
    boolean offer(E e);

    /**
     * Retrieves and removes the head of this queue.  This method differs
     * from {@link #poll poll} only in that it throws an exception if this
     * queue is empty.
     *
     * @return the head of this queue
     * @throws NoSuchElementException if this queue is empty
     */
    // 移除并返回队首对象，如果队列为空，则抛出异常
    E remove();

    /**
     * Retrieves and removes the head of this queue,
     * or returns {@code null} if this queue is empty.
     *
     * @return the head of this queue, or {@code null} if this queue is empty
     */
    // 移除并返回队首对象，如果队列为空则返回null
    E poll();

    /**
     * Retrieves, but does not remove, the head of this queue.  This method
     * differs from {@link #peek peek} only in that it throws an exception
     * if this queue is empty.
     *
     * @return the head of this queue
     * @throws NoSuchElementException if this queue is empty
     */
    // 返回队列头部（不移除），如果队列为空，抛出异常
    E element();

    /**
     * Retrieves, but does not remove, the head of this queue,
     * or returns {@code null} if this queue is empty.
     *
     * @return the head of this queue, or {@code null} if this queue is empty
     */
    // 返回队首对象（不移除），队列为空返回null
    E peek();
}

```



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

## PriorityQueue

优先队列；

非线程安全；

基于二叉堆实现（数组），支持自动扩容；

二叉堆原理参考下文 `PriorityBlockingQueue`

## DelayQueue

延迟队列；

线程安全，通过 `ReentrantLock` 实现，读写共用锁；

通过 `ObjectCondition（available）` 实现出队的等待与唤醒（只有出队操作会产生等待）；

数据存储基于 `PriorityQueue` ；出队操作支持取出延迟最大或者最小的对象；

存储对象需要实现 `Delayed` 接口；

**使用场景：**

　1. 订单业务：下单之后如果三十分钟之内没有付款就自动取消订单。 
　2. 订餐通知:下单成功后60s之后给用户发送短信通知。
 　3. 关闭空闲连接：服务器中，有很多客户端的连接，空闲一段时间之后需要关闭之。

　4.缓存：缓存中的对象，超过了空闲时间，需要从缓存中移出。

　5.任务超时处理：在网络协议滑动窗口请求应答式交互时，处理超时未响应的请求等。

## ConcurrentLinkedQueue

无界队列；

数据结构单向链表；

线程安全（CAS实现）；

## PriorityBlockingQueue

优先阻塞队列；

基于二叉堆实现（数组结构），支持自动扩容，默认容量 11；

线程安全，通过 ReentrantLock 保证线程安全，读写操作共用一个锁对象；

通过 ObjectCondition（notEmpt ） 实现出队的等待与唤醒（只有出队操作会产生等待）；

通过 **Comparator接口** 或者 **实现Comparable接口compare方法** 实现数组的**二叉堆排序**；

### 二叉堆

二叉堆本质上是一颗完全二叉树，可分为最大堆和最小堆；

最大堆，父节点都大于等于子节点，即最大值位于堆顶；最小堆，父节点都小于等于直接点，即最小值位于堆顶；

堆顶，二叉树的根节点；

二叉堆一般用数组表示，如下图，实际数据结构并不存在父子节点间的引用，而是采用如下计算公式：

> **左子节点：** 2 * n + 1  或者   n << 1 + 1
>
> **右子节点：**2 * n  + 2  或者  n << 1 + 2
>
> **父节点：** ( n - 1 ) / 2  或者  ( n - 1 )  >>> 1
>
> n 表示数组下标，从0开始；

![](https://static.laop.cc/images/20210624171645.png)

**以下说明均以最大堆为例**

#### 插入节点

首先按顺序插入节点，然后对比父节点；如果大于父节点，则与之交换位置；之后继续与父节点对比，直到根节点；如果小于父节点，则完成插入；这个过程称为**上浮**；

插入过程可以将最大值上浮到堆顶（根节点）；

![](https://static.laop.cc/images/20210622143844.png)

#### 删除节点

删除节点，则与插入过程相反，先移出根节点，之后先取堆的最末尾节点替代根节点；然后与其左右子节点对比，如果小于子节点，则与子节点对调（优先对调较大子节点）；之后继续与子节点对比，并对调位置；

![](https://static.laop.cc/images/20210622152451.png)

## LinkedTransferQueue

无界队列；

采用 CAS 实现线程安全；

队列实现了`TransferQueue` 接口，而 `TransferQueue` 又继承了 `BlockingQueue` ; 相对于其他阻塞队列，`LinkedTransferQueue`多了 `tryTransfer` 、`transfer` 等方法；

LinkedTransferQueue采用一种预占模式。意思就是消费者线程取元素时，如果队列不为空，则直接取走数据，若队列为空，那就生成一个节点（节点元素为null）入队，然后消费者线程被等待在这个节点上，后面生产者线程入队时发现有一个元素为null的节点，生产者线程就不入队了，直接就将元素填充到该节点，并唤醒该节点等待的线程，被唤醒的消费者线程取走元素，从调用的方法返回。

`LinkedTransferQueue` 是 ConcurrentLinkedQueue、SynchronousQueue（公平模式）、LinkedBlockingQueue（阻塞Queue的基本方法）的超集。而且LinkedTransferQueue更好用，因为它不仅仅综合了这几个类的功能，同时也提供了更高效的实现。



## SynchronousQueue

SynchronousQueue  内部没有对象容器，一个生产线程，当它生产对象加入对象，如果当前没有线程消费对象，此生产线程则会阻塞，直到有消费线程取出对象；反之，当消费线程需要从队列取出对象，但是没有生成线程向队列加入对象，消费线程会发生阻塞，直到有对象加入队列；

当有多个生产线程执行入队操作，在没有消费线程进行出队时，生成线程都会发生阻塞；当有消费线程执行出队时，被唤醒的生产线程并不是随机的选择的，而是采用 FILO 或者 FiFO 的规则，分别为非公平模式、公平模式；

```java
    /**
     * Creates a {@code SynchronousQueue} with nonfair access policy.
     */
    public SynchronousQueue() {
        this(false);
    }

    /**
     * Creates a {@code SynchronousQueue} with the specified fairness policy.
     *
     * @param fair if true, waiting threads contend in FIFO order for
     *        access; otherwise the order is unspecified.
     */
    public SynchronousQueue(boolean fair) {
        transferer = fair ? new TransferQueue<E>() : new TransferStack<E>();
    }
```

