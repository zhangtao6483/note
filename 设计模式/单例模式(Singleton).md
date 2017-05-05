# 单例模式(Singleton)

确保一个类只有一个实例，并且提供一个安全的全局访问点。
使用Singleton的好处还在于可以节省内存，因为它限制了实例的个数，有利于Java垃圾回收。
<br>
单例模式主要有3个特点：

1. 单例类确保自己只有一个实例。
2. 单例类必须自己创建自己的实例。
3. 单例类必须为其他对象提供唯一的实例。

-------------------

**懒汉模式：**

```java
public class Singleton {
    private static Singleton instance;
    private Singleton (){}

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

这种写法能够在多线程中很好的工作，而且看起来它也具备很好的lazy loading，但是，遗憾的是，效率很低，而且线程不安全。

-------------------

**饿汉模式：**

```java
public class Singleton {
    private static Singleton instance = new Singleton();
    private Singleton (){}

    public static Singleton getInstance() {
        return instance;
    }
}
```

这种方式基于classloder机制避免了多线程的同步问题，但是没有达到lazy loading的效果。

-------------------

**双重检测：**

```java
public class Singleton {
    public static final Singleton singleton = null;
    private Singleton(){}
    public static Singleton getInstance(){
        if(singleton == null){ //如果singleton为空，表明未实例化
           synchronized (Singleton.class){
               if( singleton == null ) { // double check 进来判断后再实例化。
                   singleton = new Singleton();
               }
        }
        return singleton;
    }
}
```

当两个线程执行完第一个 singleton == null 后等待锁， 其中一个线程获得锁并进入synchronize后，实例化了，然后退出释放锁，另外一个线程获得锁，进入又想实例化，会判断是否进行实例化了，如果存在，就不进行实例化了。

-------------------

**枚举：**

```java
public enum Singleton {  
    INSTANCE;  
    public void whateverMethod() {  
    }  
}  
```

这种方式是Effective Java作者Josh Bloch 提倡的方式，它不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象。

通过这种方式，不能通过反射和序列化来获取一个实例，因为所有的枚举类都继承自java.lang.Enum类, 而不是Object类：

```java
protected final Object clone() throws CloneNotSupportedException {  
    throw new CloneNotSupportedException();  
} 

private void readObject(ObjectInputStream in) throws IOException,  
        ClassNotFoundException {  
            throw new InvalidObjectException("can't deserialize enum");  
}  
  
private void readObjectNoData() throws ObjectStreamException {  
        throw new InvalidObjectException("can't deserialize enum");  
}  
```

## 代码示例

在Spring中，bean可以被定义为两种模式：prototype（多例）和singleton（单例）
<br>
**singleton（单例）**：scope="singleton",只有一个共享的实例存在，所有对这个bean的请求都会返回这个唯一的实例。
<br>
**prototype（多例）**：scope="prototype",对这个bean的每次请求都会创建一个新的bean实例。
<br>
Spring bean 默认是单例模式。

**实现代码：**

```java

private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);


protected Object getSingleton(String beanName, boolean allowEarlyReference) {
	Object singletonObject = this.singletonObjects.get(beanName);
	if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
		synchronized (this.singletonObjects) {
			singletonObject = this.earlySingletonObjects.get(beanName);
			if (singletonObject == null && allowEarlyReference) {
				ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
				if (singletonFactory != null) {
					singletonObject = singletonFactory.getObject();
					this.earlySingletonObjects.put(beanName, singletonObject);
					this.singletonFactories.remove(beanName);
				}
			}
		}
	}
	return (singletonObject != NULL_OBJECT ? singletonObject : null);
}
```

