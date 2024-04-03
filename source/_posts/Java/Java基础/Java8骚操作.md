---
title: Jave8骚操作
date: 2020-08-18 14:04:06
tags:
  - Java8
  - lambda表达式
  - Stream
categories:
  - Java
  - Java前沿技术
TODO: Date/TimeAPI完善
---

# Lambda表达式

## lambda表达式是什么

> **为什么需要Lambda表达式?**
>
> 在Java8之前，一个方法能接收的参数都是变量，如果你想传递一段代码（动作/方法）到另一个方法里，你需要构建一个属于某个类的对象，由它的某个方法来放置你想传递的代码块。这种方式很不方便


lambda表达式允许函数作为一个方法的参数。lambda表达式其实完成了定义接口并且实现接口里的方法这一功能，也可以认为lambda表达式代表一种动作，可以直接传递这种特殊的动作。

## lambda表达式的语法

`(parameters) -> expression`

`(parameters) ->{ statements; }` 

 `parameter -> expression`

`parameter ->{ statements; }`

- **可选类型声明：**不需要声明参数类型，编译器可以统一识别参数值。
- **可选的参数圆括号：**一个参数无需定义圆括号，但多个参数需要定义圆括号。
- **可选的大括号：**如果主体包含了一个语句，就不需要使用大括号。
- **可选的返回关键字：**如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。

  

## 有无lambda对比

```java
// 无lambda表达式版，为了开启新的线程创建了myThread
public static void main (String[] args) {    
	myThread t = new myThread(); 
} 
class myThread implements Runnable {    
    @Override 
    public void run () {
        System.out.println("放入你想执行的代码");    
    }
}

// lambda表达式版，无需创建新的类
public static void main(String[] args) {
    new Thread(() -> {
        for (int i = 0; i < 100; i++) {
            System.out.println("这是一个线程" + i);
        }
    }).start();
}
```

## 使用案例

### 使用lambda表达式对列表进行迭代

```java
// Java 8之前：
List features = Arrays.asList("Lambdas", "Default Method", "Stream API", "Date and Time API");
for (String feature : features) {
    System.out.println(feature);
}

// Java 8之后：
List features = Arrays.asList("Lambdas", "Default Method", "Stream API", "Date and Time API");
features.forEach(n -> System.out.println(n));
 
// 使用Java 8的方法引用更方便，方法引用由::双冒号操作符标示
features.forEach(System.out::println);
```

### 使用lambda表达式替代匿名内部类

```java
// Java 8之前：
new Thread(new Runnable() {
    @Override
    public void run() {
    System.out.println("Before Java8, too much code for too little to do");
    }
}).start();

//Java 8方式：使用()->{}替代了匿名内部类
new Thread( () -> System.out.println("In Java8, Lambda expression rocks !!") ).start();
```

### 使用lambda表达式的Map和Reduce示例

map允许将对象进行转换。例如在本例中，我们将 costBeforeTax 列表的每个元素转换成为税后的值。将lambda表达式传到 map() 方法，后者将其应用到流中的每一个元素。然后用 forEach() 将列表元素打印出来。

```java
// 不使用lambda表达式为每个订单加上12%的税
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
for (Integer cost : costBeforeTax) {
    double price = cost + .12*cost;
    System.out.println(price);
}
 
// 使用lambda表达式
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
costBeforeTax.stream().map((cost) -> cost + .12*cost).forEach(System.out::println);


// 不使用lambda表达式为每个订单加上12%的税并计算总价
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
double total = 0;
for (Integer cost : costBeforeTax) {
    double price = cost + .12*cost;
    total = total + price;
}
System.out.println("Total : " + total);
 
// 使用lambda表达式
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
double bill = costBeforeTax.stream().map((cost) -> cost + .12*cost).reduce((sum, cost) -> sum + cost).get();
System.out.println("Total : " + bill);
```

### 复制不同的值，创建一个子列表

