# spring cloud服务调用与熔断



## Feign框架



## 服务调用几种方式

服务调用实现的几种方式，都是声明式注解的方式

| 实现框架                          | 使用场景           | 请求映射注解    |
| --------------------------------- | ------------------ | --------------- |
| Feign                             | 客户端             | @RequestLine()  |
| spring cloud OpenFeign(Feign扩展) | 客户端             | @RequestMapping |
| JAXRS标准1，2(Jersey,RestEasy)    | 客户端，服务端声明 |                 |
| spring web mvc                    | 服务端             | @RequestMapping |



> spring cloud openfeign 是Feign的可扩展，使用spring web mvc注解来声明客户端接口



## 使用spring cloud OpenFeign基本步骤

是通过java接口的方式来声明服务端请求元信息，也就是通过调用java接口来实现HTTP/REST通信

1. ### 添加依赖

   ``` org.springframework.org:spring-cloud-starter-openfeign```

2. ### 激活Feign客户端

   ```java
   @SpringBootApplication
   @EnableFeignClients
   public class BootStrapApplication{
     //....
   }
   ```

   

3. ### 定义Feign接口

```java
@FeignClient("stores")//服务名称
public interface StoreClient {
    @RequestMapping(method = RequestMethod.GET, value = "/stores")// 使用mvc注解定义服务请求元信息
    List<Store> getStores();

    @RequestMapping(method = RequestMethod.GET, value = "/stores")
    Page<Store> getStores(Pageable pageable);

    @RequestMapping(method = RequestMethod.POST, value = "/stores/{storeId}", consumes = "application/json")
    Store update(@PathVariable("storeId") Long storeId, Store store);
}
```



## 核心实现分析

### 注册所有标注@FeignClient注解的配置接口

核心源码```org.springframework.cloud.openfeign.FeignClientsRegistrar#registerFeignClients```

过程如下：

- 通过```ClassPathScanningCandidateComponentProvider```的扫描器获取指定basePackages下的所有带有@FeignClient注解的接口
- 如果有结果则返回AnnotatedBeanDefinitiion，其中@FeignClient注解的元信息保存在AnnotationMetadata对象中。
- 重新注册FeignClientFactoryBean的BeanDefinition

### @EnableFeignClients注解驱动



## Spring Cloud OpenFeign的大致流程

- Spring WEB MVC注解原信息解析
- 通过@FeignClient所生成的代理对象方法的调用来实现HTTP调用
- 通过SpringDecoder实现Response与接口返回类型的反序列化
- 负载均衡？？
- 失败重试？？



## 基础知识点准备汇总

- FactoryBean

- EnableXXX等的模块驱动

  - 三种实现方式

    - 实现org.springframework.context.annotation.ImportBeanDefinitionRegistrar接口的方式

    ```org.springframework.cloud.openfeign.FeignClientsRegistrar```

    - 实现org.springframework.context.annotation.ImportSelector接口

      ```java
      @Import(CacheConfigurationImportSelector.class)
      ```

    - @Configuration注解

- 动态代理

- 序列化（各种序列化方式的对照）

- 远程过程调用RPC
  - 协议
    - java RMI二进制协议
      - serializable接口
      - kryo
    - 文本协议
    - web service
      - xml约束的实现两种方式
        - DTD：数据类型定义，mybatis
        - XSD：schema方式-强类型
    - JOSN
  - 消息传递方式
    - 传统请求-响应方式
    - reactive的流的方式：消息也可以看做是一种流式响应
      - gRpc支持流的方式

- HATEOAS：Hypermedia as the engine of applicatin state



## spring cloud服务调用选择

- 服务注册与发现：Eureka，Zk，Consul，Nacos等
- 服务调用：OpenFeign（唯一选择）
- 负载均衡：Netflix Ribbon（唯一选择）





## 参考文档或链接

1. [Jersey官网](https://eclipse-ee4j.github.io/jersey/)
2. [原生Feign的Github地址](https://github.com/OpenFeign/feign)
3. 





- 

