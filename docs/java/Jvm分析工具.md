# JVM分析工具

## jps

jps (jvm process status tool)，用户查询系统上允许的JVM进程信息；支持输出PID、主类、启动参数等

```bash
usage: jps [-help]
       jps [-q] [-mlvV] [<hostid>]

Definitions:
    <hostid>:      <hostname>[:<port>]
```

参数说明：

> 输出信息选项，默认省略 -lq
>
> -q	只输出jvmid
>
> -l	输出主类全名或者路径
>
> -m	输出启动main方法参数
>
> -v	输出显示指定的jvm参数
>
> hostid，远程查询

## jstat

jstat (jvm statistics monitoring ) 监控虚拟机运行状态，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译(Just In Time Compiler, 即时编译器)等运行数据。

```bash
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

Definitions:
  <option>      An option reported by the -options option
  <vmid>        Virtual Machine Identifier. A vmid takes the following form:
                     <lvmid>[@<hostname>[:<port>]]
                Where <lvmid> is the local vm identifier for the target
                Java virtual machine, typically a process id; <hostname> is
                the name of the host running the target Java virtual machine;
                and <port> is the port number for the rmiregistry on the
                target host. See the jvmstat documentation for a more complete
                description of the Virtual Machine Identifier.
  <lines>       Number of samples between header lines.
  <interval>    Sampling interval. The following forms are allowed:
                    <n>["ms"|"s"]
                Where <n> is an integer and the suffix specifies the units as
                milliseconds("ms") or seconds("s"). The default units are "ms".
  <count>       Number of samples to take before terminating.
  -J<flag>      Pass <flag> directly to the runtime system.
```

options:

```
$ jstat -options
-class			// classloader 行为统计
-compiler		// Hotspot JIT 行为统计
-gc				// GC行为统计
-gccapacity		// 堆分代空间容量统计，
-gccause		// 垃圾收集统计概述（同-gcutil），附加最近两次垃圾回收事件的原因
-gcmetacapacity		
-gcnew			//新生代行为统计
-gcnewcapacity	//新生代空间容量分析
-gcold			// 
-gcoldcapacity
-gcutil			//垃圾回收统计概述
-printcompilation	//HotSpot编译方法统计
```

1、查看类加载统计

```bash
$ jstat -class 8624
Loaded  Bytes  Unloaded  Bytes     Time
 82901 174475.8     2644  3142.4     575.01

# loaded 已加载类数量
# bytes class大小
# unload  未加载类数量
# times 加载时间
```

2、查看GC统计

```bash
$ jstat -gc 8624
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
22272.0 22272.0 14023.1  0.0   178688.0 134360.1  768380.0   566965.4  559996.0 537215.0 74976.0 66134.8   1790   17.063   4      3.733   20.796
# 后缀C/U/GC/GCT分别表示：Capacity总容量、Used已使用、GC次数、GC总耗时
# S0 / S1 / E(新生代) / O(老年代) / M(元空间) / Y(新生代) / F(老年代)
# GCT 垃圾回收总耗时

```

3、查看JVM内存空间大小

```bash
$ jstat -gccapacity 8624
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC
   192.0 340736.0 340736.0 34048.0 34048.0 272640.0       64.0  1744128.0   768380.0   768380.0      0.0 1533952.0 560252.0      0.0 1048576.0  74976.0   1790     5

# 后缀MN/MX/C分别表示 最小空间、最大空间、总空间
```

## jmap

jmap (jvm memory map) 用于生成heapdump文件；如果不使用这个命令，还阔以使用 `-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/heapdump.hprof` 参数来让虚拟机出现OOM的时候自动生成dump文件。 jmap不仅能生成dump文件，还阔以查询finalize执行队列、Java堆和永久代的详细信息，如当前使用率、当前使用的是哪种收集器等。

