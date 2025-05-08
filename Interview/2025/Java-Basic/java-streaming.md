# Java Stream API 详解

Java Stream API 是 Java 8 引入的一个强大的数据处理工具，它允许你以声明式方式处理数据集合（如集合、数组等），支持并行操作，提高了代码的可读性和简洁性。

## 一、Stream 概述

Stream 不是数据结构，而是一个来自数据源（如集合、数组）的元素序列，支持聚合操作：

- **不存储数据**：Stream 本身不存储元素，元素存储在底层集合或按需生成
- **函数式风格**：支持 lambda 表达式和方法引用
- **延迟执行**：许多 Stream 操作（如过滤、映射）可以延迟执行
- **可消费性**：Stream 只能被消费一次，类似迭代器

## 二、创建 Stream

### 1. 从集合创建

```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();       // 顺序流
Stream<String> parallelStream = list.parallelStream(); // 并行流
```

### 2. 从数组创建

```java
String[] array = {"a", "b", "c"};
Stream<String> stream = Arrays.stream(array);
```

### 3. 使用 Stream.of()

```java
Stream<String> stream = Stream.of("a", "b", "c");
```

### 4. 使用 Stream.generate() 或 Stream.iterate()

```java
// 生成无限流
Stream<String> generated = Stream.generate(() -> "element").limit(10);

// 迭代无限流
Stream<Integer> iterated = Stream.iterate(1, n -> n + 1).limit(5);
// 1, 2, 3, 4, 5
```

### 5. 基本类型流

```java
IntStream intStream = IntStream.range(1, 5);      // 1, 2, 3, 4
LongStream longStream = LongStream.rangeClosed(1, 5); // 1, 2, 3, 4, 5
```

## 三、中间操作 (Intermediate Operations)

中间操作返回一个新的 Stream，可以进行链式调用。

### 1. filter() - 过滤

```java
List<String> filtered = list.stream()
    .filter(s -> s.startsWith("a"))
    .collect(Collectors.toList());
```

### 2. map() - 映射/转换

```java
List<String> upperCase = list.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 3. flatMap() - 扁平化映射

```java
List<List<String>> listOfLists = Arrays.asList(
    Arrays.asList("a", "b"),
    Arrays.asList("c", "d")
);

List<String> flatList = listOfLists.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList());
// ["a", "b", "c", "d"]
```

### 4. distinct() - 去重

```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3);
List<Integer> distinct = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
// [1, 2, 3]
```

### 5. sorted() - 排序

```java
List<String> sorted = list.stream()
    .sorted()
    .collect(Collectors.toList());

// 自定义排序
List<String> customSorted = list.stream()
    .sorted((s1, s2) -> s2.compareTo(s1))
    .collect(Collectors.toList());
```

### 6. peek() - 调试/查看元素

```java
List<String> peeked = list.stream()
    .filter(s -> s.length() > 2)
    .peek(s -> System.out.println("Filtered: " + s))
    .map(String::toUpperCase)
    .peek(s -> System.out.println("Mapped: " + s))
    .collect(Collectors.toList());
```

### 7. limit() 和 skip() - 限制和跳过

```java
List<Integer> limited = IntStream.range(1, 10)
    .limit(5)
    .boxed()
    .collect(Collectors.toList());
// [1, 2, 3, 4, 5]

List<Integer> skipped = IntStream.range(1, 10)
    .skip(5)
    .boxed()
    .collect(Collectors.toList());
// [6, 7, 8, 9]
```

## 四、终端操作 (Terminal Operations)

终端操作会消耗 Stream，产生一个结果或副作用。

### 1. forEach() - 遍历

```java
list.stream().forEach(System.out::println);
```

### 2. collect() - 收集为集合

```java
List<String> collected = list.stream()
    .filter(s -> s.length() == 3)
    .collect(Collectors.toList());

// 其他收集方式
Set<String> set = list.stream().collect(Collectors.toSet());
Map<String, Integer> map = list.stream()
    .collect(Collectors.toMap(s -> s, String::length));
```

### 3. toArray() - 转为数组

```java
String[] array = list.stream().toArray(String[]::new);
```

### 4. reduce() - 归约

```java
Optional<String> reduced = list.stream()
    .reduce((s1, s2) -> s1 + "#" + s2);
// "a#b#c"

// 带初始值的reduce
Integer sum = Arrays.asList(1, 2, 3).stream()
    .reduce(0, (a, b) -> a + b);
// 6
```

### 5. min(), max(), count()

```java
Optional<String> min = list.stream()
    .min(Comparator.naturalOrder());

Optional<String> max = list.stream()
    .max(Comparator.reverseOrder());

long count = list.stream().count();
```

### 6. anyMatch(), allMatch(), noneMatch()

```java
boolean anyStartsWithA = list.stream()
    .anyMatch(s -> s.startsWith("a"));

boolean allStartsWithA = list.stream()
    .allMatch(s -> s.startsWith("a"));

boolean noneStartsWithZ = list.stream()
    .noneMatch(s -> s.startsWith("z"));
