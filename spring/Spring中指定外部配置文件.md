# Spring中指定外部配置文件及命令行启动

首先声明，此处Spring特指Spring boot 2.x的项目。对应之前遇到的坑做个总结

## 前情回顾

命令行方式启动jar包并指定外部配置文件的时候，出现mybatis的classpath未找到，从而导致mapper.xml未被加载，导致项目启动的时候出现下面的错误：*org.apache.ibatis.binding.BindingException: Invalid bound statement (not found)* 

使用的启动命令`` java -jar xx.jar --spring.config.location=/config/application.properties --server.port=80``

#### 顺便提一下启动时设置端口的几种方式的优先级

启动命令应用参数 > 启动命令 JDK 参数 > 环境变量

```bash
SERVER_PORT=3344 java -Dserver.port=5566 -jar <path/to/my/jar> --server.port=7788
# 上述给定参数，生效的只会是--server.port=7788的应用选项参数
```

## 先说结论

```
在spring boot2.x之后，--spring.config.location的属性配置默认做了调整，会覆盖默认的配置，也就是说一旦指定了这个选项，项目中的各种application.properties文件如果不显示指定的话，将不会被加载。
```

## 解决方案1

```
// 将项目中用到的各种配置文件显示添加到此配置项后面
--spring.config.location=classpath:/application.properties,/config/application.properties,\
/mnt/config/application.properties
```

## 解决方案2（推荐）

```
为了适配spring boot 1.x系列上使用--spring.config.location，为了达到不会覆盖默认配置文件的目的，提供了另外一个属性
--spring.config.additional-location=/mnt/application.properties
这样就不会覆盖默认配置，仅仅是相同的key后者内容会覆盖前者内容，默认的配置还是可以生效
```



## 顺便梳理一下命令行启动时指定参数的几种方式

- --server.port=8088 ：spring boot启动时的应用参数等价于在application.properties中的server.port=8088

- -Dserver.port=8088 ：Set a system property 参数-D设置系统属性

这里有一个巨坑，提醒一下：

```
通过--传递选项参数时，需要写在执行jar包之后，否则会提示错误
通过-D传递系统参数时，务必放置在待执行的jar包之前，否则不会报错，但是也不会执行
```

- java -version :标准参数，所有jvm必须支持
- java -X : 非标准参数，默认jvm实现，但并非所有jvm实现都满足
- java -XX : 非stable参数，各个参数实现不同，可能随时会取消
  - -Xms512m : jvm堆内存初始大小512M
  - -Xmx512m: jvm最大堆内存512M
  - -Xmn200m:设置年轻代大小200M。

参考文章：

- [当我指定 spring.config.location 时，Spring Boot 2.x 不扫描 application.properties](https://www.coder.work/article/1795285)

- [Stack Overflow springboot不加载application.properties案例](https://stackoverflow.com/questions/51888310/spring-boot-2-x-doesnt-scan-application-properties-when-i-specify-spring-config)

- [Spring官方外部配置解释](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)

- [Spring Boot启动参数详解](https://www.choupangxia.com/2019/12/22/spring-boot-arguments/)

- [Spring boot修改启动端口的4种方式](https://segmentfault.com/a/1190000023361229)

  





