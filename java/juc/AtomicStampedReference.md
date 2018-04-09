# 1 AtomicStampedReference

- 使用时间戳解决ABA问题
- 使用Pair维护对象值和版本号(int stamp)

```java
private static class Pair<T> {
    final T reference;
    final int stamp;
    private Pair(T reference, int stamp) {
        this.reference = reference;
        this.stamp = stamp;
    }
    static <T> Pair<T> of(T reference, int stamp) {
        return new Pair<T>(reference, stamp);
    }
}
```

- 等到修改的时候，比较当前版本号与当前线程持有的版本号是否一致，如果一致，则进行修改，并且修改版本号

```java
public boolean compareAndSet(V   expectedReference,
                             V   newReference,
                             int expectedStamp,
                             int newStamp) {
    Pair<V> current = pair;
    return
        expectedReference == current.reference &&
        expectedStamp == current.stamp &&
        ((newReference == current.reference &&
          newStamp == current.stamp) ||
         casPair(current, Pair.of(newReference, newStamp)));
}
```

# 2 AtomicMarkableReference

- 通过一个boolean来标记是否更改，本质就是只有true和false两种版本来回切换
- 只能降低aba问题发生的几率，并不能阻止aba问题的发生