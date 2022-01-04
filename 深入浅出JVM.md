## 深入理解JVM

首先声明以下分析是以HotSpot虚拟机为说明对象，其他虚拟机产品会有差别

##### 内容大纲：首先列出需要额外关注的点基本涵盖了jvm绝大部分内容

- 64位虚拟机与32位虚拟

- 程序编译与代码优化
  - 编译器优化 C1
    - 了解javac源码（java写的前端编译器，将源码转换为字节码）
    - 各种语法糖
      - 泛型（字节码类型擦除），自动装箱拆箱
  - 运行期优化 C2
    - JIT编译器将字节码转换为机器码
      - 哪些是热点代码？①多次被调用的方法；②方法内多次循环执行的循环体
      - 热点代码的2种探测方式：①采样方式(定期采集栈顶执行方法)；②方法调用计数

- 虚拟机执行子系统
  
  - 类文件结构(主要记住几个关键的结构)
  - **虚拟机加载机制**
    - 记忆宫殿法则7个字：家(加载)宴(验证)准(准备)解(解析)初(初始化)使(使用)蟹(卸载)
  - 虚拟机字节码执行引擎
  
- 自动内存管理机制
  
  - 运行时数据区域
  - GC相关
    - 垃圾收集算法
    - **垃圾收集器GC**
    - 内存分配与回收策略
  
- 高效并发
  - **java内存模型**与线程
  - 线程安全与锁优化
  
- 监控与故障处理
  - jdk工具
    - jps
    - jstat
    - jinfo
    - jmap
    - jhat
    - jstack
    - javap -v 查看类的详细字节码
    - jConsole可视化
    - VisualVM可视化
  - 调优案例与实战



### 虚拟机执行子系统

#### class文件结构

class文件是一组以字节(8位)为基础的二进制流

class文件属于一种数据结构，类似于C中结构体。

类型只包含2种：无符号整数(u1,u2,u4,u8可以用来表示数字，索引引用，数值，utf编码的字符串)以及table类型(组合数据类型)，本质上整个class文件就是一个table。

<img src="/Users/songke/work/TecPorter/imges/class文件结构.png" style="zoom: 50%;" />

通过记忆宫殿记忆住class文件的格式以及顺序

```
一个会魔法的女人去一个魔法村探险，
她和老婆一样发音不标准把吃经常说成ci，
每天吃的是版本，而且是煮着吃的。
刚到这个村入口就看到个招牌上写着前面有多个温泉大池子，又往前走了2分钟，确实发现了好多咕嘟咕嘟冒气泡的温泉池子。
于是她就想走近去试一下这些池子的水，走近一看，池子前都有访问标记，禁止触摸。
于是继续往前走，又走了2分钟，说了一句，我当前是真的累啊！
可是周围没有歇脚地方，就继续走了2分钟，又说了句：真是超级累啊！
紧接着又走了2分钟，感觉口渴的不行，发现一个水房，上面画了好多个水龙头，进去一看一望无际的水龙头都在不停地流着水。
喝饱了后继续走了2分钟，看到有好多个田地的标记，往远处一看，一望无尽的田野尽收眼底。
正在看时，突然一个老爷爷老了，说他老子这个村子名叫小方。于是我问他你们村有几个叫小方的，他说有好多个，没一会儿所有的小方都被叫了过来。
没敢耽搁继续走，终于走到村子入口的大牌坊。
牌坊隐约看见有好多个题字署名，凑近一看各种名人的名字都在牌坊上，
更神奇的是自己的名字竟然清晰地写在最前面，女人边奇怪，边思考走进了这个神奇魔法村。

```



#### 常量表（各种常量表）

字面量：字符串或者被final修饰的变量

符号引用：编译时用到的信息，主要是字段方法类以及接口等的元信息；这里涉及动态连接，jvm运行时首先会获得符号引用，在类创建或者运行时会将符号引用转换为具体的内存地址。

#### 常量池（Constant_Value）

<img src="/Users/songke/work/TecPorter/imges/constant_value结构.png" style="zoom:80%;" />



#### 字段表

三种字符串

- 简单名称
- 完全限定名
- 描述符



#### 属性表集合

在class文件，字段表，方法表中都可以携带属性表集合，以便表示特殊场景下专有信息

不再严格要求顺序性，可以自定义属性

<img src="/Users/songke/work/TecPorter/imges/虚拟机规范定义的属性.png" style="zoom:50%;" />