```java
// 用所有不同的数字创建一个正方形列表
List<Integer> numbers = Arrays.asList(9, 10, 3, 4, 7, 3, 4);
List<Integer> distinct = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
System.out.printf("Original List : %s,  Square Without duplicates : %s %n", numbers, distinct);
```

## 总结

总的来说，所有的lambda表达式都是延迟执行的，如果你希望立即执行一段代码，那就没必要使用lambda表达式了，延迟执行代码的原因有很多种：

- 在另一个线程中运行代码
- 多次运行代码
- 在某个算法的正确时间点上运行代码（如排序中的比较逻辑）
- 某些条件触发时运行代码（数据到达，接口回调等）

# 函数式接口

## 定义

函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。
Java中的函数式编程体现就是Lambda表达式，所以函数式接口就是可以适用于Lambda使用的接口。如果方法的参数是一个函数式接口， 我们可以使用 lambda表达式作为参数传递

函数式接口都用@FunctionalInterface注解进行标注了，当一个接口打上@FunctionalInterface注解之后就声明为一个函数式接口，这个接口中就只能有一个抽象方法，大于一个抽象方法就会报错。

**如何检测一个接口是不是函数式接口**
@Functionallnterface放在接口定义的上方:如果接口是函数式接口，编译通过;如果不是,编译失败

自己定义 函数式接口的时候，@Functionallnterface是可选的， 就算我不写这个注解，只要保证满足函数式接口定
义的条件，也照样是函数式接口。但是,建议加上该注解



## 常用的四个内置函数式接口

Java 8在java.util.function包下预定义了大量的函数式接口供我们使用

| 接口名    | 描述                                                         |                                                              |
| --------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Supplier  | **供给型接口**<br />无参数，返回任意泛型值<br />`T get()`    | `Supplier<String> supplier=()->{return "供给";}`<br />`System.out.println(supplier.get())` |
| Consumer  | **消费型接口**<br />一个参数，无返回值<br />`void accept (T t)` | `Consumer<String> consumer = x->System.out.println(x);`<br />`consumer.accept("消费")` |
| Predicate | **断言型接口**<br />用于做判断操作<br />一个参数，返回布尔值<br />`boolean test(T t)` | `Predicate<String> p = (x)->{return x.contains("a")};`<br />`System.out.println(p.test("hha"));` |
| Function  | **函数型接口**<br />一个参数，返回任意泛型值<br />`R apply(T t)` | `Function<String,Integer> f=(x)->{return x.length();}`<br />`System.out.println(f.apply("haha"))` |

# Stream

Java 8 中的 Stream 是对集合（Collection）对象功能的增强，它专注于对集合对象进行各种非常便利、高效的聚合操作（aggregate operation），或者大批量数据操作 (bulk data operation)

## Stream相关定义

Stream 不是集合元素，它不是数据结构并不保存数据，它是有关算法和计算的。它更像一个高级版本的Iterator，单向，不可往复，数据只能遍历一次，遍历过一次后即用尽了，就好比流水从面前流过，一去不复返。

> **Iterator**
>
> - 只能显式地一个一个遍历元素并对其执行某些操作。 
> - 只能命令式地、串行化操作
>
> **Stream**
>
> - 只要给出需要对其包含的元素执行什么操作，比如“获取每个字符串的首字母”等，Stream 会隐式地在内部进行遍历，做出相应的数据转换。
> - 能并行化操作，使用并行去遍历时，数据会被分成多个段，其中每一个都在不同的线程中处理，然后将结果一起输出。（依赖于 Java7 中引入的 Fork/Join 框架来拆分任务和加速处理过程）



#### 操作符

什么是操作符呢？操作符就是对数据进行的一种处理工作，一道加工程序；就好像工厂的工人对流水线上的产品进行一道加工程序一样。

## Stream的使用

当我们使用一个流的时候，通常包括三个基本步骤：

1. 获取一个数据源（source）
2. 数据转换
3. 执行操作获取想要的结果

每次转换原有 Stream 对象不改变，返回一个新的 Stream 对象（可以有多次转换），这就允许对其操作可以像链条一样排列，变成一个管道