```bash
$ jmap -help
Usage:
    jmap [option] <pid>
        (to connect to running process)
    jmap [option] <executable <core>
        (to connect to a core file)
    jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    <none>               to print same info as Solaris pmap
    -heap                to print java heap summary
    -histo[:live]        to print histogram of java object heap; if the "live"
                         suboption is specified, only count live objects
    -clstats             to print class loader statistics
    -finalizerinfo       to print information on objects awaiting finalization
    -dump:<dump-options> to dump java heap in hprof binary format
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format
                           file=<file>  dump heap to <file>
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   force. Use with -dump:<dump-options> <pid> or -histo
                         to force a heap dump or histogram when <pid> does not
                         respond. The "live" suboption is not supported
                         in this mode.
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system
```

1、生成虚拟机内存快照

```bash
$ jmap -dump:live,format=b,file=/home/javaheap.hprof 8624
# live表示活着的对象，format=b表示二进制格式，file输出文件路径
```

2、查看堆概要信息

```bash
$ jmap -heap 10584
Attaching to process ID 10584, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.251-b08

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 4269801472 (4072.0MB)
   NewSize                  = 89128960 (85.0MB)
   MaxNewSize               = 1422917632 (1357.0MB)
   OldSize                  = 179306496 (171.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 1302855680 (1242.5MB)
   used     = 707270536 (674.505744934082MB)
   free     = 595585144 (567.994255065918MB)
   54.28617665465449% used
From Space:
   capacity = 54525952 (52.0MB)
   used     = 25590072 (24.40459442138672MB)
   free     = 28935880 (27.59540557861328MB)
   46.93191234882061% used
To Space:
   capacity = 57147392 (54.5MB)
   used     = 0 (0.0MB)
   free     = 57147392 (54.5MB)
   0.0% used
PS Old Generation
   capacity = 200278016 (191.0MB)
   used     = 44070976 (42.02935791015625MB)
   free     = 156207040 (148.97064208984375MB)
   22.004899429401178% used

38998 interned Strings occupying 4493072 bytes.

# From Space（S0）/ To Space（S1）
```

3、打印堆中对象的统计信息；包括对象数量、内存大小、类型等

```bash
$ jmap -histo:live 10584 | head -10

 num     #instances         #bytes  class name
----------------------------------------------
   1:        154606       18545680  [C
   2:         62698        5517424  java.lang.reflect.Method
   3:        153226        3677424  java.lang.String
   4:         81115        2595680  java.util.concurrent.ConcurrentHashMap$Node
   5:          8016        2374488  [B
   6:         26494        2354008  [Ljava.lang.Object;
   7:         13769        1547912  java.lang.Class

# head -10 表示输出头部10条数据
# live 表示只统计存活的对象
```

classname 中基本数据类型对应如下：

```
[B	-> byte
[C	-> char
[D  -> double
[F  -> float
[I  -> int
[J  -> long
[S	-> short
[Z  -> boolean
```

## jhat

jhat (java heap analytisis tool) Java堆内存分析工具，与jmap配合使用，用于分析jmap生成的dump文件； jhat内置了http服务器，支持在线查看分析报告；

```bash
$ jhat -h
Usage:  jhat [-stack <bool>] [-refs <bool>] [-port <port>] [-baseline <file>] [-debug <int>] [-version] [-h|-help] <file>

        -J<flag>          Pass <flag> directly to the runtime system. For
                          example, -J-mx512m to use a maximum heap size of 512MB
        -stack false:     Turn off tracking object allocation call stack.
        -refs false:      Turn off tracking of references to objects
        -port <port>:     Set the port for the HTTP server.  Defaults to 7000
        -exclude <file>:  Specify a file that lists data members that should
                          be excluded from the reachableFrom query.
        -baseline <file>: Specify a baseline object dump.  Objects in
                          both heap dumps with the same ID and same class will
                          be marked as not being "new".
        -debug <int>:     Set debug level.
                            0:  No debug output
                            1:  Debug hprof file parsing
                            2:  Debug hprof file parsing, no server
        -version          Report version number
        -h|-help          Print this help and exit
        <file>            The file to read

```

