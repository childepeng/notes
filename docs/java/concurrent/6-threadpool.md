# Java线程池

线程的生命周期中包括多种的状态的切换，new/runnable/running/blocked/dead；系统创建线程需要为线程分配运行资源，比如内存；线程销毁需要收回线程占用的资源；当出现频繁的创建销毁线程的操作时，会占用大量的系统资源，造成系统资源浪费。
线程池可以控制线程的创建和回收、限制同时执行的线程数量，总结一下线程池有如下作用：

1. 降低资源消耗：重用已经创建的线程，降低线程创建和销毁的系统消耗
2. 提高响应速度：任务无需等待线程创建和销毁
3. 管理线程：对线程进行统一管理、分配、调优。

## 线程池的实现：ExecutorService
Java中线程池的接口是ExecutorService，比较重要的几个实现类如下：
1. ScheduledExecutorService：能和Timer/TimerTask类似，解决那些需要任务重复执行的问题
2. ThreadPoolExecutor：ExecutorService的默认实现
3. ScheduledThreadPoolExecutor：继承ThreadPoolExecutor的ScheduledExecutorService接口实现，周期性任务调度的类实现

## ThreadPoolExecutor
这里我们先只看ThreadPoolExecutor，要配置一个线程池，需要理解几个核心参数：
1. corePoolSize
 - 线程池中允许长期存活的最大线程数；
 - 当线程数小于核心线程数时，池中即时用空闲线程，也会创新新的线程
 - 当线程数大于核心线程数、且任务队列未满时，新添加的任务会加入到任务队列，等待空闲线程
2. maximumPoolSize
 - 线程池中允许的最大线程数，也是线程池最大执行线程数
 - 当线程数大于核心线程数，且任务队列已满时，新添加的任务会直接创建新的线程执行，直到线程数量超过最大线程数。
 - 当线程数等于最大线程数，且任务队列已满，线程池将拒绝新添加的任务。
3. keepAliveTime：非核心线程空闲时间，超过这个时间后，线程会被回收释放
4. TimeUnit：空间时间的单位
5. workQueue：线程池中的任务队列，常用队列：SynchronousQueue,LinkedBlockingDeque,ArrayBlockingQueue
6. threadFactory：接口，线程工厂，提供线程创建的功能
7. RejectedExecutionHandler：接口，新添加的任务被拒绝之后的处理方式，会调用rejectedExecution方法；

### workQueue(BlockingQueue) 选择说明

workQueue 类型是 BlockingQueue ，常用可选队列有： SynchronousQueue, LinkedBlockingDeque, ArrayBlockingQueue

1.  `SynchronousQueue`

   不缓存任何元素的阻塞队列，每次插入操作都会阻塞等待另一个线程执行移除操作；移除操作同样会阻塞等待其他线程进行插入数据；类似生产者和消费者是直接进行交易，缺少任何一方，另一方都需要进行等待；静态工厂方法`Executors.newCachedThreadPool` 创建线程池就使用的这种队列；

2. `LinkedBlockingDeque`

   基于链表结构的有界阻塞队列，创建可以指定长度，不指定长度的时候默认长度为 `Integer.MAX_VALUE` (2^31 - 1)

3. `ArrayBlockingQueue`

   基于数组结构的有界阻塞队列，创建时需要指定队列长度；

## Executors
通常情况下，不需要使用ThreadPoolExecutor来创建线程池，Java中提供了工具类可通过简单的参数创建不同特性的线程池：Executors，比如：
1. Executors.newFixedThreadPool(int nthreads): 创建固定大小的线程池，核心线程数与最大线程是相同，采用无界队列；超过核心线程的空闲线程立即回收
2. Executors.newCachedThreadPool()： 不设置核心线程数，采用无界队列，线程执行完成一分钟后回收