## 生成Stream的方式

- 从Collection 和数组生成
    `Collection.stream()`
    `Collection.parallelStream()`
    `Arrays.stream(T array)`
    `Stream.of(T t)`
- 从 BufferedReader
    `java.io.BufferedReader.lines()`
- 静态工厂
    `java.util.stream.IntStream.range()`
    `java.nio.file.Files.walk()`
- 自己构建
    `java.util.Spliterator`
- 其它
    `Random.ints()`
    `BitSet.stream()`
    `Pattern.splitAsStream(java.lang.CharSequence)`
    `JarFile.stream()`

```java
// 1、从Collection 和数组生成
Stream stream = Stream.of("a", "b", "c");

String [] strArray = new String[] {"a", "b", "c"};
stream = Stream.of(strArray);
stream = Arrays.stream(strArray);

List<String> list = Arrays.asList(strArray);
stream = list.stream();

// 2、数值流的构造
// 对于基本数值型，目前有三种对应的包装类型 Stream：IntStream、LongStream、DoubleStream
IntStream.of(new int[]{1, 2, 3}).forEach(System.out::println);
IntStream.range(1, 3).forEach(System.out::println);

 // 3、流转换为其他数据结构
Stream<String> stream3 = Stream.of(new String[]{"1", "2", "3"});
String str = stream3.collect(Collectors.joining());
System.out.println(str);
```

## Stream的使用

### Stream的操作类型

- 中间操作(Intermediate Operation)：一个流可以后面跟随零个或多个 intermediate 操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。
- 终止操作(Terminal Operation)：一个流只能有一个 terminal 操作，当这个操作执行后，流就被使用“光”了，无法再被操作。所以这必定是流的最后一个操作。Terminal 操作的执行，才会真正开始流的遍历，并且会生成一个结果。
- 无状态：指元素的处理不受之前元素的影响；
- 有状态：指该操作只有拿到所有元素之后才能继续下去。
- 非短路操作：指必须处理所有元素才能得到最终结果；
- 短路操作：指遇到某些符合条件的元素就可以得到最终结果，如 A || B，只要A为true，则无需判断B的结果。

### 中间操作

| 操作                                                   | 详情                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| `map(mapToInt,mapToLong,mapToDouble) `                 | 转换操作符，把比如A->B，这里默认提供了转int，long，double的操作符。 |
| `flatmap(flatmapToInt,flatmapToLong,flatmapToDouble) ` | 拍平操作符，把 int[]{2,3,4} 拍平 变成 2，3，4 也就是从原来的一个数据变成了3个数据，这里默认提供了拍平成int,long,double的操作符。 |
| `limit(int n) `                                        | 限流操作符，用于获取指定数量的流，比如数据流中有10个 我只要出前3个就可以使用。 |
| `distint`                                              | 去重操作，对重复元素去重，底层使用了equals方法。             |
| `filter`                                               | 过滤操作，把不想要的数据过滤。                               |
| `peek`                                                 | 挑出操作，如果想对数据进行某些操作，如：读取、编辑修改等。   |
| `skip`                                                 | 跳过操作，跳过某些元素。                                     |
| `sorted(unordered)`                                    | 排序操作，对元素排序，前提是实现Comparable接口，当然也可以自定义比较器。 |



### 结束操作

| 操作                          | 详情                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| `collect`                     | 收集操作，将所有数据收集起来，这个操作非常重要，官方的提供的Collectors 提供了非常多收集器，可以说Stream 的核心在于Collectors |
| `count`                       | 统计操作，统计最终的数据个数                                 |
| `findFirst findAny`           | 查找操作，查找第一个、查找任何一个 返回的类型为Optional      |
| `noneMatch allMatch anyMatch` | 匹配操作，数据流中是否存在符合条件的元素 返回值为bool 值     |
| `min max`                     | 最值操作，需要自定义比较器，返回数据流中最大最小的值         |
| `reduce`                      | 规约操作，将整个数据流的值规约为一个值，count、min、max底层就是使用reduce |
| `forEach forEachOrdered`      | 遍历操作，这里就是对最终的数据进行消费了                     |
| `toArray`                     | 数组操作，将数据流的元素转换成数组                           |



