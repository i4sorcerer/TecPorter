## 康康《为什么说Java正在死去》

### 前言说明

原文《Why Java Is Dying》链接：https://betterprogramming.pub/why-java-is-dying-b02b5fd44db9

作者：Komal Venkatesh Ganesan

暂不清楚此人在国际上的地位及水平,只找到如下描述信息：

```text
Engineer — Software / AI / Electronics / Technology. 
In pursuit of fundamental understanding of elemental physics/science 

领英地址: LinkedIn: https://bit.ly/2DN8rfP
```



### 主要观点

- 专注于仪式或形式上的java和spring中的概念或语法（称下面这些都是无意义并且处理他们很痛苦）
  - Application Context
  - 错综复杂的bean injections，autowiring，POJO mappers
  - memory-hungry JVM 
  - 尤其是臭名昭著的classloader
- 臭名昭著的Spring
- 不满足KISS原则
- 大量注解的使用，搞不清执行流程
  - 举了个Lombok帮我们自动生成Getter和Setter以及Builder的例子。讲到如果我们根据Spring自动注入的知识去找代码中的Getter和Setter方法，会发现仅仅是一个@NoArgSConstructor注解，连贯性或一致性何在？(这个人简直就是顽固不化的老古董，去让他写机器码吧)
- java中傻瓜规则
  - 类名规定
  - 包名规定
  - 重点抨击了访问修饰符的无意义，无用性。并且引用了Python官方描述其抛弃访问修饰符的原因``` We’re all adults here``` 
- 微服务的广泛应用

观点完全站不住脚，因为使用微服务我们需要把复杂的业务逻辑分割的足够小，足够简单。

以至于我们不需要进行状态管理，不需要关心并发和多线程的噩梦。

所以，java中有这些提供这些高级的功能，就说java即将死去，用她自己的话说，简直是太silly了。













### 引申出问题

#### 什么是框架framework？什么是library？

#### 什么是KISS原则？

keep it simple，stupid











