# 线程通信
线程是程序执行的最小单元，在系统独立执行，实现线程间的通信本质上都是采用的“共享内存”的方式，在Java中通常使用共享对象实现线程间的通信。

## Object类的wait、notify、notifyAll
wait: 线程释放该对象的锁，并进入等待状态；
notify: 线程唤醒一条在该对象上进入等待最久的线程；
notifyAll: 唤醒所有在该对象上等待的线程。
这三个方法在执行之前线程必须先获取对象锁，所以必须放在synchronized代码块中执行。

```java
public class ThreadCommunication {

    private Entity entity = new Entity();

    public static class Entity {
        String message;
    }

    public void threadA() {
        new Thread(() -> {
            synchronized (entity) {
                try {
                    entity.wait();
                    System.out.println("ThreadA message: " + entity.message);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    public void threadB() {
        new Thread(() -> {
            synchronized (entity) {
                entity.message = "Hello world";
                entity.notify();
            }
        }).start();
    }

    public static void main(String[] args) throws InterruptedException {
        ThreadCommunication tc = new ThreadCommunication();
        tc.threadA();
        tc.threadB();
        Thread.sleep(5000);
    }
}
```

## Exchanger
Exchanger类在java.util.concurrent包下，在多线程环境中用于线程间的数据交换。
```java
public class ExchangerTest {

    private static Exchanger<String> changer = new Exchanger<>();

    public static void threadA() {
        new Thread(() -> {
            try {
                String msg = changer.exchange("I am A!");
                System.out.println("threadA: " + msg);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }

    public static void threadB() {
        new Thread(() -> {
            try {
                String msg = changer.exchange("I am B!");
                System.out.println("threadB: " + msg);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }

    public static void main(String[] args) throws InterruptedException {
        threadA();
        threadB();
        Thread.sleep(2000);
    }
}
```