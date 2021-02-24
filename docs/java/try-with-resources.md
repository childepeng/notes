# try-with-resources

在IO操作中，一般需要使用到 `try-catch-finally` 来处理IO处理过程中的异常，finally代码块通常用于关闭io对象。由于close也会抛出IOException，所以我们不得不再finally中使用`try-catch`来处理close操作，或者将close操作的异常抛出；如果 try 代码块 和 finally 代码块中都存在异常，思考一下，最终抛出的是哪个异常？

```java
public void fun() {
    OutputStream os = null;
    try {
        os = new FileOutputStream("test.txt");
        os.write("hello".getBytes());
    } catch (IOException e) {
        // throw e;
    } finally {
        // os.close();
        try {
            os.close();
        } catch(IOException e){
            // throw e;
        }
    }
}
```

如果都出现了异常，我们将 try块 和 finally 中的异常都抛出，外层方法只能接收到 finally 代码块的异常，try中的异常被抑制(suppress)了； 当然正常情况下，使用 `try-catch-finally` 异常都在方法内就会被处理掉，毕竟这么写的目的就是处理异常；不过 finally 中的close操作还是会看起来很不优雅；

> 被抑制的异常可以通过当前异常对象的 getSuppressed() 方法获取；

Java7提供了新的处理方式 `try-with-resources` ，上述代码改造如下：

```java
public void fun() {
    try (OutputStream os = new FileOutputStream("test.txt")) {
        os.write("hello".getBytes());
    } catch (IOException e) {
        // throw e;
    } 
}
```

Java在 try代码块执行完成之后，自动为我们调用 `close` 方法；如果还是上述的异常场景， close 方法产生的异常则会被抑制，最终抛出的是 try 块的异常；

try 后面可以声明多个对象，最终执行会按照对象声明顺序反向执行close；

```java
try (OutputStream os = new FileOutputStream("test.txt");
     OutputStream os2 = new FileOutputStream("test.txt")) {
    os.write("hello".getBytes());
}
```

`try-with-resources` 并不是完全针对的IO场景，所有继承自接口 `java.io.Closeable` 的对象都可以使用其进行自动关闭。

官方解释如下：

> The try-with-resources statement is a try statement that declares one or more resources. A resource is as an object that must be closed after the program is finished with it. The try-with-resources statement ensures that each resource is closed at the end of the statement. Any object that implements java.lang.AutoCloseable, which includes all objects which implement java.io.Closeable, can be used as a resource.

