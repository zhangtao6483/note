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

## 2. StringBuffer

### 2.1 

- 继承AbstractStringBuilder,使用char数组存储（ *char[] value;*）
- 方法使用synchronized保证线程安全
- toString()方法有toStringCache作为缓存，多次调用toString方法可以减少Arrays.copyOfRange的操作<br>

```
private transient char[] toStringCache;

public synchronized String toString() {
    if (toStringCache == null) {
        toStringCache = Arrays.copyOfRange(value, 0, count);
    }
    return new String(toStringCache, true);
}
```

### 2.2 扩容

- 默认的初始化容量16
- 如果用指定的String对象进行初始化则是将字符串的长度+16作为初始化的容量大小
- append一个空指针的时候会添加’n','u','l','l'四个字符，长度也同样增加4
- 每次需要扩容的时候都是按照"旧容量*2+2"进行扩容，如果扩容之后仍不满足所需容量，则直接扩容到所需容量

```java
void expandCapacity(int minimumCapacity) {
    int newCapacity = value.length * 2 + 2;
    if (newCapacity - minimumCapacity < 0)
        newCapacity = minimumCapacity;
    if (newCapacity < 0) {
        if (minimumCapacity < 0) // overflow
            throw new OutOfMemoryError();
        newCapacity = Integer.MAX_VALUE;
    }
    value = Arrays.copyOf(value, newCapacity);
}
```

## 3. StringBuilder

### 3.1 

- 字符串变量（非final修饰）通过 "+" 进行拼接，在编译过程中会转化为StringBuilder对象的append操作
- String str1 = "te"; String str2 = "st"; String str3 = sttr1 + str2 == new StringBuilder().append(str1).append(str2);
- 在for循环中，千万不要使用 "+" 进行字符串拼接，每次循环都会重新初始化StringBuilder对象，导致性能问题的出现。

## 4. 

**性能方面**：StringBuilder > StringBuffer > String (+) (for循环里字符串拼接)<br>
**线程安全**：StringBuilder（非线程安全，速度快），StringBuffer（线程安全，速度慢）<br>

- 连接2或3个String时，使用String.concat();
- 如果连接多于3个String，并且能够精确预测出结果里的长度，使用StringBuilder/StringBuffer，并且初始化容量
- 如果连接多于3个String，但是不能精确预测出最终结果的长度，使用StringBundler。*StringBundler在append()的时候，不急着往char[]里塞东西，而是先拿一个String[]把它们都存起来，到了最后才把所有String的length加起来，构造一个合理长度的StringBuilder。*


## 5. 参考
http://calvin1978.blogcn.com/articles/stringbuilder.html<br>
https://www.jianshu.com/p/160c9be0b132<br>
http://rednaxelafx.iteye.com/blog/774673