### Stream的典型用法

```java
      // 4、流的典型用法
        // 1> map/flatMap
        // map 生成的是个 1:1 映射，每个输入元素，都按照规则转换成为另外一个元素
        Stream<String> stream4 = Stream.of(new String[]{"a", "b", "c"});
        stream4.map(String::toUpperCase).forEach(System.out::println);
        // 还有一些场景，是一对多映射关系的，这时需要 flatMap
        Stream<List<Integer>> inputStream = Stream.of(
                Arrays.asList(1),
                Arrays.asList(2, 3),
                Arrays.asList(4, 5, 6)
        );
//        Stream<Integer> mapStream = inputStream.map(List::size);
//        mapStream.forEach(System.out::println);
        Stream<Integer> flatMapStream = inputStream.flatMap(Collection::stream);
        flatMapStream.forEach(System.out::println);

        // 2> filter
        // filter 对原始 Stream 进行某项测试，通过测试的元素被留下来生成一个新 Stream
        Integer[] nums = new Integer[]{1,2,3,4,5,6};
        Arrays.stream(nums).filter(n -> n<3).forEach(System.out::println);
 
        // 3> forEach
        // forEach 是 terminal 操作，因此它执行后，Stream 的元素就被“消费”掉了,无法对一个 Stream 进行两次terminal 运算
        Stream stream13 = Arrays.stream(nums);
        stream13.forEach(System.out::print);
//        stream13.forEach(System.out::print); // 上面forEach已经消费掉了，不能再调用
        System.out.println();
        // 具有相似功能的 intermediate 操作 peek 可以达到上述目的
        Stream stream14 = Arrays.stream(nums);
        stream14.peek(System.out::print)
			   .peek(System.out::print)
              	.collect(Collectors.toList());
        System.out.println();

        // 4> reduce 主要作用是把 Stream 元素组合起来,字符串拼接、数值的 sum、min、max、average 都是特殊的 reduce
        // Stream 的 sum 就相当于：
        Integer sum = Arrays.stream(nums).reduce(0, (integer, integer2) -> integer + integer2);
        System.out.println(sum);
        // 有初始值
        Integer sum1 = Arrays.stream(nums).reduce(0, Integer::sum);
        // 无初始值
        Integer sum2 = Arrays.stream(nums).reduce(Integer::sum).get();

        // 5> limit/skip
        // limit 返回 Stream 的前面 n 个元素；skip 则是扔掉前 n 个元素。
        Arrays.stream(nums).limit(3).forEach(System.out::print);
        System.out.println();
        Arrays.stream(nums).skip(2).forEach(System.out::print);
        System.out.println();

        // 6> sorted
//        对 Stream 的排序通过 sorted 进行，它比数组的排序更强之处在于你可以首先对 Stream 进行各类 map、filter、
//        limit、skip 甚至 distinct 来减少元素数量后，再排序，这能帮助程序明显缩短执行时间。
        Arrays.stream(nums).sorted((i1, i2) -> i2.compareTo(i1)).limit(3).forEach(System.out::print);
        System.out.println();
        Arrays.stream(nums).sorted((i1, i2) -> i2.compareTo(i1)).skip(2).forEach(System.out::print);
        System.out.println();

        // 7> min/max/distinct
        System.out.println(Arrays.stream(nums).min(Comparator.naturalOrder()).get());
        System.out.println(Arrays.stream(nums).max(Comparator.naturalOrder()).get());
        Arrays.stream(nums).distinct().forEach(System.out::print);
        System.out.println();

        // 8> Match
//        Stream 有三个 match 方法，从语义上说：
//        allMatch：Stream 中全部元素符合传入的 predicate，返回 true
//        anyMatch：Stream 中只要有一个元素符合传入的 predicate，返回 true
//        noneMatch：Stream 中没有一个元素符合传入的 predicate，返回 true
//        它们都不是要遍历全部元素才能返回结果。例如 allMatch 只要一个元素不满足条件，就 skip 剩下的所有元素，返回 false。
        Integer[] nums1 = new Integer[]{1, 2, 2, 3, 4, 5, 5, 6};
        System.out.println(Arrays.stream(nums1).allMatch(integer -> integer < 7));
        System.out.println(Arrays.stream(nums1).anyMatch(integer -> integer < 2));
        System.out.println(Arrays.stream(nums1).noneMatch(integer -> integer < 0));
```