##### code属性结构

<img src="/Users/songke/work/TecPorter/imges/code属性表结构.png" style="zoom:50%;" />



#### 类加载器

实现给定类的完全限定名找到其class二级制字节流并加载到内存的功能组件

##### 分类

- 启动类加载器bootstrap classloader（c++编写，是jvm的一部分，加载lib目录下jar包）
- 扩展类加载器ext classloader（java编写，加载ext目录中jar包）
- 应用程序类加载器applicacation classloader(系统类加载器，java编写，加载classpath目录下文件)



##### 类与类加载器关系：

类的比较首先比较是否是同一个类加载器加载的，如果不是同一个类加载的即便是相同类也是不相等的，类比较结果都是false

包括equals,instanceof ,isAssignableFrom等等



##### 双亲委派模式

- 并不是强制性的，仅仅是推荐的一种实现方式而已

- class伴随着类加载器一起具备了带有优先级的层次关系
- 保证java程序的稳定性，

##### 双亲委派模式破坏

1. 提供了`` protected Class<?> loadClass(String name, boolean resolve)``方法供子类可以实现，可以破坏破模式

2. 线程上下文类加载器thread context classloader：基础类调用用户类的代码

3. 为了实现热部署，热加载

   ```
   （典型的实现是OSGI：open servie gateway Initiative为java提供动态模块化的系统）
   每个模块都有属于自己的classpath和类加载器，模块之间通过包暴露和引入进行关联，每个模块有着自己独立的生命周期，我们可以动态地对模块进行加载、卸载、更新
   
   ```

   

#### 字节码执行引擎

##### 运行时数据栈帧结构

- 局部变量表，
- 操作数栈
- 动态连接：每个栈帧都包含一个执行运行时常量池中该栈帧所属方法的引用
- 返回地址

提高一个实际问题：SLOT变量虽然超出作用域但是无法得到有效的释放内存问题？？

主要是后续没有对此Slot再次使用，未被覆盖，当前gc roots依然有效，未被gc。

主动进行 变量=null的设置，有时候是有意义的，但是绝大多数情况下是没有必要的。







### 垃圾收集器

<img src="/Users/songke/work/TecPorter/imges/hotspot-jvm垃圾收集器.png" style="zoom:50%;" />



#### 判断对象已死？

- 引用计数法（java中并没有使用）
- GC Roots搜索法（主流语言都是使用此方法）
  - 哪些可能会作为GC Roots对象：
  - 通过GC Roots去tracing，不能被root引用的对象不可达，可以被回收

#### 垃圾收集算法

- 复制算法
- 标记清除（mark-sweep）
- 标记整理（mark-compact）

#### java中几种对象引用方式

- 强
- 软
- 弱
- 虚

#### hot spot虚拟机中垃圾收集算法的实现

```
查看当前使用的垃圾回收器 java -XX:+PrintCommandLineFlags -version
```

##### 新生代

- Serial
- ParNew
- Parallel Scavenge

##### 老年代

- Serial Old
- Parallel Old
- CMS
  - 缺点1：CMS收集器对CPU资源敏感
  - 缺点2：CMS收集器无法处理浮动垃圾(Floating Garage)，可能出现Concurrent Mode Failure失败导致另一次Full GC的产生
  - 缺点3：CMS收集器收集结束后可能产生大量空间碎片(标记-清除算法)
- G1 （驾驭一切的垃圾收集器，与cms最大的不同是可以指定预期的停顿时间）
  - -XX:MaxGCPauseMillis ：停顿时间，通过预测模型实现的，根据用户指定的停顿时间，选取一定数量的Region进行回收
  - -XX:G1HeapRegionSize: 指定G1 Region大小，新生代老年代的内存不是连续的，分割成独立的内存连续的Region

一文传递搞懂G1垃圾收集器：

