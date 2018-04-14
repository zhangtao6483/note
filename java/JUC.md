# JUC

java 多线程demo

## 1 生产者消费者模型

### 1.1 synchronized实现

```java

public class Demo1 {

    public static void main(String[] args) {
        Clerk clerk = new Clerk();

        Producer pro = new Producer(clerk);
        Consumer cus = new Consumer(clerk);

        new Thread(pro, "生产者A").start();
        new Thread(cus, "消费者B").start();

        new Thread(pro, "生产者C").start();
        new Thread(cus, "消费者D").start();
    }
}

class Clerk {
    private int product = 0;

    public synchronized void produce() {
        while (product >= 10) {
            System.out.println("产品已满");
            try {
                this.wait();
            } catch (InterruptedException e) {
            }
        }
        System.out.println(Thread.currentThread().getName() + ":" + ++product);
        this.notifyAll();
    }

    public synchronized void consume() {
        while (product <= 0) {
            System.out.println("缺货");
            try {
                this.wait();
            } catch (InterruptedException e) {
            }
        }
        System.out.println(Thread.currentThread().getName() + ":" + --product);
        this.notifyAll();
    }
}


class Producer implements Runnable {
    private Clerk clerk;

    public Producer(Clerk clerk) {
        this.clerk = clerk;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            clerk.produce();
        }
    }
}

class Consumer implements Runnable {
    private Clerk clerk;

    public Consumer(Clerk clerk) {
        this.clerk = clerk;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            clerk.consume();
        }
    }
}
```

### 1.2 lock实现

```java
class Clerk {
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    private int product = 0;

    public void produce() {
        lock.lock();
        try {
            while (product >= 10) {
                condition.await();
                System.out.println("产品已满");
            }
            System.out.println(Thread.currentThread().getName() + ":" + ++product);
            condition.signalAll();
        } catch (InterruptedException e) {
        } finally {
            lock.unlock();
        }
    }

    public synchronized void consume() {
        lock.lock();
        try {
            while (product <= 0) {
                condition.await();
                System.out.println("缺货");
            }
            System.out.println(Thread.currentThread().getName() + ":" + --product);
            condition.signalAll();
        } catch (InterruptedException e) {
        } finally {
            lock.unlock();
        }
    }
}


class Producer implements Runnable {
    private Clerk clerk;

    public Producer(Clerk clerk) {
        this.clerk = clerk;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            clerk.produce();
        }
    }
}

class Consumer implements Runnable {
    private Clerk clerk;

    public Consumer(Clerk clerk) {
        this.clerk = clerk;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            clerk.consume();
        }
    }
}
```

## 2 线程通信

### 2.1 两个线程间

一个线程打印 1~52，另一个线程打印字母A-Z。打印顺序为12A34B56C……5152Z。

```java
public class Demo2 {

    public static void main(String[] args) {
        Print print = new Print();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 26; i++) {
                    print.printNumber();
                }
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 26; i++) {
                    print.printCharacter();
                }

            }
        }).start();
    }
}

class Print {

    private int number = 1;
    private char aChar = 'A';

    public synchronized void printNumber() {
        try {
            System.out.print(number++);
            System.out.print(number++);
            notifyAll();
            if (number <= 52) {
                wait();
            }


        } catch (InterruptedException e) {
        }
    }

    public synchronized void printCharacter() {
        try {
            System.out.println(aChar++);
            notifyAll();
            if (aChar <= 'Z') {
                wait();
            }
        } catch (InterruptedException e) {
        }
    }
}

```

### 2.2 多个线程间

启动3个线程打印递增的数字, 线程1先打印1,2,3,4,5, 然后是线程2打印6,7,8,9,10, 然后是线程3打印11,12,13,14,15. 接着再由线程1打印16,17,18,19,20....以此类推, 直到打印到75.

```java
public class Demo3 {

    public static void main(String[] args) {
        AlternateDemo alternateDemo = new AlternateDemo();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                alternateDemo.loopA();
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                alternateDemo.loopB();
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                alternateDemo.loopC();
            }
        }, "C").start();
    }

}

class AlternateDemo {

    private int number = 1;

    private Lock lock = new ReentrantLock();
    private Condition conditionA = lock.newCondition();
    private Condition conditionB = lock.newCondition();
    private Condition conditionC = lock.newCondition();

    public void loopA() {
        lock.lock();

        try {
            if (number != 1) {
                conditionA.await();
            }

            System.out.println(Thread.currentThread().getName() + ":" + number);

            number = 2;
            conditionB.signal();
        } catch (Exception e) {
        } finally {
            lock.unlock();
        }
    }

    public void loopB() {
        lock.lock();

        try {
            if (number != 2) {
                conditionB.await();
            }

            System.out.println(Thread.currentThread().getName() + ":" + number);

            number = 3;
            conditionC.signal();
        } catch (Exception e) {
        } finally {
            lock.unlock();
        }
    }

    public void loopC() {
        lock.lock();

        try {
            if (number != 3) {
                conditionC.await();
            }

            System.out.println(Thread.currentThread().getName() + ":" + number);

            number = 1;
            conditionA.signal();
        } catch (Exception e) {
        } finally {
            lock.unlock();
        }
    }
}
```
