# java8中的Streams处理系列之一



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



具体代码示例不在列举



### stream支持的各种操作

各种操作可大致分为2类

- filter，sorted ，map：这些可以被链接在一起可以组成pipeline
- colelct：结束pipeline并且返回一个结果



后续待处理





















*文章来源* 

[Processing Data with Java SE 8 Streams, Part 1]: https://www.oracle.com/technical-resources/articles/java/ma14-java-se-8-streams.html

