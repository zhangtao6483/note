# 1 Striped64

- @sun.misc.Contended 避免伪共享<br>
user classpath使用此注解默认是无效的，需要在jvm启动时设置-XX:-RestrictContended 
- Cell类中仅有一个保存计数的变量value，并且为该变量提供了CAS操作方法

longAccumulate/doubleAccumulate<br>

- 当没有竞争时，所有的更新都作用到 base 字段。
- 根据第一次竞争（更新 base 的 CAS 失败），表被初始化为大小 2。表的大小根据更多的竞争加倍，直到大于或等于CPU数量的最小的 2 的幂。
- 一个单独的自旋锁（“cellsBusy”）用于初始化和resize表，还有用新的Cell填充槽。不需要阻塞锁，当锁不可得，线程尝试其他槽（或 base）
- 通过 ThreadLocalRandom 维护线程探针字段，作为每线程的哈希码。



# 2 LongAdder

# 3 LongAccumulator

https://coderbee.net/index.php/concurrent/20140518/931?spm=a2c4e.11153940.blogcont20441.4.635576b0H3hSZ7