用例

```bash
$ jhat javaheap.hprof
# 分析完成之后，查看报告：http://localhost:7000
```

## jstack

jvm 线程分析工具，可以生成当前时刻的线程快照；线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。 如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。另外，jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息。

```bash
$ jstack
Usage:
    jstack [-l] <pid>
        (to connect to running process)
    jstack -F [-m] [-l] <pid>
        (to connect to a hung process)
    jstack [-m] [-l] <executable> <core>
        (to connect to a core file)
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
        (to connect to a remote debug server)

Options:
    -F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
    -m  to print both java and native frames (mixed mode)
    -l  long listing. Prints additional information about locks
    -h or -help to print this help message

```

用例

```bash
$ jstack 10584 > thread_dump.txt
# 生成线程快照，并输出到文件中
```

线程堆栈分析说明

### 线程堆栈解读

比如如下一段代码：

```java
public class ThreadTest {
    private static synchronized void fun() {
        while (true) {
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    public static void main(String[] args) throws IOException {
        new Thread(() -> fun(), "T1").start();
        new Thread(() -> fun(), "T2").start();
        System.in.read();
    }
}
```

以下是提取的部分线程信息

```
2021-01-05 16:42:31
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.251-b08 mixed mode):

"T2" #12 prio=5 os_prio=0 tid=0x000000001e317000 nid=0x316c waiting for monitor entry [0x000000001ed5f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at cc.laop.jvm.ThreadTest.fun(ThreadTest.java:15)
	- waiting to lock <0x000000076b59f0a0> (a java.lang.Class for cc.laop.jvm.ThreadTest)
	at cc.laop.jvm.ThreadTest.lambda$main$1(ThreadTest.java:24)
	at cc.laop.jvm.ThreadTest$$Lambda$2/1078694789.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)

"T1" #11 prio=5 os_prio=0 tid=0x000000001e312000 nid=0x4790 waiting on condition [0x000000001ec5e000]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
	at java.lang.Thread.sleep(Native Method)
	at cc.laop.jvm.ThreadTest.fun(ThreadTest.java:15)
	- locked <0x000000076b59f0a0> (a java.lang.Class for cc.laop.jvm.ThreadTest)
	at cc.laop.jvm.ThreadTest.lambda$main$0(ThreadTest.java:23)
	at cc.laop.jvm.ThreadTest$$Lambda$1/1324119927.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)

"Service Thread" #10 daemon prio=9 os_prio=0 tid=0x000000001e04f800 nid=0x240c runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread2" #9 daemon prio=9 os_prio=2 tid=0x000000001dfbc800 nid=0x4514 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" #8 daemon prio=9 os_prio=2 tid=0x000000001dfab000 nid=0x2b5c waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #7 daemon prio=9 os_prio=2 tid=0x000000001dfa3800 nid=0x4fb0 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Monitor Ctrl-Break" #6 daemon prio=5 os_prio=0 tid=0x000000001dfa2000 nid=0x446c runnable [0x000000001e65e000]
   java.lang.Thread.State: RUNNABLE
	at java.net.SocketInputStream.socketRead0(Native Method)
	at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
	at java.net.SocketInputStream.read(SocketInputStream.java:171)
	at java.net.SocketInputStream.read(SocketInputStream.java:141)
	at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:284)
	at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:326)
	at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:178)
	- locked <0x000000076b487540> (a java.io.InputStreamReader)
	at java.io.InputStreamReader.read(InputStreamReader.java:184)
	at java.io.BufferedReader.fill(BufferedReader.java:161)
	at java.io.BufferedReader.readLine(BufferedReader.java:324)
	- locked <0x000000076b487540> (a java.io.InputStreamReader)
	at java.io.BufferedReader.readLine(BufferedReader.java:389)
	at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:64)

"Attach Listener" #5 daemon prio=5 os_prio=2 tid=0x000000001cbb0000 nid=0x2e1c waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 tid=0x000000001df68800 nid=0x4534 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=1 tid=0x00000000035ac000 nid=0x46c8 in Object.wait() [0x000000001deff000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x000000076b308ee0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
	- locked <0x000000076b308ee0> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
	at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)

"Reference Handler" #2 daemon prio=10 os_prio=2 tid=0x00000000035a3000 nid=0x4f2c in Object.wait() [0x000000001ddff000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(Native Method)
	- waiting on <0x000000076b306c00> (a java.lang.ref.Reference$Lock)
	at java.lang.Object.wait(Object.java:502)
	at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
	- locked <0x000000076b306c00> (a java.lang.ref.Reference$Lock)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"main" #1 prio=5 os_prio=0 tid=0x00000000034b3800 nid=0x4ff0 runnable [0x000000000338f000]
   java.lang.Thread.State: RUNNABLE
	at java.io.FileInputStream.readBytes(Native Method)
	at java.io.FileInputStream.read(FileInputStream.java:255)
	at java.io.BufferedInputStream.fill(BufferedInputStream.java:246)
	at java.io.BufferedInputStream.read(BufferedInputStream.java:265)
	- locked <0x000000076b35ad10> (a java.io.BufferedInputStream)
	at cc.laop.jvm.ThreadTest.main(ThreadTest.java:25)

"VM Thread" os_prio=2 tid=0x000000001cb67800 nid=0x3fbc runnable 

"GC task thread#0 (ParallelGC)" os_prio=0 tid=0x00000000034c8800 nid=0x4f40 runnable 

"GC task thread#1 (ParallelGC)" os_prio=0 tid=0x00000000034ca800 nid=0x3180 runnable 

"GC task thread#2 (ParallelGC)" os_prio=0 tid=0x00000000034cc000 nid=0x2424 runnable 

"GC task thread#3 (ParallelGC)" os_prio=0 tid=0x00000000034ce800 nid=0x2b84 runnable 

"VM Periodic Task Thread" os_prio=2 tid=0x000000001e050000 nid=0x453c waiting on condition 

JNI global references: 317


```

