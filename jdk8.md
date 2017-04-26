# jdk8新特性

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/jdk/jdk8_0.png)

## 函数式接口

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/jdk/jdk8_2.png)

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

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/jdk/jdk8_1.png)

## 方法引用/构造函数引用

1. 引用静态方法 
<br>
ContainingClass::staticMethodName
<br>
例子: String::valueOf，对应的Lambda：(s) -> String.valueOf(s) 

```java
users.forEach(User::collide);
users.forEach(u -> User.collide(u));
```

2. 引用特定对象的实例方法 
<br>
containingObject::instanceMethodName 
<br>
例子: x::toString，对应的Lambda：() -> this.toString()

```java
final User police = CreateFactory.create(User::new);
users.forEach(police::follow);
```

3. 引用构造函数
<br>
ClassName::new 
<br>
例子: String::new，对应的Lambda：() -> new String() 

```java
User user = CreateFactory.create(User::new);
```

4. 引用特定类型的任意对象的实例方法 
<br>
ContainingType::methodName 
<br>
例子: String::toString，对应的Lambda：(s) -> s.toString()

```java
users.forEach(User::repair);
```
---

```java
public class User {
	private String name;

	private String age;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getAge() {
		return age;
	}

	public void setAge(String age) {
		this.age = age;
	}

	public static void collide(final User user) {
		System.out.println("Collided " + user.getName());
	}

	public void follow(final User another) {
		System.out.println("Following the " + another.getName());
	}

	public void repair() {
		System.out.println("Repaired " + this.toString());
	}
}
```
```java
import java.util.function.Supplier;

public interface CreateFactory {
	static <T> T create(final Supplier<T> supplier) {
		return supplier.get();
	}
}
``` 
```java
User user = CreateFactory.create(User::new);
user.setName("Neal");
user.setAge("20");
List<User> users = Arrays.asList(user);
```

## 默认方法

```java
public interface Iterable<T> {
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
}
```
forEach 使用了一个java.util.function.Consumer功能接口类型的参数，它使得我们可以传入一个lambda表达式或者一个方法引用:

```java
List<?> list = …
list.forEach(System.out::println);
```

## Optional

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/jdk/jdk8_3.png)

## CompletableFuture

## 新的时间和日期API

```java
//创建日期
LocalDate date = LocalDate.of(2017,1,21); //2017-01-21
int year = date.getYear() //2017
Month month = date.getMonth(); //JANUARY
int day = date.getDayOfMonth(); //21
DayOfWeek dow = date.getDayOfWeek(); //SATURDAY
int len = date.lengthOfMonth(); //31(days in January)
boolean leap = date.isLeapYear(); //false(not a leap year)

//时间的解析和格式化
LocalDate localDate = LocalDate.parse("2017-05-01");
LocalTime time = LocalTime.parse("13:45:00");

LocalDateTime now = LocalDateTime.now();
now.format(DateTimeFormatter.BASIC_ISO_DATE);

//合并日期和时间
LocalDateTime dt1 = LocalDateTime.of(2017, Month.JANUARY, 21, 18, 7);
LocalDateTime dt2 = LocalDateTime.of(localDate, time);
LocalDateTime dt3 = localDate.atTime(13,45,20);
LocalDateTime dt4 = localDate.atTime(time);
LocalDateTime dt5 = time.atDate(localDate);

//操作日期
LocalDate date1 = LocalDate.of(2014,3,18); //2014-3-18
LocalDate date2 = date1.plusWeeks(1); //2014-3-25
LocalDate date3 = date2.minusYears(3); //2011-3-25
LocalDate date4 = date3.plus(6, ChronoUnit.MONTHS); //2011-09-25
```

---

参考：<br>
http://www.importnew.com/22417.html<br>

http://listenzhangbin.com/post/2017/01/java8-learning-notes/<br>