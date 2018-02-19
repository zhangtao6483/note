# Vector,ArrayList,LinkedList

## 1. Vector

- Object数组存储（*protected Object[] elementData;*）
- 同步方法synchronized，保证线程安全
- 默认初始容量10（*protected int elementCount;*），capacityIncrement保存扩容增长步长，如果capacityIncrement==0，增长2倍 
- 扩容方法<br>

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

## 2. ArrayList

- Object数组存储（*transient Object[] elementData; // non-private to simplify nested class access*）
- 默认初始容量10
- 按数组下标访问元素－get（i）、set（i,e）、add(e) 的性能很高
- 如果按下标插入元素、删除元素－add（i,e）、 remove（i）、remove（e），则要用System.arraycopy（）来复制移动部分受影响的元素，性能就变差了<br>

```java
public void add(int index, E element) {
        rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
```

- trimToSize()方法用来缩小elementData数组的大小，这样可以节约内存<br>

```java
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0)
          ? EMPTY_ELEMENTDATA
          : Arrays.copyOf(elementData, size);
    }
}
```

## 3. LinkedList

- 以双向链表实现。<br>

```java
transient Node<E> first;
transient Node<E> last;
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

- 按下标访问元素－get（i）、set（i,e） 要部分遍历链表将指针移动到位 （如果i>数组大小的一半，会从末尾移起）。<br>

```java
Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

- 插入、删除元素时修改前后节点的指针即可，不再需要复制移动。但还是要部分遍历链表的指针才能移动到下标所指的位置。
- 只有在链表两头的操作 add（）、addFirst（）、removeLast（）或用iterator（）上的remove（）能省掉指针的移动。