以上信息，每一段代表一条线程，其中包括用户创建的线程，也有像GC线程这样的虚拟机创建的线程；

第一行表示线程状态信息； 

`at ...` 信息表示线程当前运行的方法栈，第一条表示当前正在运行的方法；

`java.lang.Thread.State: RUNNABLE` 表示线程状态；

```
"NettyServerWorker-3-2"		// 线程名称
#111 						// 编号
daemon 						// 线程运行方式， daemon
prio=5 						// 线程优先级
os_prio=0 					// 内核线程优先级
tid=0x00000000236cd000 		// 线程ID(jvm虚拟机)
nid=0x3334 					// 线程ID(系统)，16进制表示
runnable 					// 线程状态
[0x000000002f5ae000]		// 线程内存地址
```

线程状态：

> New	新建Thread
>
> Runnable	运行状态
>
> Waitting		无限等待状态，线程状态不会自动切换，需要其他线程唤醒，比如，线程调用了方法 wait() ；
>
> Timed Waitting	有限时间的等待，虚拟机会自动在一段时间之后，唤醒线程，比如，线程调用wait(timeout)指定等待时间；
>
> Block	阻塞状态，线程获取已被其他线程持有的对象锁，会进入阻塞状态，比如 在进入 Synchronized 代码块的时候；
>
> Terminated	线程运行结束；



上面的堆栈信息

第7行表示线程等待获取对象的锁，对象内存地址 `0x000000076b59f0a0` , 对象类型 `ThreadTest` ，由于线程当前在等待获得对象锁进入Synchronized代码块，因此此时线程状态为 `BLOCKED`

```
- waiting to lock <0x000000076b59f0a0> (a java.lang.Class for cc.laop.jvm.ThreadTest)
```

