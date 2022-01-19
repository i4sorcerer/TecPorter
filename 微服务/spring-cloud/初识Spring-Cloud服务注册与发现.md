# Spring Cloud服务注册与发现





## 服务发现与注册协议

### REST

HATEOS

### Webservice

UDDI 

都是属于IDL：interface description language



## 服务注册

- Spring cloud不支持多注册中心，折中解决方案可以通过spring.profiles.active的方式来进行切换。从而实现相同服务，不同实例使用不同的注册中心。

  - 其中涉及的问题：需要先默认把各个服务发现enable=false，然后再profiles中设置enable=true
  - 使用随机端口来进行多实例启动时问题：

  ```text
  在使用server.port=0来实现启动实例时使用随机端口的情况时，发现有bug，生成的客户端的port没有使用真实的端口，而是直接使用0作为端口号，然后url访问的时候会定位到真实的端口上。
  ```

- 

### Spring Cloud zookeeper：java客户端，C语言客户端 ZAB协议

《从PAXOS到Zookeeper》

### Spring Cloud Netflix Eureka ：

#### 到底什么是Eureka？

引述官方说明：包含2个部分

1. Eureka服务端

是基于REST服务的主要应用在云中进行服务定位，从而实现负载均衡和中间件层的故障转移failover

```text
Eureka is a REST (Representational State Transfer) based service that is primarily used in the AWS cloud for locating services for the purpose of load balancing and failover of middle-tier servers.
```

2. Eureka客户端

- 提供java-based的客户端从而简化与服务端的交互
- 提供客户端默认负载均衡：round-robin策略

#### Eureka 服务端和客户端如何通信？

下面引述官方文字描述：通信技术可以任意选择你喜欢的。Eureka帮你找到你想交流的服务的信息，但并不限制使用何种协议，何种方法去通信。例如：你可以获得目标服务器地址和端口后，然后利用thrift,http(s)或其他任一RPC机制。

```text
The communication technology could be anything you like. Eureka helps you find the information about the services you would want to communicate with but does not impose any restrictions on the protocol or method of communication. For instance, you can use Eureka to obtain the target server address and use protocols such as thrift, http(s) or any other RPC mechanisms.
```

#### Eureka包含哪些特性？

- 只有启用@EnableEurekaServer注解，并且添加相关配置，即可自动启动内置的eureka server。
- 基于内存的，重启之后注册信息不存在
- 提供TEST api方式，天然支持跨平台特性，与具体语言无关

## Eureka Client的使用

- native方式：直接使用EurekaClient
- spring cloud抽象的方式：DiscoveryClient
- spring cloud REST client builder：Feign
- spring RestTemplate

### spring cloud consul ：go语言实现

- spring cloud 不支持同时多注册中心，启动时会出现注册失败。但是可以通过spring.profiles的方式来实现注册中心的自由切换

## 服务发现



## 高可用架构

### 基本原则

- 消灭单点故障
- 可靠性传递
- 故障探测



## 各种注册中心对比

| 注册中心  | CAP   | 推荐实例规模（仅供参考） |
| --------- | ----- | ------------------------ |
| Eureka    | AP    | <30K                     |
| Zookeeper | CP    | <30K                     |
| Consul    | AP/CP | <5K                      |
| Nacos     | AP/CP | 100K~200K                |







''

## 参考文章说明

1. [Eureka官方说明](https://github.com/Netflix/eureka/wiki/Eureka-at-a-glance)
2. [spring cloud netflix(eureka clients)  3.1.0 GA ](https://docs.spring.io/spring-cloud-netflix/docs/current/reference/html/#service-discovery-eureka-clients)
3. 

