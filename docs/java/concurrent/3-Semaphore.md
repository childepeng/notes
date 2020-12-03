# Semaphore信号量
在操作系统中，信号量是个很重要的概念，操作系统通过信号量控制可同时访问某些共享资源的程序数量。在Java中同样有信号量控制机制，就是Semaphore；Semaphore可控制同时使用某些资源的线程数量。比如，通过互斥锁（synchronized），可以保证某段代码同时仅允许一条线程执行，Semaphore可允许指定数量的线程访问资源。
```
// 初始化，并设置信号量
Semaphore sph = new Semaphore(5);

for (int i = 0; i < 7; i++) {
    new Thread(() -> {
        try {
            sph.acquire(); //获取一个信号位
            System.out.println(Thread.currentThread() + " --- " + new Date());
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            sph.release(); //释放一个信号位
        }
    }).start();
}
```
```
Thread[Thread-0,5,main] --- Tue Jan 09 19:36:07 CST 2018
Thread[Thread-1,5,main] --- Tue Jan 09 19:36:07 CST 2018
Thread[Thread-3,5,main] --- Tue Jan 09 19:36:07 CST 2018
Thread[Thread-2,5,main] --- Tue Jan 09 19:36:07 CST 2018
Thread[Thread-4,5,main] --- Tue Jan 09 19:36:07 CST 2018
Thread[Thread-6,5,main] --- Tue Jan 09 19:36:17 CST 2018
Thread[Thread-5,5,main] --- Tue Jan 09 19:36:18 CST 2018
```
代码执行分析，前五条线程基本同时执行，最后两条线程输出比前五条线程晚10秒，是因为设置的信号量为5，当没有信号位的时候，线程执行acquire会进入等待。

## 主要方法说明
Semaphore(int permits): 构造，初始化并设置允许的信号量
Semaphore(int permits, boolean fair): 构造，初始化并设置允许的信号量和信号竞争方式（公平和非公平）
acquire()： 申请获取信号位
tryAcquire(): 尝试申请获取信号位，并立即返回，能获取到返回true，否则返回false
tryAcquire(long timeout, TimeUnit unit): 尝试申请获取信号位，并在指定时间内返回，能获取到返回true，否则返回false
acquireUninterruptibly(): 申请获取信号位，并且等待不可中断
release(): 释放

