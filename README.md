对于Java开发者来说，Java8的版本显然是一个具有里程碑意义的版本，蕴含了许多令人激动的新特性，如果能利用好这些新特性，能够大大提升我们的开发效率。Java8的函数式编程能够大大减少代码量和便于维护，同时，还有一些跟并发相关的功能。开发中常用到的新特性如下：

[接口的默认方法和静态方法](#1)

[函数式接口FunctionInterface与lambda表达式](#2)

[方法引用](#3)

[Stream](#4)

[Optional](#5)

[Date/time API的改进](#6)

[其他改进](#7)


 <span id="1"></span>


#  1. 接口的默认方法和静态方法


在Java8之前，接口中**只能**包含抽象方法。那么这有什么样弊端呢？比如，想再Collection接口中添加一个spliterator抽象方法，那么也就意味着之前所有实现Collection接口的实现类，都要重新实现spliterator这个方法才行。而接口的默认方法就是**为了解决接口的修改与接口实现类不兼容的问题，作为代码向前兼容的一个方法**。

那么如何在接口中定义一个默认方法呢？来看下JDK中Collection中如何定义spliterator方法的：

    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }

可以看到定义接口的默认方法是通过**default**关键字。因此，在Java8中接口能够包含抽象方法外还能够包含若干个默认方法（即有完整逻辑的实例方法）。


	public interface IAnimal {
	    default void breath(){
	        System.out.println("breath!");
	    };
	}


	public class DefaultMethodTest implements IAnimal {
	    public static void main(String[] args) {
	        DefaultMethodTest defaultMethod = new DefaultMethodTest();
	        defaultMethod.breath();
	    }
	
	}


	输出结果为：breath!

可以看出**IAnimal**接口中有由default定义的默认方法后，那么其实现类DefaultMethodTest也同样能够拥有实例方法**breath**。但是如果一个类继承多个接口，多个接口中有相同的方法就会产生冲突该如何解决？实际上默认方法的改进，使得java类能够拥有类似多继承的能力，即一个对象实例，将拥有多个接口的实例方法，自然而然也会存在方法重复冲突的问题。

下面来看一个例子：

	public interface IDonkey{
	    default void run() {
	        System.out.println("IDonkey run");
	    }
	}
	
	public interface IHorse {
	
	    default void run(){
	        System.out.println("Horse run");
	    }
	
	}
	
	public class DefaultMethodTest implements IDonkey,IHorse {
	    public static void main(String[] args) {
	        DefaultMethodTest defaultMethod = new DefaultMethodTest();
	        defaultMethod.breath();
	    }
	
	
	
	}

定义两个接口：IDonkey和IHorse，这两个接口中都有相同的run方法。DefaultMethodTest实现了这两个接口，由于这两个接口有相同的方法，因此就会产生冲突，不知道以哪个接口中的run方法为准，编译会出错：`inherits unrelated defaults for run.....`



> 解决方法

针对由默认方法引起的方法冲突问题，**只有通过重写冲突方法，并方法绑定的方式，指定以哪个接口中的默认方法为准**。

	public class DefaultMethodTest implements IAnimal,IDonkey,IHorse {
	    public static void main(String[] args) {
	        DefaultMethodTest defaultMethod = new DefaultMethodTest();
	        defaultMethod.run();
	    }
	
	    @Override
	    public void run() {
	        IHorse.super.run();
	    }
	}

DefaultMethodTest重写了run方法，并通过 `IHorse.super.run();`指定以IHorse中的run方法为准。

**静态方法**

在Java8中还有一个特性就是，接口中还可以声明静态方法，如下例：

	public interface IAnimal {
	    default void breath(){
	        System.out.println("breath!");
	    }
	    static void run(){}
	}


 <span id="2"></span>
# 2.函数式接口FunctionInterface与lambda表达式 #

> 函数式接口

Java8最大的变化是引入**了函数式思想，也就是说函数可以作为另一个函数的参数**。函数式接口，要求接口中**有且仅有一个抽象方法**，因此经常使用的Runnable，Callable接口就是典型的函数式接口。可以使用`@FunctionalInterface`注解，声明一个接口是函数式接口。如果一个接口满足函数式接口的定义，会默认转换成函数式接口。但是，最好是使用`@FunctionalInterface`注解显式声明。这是因为函数式接口比较脆弱，如果开发人员无意间新增了其他方法，就破坏了函数式接口的要求，如果使用注解`@FunctionalInterface`，开发人员就会知道当前接口是函数式接口，就不会无意间破坏该接口。下面举一个例子：

	@java.lang.FunctionalInterface
	public interface FunctionalInterface {
	    void handle();
	}

该接口只有一个抽象方法，并且使用注解显式声明。但是，函数式接口要求只有一个抽象方法却可以拥有若干个默认方法的（实例方法），比如下例：

	@java.lang.FunctionalInterface
	public interface FunctionalInterface {
	    void handle();
	
	    default void run() {
	        System.out.println("run");
	    }
	}

该接口中，除了有抽象方法handle外，还有默认方法（实例方法）run。另外，**任何被Object实现的方法都不能当做是抽象方法**。

> lambda表达式

lambda表达式是函数式编程的核心，lambda表达式即匿名函数，是一段没有函数名的函数体，可以作为参数直接传递给相关的调用者。lambda表达式极大的增加了Java语言的表达能力。lambda的语法结构为：


	(parameters) -> expression
	或
	(parameters) ->{ statements; }



- **可选类型声明**：不需要声明参数类型，编译器可以统一识别参数值。


- **可选的参数圆括号**：一个参数无需定义圆括号，但多个参数需要定义圆括号。


- **可选的大括号**：如果主体包含了一个语句，就不需要使用大括号。


- **可选的返回关键字**：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。

完整示例为（摘自[菜鸟](http://www.runoob.com/java/java8-lambda-expressions.html)）


	public class Java8Tester {
	   public static void main(String args[]){
	      Java8Tester tester = new Java8Tester();
	        
	      // 类型声明
	      MathOperation addition = (int a, int b) -> a + b;
	        
	      // 不用类型声明
	      MathOperation subtraction = (a, b) -> a - b;
	        
	      // 大括号中的返回语句
	      MathOperation multiplication = (int a, int b) -> { return a * b; };
	        
	      // 没有大括号及返回语句
	      MathOperation division = (int a, int b) -> a / b;
	        
	      System.out.println("10 + 5 = " + tester.operate(10, 5, addition));
	      System.out.println("10 - 5 = " + tester.operate(10, 5, subtraction));
	      System.out.println("10 x 5 = " + tester.operate(10, 5, multiplication));
	      System.out.println("10 / 5 = " + tester.operate(10, 5, division));
	        
	      // 不用括号
	      GreetingService greetService1 = message ->
	      System.out.println("Hello " + message);
	        
	      // 用括号
	      GreetingService greetService2 = (message) ->
	      System.out.println("Hello " + message);
	        
	      greetService1.sayMessage("Runoob");
	      greetService2.sayMessage("Google");
	   }
	    
	   interface MathOperation {
	      int operation(int a, int b);
	   }
	    
	   interface GreetingService {
	      void sayMessage(String message);
	   }
	    
	   private int operate(int a, int b, MathOperation mathOperation){
	      return mathOperation.operation(a, b);
	   }
	}

另外，lambda还可以访问外部局部变量，如下例所示：

	int adder = 5;
	Arrays.asList(1, 2, 3, 4, 5).forEach(e -> System.out.println(e + adder));

实际上在lambda中访问类的成员变量或者局部变量时，**会隐式转换成final类型变量**，所以上例实际上等价于：

	final int adder = 5;
	Arrays.asList(1, 2, 3, 4, 5).forEach(e -> System.out.println(e + adder));


<span id="3"></span>
# 3. 方法引用 #

方法引用是为了进一步简化lambda表达式，通过类**名或者实例名与方法名的组合来直接访问到类或者实例已经存在的方法或者构造方法**。方法引用使用**::**来定义，**::**的前半部分表示类名或者实例名，后半部分表示方法名，如果是构造方法就使用`NEW`来表示。

方法引用在Java8中使用方式相当灵活，总的来说，一共有以下几种形式：

- 静态方法引用：ClassName::methodName;
- 实例上的实例方法引用：instanceName::methodName;
- 超类上的实例方法引用：supper::methodName;
- 类的实例方法引用：ClassName:methodName;
- 构造方法引用Class:new;
- 数组构造方法引用::TypeName[]::new

下面来看一个例子：


	public class MethodReferenceTest {
	
	    public static void main(String[] args) {
	        ArrayList<Car> cars = new ArrayList<>();
	        for (int i = 0; i < 5; i++) {
	            Car car = Car.create(Car::new);
	            cars.add(car);
	        }
	        cars.forEach(Car::showCar);
	
	    }
	
	    @FunctionalInterface
	    interface Factory<T> {
	        T create();
	    }
	
	    static class Car {
	        public void showCar() {
	            System.out.println(this.toString());
	        }
	
	        public static Car create(Factory<Car> factory) {
	            return factory.create();
	        }
	    }
	}


	输出结果：
	
	learn.MethodReferenceTest$Car@769c9116
	learn.MethodReferenceTest$Car@6aceb1a5
	learn.MethodReferenceTest$Car@2d6d8735
	learn.MethodReferenceTest$Car@ba4d54
	learn.MethodReferenceTest$Car@12bc6874

在上面的例子中使用了`Car::new`，即通过构造方法的方法引用的方式进一步简化了lambda的表达式，`Car::showCar`，即表示实例方法引用。

<span id="4"></span>
# 4. Stream #
Java8中有一种新的数据处理方式，那就是流Stream，结合lambda表达式能够更加简洁高效的处理数据。Stream使用一种类似于SQL语句从数据库查询数据的直观方式，对数据进行如筛选、排序以及聚合等多种操作。

## 4.1 什么是流Stream ##

Stream是一个来自数据源的元素队列并支持聚合操作，更像是一个更高版本的Iterator,原始版本的Iterator，只能一个个遍历元素并完成相应操作。而使用Stream，只需要指定什么操作，如“过滤长度大于10的字符串”等操作，Stream会内部遍历并完成指定操作。

Stream中的元素在管道中经过中间操作（intermediate operation）的处理后，最后由最终操作（terminal operation）得到最终的结果。

- 数据源：是Stream的来源，可以是集合、数组、I/O channel等转换而成的Stream；
- 基本操作：类似于SQL语句一样的操作，比如filter,map,reduce,find,match,sort等操作。

当我们操作一个流时，实际上会包含这样的执行过程：

**获取数据源-->转换成Stream-->执行操作，返回一个新的Stream-->再以新的Stream继续执行操作--->直至最后操作输出最终结果**。


## 4.2 生成Stream的方式 ##

生成Stream的方式主要有这样几种：

1. 从接口Collection中和Arrays：
	
	- Collection.stream();
	- Collection.parallelStream(); //相较于串行流，并行流能够大大提升执行效率
	- Arrays.stream(T array);
2. Stream中的静态方法：
	
	- Stream.of()；
	- generate(Supplier<T> s);
	- iterate(T seed, UnaryOperator<T> f);
	- empty();

3. 其他方法
	- Random.ints()
	- BitSet.stream()
	- Pattern.splitAsStream(java.lang.CharSequence)
	- JarFile.stream()
	- BufferedReader.lines()

下面对前面常见的两种方式给出示例：

	public class StreamTest {
	
	
	    public static void main(String[] args) {
	        //1.使用Collection中的方法和Arrays
	        String[] strArr = new String[]{"a", "b", "c"};
	        List<String> list = Arrays.asList(strArr);
	        Stream<String> stream = list.stream();
	        Stream<String> stream1 = Arrays.stream(strArr);
	
	        //2. 使用Stream中提供的静态方法
	        Stream<String> stream2 = Stream.of(strArr);
	        Stream<Double> stream3 = Stream.generate(Math::random);
	        Stream<Object> stream4 = Stream.empty();
	        Stream.iterate(1, i -> i++);
	
	    }
	}	

## 4.3 Stream的操作 ##

常见的Stream操作有这样几种：

1. Intermediate（中间操作）:中间操作是指对流中数据元素做出相应转换或操作后依然返回为一个流Stream，仍然可以供下一次流操作使用。常用的有：map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip。
2. Termial（结束操作）：是指最终对Stream做出聚合操作，输出结果。


**中间操作**

> filter：对Stream中元素进行过滤

过滤元素为空的字符串：

	long count = stream.filter(str -> str.isEmpty()).count();

> map：对Stream中元素按照指定规则映射成另一个元素

将每一个元素都添加字符串“_map”

    stream.map(str -> str + "_map").forEach(System.out::println);

map方法是一对一的关系，将stream中的每一个元素按照映射规则成另外一个元素，而如果是一对多的关系的话就需要使用flatmap方法。

> concat：对流进行合并操作

concat方法将两个Stream连接在一起，合成一个Stream。若两个输入的Stream都时排序的，则新Stream也是排序的；若输入的Stream中任何一个是并行的，则新的Stream也是并行的；若关闭新的Stream时，原两个输入的Stream都将执行关闭处理。

	Stream.concat(Stream.of(1, 2, 3), Stream.of(4, 5, 6)).
		forEach(System.out::println);

> distinct：对流进行去重操作

去除流中重复的元素

	Stream<String> stream = Stream.of("a", "a", "b", "c");
	        stream.distinct().forEach(System.out::println);
	
	输出结果：
	a
	b
	c

> limit：限制流中元素的个数

截取流中前两个元素：

	Stream<String> stream = Stream.of("a", "a", "b", "c");
	        stream.limit(2).forEach(System.out::println);
	
	输出结果：
	a
	a


> skip：跳过流中前几个元素

丢掉流中前两个元素：

	Stream<String> stream = Stream.of("a", "a", "b", "c");
	        stream.skip(2).forEach(System.out::println);
	输出结果：
	b
	c

> peek：对流中每一个元素依次进行操作，类似于forEach操作

JDK中给出的例子：

	Stream.of("one", "two", "three", "four")
	            .filter(e -> e.length() > 3)
	            .peek(e -> System.out.println("Filtered value: " + e))
	            .map(String::toUpperCase)
	            .peek(e -> System.out.println("Mapped value: " + e))
	            .collect(Collectors.toList());
	输出结果：
	Filtered value: three
	Mapped value: THREE
	Filtered value: four
	Mapped value: FOUR

> sorted：对流中元素进行排序，可以通过sorted(Comparator<? super T> comparator)自定义比较规则

	Stream<Integer> stream = Stream.of(3, 2, 1);
	        stream.sorted(Integer::compareTo).forEach(System.out::println);
	输出结果：
	1
	2
	3


> match：检查流中元素是否匹配指定的匹配规则

Stream 有三个 match 方法，从语义上说：

- allMatch：Stream 中全部元素符合传入的 predicate，返回 true；
- anyMatch：Stream 中只要有一个元素符合传入的 predicate，返回 true；
- noneMatch：Stream 中没有一个元素符合传入的 predicate，返回 true。

如检查Stream中每个元素是否都大于5：

	Stream<Integer> stream = Stream.of(3, 2, 1);
	boolean match = stream.allMatch(integer -> integer > 5);
	System.out.println(match);
	输出结果：
	false

**结束操作**


[Collectors中常见归约方法总结](https://blog.csdn.net/u013291394/article/details/52662761)

[重构和定制收集器Collectors](https://blog.csdn.net/io_field/article/details/54971555)

> count：统计Stream中元素的个数

	long count = stream.filter(str -> str.isEmpty()).count();



> max/min：找出流中最大或者最小的元素


	Stream<Integer> stream = Stream.of(3, 2, 1);
	    System.out.println(stream.max(Integer::compareTo).get());
	
	输出结果：
	3

> forEach

forEach方法前面已经用了好多次，其用于遍历Stream中的所元素，避免了使用for循环，让代码更简洁，逻辑更清晰。

示例：

	Stream.of(5, 4, 3, 2, 1)
	    .sorted()
	    .forEach(System.out::println);
	    // 打印结果
	    // 1，2，3,4,5

> reduce

[Stream中的Reduce讲解](https://blog.csdn.net/IO_Field/article/details/54971679)


<span id="5"></span>
# 5. Optional #

为了解决空指针异常，在Java8之前需要使用if-else这样的语句去防止空指针异常，而在Java8就可以使用Optional来解决。Optional可以理解成一个数据容器，甚至可以封装null，并且如果值存在调用isPresent()方法会返回true。为了能够理解Optional。先来看一个例子：


	public class OptionalTest {
	
	
	    private String getUserName(User user) {
	        return user.getUserName();
	    }
	
	    class User {
	        private String userName;
	
	        public User(String userName) {
	            this.userName = userName;
	        }
	
	        public String getUserName() {
	            return userName;
	        }
	    }
	}


事实上，getUserName方法对输入参数并没有进行判断是否为null，因此，该方法是不安全的。如果在Java8之前，要避免可能存在的空指针异常的话就需要使用`if-else`进行逻辑处理，getUserName会改变如下：

	private String getUserName(User user) {
	    if (user != null) {
	        return user.getUserName();
	    }
	    return null;
	}


这是十分繁琐的一段代码。而如果使用Optional则会要精简很多：

	private String getUserName(User user) {
	    Optional<User> userOptional = Optional.ofNullable(user);
	    return userOptional.map(User::getUserName).orElse(null);
	}

Java8之前的if-else的逻辑判断，这是一种命令式编程的方式，而使用Optional更像是一种函数式编程，关注于最后的结果，而中间的处理过程交给JDK内部实现。

到现在，可以直观的知道Optional对避免空指针异常很有效，下面，对Optional的API进行归纳：


> 创建Optional

1. Optional.empty():通过静态工厂方法Optional.empty，创建一个空的Optional对象；
2. Optional<T> of(T value):如果value为null的话，立即抛出NullPointerException；
3. Optional<T> ofNullable(T value)：使用静态工厂方法Optional.ofNullable，你可以创建一个允许null值的Optional对象。

实例代码：

	//创建Optional
	Optional<Object> optional = Optional.empty();
	Optional<Object> optional1 = Optional.ofNullable(null);
	Optional<String> optional2 = Optional.of(null);


> 常用方法

	1.	boolean equals(Object obj)：判断其他对象是否等于 Optional；
	2. Optional<T> filter(Predicate<? super <T> predicate)：如果值存在，并且这个值匹配给定的 predicate，返回一个Optional用以描述这个值，否则返回一个空的Optional；
	3. <U> Optional<U> flatMap(Function<? super T,Optional<U>> mapper)：如果值存在，返回基于Optional包含的映射方法的值，否则返回一个空的Optional；
	4. T get()：如果在这个Optional中包含这个值，返回值，否则抛出异常：NoSuchElementException；
	5. int hashCode()：返回存在值的哈希码，如果值不存在 返回 0；
	6. void ifPresent(Consumer<? super T> consumer)：如果值存在则使用该值调用 consumer , 否则不做任何事情；
	7. boolean isPresent()：如果值存在则方法会返回true，否则返回 false；
	8. <U>Optional<U> map(Function<? super T,? extends U> mapper)：如果存在该值，提供的映射方法，如果返回非null，返回一个Optional描述结果；
	9. T orElse(T other)：如果存在该值，返回值， 否则返回 other；
	10. T orElseGet(Supplier<? extends T> other)：如果存在该值，返回值， 否则触发 other，并返回 other 调用的结果；
	11. <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier)：如果存在该值，返回包含的值，否则抛出由 Supplier 继承的异常；
	12. String toString()：返回一个Optional的非空字符串，用来调试


<span id="6"></span>
# 6. Date/time API的改进 #

在Java8之前的版本中，日期时间API存在很多的问题，比如：
- 线程安全问题：java.util.Date是非线程安全的，所有的日期类都是可变的；
- 设计很差：在java.util和java.sql的包中都有日期类，此外，用于格式化和解析的类在java.text包中也有定义。而每个包将其合并在一起，也是不合理的；
- 时区处理麻烦：日期类不提供国际化，没有时区支持，因此Java中引入了java.util.Calendar和Java.util.TimeZone类；


针对这些问题，Java8重新设计了日期时间相关的API，Java 8通过发布新的Date-Time API (JSR 310)来进一步加强对日期与时间的处理。在java.util.time包中常用的几个类有：


- 它通过指定一个时区，然后就可以获取到当前的时刻，日期与时间。Clock可以替换System.currentTimeMillis()与TimeZone.getDefault()
- Instant:一个instant对象表示时间轴上的一个时间点，Instant.now()方法会返回当前的瞬时点（格林威治时间）；
- Duration:用于表示两个瞬时点相差的时间量；
- LocalDate:一个带有年份，月份和天数的日期，可以使用静态方法now或者of方法进行创建；
- LocalTime:表示一天中的某个时间，同样可以使用now和of进行创建；
- LocalDateTime：兼有日期和时间；
- ZonedDateTime：通过设置时间的id来创建一个带时区的时间；
- DateTimeFormatter：日期格式化类，提供了多种预定义的标准格式；

示例代码如下：

	public class TimeTest {
	    public static void main(String[] args) {
	        Clock clock = Clock.systemUTC();
	        Instant instant = clock.instant();
	        System.out.println(instant.toString());
	
	        LocalDate localDate = LocalDate.now();
	        System.out.println(localDate.toString());
	
	        LocalTime localTime = LocalTime.now();
	        System.out.println(localTime.toString());
	
	        LocalDateTime localDateTime = LocalDateTime.now();
	        System.out.println(localDateTime.toString());
	
	        ZonedDateTime zonedDateTime = ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));
	        System.out.println(zonedDateTime.toString());
	    }
	}
	输出结果为：
	2018-04-14T12:50:27.437Z
	2018-04-14
	20:50:27.646
	2018-04-14T20:50:27.646
	2018-04-14T20:50:27.647+08:00[Asia/Shanghai]



<span id="7"></span>
# 7. 其他改进 #

Java8还在其他细节上也做出了改变，归纳如下：

1. 之前的版本，注解在同一个位置只能声明一次，而Java8版本中提供@Repeatable注解，来实现可重复注解；
2. String类中提供了join方法来完成字符串的拼接；
3. 在Arrays上提供了并行化处理数组的方式，比如利用Arrays类中的parallelSort可完成并行排序；
4. 在Java8中在并发应用层面上也是下足了功夫：（1）提供功能更强大的Future：CompletableFuture；（2）StampedLock可用来替代ReadWriteLock；（3）性能更优的原子类：：LongAdder,LongAccumulator以及DoubleAdder和DoubleAccumulator；
5. 编译器新增一些特性以及提供一些新的Java工具


> 参考资料

Stream的参考资料：

[Stream API讲解](https://blog.csdn.net/u010425776/article/details/52344425)

[Stream讲解系列文章](https://blog.csdn.net/io_field/article/details/54971761)

[对Stream的讲解很细致](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/ "https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/")

[Java8新特性之Stream API](https://blog.csdn.net/u013291394/article/details/52624966)

Optional的参考资料：

[Optional的API讲解很详细](http://www.importnew.com/6675.html)

[对Optional部分的讲解还不错，值得参考](http://www.importnew.com/26144.html)

Java8新特性的介绍：

[Java8新特性指南，很好的资料](http://www.importnew.com/11908.html)

[跟上 Java 8 – 你忽略了的新特性](http://www.importnew.com/26144.html)

[Java8新特性学习系列教程](http://www.runoob.com/java/java8-new-features.html)
