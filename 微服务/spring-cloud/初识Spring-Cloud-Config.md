## 初识Spring Cloud Config

### 前言说明

要想理解Spring Cloud的实现，必须一步步理解在Spring Could出现之前，先驱们已经做了哪些实现。

这些实现中有些是共性的，Spring中也会采用，有些没有解决的问题在Spring Config中得到了实现。

### Java原生配置支持

#### 原生API

##### 存取支持

- java.util.Properties
  - 支持对.properties文件解析
  - 支持xml格式配置文件解析：storeToXML(OutputStream os,String comment)

##### 类型转换支持

- java.lang.Long#getLong(java.lang.String, long)
- java.lang.Integer#getInteger(java.lang.String, java.lang.Integer)
- java.lang.Boolean#getBoolean

### Apache Commons Configuration相关包支持

- 读取配置源
- 合并配置源
- 定义顺序

> 提供完整的配置API，并且通过组合模式来组合多种配置来源，来达到统一配置目的。
>
> org.apache.commons.configuration2.CompositeConfiguration

### Netflix Archaius Configuration

> 扩展了apache commons configuration中的接口，实现了动态更新与写入



### Spring Framwork Environment抽象

#### 配置源

##### API方式

- 单一配置源  -org.springframework.core.env.PropertySource
- 多配置源 -org.springframework.core.env.PropertySources

##### 注解方式

- 单一配置源 -org.springframework.context.annotation.PropertySource
- 多配置源 -org.springframework.context.annotation.PropertySources

#### SpringProfile

...

#### 类型转换

- org.springframework.core.convert.converter.ConverterRegistry

- org.springframework.core.convert.ConversionService
- org.springframework.core.env.PropertyResolver

### Spring Cloud Configuration

- 如何控制读取远程配置在本地配置之前？

#### 1个带顺序的ApplicationContextListener

通过集成Order接口，并且通过最高优先级+N的方式，来指定不同属性源之间的相对顺序

#### 1个资源定位PropertySourceLocator

其他具体组件只要实现此接口

