## Stream总结

1. 不是数据结构，它没有内部存储，它只是用操作管道从 source（数据结构、数组、generator function、IO channel）抓取数据
2. 它绝不修改自己所封装的底层数据结构的数据。例如 Stream 的 filter 操作会产生一个不包含被过滤元素的新 Stream，而不是从 source 删除那些元素
3. 所有 Stream 的操作必须以 lambda 表达式为参数
4. 惰性化，很多 Stream 操作是向后延迟的，一直到它弄清楚了最后需要多少数据才会开始，Intermediate操作永远是惰性化的
5. 当一个 Stream 是并行化的，就不需要再写多线程代码，所有对它的操作会自动并行进行的

# 接口的默认方法

在接口中用**default**修饰的方法称为**默认方法**。
接口中的默认方法一定要有默认实现（方法体），接口实现者可以继承它，也可以覆盖它。

```java
/**
 * 接口的默认方法
 * @param s
 * @return
 */
default String methodDefault(String s) {
    System.out.println(s);
    return "res--" + s;
}
```

在接口中用**static**修饰的方法称为**静态方法**。

```java
/**
 * 接口的静态方法
 * @param a
 * @param b
 * @return
 */
static String methodStatic(String a, String b) {
    return a + b;
}
```

# 方法引用

|              | 引用方式                                                     | 示例                                                         |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 引用方法     | `实例对象::实例方法名`<br />`类名::静态方法名` <br />`类名::实例方法名` | `Consumer<String> consumer1 = s -> System.out.println(s);`<br/>`Consumer<String> consumer2 = System.out::println;`<br/>`consumer2.accept("呵呵");` |
| 引用构造方法 | `类名::new`                                                  | `Function<Integer, StringBuffer> fun = n -> new StringBuffer(n);`<br/>`Function<Integer, StringBuffer> fun = StringBuffer::new;` |
| 引用数组     | `类型[]::new`                                                | `Function<Integer, int[]> fun = n -> new int[n];`<br/>`Function<Integer, int[]> fun1 = int[]::new;` |

# Optional

Optional实际上是个容器：它可以保存类型T的值，或者仅仅保存null。

Optional提供很多有用的方法，这样我们就不用显式进行空值检测。

## 创建Optional对象方法

1. `Optional.of(T value)`， 返回一个Optional对象，value不能为空，否则会出空指针异常
2. `Optional.ofNullable(T value)`， 返回一个Optional对象，value可以为空
3. `Optional.empty()`，代表空

## API

其他API:

1.  `optional.isPresent()`，是否存在值（不为空）
2.  `optional.ifPresent(Consumer<? super T> consumer)`, 如果存在值则执行consumer
3.  `optional.get()`，获取value
4.  `optional.orElse(T other)`，如果没值则返回other
5.  `optional.orElseGet(Supplier<? extends T> other)`，如果没值则执行other并返回
6.  `optional.orElseThrow(Supplier<? extends X> exceptionSupplier)`，如果没值则执行`exceptionSupplier`，并抛出异常

> 使用 `Optional` 时尽量不直接调用 `Optional.get() `方法, `Optional.isPresent()` 更应该被视为一个私有方法, 应依赖于其他像 `Optional.orElse()`, `Optional.orElseGet()`, `Optional.map() `等这样的方法。

高级API：

