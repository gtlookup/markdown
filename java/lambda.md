# 与 C# 对比

| C#             | java                                      |
| -------------- | ----------------------------------------- |
| Any            | anyMatch                                  |
| All            | allMatch                                  |
| where          | filter                                    |
| Select         | map                                       |
| SelectMany     | flatMap                                   |
| FirstOrDefault | filter(x -> ...).findFirst().orElse(null) |

# jdk9 新增

takeWhile dropWhile ofNullable iterate(新重载方法)

# 函数式接口定义

- 任何接口若只包含唯一一个抽象方法，那么它就是函数式接口

  ```java
  public interface Runnable { public abstract void run(); }
  ```
  
- 对于函数式接口我们可以通过lambda表达式来创建该接口的对象

- Lambda 简化过程

  ```java
  // 定义函数式接口
  interface IStep { void step();}
  // 1. 实现类
  class Step1 implements IStep {
      @Override
      public void step() { System.out.println("step 1"); }
  }
  public class LambdaStep {
      // 2. 静态内部类
      static class Step2 implements IStep {
          @Override
          public void step() { System.out.println("step 2"); }
      }
      public static void main(String[] args) {
          new Step1().step(); // 1
          new Step2().step(); // 2
  
          // 3. 局部内部类
          class Step3 implements IStep {
              @Override
              public void step() { System.out.println("step 3"); }
          }
          new Step3().step();
  
          // 4. 匿名内部类，没有类的名称，必须借助接口或者父类
          new IStep() {
              @Override
              public void step() { System.out.println("step 4"); }
          }.step();
  
          // 5. Lambda
          IStep lmb = () -> { System.out.println("step 5"); };
          lmb.step();
      }
  }
  ```




# 原生函数式接口

## Function<T, R>

- 一个入参，一个返回值

```java
//Function<T, R>
Function f = s -> s;
// 调用
f.apply("abc");
f.compose(b).apply("abc"); // 先 b 后 f
f.andThen(b).apply(1);     // 先 f 后 b
```

## BiFunction<T, U, R>

- 两个入参，一个返回值

```java
BiFunction<Integer, Integer, Boolean> f = (x, y) -> x > y;
f.apply(1, 2);  // false
```

## Predicate< T >

- 判定型。一个入参，返回bool值

```java
// Predicate<T>
Predicate<String> pred = s -> s.isEmpty();
// 判断字符串是否为空，结果：true
pred.test("");

// 如果 if1 不满足则不会走 if2
// 当 if12 都满足才返回true
Predicate<Integer> if1 = i -> {System.out.println("if1");return i == 1;};
Predicate<Integer> if2 = i -> {System.out.println("if2");return i == 2;};
System.out.println(if1.and(if2).test(2));
```

## Consumer< T >

- 消费型。一个入参，没有返回

```java
// Consumer<T>
Consumer c = s -> System.out.println(s);
// 调用，结果：haha
c.accept("haha");
```

## Supplier< T >

- 供给型。无参，只有一个返回值

```java
// Supplier<T>
Supplier<Integer> s = () -> 1000;
// 调用。结果：1000
s.get();
// 重点：将某个类的方法赋给代理用两个冒号（::）
Supplier<String> sl = String::new;
```



# 应用

## sorted

```java
// Comparator.reverseOrde 表示降序
list.stream().sorted(Comparator.comparing(类::属性一,Comparator.reverseOrder()));
// 先以 属性一 降序,再进行 属性二 降序
list.stream()
    .sorted(Comparator.comparing(类::属性一,Comparator.reverseOrder())
    .thenComparing(类::属性二,Comparator.reverseOrder()));
// List<Date> list = ...
list.stream()
    .sorted((x, y) -> x.compareTo(y))
    .map(x -> x.getYear() + "/" + x.getMonth() + "/" + x.getDate())
    .collect(Collectors.toList());
```

## boxed

```java
// 将LongStream、IntStream、DoubleStream转换成对应类型的Stream<T>
```

## collect

```java
// 收集stream元素,通过Collectors.toList()将stream转换为List
```

## map => Stream

```java
map.entrySet().stream();
```

## groupingBy

