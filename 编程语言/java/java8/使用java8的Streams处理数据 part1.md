# java8中Streams处理系列之一



## 引言：stream用来表达复杂的数据处理过程

开篇当头一问：如果没有集合世界将会怎样？

答曰：几乎所有的java应用程序都会产生和处理集合。他们对于某些编程任务是基础的：分组以及处理数据。

例如：

场景1：你可能会创建一个银行交易集合来代表客户的对账单

场景2：你可能会遍历整个集合来找出客户总共花费多少钱

。。。。

但是，抛开他们的重要性来看，java中的集合操作远远没有达到使用完美的地步。

下面具体来分析为什么这样说：

1. 典型的处理数据的模式应当是SQL-Like的操作，比如：查找（查询集合中的最大值），分组（以杂货店对所有交易集合进行分组）。因为绝大多数的数据库都是支持类似的生命操作的，在sql中我们很容易写出如下操作声明：

   ```sql
   找出值最大交易
   select id,max(value) from transactions
   ```

   正如我们熟知的，sql中我们根本不需要实现如何从集合中找到最大值（比如，使用循环遍历并且记录最大值的方式）。我们只需要表达出来我们所希望的结果即可。这种很简单的想法意思是你不用关系J具体的实现，因为这对你来说也是困难的。

   再次发问：

   为什么我们在java的集合上不能做类似的事情呢（事实证明一切创造性动力的来源是怀疑）？

   为什么我我们要反复的一次次重复的实现对集合的巡护操作呢？

2. 我们如何高效的处理大量的数据呢？最直接的想法是利用多核cpu来并行处理从而加速处理过程。但是，对于大多数开发者来说，写一个好的并发代码，是困难的，而且极易出错的。



-----------------------------**大救星Java SE 8闪亮登场！！**-----------------------------

java API的设计者们正在密谋设计一种新的概念”Stream“ 能让我们以一种声明的方式来处理数据。

而且，streams能够利用多核架构的优势，然鹅你却不用操心任何一行多线程相关的code。

听起来是不是很棒？这就是本篇文章将要和你一起探索的内容。

在真正开始之前，先让我们通过离例子感受一下，java8 streams的魅力。

让我们假设一种需求：找到所有交易类型为grocery的交易，并且返回结果以交易值倒叙排列。

**java7写法**

```java
List<Transaction> groceryTransactions = new Arraylist<>();
for(Transaction t: transactions){
  if(t.getType() == Transaction.GROCERY){
    groceryTransactions.add(t);
  }
}
Collections.sort(groceryTransactions, new Comparator(){
  public int compare(Transaction t1, Transaction t2){
    return t2.getValue().compareTo(t1.getValue());
  }
});
List<Integer> transactionIds = new ArrayList<>();
for(Transaction t: groceryTransactions){
  transactionsIds.add(t.getId());
}
```



**Java8写法**

```java
List<Integer> transactionsIds = 
    transactions.stream()
                .filter(t -> t.getType() == Transaction.GROCERY)
.sorted(comparing(Transaction::getValue).reversed())
                .map(Transaction::getId)
                .collect(toList());
```



图1：下图展示了java8代码的执行过程。

1. 通过stream()方法从list数据中获取一个stream
2. 若干个支持的操作连接在一起来生成一个管道pipeline：filter,sorted,map,collect等。这些操作可以理解为在数据集上执行特定的查询操作

