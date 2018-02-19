# String,StringBuffer,StringBuilder

## 1. String

### 1.1 

- String对象中使用char数组存储字符串（*private final char value[]*），jdk9使用byte数组存储（*private final byte[] value*）+ 编码（*private final byte coder*），如果String只包含Latin1字符，可以压缩存储空间，只使用1字节存一个字符
- String a = "test" == String b = "test" == String c = "te" + "st"
- String a = "test" != String b = new String("test");
- String a = "te"; String b = "st"; String c = a + b;字符串变量的连接动作，在编译阶段会被转化成StringBuilder的append操作
- final String a = "te"; final String b = "st"; String c = a + b;当final修饰的变量发生连接动作时，虚拟机会进行优化，将表达式结果直接赋值给目标变量
- <br>![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/java/string1.png)
- Java代码被编译成class文件时，会生成一个常量池（Constant pool）的数据结构，用以保存字面常量和符号引用（类名、方法名、接口名和字段名等）

### 1.2 方法

- equals

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

- hashcode

``` java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

### 1.2 String 不可变

1. 只有当字符串是不可变的，字符串池才有可能实现。字符串池的实现可以在运行时节约很多heap空间，因为不同的字符串变量都指向池中的同一个字符串。
2. 如果字符串是可变的，那么会引起很严重的安全问题。（改变字符串指向的对象的值，造成安全漏洞。）
3. 因为字符串是不可变的，所以是多线程安全的。
4. 类加载器要用到字符串，不可变性提供了安全性，以便正确的类被加载。
5. 因为字符串是不可变的，所以在它创建的时候hashcode就被缓存了，不需要重新计算。

### 1.3 String.intern() 

- String.intern()是一个Native方法，底层调用C++的 StringTable::intern 方法
- 当调用 intern 方法时，如果常量池中已经该字符串，则返回池中的字符串；否则将此字符串添加到常量池中，并返回字符串的引用。

---
1. jdk6及之前版本中，常量池的内存在永久代PermGen进行分配，所以常量池会受到PermGen内存大小的限制。
2. jdk7中，常量池的内存在Java堆上进行分配，意味着常量池不受固定大小的限制了。
3. jdk8中，移除了永久代PermGen。

## 2. StringBuffer

## 3. StringBuilder

## 4. 

**性能方面**：StringBuilder > StringBuffer > String (+) (for循环里字符串拼接)
**线程安全**：StringBuilder（非线程安全，速度快），StringBuffer（线程安全，速度慢）

