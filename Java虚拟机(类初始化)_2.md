# Java虚拟机(类初始化)_2

[toc]

---
@(Java)[Java]

#1 类初始化
虚拟机规范严格规定了有且只有四种情况必须立即对类进行初始化：

- 遇到new、getstatic、putstatic、invokestatic这四条字节码指令时，如果类还没有进行过初始化，则需要先触发其初始化。生成这四条指令最常见的Java代码场景是：使用new关键字实例化对象时、读取或设置一个类的静态字段（static）时（被static修饰又被final修饰的，已在编译期把结果放入常量池的静态字段除外）、以及调用一个类的静态方法时。
- 使用Java.lang.refect包的方法对类进行反射调用时，如果类还没有进行过初始化，则需要先触发其初始化。
- 当初始化一个类的时候，如果发现其父类还没有进行初始化，则需要先触发其父类的初始化。
- 当虚拟机启动时，用户需要指定一个要执行的主类，虚拟机会先执行该主类。

##1. 1 通过子类引用父类中的静态字段，这时对子类的引用为被动引用，因此不会初始化子类，只会初始化父类
```java
public class SuperClass {

	/**
	 * 通过子类引用父类的静态字段，不会导致子类初始化
	 */
	static {
		System.out.println("SuperClass init!");
	}

	public static int value = 123;

}

public class SubClass extends SuperClass {

	static {
		System.out.println("SubClass init!");
	}
}

public class NotInitalization {
	public static void main(String[] args) {
		System.out.println(SubClass.value);
	}
}

```

 **输出：SuperClass init!**

##1.2 常量在编译阶段会存入调用它的类的常量池中，本质上没有直接引用到定义该常量的类，因此不会触发定义常量的类的初始化
```java
public class ConstClass {
	
	static {
		System.out.println("ConstClass init!");
	}

	public static final String HELLOWORLD = "hello world!";
	
}

public class NotInitalization {
	public static void main(String[] args) {
	System.out.println(ConstClass.HELLOWORLD);
	}
}

```

**没有输出“ConstClass init!”**

##1.3 通过数组定义来引用类，不会触发类的初始化

```java
public class NotInitalization {
	public static void main(String[] args) {
			SuperClass[] sac = new SuperClass[10];
		
	}
}

```

**执行后不输出任何信息，说明Const类并没有被初始化。**