![](https://www.oracle.com/ocom/groups/public/@otn/documents/digitalasset/2179048.jpg)



对于如何并发化我们的代码，jav8的方式是非常easy的。只需要用pallelStream()替换stream()，内部实现会自动将你的查询分散到多个核心上，充分利用多核心架构的优势。

```java
List<Integer> transactionsIds = 
    transactions.parallelStream()
                .filter(t -> t.getType() == Transaction.GROCERY)
                .sorted(comparing(Transaction::getValue).reversed())
                .map(Transaction::getId)
                .collect(toList());
```



看了上面代码的写法，也许你会二丈和尚摸不着头脑，下一节将带你领略它是如何工作的。在此之前，您应当对下面内容已经很熟悉了

- lambda表达式
- 方法引用

如想巩固上述内容可以查看 Java Magazine中其他文章以及末尾列举的相关文章。

到现在为止，你可以把stream看成是表达高效的，类似sql的作用于集合数据之上的操作。另外，这些操作可以简洁的使用lambda表达式



## 正式开启Streams探索之旅

### 理解streams的定义

我们该如何定义一个stream呢？一个简单的定义是

 ```"a sequence of elements from a source that supports aggregate operations"```

让我们拆开来解释

- sequence of elements: stream提供了对特定类型的序列化的数据集的接口，并不实际存储元素，他们是实时按需计算的。
- source ：streams消费的数据来自数据提供源，比如：集合，数组，以及I/O资源等
- aggregate operations ：streams支持类似sql的操作以及来自函数编程的常用操作，比如：filter,map,reduce,find,match,sorted等

下面2个是根本区别与集合的

- Pipeline ：大多数的stream操作返回一个stream对象自身。这就可以使得多个操作被链接成更大的管道。这其中包含一些优化，比如：懒加载，短路等
- internal iteration：对比集合外部显示的迭代，steam是在内部迭代帮你屏蔽了迭代的过程

让我们用下面的图再来详细解释下上面的java8的代码示例执行过程。

![](https://www.oracle.com/ocom/groups/public/@otn/documents/digitalasset/2179051.jpg)

具体过程

1. 调用stream方法获得交易stream，数据源会被处理成元素序列传递到stream中
2. 执行了多个聚合操作，他们链接成pipeline，可以被当做是对源数据的一种查询操作
3. 直到调用collect，任何数据操作都不会执行。collect操作会处理上述pipeline来返回需要的结果。关于colect会在另外文章专门阐述，这里把colellect当做是一个将stream变换为聚合结果的形形色色的配方，toList指将stream转换为list



### 对比streams和collections

简而言之，collections是关于数据而streams是关于计算

举个生活中的例子：

一个存储在dvd中的电影，是一个集合，因为它存储整个数据结构。

当在internet上看同一个被流化的电影，这是stream。流视频播放器仅仅需要提前下载用户正在观看的数个帧的数据，这样你就可以在大部分流中的数据还没有被下载的时候，从头开始一步步享受这个完整的视觉盛宴（想想一下流化一个实况足球游戏）

粗略的来讲他们之间的区别，主要在于计算何时进行。

- 集合是内存数据结构，保存当前状态的完整数据集，每一个数据元素在加入集合之前就应该被执行过计算。
- stream是概念上一致的数据结构，其元素会被实时按需被计算



迭代的区别

- Collection接口需要用户自己去遍历，这种称为外部遍历
- streams使用内部遍历，帮你遍历并且保存遍历后的结果流，你仅仅需要提供一个方法告诉stream需要做什么。



具体代码示例不再列举



### stream支持的各种操作

各种操作可大致分为2类

#### Intermediate operations(中间操作)

stream中可以被连接在一起的操作称为中间操作，之所以能够连接在一起是以为他们的返回值都是steam。

filter，sorted ，map：这些可以被链接在一起可以组成pipeline

#### Terminal operations（结束操作）

关闭pipleline生成最终结果的操作，最终结果一般都不是steam



### 总结stream使用时3个步骤

- 用来执行query的源数据
- 用来组装pipeline的中间操作
- 用来执行pipeline并且产生结果的结束操作

### 具体使用示例

#### 过滤filtering

- filter(Predicate) 
- distinct 
- limit(n)
- skip(n)

#### 发现和匹配finding and matching（terminal operations）

常见的数据处理是判断某些元素是不是满足指定的属性

- findFirst：返回结果都是Optional对象

- findAny：返回结果都是Optional对象

  

- anyMatch：判断stream中元素是否满足给定的predicate

- allMatch

- noneMatch

#### 映射mapping

- map ：接收Function类型参数，目的是将元素转换成另一种类型。指定的Function作用于每一个元素，转换为另一种新元素

### Reducing

目前为止，我们使用的terminal operations都是要么返回boolean要么干活void要么是object。我们也可以使用collect来将stream中的元素组装成一个list

- reduce ：对于每一个元素重复的应用某一个操作，直到结果产生。这在函数编程中通常也被叫作fold 操作。

```
It’s often called a fold operation in functional programming because you can view this operation as “folding” repeatedly a long piece of paper (your stream) until it forms one little square, which is the result of the fold operation.

这里因为原文描述的特别形象，因此直接拿过来贴在这里
译文：这在函数编程中又被叫做折叠操作 ，打个比喻就好比你重复的折叠一长条纸（类似你的stream）直到它变成你想要的一个小方块，这就是折叠之后的结果。

```



场景：找出交易流水号最大的id？计算交易值的sum值？

```java

// 使用reduce实现
int product = numbers.stream().reduce(1,(a,b)->a*b);
int product = numbers.stream().reduce(1,Integer::Max());
```

#### 数值型streams

对于数值类型来说可以使用reduce方式进行计算，但是会出现大量的装箱操作，如果能使用sum方式是不是会更好的呢？

```java
int statement = 
    transactions.stream()
                .map(Transaction::getValue)
                .sum(); // error since Stream has no sum method
```



java8中针对基本类型专门定义了几种stream，IntStream，LongStream，DoubleStream，他们分别专门针对于元素为int，long和double的stream。

将一种流转变成另一程特殊的流，最可能常用的方法为mapToInt，mapToLong，mapToDouble。这些方法和之前使用的map是一致的，只是此处返回的是特定的stream而map返回的是stram<T>类型



最后，还有一种比较常用的方法叫做，数值范围。IntStram，LongStream、DoubleStream提供了2个静态方法，range和rangeClosed

range是开区间，rangeColsed是闭区间



#### 创建stream(building streams)

创建stream有很多中方式。前面已经说过从collectiong中获取stream的方式。同样的你也可以从数组文件中生成stream。

文件生活曾stream

```java
long numberOfLines = 
    Files.lines(Paths.get(“yourFile.txt”), Charset.defaultCharset())
         .count();
```



另外也可以生成无穷大stream

#### 无穷大streams(infinite stream)

我们可以通过函数窗创建无穷大stream，Stream.iterate Stream.generate方法，因为元素是按需被计算出来的，因此能够持续生成元素，这样的stream没有固定的大小，像前面我们从集合中创建的都是固定长度的流。

```java
// 所有的10的倍数的元素流
Stream<Integer> numbers = Stream.iterate(0, n -> n + 10);

// 通过limit将无穷大stream转换为固定场stream
numbers.limit(5).forEach(System.out::println); // 0, 10, 20, 30, 40
```

### 总结

java8中发布的stream api使我们可以表述复杂的数据查询过程。通过本文我们知道stream中支持的各种操作：filter，map，reduce 以及iterate，而这些方法可以相互组合来写出更高效的数据处理查询过程。这种新的书写方式和你之前操作集合的方式是大大的不同的。

好处也是显而易见的：

- stream使用懒技术和短路技术俩优化你的查询过程
- stream能够自动的并行化处理以此来重复利用计算机的多核心架构优势



下一节，我们将介绍一些更高级的操作，比如flatMap以及collect

别走开，下节更精彩。。。。







作者：

**Raoul-Gabriel Urma** 目前正在剑桥大学攻读计算机博士学位，研究程序语言。另外，他是*Java 8 in Action: Lambdas, Streams and Functional-style Programming*这本书的作者。



*文章来源* 

[Processing Data with Java SE 8 Streams, Part 1]: https://www.oracle.com/technical-resources/articles/java/ma14-java-se-8-streams.html

