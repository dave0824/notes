## 前言

本篇博客对 java 8 的流库进行一个总结

### 1. 从迭代到流

在处理集合时，我们通常会迭代遍历它的元素，并在每个元素上执行某项操作，列如假设我们想统计某本书的所有长单词数（单词长度大于10）：

```java
package com.dave.steams;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Arrays;
import java.util.List;

/**
 * @author dave
 * @date 2021/1/3 22:01
 * @description 使用流统计单词数
 */
public class CountLongWords {

    public static void main(String[] args) throws IOException {
        String contents = new String(Files.readAllBytes(
                Paths.get("G:/javaFileBooks/The Nightingale and the Rose.txt")
        ),StandardCharsets.UTF_8);
        List<String> words = Arrays.asList(contents.split("\\PL+"));

        long count = 0;
        // 迭代遍历单词长度大于10的单词数
        for (String word : words) {
            if (word.length() > 10){
                count ++;
            }
        }
        System.out.println(count);
    }
}
```

如果使用流的话，迭代部分的操作将是这样的：

```java
count = words.stream().filter(word -> word.length() > 10).count();
```

1. 流的操作比起迭代的操作更易于阅读，通过方法名就可以知道代码的操作时干啥的。
2. 循环迭代是指定操作的顺序的，流可以并行的计算，只要保证结果是正确的即可。
3. 流并不存储元素。这些元素可能存储在底层的集合中。
4. 流的操作不会修改数据源。列如，filter 方法不会从流中将元素移除，而是会重新生成一个新的流，其不包括被过滤了的流。
5. 流的操作尽可能使惰性执行的。这意味着直至需要结果时，操作才会执行，并得到想要的结果时就会停止过滤。

使用并行流进行统计长单词操作：

```java
count = words.parallelStream().filter(word -> word.length() > 10).count();
```

**操作流时的典型流程：**

1. 创建一个流
2. 指定将初始流转换为其它流的中间操作。
3. 应用终止操作，从而产生结果。这个操作会强制执行之前的惰性操作，从此之后，这个流就不能使用了。

### 2. 流的创建

Collection 接口的 stream 方法，所有的集合使用 stream() 就可以创建一个流，使用 parallelStream() 就可以创建一个并行流。

#### 2.1 Steam.of() 

```java
// 产生一个元素为其给定值的流，事实上是调用 Arrays.stream(values) 实现的
public static<T> Stream<T> of(T... values) {
        return Arrays.stream(values);
    }
```

#### 2.2 Arrays.stream()

```java
// 返回以指定数组为源的顺序Stream 
public static <T> Stream<T> stream(T[] array) {
        return stream(array, 0, array.length);
    }
```

#### 2.3 Array.steam(array, from, to)

```java
// 返回以指定数组的指定范围为源的顺序Stream 。
public static <T> Stream<T> stream(T[] array, int startInclusive, int endExclusive) {
        return StreamSupport.stream(spliterator(array, startInclusive, endExclusive), false);
    }
```

#### 2.4 Stream.empty()

```java
// 返回一个空的顺序Stream 。
public static<T> Stream<T> empty() {
        return StreamSupport.stream(Spliterators.<T>emptySpliterator(), false);
    }
```

#### 2.5 Stream.generate()

```java
// 返回无限顺序无序流，其中每个元素由提供的Supplier生成。 这适用于生成恒定流，随机元素流等。
public static<T> Stream<T> generate(Supplier<T> s) {
        Objects.requireNonNull(s);
        return StreamSupport.stream(
                new StreamSpliterators.InfiniteSupplyingSpliterator.OfRef<>(Long.MAX_VALUE, s), false);
    }
```

```java
// 生成1000个随机数
Stream.generate(Math::random).limit(1000).collect(Collectors.toList())
```

### 3. filter、map和flatMap 方法

#### 3.1 filter

filter 方法会引元是 Predicate<T>, 即从 T 到 Boolean 的函数。对流按照某种条件进行过滤。

```java
count = words.stream().filter(word -> word.length() > 10).count();
```

#### 3.2 map

map 方法为按某种方式来转化流中的值。

```java
// 将单词大写转换成小写
List<String> lowerCaseWords = words.parallelStream().map(String::toLowerCase).collect(Collectors.toList());
```

#### 3.3 flatMap

flatMap 方法将所有的流合并成一个流

