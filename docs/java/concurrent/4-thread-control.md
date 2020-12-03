# CountDownLatch 
CountDownLatch类位于java.util.concurrent包下，它类似一个线程计数器。
比如学生考试的场景：当所有的学生都做完了，老师才开始收试卷。
```
    public static CountDownLatch cdl;

    public static class Student extends Thread {
        public void run() {
            try {
                Thread.sleep(1000);
                System.out.println(Thread.currentThread() + " -- finished !");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                cdl.countDown();
            }
        }
    }

    public static class Teacher {
        public void finnish() {
            try {
                cdl.await();
                System.out.println(Thread.currentThread() + " -- 考试结束，收卷");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        cdl = new CountDownLatch(10);
        for (int i = 0; i < 10; i++) {
            Student stu = new Student();
            stu.start();
        }
        Teacher tch = new Teacher();
        tch.finnish();
    }

```

# CyclicBarrier 
CyclicBarrier类同样位于java.util.concurrent包下，它强制多条线程都达到某一状态之后才继续往下执行。
比如考试的场景：当所有的学生都考完之后才能离开考场。
```
public class CyclicBarrierTest {

    public static class Student extends Thread {

        private CyclicBarrier cb;

        public Student(CyclicBarrier cb){
            this.cb = cb;
        }

        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread() + ". Begin...");
                Thread.sleep(1000);
                System.out.println("Finish.");
                cb.await();
                System.out.println(Thread.currentThread() + ". I get out");
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        CyclicBarrier cb = new CyclicBarrier(10);
        for (int i = 0; i < 10; i++) {
            new Student(cb).start();
        }
        Thread.sleep(5000);
    }
}
```

# Phaser
Phaser类位于java.util.concurrent包下，实现的是一个线程阶段器，功能类似于CyclicBarrier，不同的是Phaser强制线程阶段性同步执行。
比如仍然是考试的场景：总共若干道题，只有所有的人都答完某题之后才能进入下一题的解答。
```
public class PhaserTest {

    public static class Student extends Thread {
        private Phaser ps;

        public Student(Phaser ps) {
            this.ps = ps;
        }

        public void run() {
            try {
                answer("first question");
                answer("second question");
                answer("third question");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        public void answer(String question) throws InterruptedException {
            Thread.sleep(1000);
            System.out.println(Thread.currentThread() + ": finish " + question);
            ps.arriveAndAwaitAdvance();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Phaser ps = new Phaser(10);
        for (int i = 0; i < 10; i++) {
            new Student(ps).start();
        }
        Thread.sleep(10000);
    }
}
```
