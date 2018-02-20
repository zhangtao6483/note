# HashMap

## 1 

- 使用Node<K,V>[]数组实现哈希桶数组。（*transient Node<K,V>[] table;*）取模用与操作（hash & （arrayLength-1））
- hash方法： (h = key.hashCode()) ^ (h >>> 16);
- jdk8将原本Map.Entry接口的实现类Entry改名为了Node<br>

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
}
```

- 当链表长度大于阈值（*static final int TREEIFY_THRESHOLD = 8; 必须大于2，至少大于8*）时，将链表转化为红黑树，以减少搜索时间最好 O（1），最差 O（n），红黑树 O（logn）

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
}
```

- 阈值（*int threshold;*），等于加载因子（*final float loadFactor;* 默认：*static final float DEFAULT_LOAD_FACTOR = 0.75f;*） * 桶数量（默认：*static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16*），当实际大小超过阈值则进行扩容
- resize()
  - threshold >= 2^30 时 threshold = Integer.MAX_VALUE
  - 哈希桶扩容2倍容量扩容，开辟新空间来创建数组。
  - Node在新的数组中在原来的桶位置，或者移动到新的桶位置下（*(e.hash & oldCap) == 0*，所以计算桶位置的时候使用 & 而没有取模 %）<br>

```java
do {
    next = e.next;
    if ((e.hash & oldCap) == 0) {
        if (loTail == null)
            loHead = e;
        else
            loTail.next = e;
        loTail = e;
    }
    else {
        if (hiTail == null)
            hiHead = e;
        else
            hiTail.next = e;
        hiTail = e;
    }
} while ((e = next) != null);
```

- 如果两条Key落在同一个桶，那么要存储到同一个数组下标位置，这个现象就叫哈希碰撞。<br>

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null) //如果该桶下没值，则存储到这个位置
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p; //如果哈希值相同而且key相同， 则更新键值
        else if (p instanceof TreeNode) //如果是TreeNode类型，则将新数据添加到红黑树中。
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null); //将新Node添加到链表末尾
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st //如果链表个数达到8个时，将链表修改为红黑树结构
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```