```java
// 简单分组
Map<String, List<String>> m = x.stream().collect(Collectors.groupingBy(x -> x.getxx()));

// *1: 
//    参数1：创建返回结果的方法（通常是 new）
//    参数2：将元素合并到结果集的方法
//    参数3：合并两个结果集的方法
// <R> R collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner);

// 取分组后每组元素大于1个的
List<String> list = Arrays.asList("a","b","c","c","d","e","e");
Map<String,List<String>> map = list.stream()
        .collect(Collectors.groupingBy(x -> x))
        .entrySet().stream()
        .filter(x -> x.getValue().size() > 1)
    	// 1. rlt = new HashMap();
    	// 2. m = new HashMap(); m.add();
    	// 3. rlt.putAll(m)
        .collect(HashMap::new, (m,v) -> m.put(v.getKey(),v.getValue()),HashMap::putAll); // *1
// 取分组后每组最后一个元素
list = list.stream().collect(Collectors.groupingBy(x -> x.getParentId()))
                .entrySet().stream()
                .map(x -> x.getValue().get(x.getValue().size() - 1))
    			// 1. rlt = new ArrayList();
    			// 2. l = new ArrayList(); l.add();
    			// 3. rlt.addAll(l);
                .collect(ArrayList::new, (l, v) -> l.add(v), ArrayList::addAll); // *1
```

## max

```java
// 取最大值
Stream.of(-2,-1,-4,-5,-3).max(Integer::max).get();      // 错的
Stream.of(-2,-1,-4,-5,-3).max(Integer::compare).get();  // 对的
```

## limit

```java
list.stream().limit(2); // 取前两个
```

## skip

```java
list.stream().skip(2);  // 丢弃前两个
```

## stream

```java
// 创建个空流
Stream sm = Stream.empty();
// 创建一个字符串集合流
Stream s1 = Stream.of("a", "b", "c", "d");
// 初值1，i为step，生成10个数
Stream.iterate(1, i -> i + 1).limit(10).forEach(System.out::println);
// 10 个 10
// generate：返回无限连续的流
Stream.generate(() -> 10).limit(10).forEach(System.out::println);
// 连接两个流
Stream.concat(
    Stream.generate(() -> 10).limit(10),
    Stream.iterate(1, i -> i + 1).limit(10)
).forEach(System.out::println);
```

```java
// 取集合的 index
List<String> list = Arrays.asList("a","b","c");
List<HashMap<Integer, String>> map = Stream
    .iterate(0, i -> i + 1).limit(list.size())
    .map(x -> new HashMap<Integer, String>() {{ put(x, list.get(x)); }})
    .collect(Collectors.toList());
```

## peek

是一个中间操作，类似 filter / map，只有遇到结果方法才执行

```java
// 生成1到10集合，peek 打印 x - 1，当 toList 后执行 peek
// 结果：0 到 9
Stream.iterate(1, i -> i + 1).limit(10).peek(x -> {System.out.println(x - 1);}).collect(Collectors.toList());
```

## Optional

对一个变量进行封装，防止出现空指针异常

```java
Optional<User> o = Optional.of(new A() {{setId("111");}});
String id = o.get().getId();
```

## reduce

- **Optional< T > reduce(BinaryOperator< T > accumulator)** 

  - 参数：组合两个值的代理方法。继承了 BiFunction<T,T,T>

  ```java
  Arrays.asList(1, 2, 3).stream().reduce((x, y) -> x + y).get();          // 求和
  Arrays.asList(1, 2, 3).stream().reduce((x, y) -> x >= y ? x : y).get(); // 最大值
  ```

- **T reduce(T identity, BinaryOperator< T > accumulator)**

  - 参数1：'参数2'的初值，当集合为空时返回这个值
  - 参数2：组合两个值的代理方法

  ```java
  Arrays.asList(1, 2, 3).stream().reduce(10, (x, y) -> x + y).intValue(); // 16
  // x = 10, y = 1
  // x = 10 + 1, y = 2
  // x = 10 + 1 + 2, y = 3
  ```

- **< U > U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator< U > combiner)**

  - 参数1：new 一个最终返回的集合实例
  - 参数2：合并元素的方法
  - 参数3：合并集合的方法

  ```java
  // 将 list 里的 奇数和偶数分别存到 map 0和1下标下
  List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6);
  Map<Integer, ArrayList<Integer>> map = list.stream().reduce(
      new HashMap<>(),
      // 一个 map 装偶数，一个 map 装奇数
      (m, v) -> {
          if (!m.containsKey(0)) m.put(0, new ArrayList<>());
          if (!m.containsKey(1)) m.put(1, new ArrayList<>());
          if ((v % 2) == 0) m.get(0).add(v);
          else m.get(1).add(v);
          return m;
      },
      // 将偶数和奇数 map 合并成一个 map
      (m1, m2) -> { m1.putAll(m2);return m1; }
  );
  ```


## toArray(xx[]::new)

```java
List<String> list = Arrays.asList("a", "b");
String[] ar = list.stream().toArray(String[]::new);
```