```

### 7. findFirst(), findAny()

```java
Optional<String> first = list.stream()
    .filter(s -> s.startsWith("a"))
    .findFirst();

Optional<String> any = list.parallelStream()
    .filter(s -> s.startsWith("a"))
    .findAny();
```

## 五、数值流特化

对于基本数值类型，Java 提供了特化的 Stream 类以避免装箱开销：

### 1. IntStream

```java
int sum = IntStream.range(1, 5)  // 1, 2, 3, 4
    .sum();  // 10

double avg = IntStream.of(1, 2, 3)
    .average()
    .orElse(0);
```

### 2. LongStream

```java
long sum = LongStream.rangeClosed(1, 100)
    .sum();  // 5050
```

### 3. DoubleStream

```java
double sum = DoubleStream.of(1.1, 2.2, 3.3)
    .sum();
```

## 六、并行流

Stream API 支持简单的并行处理：

```java
// 创建并行流
Stream<String> parallelStream = list.parallelStream();

// 将顺序流转为并行流
Stream<String> parallel = stream.parallel();

// 使用并行流计算
long count = list.parallelStream()
    .filter(s -> s.startsWith("a"))
    .count();
```

**注意事项**：
- 并行流使用 ForkJoinPool.commonPool()
- 不是所有操作都适合并行，某些操作（如 limit、findFirst）在并行时性能可能更差
- 确保操作是无状态的，不依赖顺序，且关联的

## 七、收集器 (Collectors)

Collectors 类提供了许多静态工厂方法，用于创建常见的收集器：

### 1. 转换为集合

```java
List<String> list = stream.collect(Collectors.toList());
Set<String> set = stream.collect(Collectors.toSet());
```

### 2. 连接字符串

```java
String joined = stream.collect(Collectors.joining());
String joinedWithDelimiter = stream.collect(Collectors.joining(", "));
String joinedWithPrefixSuffix = stream.collect(Collectors.joining(", ", "[", "]"));
```

### 3. 分组

```java
Map<Integer, List<String>> groupByLength = stream
    .collect(Collectors.groupingBy(String::length));

// 多级分组
Map<Integer, Map<Character, List<String>>> multiLevelGroup = stream
    .collect(Collectors.groupingBy(String::length, 
        Collectors.groupingBy(s -> s.charAt(0)));
```

### 4. 分区

```java
Map<Boolean, List<String>> partition = stream
    .collect(Collectors.partitioningBy(s -> s.length() > 3));
```

### 5. 统计

```java
DoubleSummaryStatistics stats = stream
    .collect(Collectors.summarizingDouble(String::length));

double average = stats.getAverage();
long count = stats.getCount();
double max = stats.getMax();
```

## 八、实战示例

### 示例1：处理用户列表

```java
List<User> users = Arrays.asList(
    new User("Alice", 25, Gender.FEMALE),
    new User("Bob", 30, Gender.MALE),
    new User("Charlie", 20, Gender.MALE),
    new User("Diana", 22, Gender.FEMALE)
);

// 获取所有男性用户的名称，按年龄排序
List<String> maleNames = users.stream()
    .filter(u -> u.getGender() == Gender.MALE)
    .sorted(Comparator.comparing(User::getAge))
    .map(User::getName)
    .collect(Collectors.toList());

// 按性别分组统计平均年龄
Map<Gender, Double> avgAgeByGender = users.stream()
    .collect(Collectors.groupingBy(
        User::getGender,
        Collectors.averagingInt(User::getAge)
    ));

// 查找年龄最大的用户
Optional<User> oldest = users.stream()
    .max(Comparator.comparing(User::getAge));
```

### 示例2：处理订单数据

```java
List<Order> orders = // 获取订单列表

// 计算每个客户的总消费金额
Map<String, Double> customerTotal = orders.stream()
    .collect(Collectors.groupingBy(
        Order::getCustomerId,
        Collectors.summingDouble(Order::getAmount)
    ));

// 找出消费金额最高的前5个客户
List<String> topCustomers = orders.stream()
    .collect(Collectors.groupingBy(
        Order::getCustomerId,
        Collectors.summingDouble(Order::getAmount)
    ))
    .entrySet().stream()
    .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
    .limit(5)
    .map(Map.Entry::getKey)
    .collect(Collectors.toList());
```

## 九、注意事项

1. **Stream 只能消费一次**：尝试重用已消费的 Stream 会抛出 IllegalStateException
2. **避免副作用**：Stream 操作应该是无状态的，不要在操作中修改外部状态
3. **性能考虑**：
   - 小数据集可能不需要 Stream
   - 某些操作（如 findFirst 在并行流中）可能比顺序流慢
4. **空指针安全**：使用 Optional 处理可能为 null 的结果
5. **调试困难**：由于链式调用，调试可能不如传统循环直观

## 十、总结

Java Stream API 提供了一种高效、声明式处理集合数据的方式，主要特点包括：

- 链式操作，代码简洁
- 支持并行处理，提高性能
- 丰富的中间和终端操作
- 强大的收集器框架
- 基本类型特化流避免装箱开销

合理使用 Stream API 可以显著提高代码的可读性和简洁性，特别适合复杂的数据处理场景。