1.  `optional.map(Function<? super T, ? extends U> mapper)`，映射，映射规则由`function`指定，返回映射值的`Optional`，所以可以继续使用`Optional`的API。
2.  `optional.flatMap(Function<? super T, Optional< U > > mapper)`，同`map`类似，区别在于`map`中获取的返回值自动被`Optional`包装，`flatMap`中返回值保持不变,但入参必须是Optional类型。
3.  `optional.filter(Predicate<? super T> predicate)`，过滤，按predicate指定的规则进行过滤，不符合规则则返回`empty`，也可以继续使用`Optional`的API。

```java
/**
 * 防止空指针，使用Optional
 * @param person
 * @return
 */
public static String getPersonNameOptional(Person person) {
    // 使用Optional对person进行包装
    return Optional.ofNullable(person).map((per) -> per.getName()).orElse(null);
}

/**
 * 防止空指针，原有实现方法
 * @param person
 * @return
 */
public static String getPersonName(Person person) {
    // 防止空指针，进行非空判断
    if (person == null) {
        return null;
    }
    return person.getName();
}
```



# Date/Time API

Java 8通过发布新的Date-Time API (JSR 310)来进一步加强对日期与时间的处理。Joda-Time是一个可替换标准日期/时间处理且功能非常强大的Java API的诞生。Java 8新的Date-Time API (JSR 310)在很大程度上受到Joda-Time的影响，并且吸取了其精髓。

## LocalDate类

## LocalTime类

## ZoneDateTime类


## Clock类


## Duration类


# 其他特性

## CompletableFuture

当我们Javer说异步调用时，我们自然会想到Future，比如：

```java
public class FutureTest {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newCachedThreadPool();
        Future<Integer> result = executor.submit(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                int sum=0;
                System.out.println(Thread.currentThread().getName()+"正在计算...");
                for (int i=0; i<100; i++) {
                    sum = sum + i;
                }
                Thread.sleep(TimeUnit.SECONDS.toSeconds(3000));
                System.out.println(Thread.currentThread().getName()+"计算完了！");
                return sum;
            }
        });
        System.out.println(Thread.currentThread().getName()+"做其他事情...");
        System.out.println(Thread.currentThread().getName()+"计算结果：" + result.get());
        System.out.println(Thread.currentThread().getName()+"事情都做完了！");
        executor.shutdown();
    }
}
```

输出结果：

```
main做其他事情...
pool-1-thread-1正在计算...
pool-1-thread-1计算完了！
main计算结果：4950
main事情都做完了！
```

那么现在如果想实现异步计算完成之后，立马能拿到这个结果继续异步做其他事情呢？这个问题就是一个线程依赖另外一个线程，这个时候Future就不方便，我们来看一下CompletableFuture的实现：

```java
public class CompletableFutureTest {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(10);
        CompletableFuture result = CompletableFuture.supplyAsync(() -> {
            int sum=0;
            System.out.println(Thread.currentThread().getName()+"正在计算...");
            for (int i=0; i<100; i++) {
                sum = sum + i;
            }
            try {
                Thread.sleep(TimeUnit.SECONDS.toSeconds(3000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"计算完了！");
            return sum;
        }, executor).thenApplyAsync(sum -> {
            System.out.println(Thread.currentThread().getName()+"打印："+sum);
            return sum;
        }, executor);
        System.out.println(Thread.currentThread().getName()+"做其他事情...");
        try {
            System.out.println(Thread.currentThread().getName()+"计算结果：" + result.get());
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName()+"事情都做完了！");
        executor.shutdown();
    }
}
```

输出结果：

```
main做其他事情...
pool-1-thread-1正在计算...
pool-1-thread-1计算完了！
pool-1-thread-2打印：4950
main计算结果：4950
main事情都做完了！
```

只需要简单的使用`thenApplyAsync`就可以实现了。

## JVM的新特性

`PermGen`空间被移除了，取而代之的是`Metaspace（JEP 122）`

JVM选项 `-XX:PermSize`与`-XX:MaxPermSize`分别被`-XX:MetaSpaceSize`与`-XX:MaxMetaspaceSize`所代替。