```java
    public static void main(String[] args) {
        /**
         * 获取所有单词的字母集合
         * 1. map 方法获取到的是所有单词流的集合
         * 2. flatMap 方法获取到的是所有单词字母的集合
         */
        List<Stream<String>> result = words.stream().map(CountLongWords::letters).collect(Collectors.toList());
        System.out.println(result); 
        // 输出：[java.util.stream.ReferencePipeline$Head@b1bc7ed, java.util.stream.ReferencePipeline$Head@7cd84586, java.util.stream.ReferencePipeline$Head@30dae81,...]
        List<String> flatMapResult = words.stream().flatMap(CountLongWords::letters).collect(Collectors.toList());
        System.out.println(flatMapResult);
        //输出： [T, h, e, N, i, g, h, t, i, n, g, a, l, e, a, n, d, t, h, e, R, o, ...]
    }

	/**
     * 返回单词的字母流
     * @param s 单词
     * @return Stream<String>
     */
    private static Stream<String> letters(String s){
        ArrayList<String> result = new ArrayList<>();
        for (int i = 0; i < s.length(); i++) {
            result.add(s.substring(i,i+1));
        }
        return result.stream();
    }
```



### 4. 抽取子流和连接流

#### 4.1 stream.limit(n)

调用 stream.limit(n) 会返回一个新的流，它在 n 个元素之后结束（原来的流更短，那么就会在流结束时结束）。这个方法对裁剪无限流尺寸会特别有用，如：

```java
Stream<String> words = Stream.generate(Math::random).limit(1000);
```

#### 4.2 stream.skip(n)

stream.skip(n) 正好与 stream.limit(n) 方法相反，它是丢弃前 n 个元素。

#### 4.3 stream.concat(a,b)

将两个流连接起来，第一个流不能是无限流，否则第二个流将永远不能得到执行。



### 5. 其它的流转化

```java
// 产生一个流，包含当前流中所有不同元素
Stream<T> distinct();
// 产生一个流，它的元素是当前流中的所有元素按照顺序排序的
Stream<T> sorted();
Stream<T> sorted(Comparator<? super T> comparator);
// 产生一个流，它与当前流中的元素相同，在获取每个元素时，会将其传递给 action
Stream<T> peek(Consumer<? super T> action);
```



### 6. 简单约简

从流中获取结果的方法被称为约简，约简是一种终结操作，它们会将流约简为可以在程序中使用的非流值。前面的count 方法就是一种简单约简方法。

常用的简单约简方法:

```java
// 使用给定的比较强指定的排序规则返回流的最大元素
Optional<T> max(Comparator<? super T> comparator)
// 使用给定的比较强指定的排序规则返回流的最小元素
Optional<T> min(Comparator<? super T> comparator)
// 返回流的第一个元素
Optional<T> findFrist()
// 返回流的任意一个元素
Optional<T> findAny()

// 分别在这个流中的任意元素、所有元素和没有任何元素匹配给定断言时返回 true
boolean anyMatch(Predicate<? super T> predicate);
boolean allMatch(Predicate<? super T> predicate);
boolean noneMatch(Predicate<? super T> predicate);
```



### 7. 收集结果

使用迭代器将某个函数应用于每个流中

```java
// 输出流中的每个元素
stream.forEach(System.out::println);
// 将流转化为数组
String[] result = stream.toArray(String[]::new);
```

针对将流中的元素收集到另一个目标中，可以使用一个便捷的方法: collect , 它会接受一个 Collector 接口的实例。

Collectors 类提供了大量用于生成公共收集器的工厂方法。列如：

```java
// 将流收集到 list 集合中
List<String> result = stream.collect(Collectors.toList());
// 将流收集到 ser 集合中
Set<String> result = stream.collect(Collectors.toSet());
// 收集到指定集合中
TreeSet<String> result = stream.collect(Collectors.toCollection(TreeSet::new));
// 连接流中的所有元素生成字符串
String result = stream.collect(Collectors.joining());
// 连接流中的所有元素生成字符串,元素间用逗号分开
String result = stream.collect(Collectors.joining(','));
String result = stream.map(Object::toString)collect(Collectors.joining(','));

// 求流的总和，平均值，最大值，最小值
// 求所有单词长度的总和，平均值，最大值，最小值
IntSummaryStatistics summary = words.stream().collect(Collectors.summarizingInt(String::length));
long count = summary.getCount();
double average = summary.getAverage();
int max = summary.getMax();
int min = summary.getMin();
```

### 8. 收集到映射表中

假设我们有一个 Stream<Person>, 并且想要将其收集到一个映射表中（Map 中），就可以使用 Collectors.toMap 方法，列如：