第16行表示当前线程持有对象的锁，对象地址 `0x000000076b59f0a0` ，类型 `ThreadTest`，由于执行了 sleep 方法，线程进入 `TIMED_WAITING`

```
- locked <0x000000076b59f0a0> (a java.lang.Class for cc.laop.jvm.ThreadTest)
```

第59行表示线程调用wait()方法，进入等待状态， `WAITING`

```
- waiting on <0x000000076b308ee0> (a java.lang.ref.ReferenceQueue$Lock)
```



### 使用线程堆栈分析具体问题

#### 死锁分析

当两个或两个以上的线程因为环路的锁依赖而形成的锁环，就会发生死锁；

如下代码，简单模拟了死锁的场景，代码运行之后，线程1持有dl1的锁同时等待获取dl2的锁，线程2持有dl2的锁同时等待获取dl1的锁；

```java
public class DeadLockTest {

    public synchronized void test(DeadLockTest dl) {
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        dl.print();
    }

    public synchronized void print() {
    }

    public static void main(String[] args) throws IOException {
        DeadLockTest dl1 = new DeadLockTest();
        DeadLockTest dl2 = new DeadLockTest();
        
        new Thread(() -> dl1.test(dl2)).start();
        new Thread(() -> dl2.test(dl1)).start();
        
        System.in.read();
    }

}
```

线程堆栈中会提示死锁发生的场景，一下是截取的部分堆栈信息

```
Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x000000000336b848 (object 0x000000076b5a8bf8, a cc.laop.java.concurrent.DeadLockTest),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x000000000336b9a8 (object 0x000000076b5a8c08, a cc.laop.java.concurrent.DeadLockTest),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
        at cc.laop.java.concurrent.DeadLockTest.print(DeadLockTest.java:22)
        - waiting to lock <0x000000076b5a8bf8> (a cc.laop.java.concurrent.DeadLockTest)
        at cc.laop.java.concurrent.DeadLockTest.test(DeadLockTest.java:18)
        - locked <0x000000076b5a8c08> (a cc.laop.java.concurrent.DeadLockTest)
        at cc.laop.java.concurrent.DeadLockTest.lambda$main$1(DeadLockTest.java:29)
        at cc.laop.java.concurrent.DeadLockTest$$Lambda$2/1747585824.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
"Thread-0":
        at cc.laop.java.concurrent.DeadLockTest.print(DeadLockTest.java:22)
        - waiting to lock <0x000000076b5a8c08> (a cc.laop.java.concurrent.DeadLockTest)
        at cc.laop.java.concurrent.DeadLockTest.test(DeadLockTest.java:18)
        - locked <0x000000076b5a8bf8> (a cc.laop.java.concurrent.DeadLockTest)
        at cc.laop.java.concurrent.DeadLockTest.lambda$main$0(DeadLockTest.java:28)
        at cc.laop.java.concurrent.DeadLockTest$$Lambda$1/1078694789.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.

```

#### Java代码导致的CPU占用过高

对于CPU占用过高的问题，首先需要查占用CPU高的线程ID；linux里面使用 `top -Hp <jvm pid>` ，根据线程ID查询堆栈中的线程信息，对照源码判断问题原因；（通过top查询的线程ID需要转换为16进制）

1、查询线程ID

2、线程ID转换为16进制

```bash
# 转10进制
printf '%d' <number> 	
# 转16进制
printf '%x' <number>
# 转八进制
printf '%o' <number>
```

3、详细线程堆栈，匹配线程信息中的 `nid` 属性；

4、对比源码确认问题原因；

#### 死循环分析

死循环一般会产生CPU的高占用；问题分析与上面的相同；

#### 性能分析

参考：http://www.hollischuang.com/archives/110

## jinfo

jinfo用于输出jvm参数；

## jvisualvm

jvisualvm 是 Oracle JDK自带的JVM状态分析工具；可实时查看JVM参数、堆栈、CPU使用、内存使用、线程状态等信息；