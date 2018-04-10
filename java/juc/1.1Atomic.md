# 1 CAS

- CAS (Compare-And-Swap) 是一种硬件对并发的支持，针对多处理器 操作而设计的处理器中的一种特殊指令，用于管理对共享数据的并发访问。
- CAS 是一种无锁的非阻塞算法的实现。
- CAS 包含了 3 个操作数:

 - 需要读写的内存值 V
 - 进行比较的值 A
 - 拟写入的新值 B
 
- 当且仅当 V 的值等于 A 时，CAS 通过原子方式用新值 B 来更新 V 的值，否则不会执行任何操作。

# 2 Unsafe

- 获取Unsafe：private static final Unsafe unsafe = Unsafe.getUnsafe();


# 3 Atomic

- 变量valueOffset，表示该变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据的。
- 变量value用volatile修饰，保证了多线程之间的内存可见性。
- lazySet：unsafe.putOrderedInt是putIntVolatile的延迟实现，不保证值的改变被其他线程立即看到， 省去了StoreLoad屏障, 只留下StoreStore，可以减少不必要的内存屏障，从而提高程序执行的效率。<br>

```java
public final void lazySet(int newValue) {
    unsafe.putOrderedInt(this, valueOffset, newValue);
}
```
- compareAndSet：<br>

```java
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

  - 如果是多处理器，为cmpxchg指令添加lock前缀。
  - 单处理器就省略lock前缀。（不需要lock前缀提供的内存屏障效果）

> intel手册对lock前缀的说明如下：<br>
> 1. 确保后续指令执行的原子性。
Intel使用缓存锁定来保证指令执行的原子性，缓存锁定将大大降低lock前缀指令的执行开销。<br>
> 2. 禁止该指令与前面和后面的读写指令重排序。<br>
> 3. 把写缓冲区的所有数据刷新到内存中。

- weakCompareAndSet：不会保证happen-before关系<br>
jdk8之前实现与compareAndSet相同<br>
jdk9：底层调用的native方法<br>

```java
@HotSpotIntrinsicCandidate
public final native boolean compareAndSetInt(Object o, long offset,
                                             int expected,
                                             int x);
```

### Array

- int base = unsafe.arrayBaseOffset(int[].class)<br>
获取该类型的数组，在对象存储时存放第一个元素的内存地址，相对于数组对象起始地址的内存偏移量。

- int scale = unsafe.arrayIndexScale(int[].class);
获取该类型的数组中元素的大小，占用多少个字节。

### FieldUpdater

- 基于反射原理实现，对指定类的 'volatile long'类型的成员进行原子更新。
- 必须用volatile修饰
- 使用<br>

```java
public class FieldUpdaterDemo {
	public static void main(String[] args) {
		Class clz = Clazz.class;
		AtomicLongFieldUpdater longFieldUpdater = AtomicLongFieldUpdater.newUpdater(clz, "vl");
		Clazz clazz = new Clazz(1L);
		longFieldUpdater.compareAndSet(clazz, 1L, 100L);
		System.out.println(clazz.getVl()); // 100
	}
}
class Clazz{
	volatile long vl;

	public Clazz(long vl) {
		this.vl = vl;
	}

	public long getVl() {
		return vl;
	}
}
```


