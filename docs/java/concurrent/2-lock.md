# Java锁机制
Java中的锁分为互斥锁和读写锁，互斥锁有分为内置锁和显示锁，同时通过锁的竞争形式分为公平锁和非公平锁；锁是每个Java对象都有的特性，在多线程的环境中通过对对象锁的操作实现线程的同步和互斥。

## synchronized
synchronized实现的是内置锁，它是Java的一个关键字，获取锁和释放锁的操作对开发者是不可见的；synchronized代码块执行结束或者出现异常，锁会自动释放。
synchronized常有两种用法：
```
synchronized (object) {
    ...
}
```
```
public synchronized void method() {
    ...
}
```
修饰对象的时候表示线程将请求获取所修饰对象的锁，修饰方法时表示获取当前对象（this）的锁；当一个线程获取该对象的锁之后，其他线程执行到修饰该对象的代码块或者该对象的同步方法时，线程将进入等待，直到获取到该对象的锁。。
线程在等待对象锁的过程是不可中断的。

## Lock & ReentrantLock
Lock是显示锁，通过调用方法lock()和unlock()来获取锁和释放锁，通过lock()获取的锁只能通过unlock()释放，否则可能发生死锁。
Lock是java.util.concurrent.locks下的接口，该接口主要有以下几个方法：
lock()： 获取当前lock对象的锁；如果锁已被其他线程获取，则进入等待，并且不可中断；
tryLock()：尝试获取当前lock对象的锁，并立即返回，如果获取成功返回true，否则返回false；
tryLock(long time, TimeUnit unit)：方法同tryLock()，只是当lock对象锁被其他线程获取后，会进入等待，并在指定时间内返回锁的获取结果；
lockInterruptibly()：方法同lock()，只是在线程等待过程可被中断
unlock()：释放当前lock对象锁
为了防止死锁，lock通常在try-catch-finally中使用：
```
try {
    lock.lock();
    ...
} catch (Exception e) {
} finally {
    lock.unlock();
}
```
ReentrantLock是其实现类

## ReadWriteLock & ReentrantReadWriteLock
ReadWriteLock是java.util.concurrent.locks包下的接口，实现读写锁；
readLock(): 获取读锁，读锁允许多线程同时进入执行，如果当前写入锁被其他线程获取，获取读锁的线程将进入等待；
writeLock(): 获取写锁，不允许多线程同时进入执行，如果当前读锁或者写锁被其他线程获取，获取写锁将进入等待；
通过这两个方法，防止在多线程的场景下同时执行对文件的读写操作。

## 公平锁 & 非公平锁
公平锁是尽量以请求锁的顺序获取锁，也就是说等待时间最久的将优先获取锁。
非公平锁是无法保证锁的获取按照顺序进行。
synchronized时非公平锁；ReentrantLock和ReentrantReadWriteLock默认为非公平锁，当可设置为公平锁。
比如：ReentrantLock
```
    ...
    /**
     * Sync object for non-fair locks
     */
    static final class NonfairSync extends Sync { ... }

    /**
     * Sync object for fair locks
     */
    static final class FairSync extends Sync { ... }

    /**
     * Creates an instance of {@code ReentrantLock}.
     * This is equivalent to using {@code ReentrantLock(false)}.
     */
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    /**
     * Creates an instance of {@code ReentrantLock} with the
     * given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```
FaiSync: 公平锁
NonfailSync： 非公平锁