[java hot spot G1 GC的关键技术 美团团队](https://tech.meituan.com/2016/09/23/g1.html)



总结G1相对于CMS的优点：

1. 有整理内存过程的垃圾收集器，不会产生很多碎片
2. STW的时间更可控，用户可以指定期望停顿时间



### 内存模型与高并发

#### 内存模型

针对8种操作必须满足的规则进行明确规定，保证工作内存和主内存之间共享变量并发访问，屏蔽硬件实现细节，使java程序在不同平台下实现一致的并发效果。

不同虚拟机可以有自己的具体不同实现。

8种操作分别如下：

- lock
- unlock
- read
- load
- assign
- use
- store
- write

#### 共享变量访问的三个特性(并发安全要处理的三个特性)

- 原子性：6种读写操作基本可以保证原子性(long double这种64位数据类型分2个指令 一般虚拟机实现都会将其实现为原子性)
- 可见性：一个线程中修改的变量能否对另一个线程立即可见
- 有序性：线程内部as-if-serial，多线程因为jit编译器和处理器指令重排序，代码无序执行

#### java中提供的并发关键字

- volatile：
  - 可见性保证：修改变量后立即刷新到主内存，读取变量强制从主内存读取；
  - 顺序性保证：添加内存屏障
  - 原子性：无法保证原子性
- synchronized：只有同步快中才会添加monitor字节码，方法上只会添加ACC_SYNCHRONIZED访问标志，虚拟再根据方法是静态还是实例来决定锁定当前this对象还是Class对象
  - 对于高版本的jdk，jvm已经对synchronized关键字进行了非常好的优化，其性能已经基本和JUC中的ReentryLock一致，所以在synchronize关键字能满足需求的情况下，直接使用原生synchronized是没有性能问题的。
  - 有序性：一个变量同一时刻只允许一个线程对其lock
  - 可见性：对一个变量执行unlock操作前必须同步回主内存(store,write操作)
  - 原子性：字节码层面monitorenter，monitorexit实现，最终是通过lock和unlock操作实现代码块的原子性。基本数据类型的原子性，上述6个操作本身就可以保证。一般指更大范围的原子性
- final ：构造函数中初始化后不运行被修改

#### happens-before原则

只要满足其规定的原则，天然保证顺序性，不会重排序

- 程序次序原则：一个线程内，程序代码顺序
- 管程锁定原则：monitor lock rule 同一个锁的unlock操作必须早于lock操作
- volatile变量原则：volatile variable rule ，对volatile变量写操作早于其后的读操作
- 线程启动原则：
- 线程终止原则
- 线程中断原则
- 对象终结原则：构造函数早于finalize方法
- 传递性：A->B B->C =》 A->C

### 线程的实现

java线程的实现是native的，是基于操作系统原生线程模型来实现的

线程的实现方式

- 内核线程：轻量级进程LWP，1：1线程模型，1个LWP对应1个KLT
- 用户线程：用户态多个UT



### 并发安全

并发安全的实现方式

- 互斥同步，阻塞同步 ，悲观锁：加锁
- 非阻塞同步，乐观锁：CAS操作，需要底层指令系统的支持
- 无同步



### 锁优化

实现基础是每个对象的对象头的mark word中存储了锁信息32bit

<img src="/Users/songke/work/TecPorter/imges/对象头-markword存储结构.png" alt="对象头markword存储结构" style="zoom:50%;" />





附件：Garbage Collection Root

```
A garbage collection root is an object that is accessible from outside the heap. The following reasons make an object a GC root:

System Class
Class loaded by bootstrap/system class loader. For example, everything from the rt.jar like java.util.* .

JNI Local
Local variable in native code, such as user defined JNI code or JVM internal code.

JNI Global
Global variable in native code, such as user defined JNI code or JVM internal code.

Thread Block
Object referred to from a currently active thread block.

Thread
A started, but not stopped, thread.

Busy Monitor
Everything that has called wait() or notify() or that is synchronized. For example, by calling synchronized(Object) or by entering a synchronized method. Static method means class, non-static method means object.

Java Local
Local variable. For example, input parameters or locally created objects of methods that are still in the stack of a thread.

Native Stack
In or out parameters in native code, such as user defined JNI code or JVM internal code. This is often the case as many methods have native parts and the objects handled as method parameters become GC roots. For example, parameters used for file/network I/O methods or reflection.

Finalizable
An object which is in a queue awaiting its finalizer to be run.

Unfinalized
An object which has a finalize method, but has not been finalized and is not yet on the finalizer queue.

Unreachable
An object which is unreachable from any other root, but has been marked as a root by MAT to retain objects which otherwise would not be included in the analysis.

Java Stack Frame
A Java stack frame, holding local variables. Only generated when the dump is parsed with the preference set to treat Java stack frames as objects.

Unknown
An object of unknown root type. Some dumps, such as IBM Portable Heap Dump files, do not have root information. For these dumps the MAT parser marks objects which are have no inbound references or are unreachable from any other root as roots of this type. This ensures that MAT retains all the objects in the dump.

```











