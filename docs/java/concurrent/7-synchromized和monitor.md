# Java Monitor机制

Synchronized 是Java的关键字，用于实现线程同步，可以强制线程在访问执行 synchronized 代码块或者方法的时候串行执行；它主要有两种使用方案，修饰方法和代码块；而它的背后是 Jvm 内部实现的 Monitor 机制。

[官方描述](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.10)

> The Java Virtual Machine supports synchronization of both methods and sequences of instructions within a method by a single synchronization construct: the *monitor*.
>
> Method-level synchronization is performed implicitly, as part of method invocation and return ([§2.11.8](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.11.8)). A `synchronized` method is distinguished in the run-time constant pool's `method_info` structure ([§4.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.6)) by the `ACC_SYNCHRONIZED` flag, which is checked by the method invocation instructions. When invoking a method for which `ACC_SYNCHRONIZED` is set, the executing thread enters a monitor, invokes the method itself, and exits the monitor whether the method invocation completes normally or abruptly. During the time the executing thread owns the monitor, no other thread may enter it. If an exception is thrown during invocation of the `synchronized` method and the `synchronized` method does not handle the exception, the monitor for the method is automatically exited before the exception is rethrown out of the `synchronized` method.
>
> Synchronization of sequences of instructions is typically used to encode the `synchronized` block of the Java programming language. The Java Virtual Machine supplies the *monitorenter* and *monitorexit* instructions to support such language constructs. Proper implementation of `synchronized` blocks requires cooperation from a compiler targeting the Java Virtual Machine ([§3.14](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-3.html#jvms-3.14)).
>
> *Structured locking* is the situation when, during a method invocation, every exit on a given monitor matches a preceding entry on that monitor. Since there is no assurance that all code submitted to the Java Virtual Machine will perform structured locking, implementations of the Java Virtual Machine are permitted but not required to enforce both of the following two rules guaranteeing structured locking. Let *T* be a thread and *M* be a monitor. Then:
>
> 1. The number of monitor entries performed by *T* on *M* during a method invocation must equal the number of monitor exits performed by *T* on *M* during the method invocation whether the method invocation completes normally or abruptly.
> 2. At no point during a method invocation may the number of monitor exits performed by *T* on *M* since the method invocation exceed the number of monitor entries performed by *T* on *M* since the method invocation.
>
> Note that the monitor entry and exit automatically performed by the Java Virtual Machine when invoking a `synchronized` method are considered to occur during the calling method's invocation.

## Synchronized 

monitor 常被翻译成监视器，这里通常被称为管程；一下通过一段简单的代码看一下`synchronized`的实现。

```java
public class LockTest {
    // 同步方法
    public synchronized void fun1(){
        System.out.println("world");
    }

    public void fun2() {
        // 同步代码块
        synchronized (this) {
            System.out.println("Hello");
        }
    }
}
```

编译之后使用 `javap -v  `   命令进行反编译，得出如下指令代码：

```java
Classfile LockTest.class
  Last modified 2020-12-15; size 571 bytes
  MD5 checksum 9415ef01ae16ca4bd15f953b3335d1c9
  Compiled from "LockTest.java"
public class LockTest
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #7.#20         // java/lang/Object."<init>":()V
   #2 = Fieldref           #21.#22        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #23            // world
   #4 = Methodref          #24.#25        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = String             #26            // Hello
   #6 = Class              #27            // LockTest
   #7 = Class              #28            // java/lang/Object
   #8 = Utf8               <init>
   #9 = Utf8               ()V
  #10 = Utf8               Code
  #11 = Utf8               LineNumberTable
  #12 = Utf8               fun1
  #13 = Utf8               fun2
  #14 = Utf8               StackMapTable
  #15 = Class              #27            // LockTest
  #16 = Class              #28            // java/lang/Object
  #17 = Class              #29            // java/lang/Throwable
  #18 = Utf8               SourceFile
  #19 = Utf8               LockTest.java
  #20 = NameAndType        #8:#9          // "<init>":()V
  #21 = Class              #30            // java/lang/System
  #22 = NameAndType        #31:#32        // out:Ljava/io/PrintStream;
  #23 = Utf8               world
  #24 = Class              #33            // java/io/PrintStream
  #25 = NameAndType        #34:#35        // println:(Ljava/lang/String;)V
  #26 = Utf8               Hello
  #27 = Utf8               LockTest
  #28 = Utf8               java/lang/Object
  #29 = Utf8               java/lang/Throwable
  #30 = Utf8               java/lang/System
  #31 = Utf8               out
  #32 = Utf8               Ljava/io/PrintStream;
  #33 = Utf8               java/io/PrintStream
  #34 = Utf8               println
  #35 = Utf8               (Ljava/lang/String;)V
{
  public LockTest();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public synchronized void fun1();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String world
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 3: 0
        line 4: 8

  public void fun2();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: dup
         2: astore_1
         3: monitorenter
         4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         7: ldc           #5                  // String Hello
         9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        12: aload_1
        13: monitorexit
        14: goto          22
        17: astore_2
        18: aload_1
        19: monitorexit
        20: aload_2
        21: athrow
        22: return
      Exception table:
         from    to  target type
             4    14    17   any
            17    20    17   any
      LineNumberTable:
        line 7: 0
        line 8: 4
        line 9: 12
        line 10: 22
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 17
          locals = [ class LockTest, class java/lang/Object ]
          stack = [ class java/lang/Throwable ]
        frame_type = 250 /* chop */
          offset_delta = 4
}
SourceFile: "LockTest.java"
```

synchronized 修饰方法时，monitor，`ACC_SYNCHRONIZED` 作为同步方法标识，线程执行到同步方法，判断 flags 是否包括 ACC_SYNCHRONIZED，如果是同步方法，执行线程将获得对象的monitor，并且在方法执行结束之后，释放对象的monitor；如果方法抛出异常，则在异常抛出前释放对象的monitor；

同步代码块中，JVM提供了 `monitorenter ` 和  `monitorexit ` 指令来实现对象monitor的获取和释放；指令中间的代码就是临界区；

`monitorenter`  [官方描述](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.monitorenter)

>The *objectref* must be of type `reference`.
>
>Each object is associated with a monitor. A monitor is locked if and only if it has an owner. The thread that executes *monitorenter* attempts to gain ownership of the monitor associated with *objectref*, as follows:
>
>- If the entry count of the monitor associated with *objectref* is zero, the thread enters the monitor and sets its entry count to one. The thread is then the owner of the monitor.
>- If the thread already owns the monitor associated with *objectref*, it reenters the monitor, incrementing its entry count.
>- If another thread already owns the monitor associated with *objectref*, the thread blocks until the monitor's entry count is zero, then tries again to gain ownership.

`monitorexit`  [官方描述](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.monitorexit)

> The *objectref* must be of type `reference`.
>
> The thread that executes *monitorexit* must be the owner of the monitor associated with the instance referenced by *objectref*.
>
> The thread decrements the entry count of the monitor associated with *objectref*. If as a result the value of the entry count is zero, the thread exits the monitor and is no longer its owner. Other threads that are blocking to enter the monitor are allowed to attempt to do so.

`monitorexit` 在上述代码中出现了两次：13和19行，因为这里有个类似`try - finaly` 的异常处理机制，保证监视器最终能够释放，防止死锁。前面如果可以正常释放监视器，后面就会通过 `goto 22`  跳到 22 行执行 `return` ; 而 `Exception table` 定义了可能出现异常的指令范围；

Java指令集参考文档：

[Orcale官方文档]: https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5
[转载]: http://www.blogjava.net/DLevin/archive/2011/09/13/358497.html



## Monitor

Java中每个对象都会关联一个monitor对象，也就是说每个对象都有锁（不包括基本数据类型）；synchronized 使用场景不同对用锁的对象也不同；

* 普通方法中，锁的是当前调用该方法的对象；
* 静态方法中，锁的是当前类的Class对象；
* 同步代码块中，锁的是括号中的对象；

在深入解释monitor之前，要先了解一下对象的数据结构；

### Java对象结构

Java对象在内存中有三个部分组成：对象头、实例数据和对齐填充字节；

#### 对象头

对象头分为三个部分：MarkWord、指向类的指针、数组长度（仅数组对象有）；32位系统中，每个部分长度位32位；64位系统为64位；

```
|-------------------------------------------------------------------------|
|                         Object Header (96 bits)                         |
|------------------------|-----------------------|------------------------|
|     Mark Word(32bits)  | class pointer(32bits) |  array length(32bits)  |
|------------------------|-----------------------|------------------------|
```

##### Mark Word

这部分存储对象自身的运行数据，比如hashcode、gc分代年龄、锁状态等；MarkWord在32位系统中，长度位32位，64位系统为64位；以32位系统为例，说明一下几种MarkWord状态，数字表示长度；

```
|-------------------------------------------------------| 
|                  Mark Word (32 bits)                  |       State        
|-------------------------------------------------------| 
| identity_hashcode:25 | age:4 | biased_lock:1 | lock:2 |       Normal（无锁）      
|-------------------------------------------------------| 
|  thread:23 | epoch:2 | age:4 | biased_lock:1 | lock:2 |       Biased（偏向锁）   
|-------------------------------------------------------| 
|               ptr_to_lock_record:30          | lock:2 | Lightweight Locked（轻量级锁）
|-------------------------------------------------------| 
|               ptr_to_heavyweight_monitor:30  | lock:2 | Heavyweight Locked（重量级锁）
|-------------------------------------------------------| 
|                                              | lock:2 |    Marked for GC（GC标记）
|-------------------------------------------------------| 
```

**lock** 是对象锁的标记位，占用最后两位长度；

**biased_lock** 标记对象锁是否位偏向锁，占用一位长度；1 表示偏向锁，0 表示没有偏向锁；结合 **lock** ，标记对象的锁状态：

| biase_lock | lock | 状态     |
| ---------- | ---- | -------- |
| 0          | 01   | 无锁     |
| 1          | 01   | 偏向锁   |
|            | 00   | 轻量级锁 |
|            | 10   | 重量级锁 |
|            | 11   | GC标记   |

**age** 对象的年龄，占用四位长度；在GC中，对象在Survivor区每复制一次，年龄加1；当年龄达到设置的值，对象会被移动到老年代；由于只有4位二进制，所以默认年龄是15；可通过`-XX:MaxTenuringThreshold` 设置；

**identity_hashcode** 占用25位长度，对象标识Hash码，采用延迟加载技术。调用方法`System.identityHashCode()`计算，并会将结果写到该对象头中。当对象被锁定时，该值会移动到管程Monitor中。

**thread** 持有当前对象偏向锁的线程ID

**epoch** 偏向时间戳

**ptr_to_lock_record** 指向栈中锁记录的指针

**ptr_to_heavyweight_monitor** 指向monitor对象的指针



##### Class Pointor

对象头中的这部分存储对象的类型指针，指向对象的类，通过这个指针确定对象是哪个类的实例；

##### Array Length

如果对象是数组，对象头中需要额外的空间的存储数组长度；普通对象没有这个占用；

### Monitor原理

Synchronized 实现的是重量级锁；所以在Java对象头部中对应的 **ptr_to_heavyweight_monitor** 存储了指向monitor的指针；

monitor的定义与初始化是由C语言实现的；具体实现有时间再写吧。。。

下面直接说结论：



![](http://static.laop.cc/images/20201215200451.png)

![](http://static.laop.cc/images/20201215200900.png)

> 1. 线程访问同步代码，需要获取monitor锁
> 2. 线程被jvm托管
> 3. jvm获取充当临界区锁的java对象
> 4. 根据java对象对象头中的重量级锁 ptr_to_heavyweight_monitor指针找到objectMonitor
> 5. 将当前线程包装成一个ObjectWaiter对象
> 6. 将ObjectWaiter加入_cxq(ContentionList)队列头部
> 7. _count++
> 8. 如果owner是其他线程说明当前monitor被占据，则当前线程阻塞。如果没有被其他线程占据，则将owner设置为当前线程，将线程从等待队列中删除，count--。
> 9. 当前线程获取monitor锁，如果条件变量不满足，则将线程放入WaitSet中。当条件满足之后被唤醒，把线程从WaitSet转移到EntrySet中。
> 10. 当前线程临界区执行完毕
> 11. Owner线程会在unlock时，将ContentionList中的部分线程迁移到EntryList中，并指定EntryList中的某个线程为OnDeck线程（一般是最先进去的那个线程）。Owner线程并不直接把锁传递给OnDeck线程，而是把锁竞争的权利交个OnDeck，OnDeck需要重新竞争锁



参考：[https://www.cnblogs.com/qingshan-tang/p/12698705.html](https://www.cnblogs.com/qingshan-tang/p/12698705.html)