```java
// 以 id 为 key,name 为 value
Map<Integer, String> collect = people.stream().collect(Collectors.toMap(Person::getId, Person::getName));
// 以 id 为键，元素本身为值
Map<Integer, Person> collect = people.stream().collect(Collectors.toMap(Person::getId, Function.identity()));
```

如果有多个元素有相同的键，那么就会存在冲突，抛出 IllegalStateException 对象。可以使用第三个引元函数来覆盖这种行为：

```java
// 存在相同的键时，取旧值，不管新值
Map<Integer, Person> collect2 = people.stream().collect(Collectors.toMap(Person::getId, Function.identity(),(existingValue, newValue) -> existingValue));

// 将键相同的名字放到一个 set 集合中，下面群组有更简便的方法
Map<Integer, Set<String>> collect = people.stream().collect(Collectors.toMap(Person::getId,
                p -> Collections.singleton(p.getName()),
                (existingValue, newValue) -> {
                    Set<String> union = new HashSet<>(existingValue);
                    union.addAll(newValue);
                    return union;
                }));
```



### 9. 群组和分区

```java
// 将键相同的名字放到一个 list 集合中
Map<Integer, List<Person>> collect1 = people.stream().collect(Collectors.groupingBy(Person::getId));
```

当分类函数是断言函数时（即返回 boolean 值的函数）时，流的元素可以分为两个列表：该函数返回 true 的元素和其它元素。在这种情况下使用 partitioningBy 比使用 groupingBy 更要高效，如下面代码将年龄大于20的分为一组，小于20的分为另一组：

```java
Map<Boolean, List<Person>> collect3 = people.stream().collect(Collectors.partitioningBy(p -> p.getAge() > 20));
        List<Person> greaterThanTwenty= collect3.get(true);
```



### 10.下游收集器

groupingBy 方法会产生一个映射表，它的每一个值都是一个列表。如果想要以某种方式来处理这些列表，就需要提供一个下游收集器。列如，如果想要获得集而不是列表，那么就可以使用 Collectors.toSet 收集器：

```java
Map<Integer, Set<Person>> collect4 = people.stream().collect(Collectors.groupingBy(Person::getId, Collectors.toSet()));

```

常用的下游收集器：

```java
// 1. counting 会产生收集到的元素的个数，如：
Map<Integer, Long> idCount = people.stream().collect(Collectors.groupingBy(Person::getId, Collectors.counting()));

// 2. summing(Int|Long|Double) 会接受一个函数引元，将该函数应用到下游元素中，并计算它们的和。如：
Map<Integer, Integer> ageSum = people.stream().collect(Collectors.groupingBy(Person::getId, Collectors.summingInt(Person::getAge)));

// 3. maxBy 和 minBy 会接受一个比较器，并产生下游元素中最大值和最小值。如：
Map<Integer, Optional<Person>> maxAge = people.stream().collect(Collectors.groupingBy(Person::getId, Collectors.maxBy(Comparator.comparing(Person::getAge))));

// 4. mapping 方法会产生将函数应用到下游结果的收集器，并将函数值传递给另一个收集器，列如：按照州将城市群组在一起。每个州的内部，我们生成各个城市的名字，并按照最大长度简约。
Map<Integer, Optional<Person>> stateToLongestCityName = cities.stream().collect(Collectors.groupingBy(City::getState, Collectors.mapping(City::getName, Collectors.maxBy(Comparator.comparing(Person::getAge))));
```



### 11. 并行流

使用 Collection.parallelStream() 方法可以从任何集合中获取并行流

流使得并行处理快操作变得很容易，这个过程几乎是自动的，但需要遵守一些规则:

1. 只要在终结方法执行时，流出于并行模式，那么所有的中间流操作都将被并行化。
2. 当流并行运行时，目标的返回结果与顺序执行时返回的结果相同。
3. 流应该可以被高效的分成若干个子部分。
4. 不要将所有流都转化为并行流。只有在对已经位于内存中的数据执行大量计算操作时，才应该使用并行流。
5. 传递给并行流的操作的任何函数都是可以安全地并行执行，远离易变状态

一段糟糕的无法完成的任务：对字符串流中的所有短单词计数：

```java
int[] shortWords = new int[12];
 words.parallelStream().forEach(s->{
     if (s.length() < 12) {shortWords[s.length()] ++;}}
 );              
```

这是一段很糟糕的代码，传递给 forEach 的函数会在多个并发线程中运行，每个都会更新共享的数组。如果多次运行这个程序，很可能每次的计数结果都不一样。

改进：

```java
Map<Integer, Long> shortWordCounts = words.parallelStream().filter(s -> s.length() < 12).collect(Collectors.groupingBy((String::length), Collectors.counting()));

```

