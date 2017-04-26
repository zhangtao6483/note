## 函数式接口

### 1. Predicate
使用 java.util.function.Predicate 函数式接口以及lambda表达式，可以向API方法添加逻辑，用更少的代码支持更多的动态行为。<br>
Predicate接口非常适用于做过滤。

```java

	public static void main(String[] args) {
		List languages = Arrays.asList("Java", "Scala", "C++", "Haskell", "Lisp");

		System.out.println("Languages which starts with J :");
		filter(languages, (str) -> str.startsWith("J"));

		System.out.println("Print no language : ");
		filter(languages, (str) -> false);

		System.out.println("Print language whose length greater than 4:");
		filter(languages, (str) -> str.length() > 4);
	}

	public static void filter(List<String> names, Predicate<String> condition) {
		for (String name : names) {
			if (condition.test(name)) {
				System.out.println(name + " ");
			}
		}
	}

//	Languages which starts with J :
//	Java
//	Print no language :
//	Print language whose length greater than 4:
//	Scala 
//	Haskell
```

lambda表达式使用Predicate

```java
Predicate<String> startsWithJ = (n) -> n.startsWith("J");
		Predicate<String> fourLetterLong = (n) -> n.length() == 4;
		languages.stream()
				.filter(startsWithJ.and(fourLetterLong))
				.forEach((n) -> System.out.print("nName, which starts with 'J' and four letter long is : " + n));
				
//nName, which starts with 'J' and four letter long is : Java
```

## Lambda表达式

### 1. 用lambda表达式实现Runnable

使用() -> {}代码块替代了整个匿名类

```java
// Java 8之前：
new Thread(new Runnable() {
    @Override
    public void run() {
    System.out.println("Before Java8, too much code for too little to do");
    }
}).start();

//Java 8方式：
new Thread( () -> System.out.println("In Java8, Lambda expression rocks !!") ).start();

```

## Stream

并行流 stream <br>
串行流 parallelStream

![]()

## 
