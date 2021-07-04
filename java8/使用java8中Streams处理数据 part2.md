# java8中的Streams处理系列之一

### 概要：组合各种操作表述复杂数据的表述（rich data）

### flatMap Operation

也是中间操作是map和flatten操作的组合

```
Strem<String[]> 转化为Stream<String>

```

这两个操作对于表达更复杂的查询时有用的。例如：你可以组合flatMap和collect来生成map，表述stream中的words的字符出现次数。

```java
import static java.util.function.Function.identity;
import static java.util.stream.Collectors.*;

Stream<String> words = Stream.of("Java", "Magazine", "is", 
     "the", "best");

Map<String, Long> letterToCount =
           words.map(w -> w.split(""))
                                   .flatMap(Arrays::stream)
                                   .collect(groupingBy(identity(), counting()));
```



我们需要的string流不是string数组流。有一个方法叫做Arrays.strem()入参一个数组产生一个stream。

```java
String[] arrayOfWords = {"Java", "Magazine"};
Stream<String> streamOfwords = Arrays.stream(arrayOfWords);
```

下面这个图例展示具体代码过程：

![](https://www.oracle.com/ocom/groups/public/@otn/documents/digitalasset/2228346.jpg)



结论：

flatMap使你可以用另一个Stream去替换一个Stream中的每个值，然后组装成一个新的Stream

flatMap是一种常用模式，在Optional以及CompletableFuture中也会用到。





### collect Operation

计算stream转化为一个合并结果

Collector：收集器

描述一种计算流元素转化为最终结果。

Collectors工厂方法中内置了多种收集器。

toList()

Collectors.toSet() : 取法指定返回set的类型

如要指定集合类型可使用toCollection(HashSet::new)，指定一个具体类的构造函数的方式

一些高级的Collectors

- 分组groupBy
- partition分区
- 生成复杂的分组



先来看些简单的内置collectors

- summarize/average取和平均值
- maxby/minby：直接获取stream中的最大最小值
- groupingBy：需要提供一个function（classification function标识函数）入参作为分组的key

![](https://www.oracle.com/ocom/groups/public/@otn/documents/digitalasset/2228348.jpg)



- partitionby：需要提供一个predicate作为入参，只是一种特殊的groupby
- 组合collectors：比如对groupby再使用count或者sum等

```java
// 从所有交易中先按找city进行分组再计算每个分组的最大值，最终返回一个map结构
Map<String, Optional<Transaction>> cityToHighestTransaction = 
           transactions.stream().collect(groupingBy(
             Transaction::getCity, maxBy(comparing(Transaction::getValue))));
```

提供了groupingby重载方法，入参1是function提取key，入参2是对于相同key中每个元素进行怎样的操作，又是一个collector。

- 复合groupby

```
Because groupingBy is a collector itself, we can create multilevel groupings by passing another groupingBy collector that defines a second criterion by which to classify the stream’s elements.
因为groupby也是一个collector，我们可以创建多层次的分组。通过传递另一个groupingby collector，此collector相当于是对每一个元素又应用了分类标准
```



![](https://www.oracle.com/ocom/groups/public/@otn/documents/digitalasset/2228349.jpg)

### 总结：

在本篇文章我们介绍了2个高级api，flatMap和colect。其中collect可以表达更丰富的数据查询。

另外你可以看到collect可以用来表达取和，分组，分区等。另外这些操作可以被组合起来，形成更复杂的查询。比如多层次的map，也就是多个维度进行分组。

本篇文章并不打算讲解所有的内置collector，前天内置collector比如mapping，joning  andthen等，可以去尝试一下，然后你会发现他们大有裨益。



对于java8中内置的收集器的说明可以参考文章：https://www.jianshu.com/p/afe6f60d492a



*文章来源* 

[Part 2: Processing Data with Java SE 8 Streams]: https://www.oracle.com/technical-resources/articles/java/architect-streams-pt2.html

[java8中的collectors](https://www.jianshu.com/p/afe6f60d492a)

