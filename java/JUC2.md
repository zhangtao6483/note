JUC2

## volatile 变量

- 内存可见性(Memory Visibility)是指当某个线程正在使用对象状态 而另一个线程在同时修改该状态，需要确保当一个线程修改了对象 状态后，其他线程能够看到发生的状态变化。
- Java 提供了一种稍弱的同步机制，即 volatile 变量，用来确保将变量的更新操作通知到其他线程。可以将 volatile 看做一个轻量级的锁，但是又与锁有些不同:
  - 对于多线程，不是一种互斥关系
  - 不能保证变量状态的“原子性操作”

重排序

```java
class ReorderExample {
int a = 0;
boolean flag = false;

public void writer() {
    a = 1;                   //1
    flag = true;             //2
}

Public void reader() {
    if (flag) {                //3
        int i =  a * a;        //4
        ……
    }
}
}
```
  
- volatile不能保证操作的原子性 eg: i=i+1;

## CAS操作

- CAS (Compare-And-Swap) 是一种硬件对并发的支持，针对多处理器 操作而设计的处理器中的一种特殊指令，用于管理对共享数据的并发访问。
- CAS 是一种无锁的非阻塞算法的实现。
- CAS 包含了 3 个操作数:  
  1. 需要读写的内存值 V
  2. 进行比较的值 A
  3. 拟写入的新值 B
- 当且仅当 V 的值等于 A 时，CAS 通过原子方式用新值 B 来更新 V 的值，否则不会执行任何操作。

```java
/*
 * 模拟 CAS 算法
 */
public class TestCompareAndSwap {

	public static void main(String[] args) {
		final CompareAndSwap cas = new CompareAndSwap();
		
		for (int i = 0; i < 10; i++) {
			new Thread(new Runnable() {
				
				@Override
				public void run() {
					int expectedValue = cas.get();
					boolean b = cas.compareAndSet(expectedValue, (int)(Math.random() * 101));
					System.out.println(b);
				}
			}).start();
		}
		
	}
	
}

class CompareAndSwap{
	private int value;
	
	//获取内存值
	public synchronized int get(){
		return value;
	}
	
	//比较
	public synchronized int compareAndSwap(int expectedValue, int newValue){
		int oldValue = value;
		
		if(oldValue == expectedValue){
			this.value = newValue;
		}
		
		return oldValue;
	}
	
	//设置
	public synchronized boolean compareAndSet(int expectedValue, int newValue){
		return expectedValue == compareAndSwap(expectedValue, newValue);
	}
}
```

非阻塞计数器

```java
@ThreadSafe
public class CasCounter {
    private SimulatedCAS value;
    
    public int getValue() {
        return value.get();
    }

    public int increment() {
        int v;
        do {
            v = value.get();
        }
        while (v != value.compareAndSwap(v, v + 1));
        return v + 1;
    }
}
```

- java.util.concurrent.atomic包下提供了一些原子操作的常用类:
  - AtomicBoolean、AtomicInteger、AtomicLong、AtomicReference 
  - AtomicIntegerArray、AtomicLongArray
  - AtomicMarkableReference
  - AtomicReferenceArray
  - AtomicStampedReference

  
- ABA问题
<br>
ABA问题的解决思路是给变量的值关联一个戳记，每次修改变量的值时同时改变戳记，那么即使变量的值不变也可以发现它是否被修改过。
- 循环时间长开销大
- 只能保证一个共享变量的原子操作

## AQS

AQS 是 java.util.concurrent.locks.AbstractQueuedSynchronizer 类的简称，它虽然只是一个类，但也是一个强大的框架，目的是为实现依赖于先进先出 (FIFO) 等待队列的阻塞锁和相关同步器（信号量、事件，等等）提供一个框架，这些类同步器都依赖单个原子 int 值来表示状态。

同步器一般包含两种方法，一种是acquire，另一种是release。acquire操作阻塞调用的线程，直到或除非同步状态允许其继续执行。而release操作则是通过某种方式改变同步状态，使得一或多个被acquire阻塞的线程继续执行。

acquire操作：

