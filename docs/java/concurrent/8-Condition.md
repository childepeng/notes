# Condition

Condition 用于代替Object的`wait\notify\notifyAll`，通过`await\signal\signalAll`实现线程间的协作；在JUC包中被广泛使用；

Condtion依赖于Lock接口，通过`Lock.newCondition()`方法创建实例，调用condition的方法必须要在lock锁定范围内；而Object的`wait\notify\notifyAll` 则需要使用 `syncronized` 获取对象锁；

await ： 当前线程调用 await 方法后，会释放锁，并进入等待状态，等待signal唤醒；

signal/signalAll: 当前线程调用signal/signalAll后，会唤醒在当前condition对象上执行await方法的线程，这些线程将重新获取锁。

## Condition 应用

### LinkedBlockingQueue

```java
...
    /** Main lock guarding all access */
    final ReentrantLock lock = new ReentrantLock();

    /** Condition for waiting takes */
    private final Condition notEmpty = lock.newCondition();

    /** Condition for waiting puts */
    private final Condition notFull = lock.newCondition();
...
```

`LinkedBlockingQueue`队列采用`ReentrantLock`对部分方法进行加锁，保证并发安全；同时采用两个`condition`实例，来控制 `take/poll/offer` 等方法的阻塞与唤醒行为；

比如，take方法，当队列为空时，会进入等待。

```java
	public E take() throws InterruptedException {
        return takeFirst();
    }

	public E takeFirst() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            E x;
            // 当x为空，此时队列为空时，通过 notEmpty.await 等待
            while ( (x = unlinkFirst()) == null)
                notEmpty.await();
            return x;
        } finally {
            lock.unlock();
        }
    }

	/**
     * Removes and returns first element, or null if empty.
     */
    private E unlinkFirst() {
        // assert lock.isHeldByCurrentThread();
        Node<E> f = first;
        if (f == null)
            return null;
        Node<E> n = f.next;
        E item = f.item;
        f.item = null;
        f.next = f; // help GC
        first = n;
        if (n == null)
            last = null;
        else
            n.prev = null;
        --count;
        // 当取出成功时，此时队列不满，通过 notFull 唤醒在这个对象上等待的线程；
        notFull.signal();
        return item;
    }

	
	/**
     * @throws NullPointerException {@inheritDoc}
     * @throws InterruptedException {@inheritDoc}
     */
    public boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException {
        return offerLast(e, timeout, unit);
    }

	/**
     * @throws NullPointerException {@inheritDoc}
     * @throws InterruptedException {@inheritDoc}
     */
    public boolean offerLast(E e, long timeout, TimeUnit unit)
        throws InterruptedException {
        if (e == null) throw new NullPointerException();
        Node<E> node = new Node<E>(e);
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (!linkLast(node)) {
                if (nanos <= 0)
                    return false;
                // 当队列已满，通过 notFull 等待指定时间；
                // while循环是因为，线程有可能被提前唤醒，但是在多线程场景下仍然无法并执行插入时，继续等待直到超时。
                nanos = notFull.awaitNanos(nanos);
            }
            return true;
        } finally {
            lock.unlock();
        }
    }
```

