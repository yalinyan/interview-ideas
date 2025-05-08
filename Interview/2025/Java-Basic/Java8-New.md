
# Java 8 新特性详解

Java 8 是 Java 语言发展史上的一个重要里程碑，引入了许多革命性的新特性，显著提升了开发效率和代码可读性。以下是 Java 8 主要新特性的详细解释：

## 1. Lambda 表达式

Lambda 表达式是 Java 8 最显著的特性，它允许将函数作为方法参数传递，使代码更加简洁。

**语法**：
```java
(parameters) -> expression
或
(parameters) -> { statements; }
```

**示例**：
```java
// 旧方式
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello World");
    }
};

// Lambda 表达式
Runnable r2 = () -> System.out.println("Hello World");
```

**特点**：
- 可替代匿名内部类
- 类型推断：编译器可以根据上下文推断参数类型
- 必须与函数式接口(Functional Interface)配合使用

## 2. 函数式接口(Functional Interface)

函数式接口是只包含一个抽象方法的接口，可以使用 `@FunctionalInterface` 注解标记。

**常见函数式接口**：

| 接口                | 参数 | 返回类型 | 描述                       |
|---------------------|------|----------|----------------------------|
| `Predicate<T>`      | T    | boolean  | 断言型接口                 |
| `Consumer<T>`       | T    | void     | 消费型接口                 |
| `Function<T,R>`     | T    | R        | 函数型接口                 |
| `Supplier<T>`       | 无   | T        | 供给型接口                 |
| `UnaryOperator<T>`  | T    | T        | 一元操作(继承Function)     |
| `BinaryOperator<T>` | (T,T)| T        | 二元操作(继承BiFunction)   |

**示例**：
```java
@FunctionalInterface
interface MyFunctionalInterface {
    void execute();
}
```

## 3. Stream API

Stream 是 Java 8 中处理集合的关键抽象概念，可以让你以声明式方式处理数据。

**特点**：
- 不是数据结构，不存储数据
- 不会修改源数据
- 惰性执行(终端操作时才执行)
- 可并行操作

**常用操作**：

| 操作类型 | 方法示例                              | 描述               |
|----------|---------------------------------------|--------------------|
| 创建     | `stream()`, `parallelStream()`        | 创建流             |
| 中间操作 | `filter()`, `map()`, `distinct()`     | 转换流             |
| 终端操作 | `forEach()`, `collect()`, `reduce()`  | 产生结果或副作用   |

**示例**：
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// 过滤并打印
names.stream()
     .filter(name -> name.startsWith("A"))
     .forEach(System.out::println);

// 转换为大写并收集到列表
List<String> upperNames = names.stream()
                              .map(String::toUpperCase)
                              .collect(Collectors.toList());
```

## 4. 方法引用

方法引用是 Lambda 表达式的简化写法，用于直接指向已有方法。

**四种形式**：
1. 静态方法引用：`ClassName::staticMethod`
2. 实例方法引用：`instance::instanceMethod`
3. 特定类型的任意对象方法引用：`ClassName::instanceMethod`
4. 构造器引用：`ClassName::new`

**示例**：
```java
// Lambda 表达式
Function<String, Integer> parser1 = s -> Integer.parseInt(s);

// 方法引用
Function<String, Integer> parser2 = Integer::parseInt;
```

## 5. 接口的默认方法和静态方法

Java 8 允许接口包含具体实现的方法。

**默认方法(Default Method)**：
```java
interface Vehicle {
    default void print() {
        System.out.println("我是一辆车!");
    }
}
```

**静态方法(Static Method)**：
```java
interface Vehicle {
    static void blowHorn() {
        System.out.println("按喇叭!!!");
    }
}
```

**特点**：
- 默认方法可以被实现类重写
- 静态方法只能通过接口名调用
- 主要目的是接口演化(向后兼容)

## 6. Optional 类

Optional 是一个容器对象，用于更优雅地处理可能为 null 的值。

**常用方法**：
- `Optional.of(T value)` - 创建非空 Optional
- `Optional.empty()` - 创建空 Optional
- `Optional.ofNullable(T value)` - 创建可能为空的 Optional
- `isPresent()` - 检查是否有值
- `get()` - 获取值(不安全)
- `orElse(T other)` - 有值则返回，否则返回 other
- `ifPresent(Consumer<? super T> consumer)` - 有值则执行 consumer

**示例**：
```java
Optional<String> optional = Optional.ofNullable(getName());

// 旧方式
if (optional.isPresent()) {
    System.out.println(optional.get());
}

// 新方式
optional.ifPresent(System.out::println);
```

## 7. 新的日期时间 API (java.time)

Java 8 引入了全新的日期时间 API，解决了旧 `java.util.Date` 和 `java.util.Calendar` 的问题。

**主要类**：
- `LocalDate` - 只包含日期
- `LocalTime` - 只包含时间
- `LocalDateTime` - 包含日期和时间
- `ZonedDateTime` - 包含时区的日期时间
- `Duration` - 时间段(基于时间)
- `Period` - 时间段(基于日期)
- `DateTimeFormatter` - 日期格式化

**示例**：
```java
// 获取当前日期
LocalDate today = LocalDate.now();

// 创建特定日期
LocalDate birthday = LocalDate.of(1990, Month.JANUARY, 1);

// 日期操作
LocalDate nextWeek = today.plusWeeks(1);

// 日期比较
boolean isAfter = today.isAfter(birthday);

// 格式化
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
String formattedDate = today.format(formatter);
```

## 8. Nashorn JavaScript 引擎

Java 8 引入了新的 JavaScript 引擎 Nashorn，替代了旧的 Rhino 引擎。

**示例**：
```java
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("nashorn");

// 执行 JavaScript 代码
engine.eval("print('Hello Nashorn!')");
```

## 9. 并行数组操作

Java 8 新增了 `Arrays.parallelSort()` 等并行数组操作方法。

**示例**：
```java
int[] numbers = {5, 3, 9, 1, 7};

// 并行排序
Arrays.parallelSort(numbers);
```

## 10. 其他改进

- **Base64 支持**：内置了 Base64 编码解码器
- **重复注解**：允许在同一声明处多次使用同一注解
- **类型注解**：注解可以用于任何类型而不仅仅是声明
- **改进的类型推断**：泛型类型推断能力增强

## 总结

Java 8 的新特性使得 Java 语言更加现代化，特别是 Lambda 表达式和 Stream API 的引入，使得函数式编程风格可以在 Java 中实现，大大提高了代码的简洁性和可读性。这些特性不仅改变了 Java 的编程方式，也为后续版本的发展奠定了基础。