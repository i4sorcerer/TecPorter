## arthas常用命令

### 共通基础部分

#### 条件表达式

```
// 方法执行前进行过滤
monitor -b -c 5 *.MathGame primeFactors "params[0] <= 2"
// 方法执行后进行过滤
monitor -c 5 *.MathGame primeFactors "params[0] <= 2"


```

#### OGNL表达式



### thread命令

- 查询当前所有线程情况

  ```
  thread
  ```

- 查询当前最忙的3个线程

  ```shell
  thread -n 3
  
  等同于使用下面原生命令：
  // 周到cpu最忙的线程ID
  top -Hp 8612
  // 返回8657
  // 打印线程ID当前的stack
  printf "%x\n" 8657
  // 返回2236
  jstack 8612|grep 2236  -A 30
  // 根据返回的结果就可以定位到具体错误的代码了
  "pool-12-thread-1" #157 prio=5 os_prio=0 tid=0x00007f7b80288800 nid=0x2236 waiting on condition [0x00007f7b527b4000]
     java.lang.Thread.State: WAITING (parking)
  	at sun.misc.Unsafe.park(Native Method)
  	- parking to wait for  <0x00000000f7e7a510> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
  	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
  	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
  	at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
  	at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
  	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
  	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
  	at java.lang.Thread.run(Thread.java:748)
  
  ```

- 找出当前阻塞其他线程的线程

  ```
  thread -b
  ```

### watch命令

方法执行数据观测

#### 可以watch范围

- 返回值
- 抛出异常
- 入参

#### 选项说明

| 参数名称            | 参数说明                                          |
| ------------------- | ------------------------------------------------- |
| *class-pattern*     | 类名表达式匹配                                    |
| *method-pattern*    | 方法名表达式匹配                                  |
| *express*           | 观察表达式，默认值：`{params, target, returnObj}` |
| *condition-express* | 条件表达式                                        |
| [b]                 | 在**方法调用之前**观察                            |
| [e]                 | 在**方法异常之后**观察                            |
| [s]                 | 在**方法返回之后**观察                            |
| [f]                 | 在**方法结束之后**(正常返回和异常返回)观察        |
| [E]                 | 开启正则表达式匹配，默认为通配符匹配              |
| [x:]                | 指定输出结果的属性遍历深度，默认为 1              |



### monitor命令

对匹配 `class-pattern`／`method-pattern`／`condition-express`的类、方法的调用进行监控。

-c 5 : 指定采样周期为5s，默认是120s

-b 选项：指定条件表达式在方法执行前进行筛选

### sc命令

- sc -d 具体class名 ：可以显示具体class对应的classloader 的hashcode

### mc命令

### jad命令

### retransform命令

### getstatic命令

对于简单的静态方法和静态字段可以直接通过此命令获取其内容

```
getstatic com.meizhilab.deepcap.rpasource.consts.GdbCsvColumnTypes GDB_CSV_COLUMN_TYPES -x 3

```

### ognl表达式

官方参考文档：https://github.com/alibaba/arthas/issues/11



非常强大的东西，可以线上执行任意代码，下面先简单看下apache官网的介绍

```
Basic OGNL expressions are very simple. 
The language has become quite rich with features, but you don't generally need to worry about the more complicated parts of the language: the simple cases have remained that way. 
What is a property? 
For example, to get at the name property of an object, the OGNL expression is simply name. To get at the text property of the object returned by the headline property, the OGNL expression is headline.text.
Roughly, an OGNL property is the same as a bean property, which means that a pair of get/set methods, or alternatively a field, defines a property (the full story is a bit more complicated, since properties differ for different kinds of objects; see below for a full explanation).
The fundamental unit of an OGNL expression is the navigation chain, usually just called "chain." The simplest chains consist of the following parts:

基础的ognl表达式是非常简单的。
ognl语言已经拥有非常丰富的特性，但是你大可不必担心它复杂的部分，简单的案例说明一下。
例如：获取对象的name属性，只要name就好;获取headline属性中的text属性，只要headline.text即可。
那么什么是属性呢？粗暴的讲，ognl属性和bean属性是一样的，通常指的是拥有一对儿get/set方法，或者一个字段
ognl表达式基础组成部分是导航链，通常直接成为链。最简单的chain由以下几部分组成：
```

#### OGNL基本组成部分

- Property names : 属性名

```
name
headline.text
```

- Method Calls ：方法调用

```
hashCode() to return the current object's hash code
```

- Array Indices：数组索引

```shell
listeners[0] to return the first of the current object's list of listeners
```



#### 链式传递

```
All OGNL expressions are evaluated in the context of a current object, and a chain simply uses the result of the previous link in the chain as the current object for the next one. You can extend a chain as long as you like. For example, this chain:

所有的表达式都是针对当前上下文对象进行evaluated，前一个link的结果是下一个link的上下文对象，并且你可以任意扩展你的链路
name.toCharArray()[0].numericValue.toString()

```

上述链路evaluate过程（这个过程仅仅适用于获取值不适应设定值）

- 提取name属性从出initial或者当前OGNL上下文的root对象
- 对提取出来的name属性string值调用toCharArray()方法，转换成字符数组
- 从字符数组结果中提取第一个字符
- 获取字符对象的numericValue属性值（字符对象，并且拥有一个getNumericValue()方法）
- 对获取的Integer对象调用toString()方法。最终的结果就是toString返回的值



#### 常量

ognl中常量包含以下几种

- 字符串常量: "测试"

- 字符常量 : ‘A’ 

- 数值文字：

  ```
  int long double类型，
  BigDecimal ：前缀b/B  
  BigInteger：前缀h/H
  ```

- 布尔值：true/false

- null



#### 属性引用



#### 变量引用

- #var ：在ognl表达式中是全局的
- #this ：在链路的任意点，代指当前的对象



#### 集合构建

##### Lists

```
// 判断name属性是否在list中存在，会自动创建一个list实例
name in {null,"title"}

```



##### native arrays（java本地数组）

```
// 这种方式和java中使用数组非常类似
new int[] {1,2,3} // 给定初始值
new int[3] // 给定初始size，初始值都为0
```



##### maps

```
// 创建一个map对象
#{“foo”:"foo value","bar":"bar value"} 
// 创建具体类型的map对象
#@java.util.LinkedHashMap@{ "foo" : "foo value", "bar" : "bar value" }


```



### 具体实战场景

- 实时查看某个方法的入参出参以及有无异常情况
- 线下修改并热部署代码
- 监控某个方法的执行情况
- 获取任意bean的成员变量
- 调用任意bean的成员方法
- 获取类的静态成员变量
- 调用类的静态成员方法



### 其他相关

- 下载arthas安装包并启动：

  ```shell
  curl -O https://arthas.aliyun.com/arthas-boot.jar
  java -jar arthas-boot.jar
  ```

  

### 参考链接

1. [ognl官方手册](https://arthas.aliyun.com/doc/ognl.html)
1. [活用OGNL表达式](https://arthas.aliyun.com/doc/advanced-use.html)
2. [apache commons-ognl](https://commons.apache.org/proper/commons-ognl/language-guide.html)
3. 