```
// 循环里不断尝试，典型的失败后重试
while (synchronization state does not allow acquire) {
     // 同步状态不允许获取，进入循环体，也就是失败后的处理
     enqueue current thread if not already queued;     // 如果当前线程不在等待队列里，则加入等待队列
     possibly block current thread;     // 可能的话，阻塞当前线程
}

// 执行到这里，说明已经成功获取，如果之前有加入队列，则出队列。
dequeue current thread if it was queued; 
```

release操作：

```java
update synchronization state;    //  更新同步状态
if (state may permit a blocked thread to acquire) // 检查状态是否允许一个阻塞线程获取
      unblock one or more queued threads;     // 允许，则唤醒后继的一个或多个阻塞线程。
```

## 线程池

hreadPoolExecutor的完整构造方法类型是：<br>ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler) .

<br>corePoolSize - 池中所保存的线程数，包括空闲线程。
<br>maximumPoolSize-池中允许的最大线程数。
<br>keepAliveTime - 当线程数大于核心时，此为终止前多余的空闲线程等待新任务的最长时间。
<br>unit - keepAliveTime 参数的时间单位。
<br>workQueue - 执行前用于保持任务的队列。此队列仅保持由 execute方法提交的 Runnable任务。
<br>threadFactory - 执行程序创建新线程时使用的工厂。
<br>handler - 由于超出线程范围和队列容量而使执行被阻塞时所使用的处理程序。
<br><br>ThreadPoolExecutor是Executors类的底层实现。

**corePoolSize和maximumPoolSize**
<br>如果池中的实际线程数小于corePoolSize,无论是否其中有空闲的线程，都会给新的任务产生新的线程
<br>如果池中的线程数>corePoolSize and <maximumPoolSize,而又有空闲线程，就给新任务使用空闲线程，如没有空闲线程，则产生新线程
<br>如果池中的线程数＝maximumPoolSize，则有空闲线程使用空闲线程，否则新任务放入workQueue。（线程的空闲只有在workQueue中不再有任务时才成立）

工具类 : Executors

- ExecutorService newFixedThreadPool() : 创建固定大小的线程池
- ExecutorService newCachedThreadPool() : 缓存线程池，线程池的数量不固定，可以根据需求自动的更改数量。
- ExecutorService newSingleThreadExecutor() : 创建单个线程池。线程池中只有一个线程
- ScheduledExecutorService newScheduledThreadPool() : 创建固定大小的线程，可以延迟或定时的执行任务。

## ConcurrentHashMap

jdk1.7

- ConcurrentHashMap采用分段锁的机制，实现并发的更新操作，底层采用数组和链表的存储结构。
- 其包含两个核心静态内部类 Segment和HashEntry。Segment继承ReentrantLock用来充当锁的角色，每个 Segment 对象守护每个散列映射表的若干个桶。HashEntry 用来封装映射表的键 / 值对；
- 每个桶是由若干个 HashEntry 对象链接起来的链表。

JDK1.8

- 利用CAS+Synchronized来保证并发更新的安全，底层采用数组和链表/红黑树的存储结构。

- 1.8中使用一个volatile类型的变量baseCount记录元素的个数，当插入新数据或则删除数据时，会通过addCount()方法更新baseCount

## CountDownLatch

- CountDownLatch 一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。
- 闭锁可以延迟线程的进度直到其到达终止状态，闭锁可以用来确保某些活 动直到其他活动都完成才继续执行
  - 确保某个计算在其需要的所有资源都被初始化之后才继续执行;
  - 确保某个服务在其依赖的所有其他服务都已经启动之后才启动;
  - 等待直到某个操作所有参与者都准备就绪再继续执行。

## CyclicBarrier

- CyclicBarrier也叫同步屏障，在JDK1.5被引入，可以让一组线程达到一个屏障时被阻塞，直到最后一个线程达到屏障时，所以被阻塞的线程才能继续执行。
- 叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。

## Semaphore
- Semaphore也叫信号量，在JDK1.5被引入，可以用来控制同时访问特定资源的线程数量，通过协调各个线程，以保证合理的使用资源。
- Semaphore内部维护了一组虚拟的许可，许可的数量可以通过构造函数的参数指定。
- 访问特定资源前，必须使用acquire方法获得许可，如果许可数量为0，该线程则一直阻塞，直到有可用许可。访问资源后，使用release释放许可。




---
引用：
http://www.importnew.com/21889.html
