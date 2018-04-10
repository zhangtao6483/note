# 1 Striped64

- @sun.misc.Contended 避免伪共享<br>
user classpath使用此注解默认是无效的，需要在jvm启动时设置-XX:-RestrictContended 
- Cell类中仅有一个保存计数的变量value，并且为该变量提供了CAS操作方法

### longAccumulate/doubleAccumulate<br>

- 当没有竞争时，所有的更新都作用到 base 字段。
- 根据第一次竞争（更新 base 的 CAS 失败），cells数组初始化大小为2。cells数组的大小根据更多的竞争加倍，直到大于或等于CPU数量的最小的 2 的幂。
- 一个CAS实现的锁，只有在需要更新cells数组的时候才会更新该值为1，如果更新失败，则说明当前有线程在更新cells数组，当前线程需要等待。
- 通过 ThreadLocalRandom 维护线程探针字段，作为每线程的哈希码。


# 2 LongAdder

- 提供add(long x)/ sum() 方法
- sumThenReset() 返回总和后重置

# 3 LongAccumulator

- 提供双目运算器，据输入的两个参数返回一个计算值，identity则是LongAccumulator累加器的初始值

```java
public LongAccumulator(LongBinaryOperator accumulatorFunction, long identity)
```
- 使用方法：(多线程处理结果不正确.)

```java
LongAccumulator accumulator = new LongAccumulator((left, right) -> left * right, 1);

accumulator.accumulate(2);
```

<br>
https://coderbee.net/index.php/concurrent/20140518/931
