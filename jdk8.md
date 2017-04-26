# jdk8新特性

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/jdk/jdk8_0.png)

## 函数式接口

函数式接口                   | 参数类型 | 返回类型 | 用途
-------------------------- | ------- | ------- | ------------- 
Consumer<T>  <br>消费型接口  |   T     |   void  | 对类型为T的对象应用操 作，包含方法: void accept(T t)
Supplier<T>  <br>供给型接口  |   无     |   T    | 返回类型为T的对象，包 含方法:T get()
Function<T, R>  <br>函数型接口  |   T     |   R  | 对类型为T的对象应用操 作，并返回结果。结果 是R类型的对象。包含方 法:R apply(T t)
Predicate<T>  <br>断定型接口  |   T     |   boolean  | 确定类型为T的对象是否 满足某约束，并返回 boolean 值。包含方法 boolean test(T t)

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

1. Stream不会存储元素
2. Stream不会改变源对象，会返回一个持有结果的新Stream
3. Stream操作是延迟执行的

Stream操作步骤：

- 创建Stream：一个数据源（集合、数组），获取一个流
- 中间操作：一个中间操作链，对数据源的数据进行处理
- 终止操作：一个终止操作，执行中间操作链，并产生结果

### 创建Stream

#### 由Collection创建流

- default Stream<E> stream() : 返回一个顺序流- default Stream<E> parallelStream() : 返回一个并行流

#### 由数组创建流

- static <T> Stream<T> stream(T[] array): 返回一个流

重载形式，能够处理对应基本类型的数组：

- public static IntStream stream(int[] array)- public static LongStream stream(long[] array)- public static DoubleStream stream(double[] array)

#### 由值创建流

public static<T> Stream<T> of(T... values) : 返回一个流

#### 由函数创建流：创建无限流

- 迭代<br>public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f)- 生成<br>public static<T> Stream<T> generate(Supplier<T> s)

### 中间操作

#### 筛选与切片

方法                |  描述
------------------- | -----filter(Predicate p) | 接收 Lambda ， 从流中排除某些元素。distinct()          | 筛选，通过流所生成元素的 hashCode() 和 equals() 去 除重复元素limit(long maxSize) | 截断流，使其元素不超过给定数量skip(long n)        | 跳过元素，返回一个扔掉了前 n 个元素的流。若流中元素 不足 n 个，则返回一个空流。与 limit(n) 互补

#### 映射

方法                             | 描述
------------------------------- | -----map(Function f)                 | 接收一个函数作为参数，该函数会被应用到每个元 素上，并将其映射成一个新的元素。mapToDouble(ToDoubleFunction f) | 接收一个函数作为参数，该函数会被应用到每个元 素上，产生一个新的 DoubleStream。mapToInt(ToIntFunction f)       | 接收一个函数作为参数，该函数会被应用到每个元 素上，产生一个新的 IntStream。mapToLong(ToLongFunction f)     | 接收一个函数作为参数，该函数会被应用到每个元 素上，产生一个新的 LongStream。flatMap(Function f)             | 接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流

#### 排序

方法                     | 描述
----------------------- | ----sorted()                | 产生一个新流，其中按自然顺序排序sorted(Comparator comp) | 产生一个新流，其中按比较器顺序排序

### 终止操作

#### 查找与匹配

方法                    | 描述
---------------------- | ----allMatch(Predicate p)  | 检查是否匹配所有元素anyMatch(Predicate p)  | 检查是否至少匹配一个元素noneMatch(Predicate p) | 检查是否没有匹配所有元素findFirst()            | 返回第一个元素findAny()              | 返回当前流中的任意元素
count()                | 返回流中元素总数max(Comparator c)      | 返回流中最大值min(Comparator c)      | 返回流中最小值forEach(Consumer c)    | 内部迭代(使用 Collection 接口需要用户去做迭 代，称为外部迭代。相反，Stream API 使用内部 迭代——它帮你把迭代做了)

#### 归约

方法                    | 描述
-------------------------------- | ----
reduce(T iden, BinaryOperator b) | 可以将流中元素反复结合起来，得到一个值。 返回 Treduce(BinaryOperator b)         | 可以将流中元素反复结合起来，得到一个值。 返回 Optional<T>

#### 收集

方法                    | 描述
-------------------------------- | ----collect(Collector c)             | 将流转换为其他形式。接收一个 Collector接口的 实现，用于给Stream中元素做汇总的方法


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

接口默认方法的”类优先”原则若一个接口中定义了一个默认方法，而另外一个父类或接口中又定义了一个同名的方法时- 选择父类中的方法。如果一个父类提供了具体的实现，那么接口中具有相同名称和参数的默认方法会被忽略。- 接口冲突。如果一个父接口提供一个默认方法，而另一个接口也提供了一个具有相同名称和参数列表的方法(不管方法是否是默认方法)，那么必须覆盖该方法来解决冲突

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
int year = date.getYear(); //2017
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