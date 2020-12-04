# Java Monitor机制

Synchronized 是Java的关键字，用于实现线程同步，可以强制线程在访问执行 synchronized 代码块的时候串行执行； 而它的背后是 Jvm 内部实现的 Monitor 机制。

## Monitor

monitor 常被翻译成监视器，这里通常被称为管程；Java中实现monitor机制的是两个指令： `monitorenter` 和 `monitorexit`

如下代码：

```java
public class LockTest {
    public void fun() {
        synchronized (this) {
            System.out.println("Hello");
        }
    }
}
```

编译之后使用 `javap` 命令进行反编译，得出如下指令代码：

```java
Compiled from "LockTest.java"
public class LockTest {
  public LockTest();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public void fun();
    Code:
       0: aload_0
       1: dup
       2: astore_1
       3: monitorenter
       4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       7: ldc           #3                  // String Hello
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
}
```

`monitorenter ` 和  `monitorexit ` 是java的原始指令码：

`monitorenter` 表示进入并获得对象的监视器

`monitorexit` 表示释放并退出对象的监视器，上述代码中出现了两次：13和19行，因为这里有个类似`try - finaly` 的异常处理机制，保证监视器最终能够释放，防止死锁。前面如果可以正常释放监视器，后面就会通过 `goto 22`  跳到 22 行执行 `return` ; 而 `Exception table` 定义了可能出现异常的指令范围；

Java指令集参考文档：

[Orcale官方文档]: https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5
[转载]: http://www.blogjava.net/DLevin/archive/2011/09/13/358497.html

