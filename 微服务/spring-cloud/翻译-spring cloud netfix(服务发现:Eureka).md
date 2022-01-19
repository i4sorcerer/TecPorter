# 翻译 spring cloud netfix(服务发现:Eureka)

翻译原文链接：[spring cloud netflix(服务发现)  3.1.0 GA ](https://docs.spring.io/spring-cloud-netflix/docs/current/reference/html/#service-discovery-eureka-clients)

本文目的有二

- 依据官方提供的文档，更进一步或更全面准确的加深对Eureka特性/机制的理解使用
- 保活自己的英语阅读能力

Netflix Project通过自动配置和绑定到Spring Environment以及Spring编程模型风格，从而为Spring Boot应用提供了OSS集成。通过一些简单的注解，就可以在你的应用中启用或配置通用模范，使用大量测试过的组件来构建大型分布式系统。提供的通用模范包括服务发现Eureka。

## 服务发现 Eureka 客户端

服务发现是微服务架构下的基本原则。

尝试手动配置每个客户端或者一些管理是困难并且是脆弱的。

Eureka指的是Nefilx  服务发现服务端和客户端。服务端可以被配置和部署为高可用，每个服务都会作为其他服务器的已注册服务的状态的备份。

### 如何引入Eureka客户端

只需要使用groupID为``` org.springframwork.cloud``` 的artifactID为``` spring-cloud-starter-netflix-eureka-client```即可。

### 如何使用Eureka来注册

客户端使用Eureka注册服务，需要提供自身的元数据，包括host，端口，健康指示器URL，主页已经其他详细信息。Eureka接收服务的每个实例返回的心跳消息。如果心跳消息没有在配置的超时时间内收到，则此实例通常会被从注册中心中移除。

下面展示最小集合的客户端应用配置：

```java
@SpringBootApplication
@RestController
public class Application {

    @RequestMapping("/")
    public String home() {
        return "Hello world";
    }

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}

```

上述代码展示的是正常的Spring Boot应用。通过在classpath中引入spring-cloud-starter-netflix-eureka-client，应用服务自动注册到Eureka 服务器。但是Eureka服务器信息需要在配置中指定，如下application.yaml配置文件所示：

```yaml
eureka: 
	client: 
		serviceUrl:
			defaultZone: http://localhost:8761/eureka
```

在上面示例中，defaultZone是一个字符串回退值，给那些没有指定优先zone的所有客户端提供默认的service URL。

```
说明：
defaultZone关键字是大小写敏感并且需要驼峰配置，原因是serviceURL类型是Map<String,String>类型。
因此其name没有采用spring惯例使用的snake-case方式 default-zone.
```

默认的应用名，虚拟host，非安全端口（Environment中获取的）分别是${spring.application.name},${spring.application.name},

${server.port}。

引入spring-cloud-start-netflix-eureka-client到应用中可以使本应用既是"实例"(注册自己)又是""客户端"(可以查询注册中心来定位其他服务)。

实例行为是通过```eureka.instance.*```相关的配置key进行驱动的，同时默认值一般是ok的如果已经指定了${spring.application.name}，这个配置是Eureka服务ID和VIP的默认值。

详细的配置信息可以参照```EurekaInstanceConfigBean``` 和```EurekaClientConfigBean``` 

如果需要关闭服务发现客户端通过``` eureka.client.enabled=false```和```eureka.cloud.discovery.enabled=false```这2个进行配置。

### 如何使用Eureka 服务端进行验证

HTTP基本验证可以通过下面serviceURL的配置自动添加到应用中

```yaml
eureka.client.serviceUrl.defaultZone=user.password@localhost:8761/eureka
```

对于更复杂的需求，可以通过创建一个```DiscoveryClientOptionalArgs```类型的Bean实例，然后将```ClientFilter```注入到其中，这个过滤器从客户端到服务端都会被调用。

When Eureka server requires client side certificate for authentication, the client side certificate and trust store can be configured via properties, as shown in following example:

当Eureka服务需要客户端的证书验证，客户端证书以及store能够通过属性配置进行配置，如下application.xml配置：

```yaml
eureka:
  client:
    tls:
      enabled: true #开启客户端TLS
      key-store: <path-of-key-store>
      key-store-type: PKCS12
      key-store-password: <key-store-password>
      key-password: <key-password>
      trust-store: <path-of-trust-store> # 如果未设置 使用JVM default trust store
      trust-store-type: PKCS12 # 如果未设置 默认PKCS12
      trust-store-password: <trust-store-password>
```



```
说明：
因为Eureka中的限制，不支持每个server的基本资格认证，所以只会应用在发现的第一个set。
（这块儿没有理解？？？？）
	Because of a limitation in Eureka, it is not possible to support per-server basic auth credentials, so only the first set that are found is used.
```

If you want to customize the RestTemplate used by the Eureka HTTP Client you may want to create a bean of `EurekaClientHttpRequestFactorySupplier` and provide your own logic for generating a `ClientHttpRequestFactory` instance.

如果需要客户化被Eureka HTTP Client的RestTemplate，需要创建Bean ``` EurekaClientHttpRequestFactorySupplier```

并且提供自定义创建```ClientHttpRequestFactory``` 实例的逻辑。

### 运行状态页面及健康指示器

status 页默认是```/info```，健康检查页默认是```/health```这是Spring Boot中的Actuator的应用。

The status page and health indicators for a Eureka instance default to `/info` and `/health` respectively, which are the default locations of useful endpoints in a Spring Boot Actuator application. 

如果需要改变这些，比如使用非默认context path或者servlet path。下面的例子显示针对这两项的默认配置：

```yaml
eureka:
  instance:
    statusPageUrlPath: ${server.servletPath}/info
    healthCheckUrlPath: ${server.servletPath}/health
```

### 注册安全应用https

如果客户端希望注册https，需要设定EurekaInstanceConfigBean中的2个设定

```yaml
eureka.instance.[nonSecurePortEnabled]=[false]
eureka.instance.[securePortEnabled]=[true]
```

这样做可以使Eureka发布实例信息时会明确显示希望安全交流。Spring Cloud```DiscoveryClient``` 总是会返回https的URI。类似的，实例健康检查也是安全的URL。

因为Eureka内部工作机制，状态页和主页的URL仍然是非安全的，除非你显示的覆盖他们的配置。你可以用占位符类配置eureka实例urls，如下面application.yaml示例表示：

```yaml
eureka:
  instance:
    statusPageUrl: https://${eureka.hostname}/info
    healthCheckUrl: https://${eureka.hostname}/health
    homePageUrl: https://${eureka.hostname}/
```

(Note that `${eureka.hostname}` is a native placeholder only available in later versions of Eureka. You could achieve the same thing with Spring placeholders as well — for example, by using `${eureka.instance.hostName}`.)

注意：${eureka.hostname}仅仅是在后续版本中可用的本地占位符。你也可以用spring中的占位符达到同样目的-例如：${eureka.instance.hostName}

### Eureka的健康检查

默认情况下，Eureka使用客户端心跳来确定客户端是否是活的。除非指定，客户端不传播应用的健康检查状态。因此，成功注册之后，Eureka总是报告应用是UP状态。这种行为可以同过启用健康检查来修改，传播应用状态给Eureka。结果是，其他任一应用都只会 把流量发给处于UP状态的应用。配置如下application.yaml所示：

```yaml
eureka:
  client:
    healthcheck:
      enabled: true
```

注意：eureka.client.healthcheck.enable=true只能设置在application.yaml中如果设置在bootstrap.yaml中会产生意料之外的结果，例如导致应用注册状态为UNKNOWN状态

### Eureka包含的实例和客户端的元数据

花一点时间来理解Eureka的元数据工作机制是很值得的，因此你可以在你的平台中以一种有意义的方式来使用。有一些标准元数据，如：hostname，IP地址，端口，状态页，以及监控检查。这些都是在服务注册中心发布的，并且可以直接方式来和服务交流。

额外的元数据信息可以通过```eureka.instance.metadataMap```来添加到实例注册中，并且这个元数据是可以被远程客户端访问的。

通常来讲，额外的元数据不会改变客户端的行为，除非客户端意识到元数据的意义。有一些特殊情况，后续会讨论，就是spring cloud已经赋予metadataMap意义。

#### 在云厂商使用Eureka客户端

云厂商拥有全局路由机制，因此所有应用实例是相同的hostname(其他拥有相似架构的的Pass解决方案拥有相同的安排)。这不是使用Eureka的必须的障碍。可是如果使用路由(推荐或必要的，这取决于你选择的平台)你需要明确指定hostname和port(安全或非安全的)，从而可以使用路由器。

可能你也希望使用实例元数据来区分不同实例(例如：在客户端负载均衡中)，默认情况，eureka.instance.stanceId是和vcap.application.instane_id是一样的。如下图application.yaml所示。

```yaml
eureka:
  instance:
    hostname: ${vcap.application.uris[0]}
    nonSecurePort: 80
```

取决于你再云厂商中健安全规则配置的方式，你可以注册并且使用host VM的IP地址来进行直接的服务到服务的调用。该功能尚未在PWS中提供。

#### 在AWS中使用Eureka

略。。。。

#### 改变Eureka 实例ID

香草 Netflix Eureka实例的注册ID和其hostname是一致的(也就是说一个host只能注册一个服务)

spring cloud eureka提供一个大小写敏感的默认值，如下所示：

```yaml
${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}
```

一个例子是：没有host:myappname:8080

使用spring cloud，你可以通过在 ``` eureka.instance.instanceID```中指定一个唯一ID，如下所示：

```yaml
eureka:
  instance:
    instanceId: ${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}
```

带有上述例子中的元数据，并且在本地部署多服务实例，上面的random value会被插入一个随机值来保证实例ID的唯一性。

在云厂商中，```vcap.application.instane_id```是在spring boot应用中自动被填充的，因此随机数不是必须的。

### 使用Eureka 客户端

一旦你的应用是服务发现客户端Discovery Client，你就可以使用它来从Eureka server发现服务实例。

方法1：使用原生的Eureka 客户端 ``` com.netflix.discovery.EurekaClient```(作为spring cloud ```DiscoveryClient```的对照)

```java
@Autowired
private EurekaClient discoveryClient;

public String serviceUrl() {
    InstanceInfo instance = discoveryClient.getNextServerFromEureka("STORES", false);
    return instance.getHomePageUrl();
}
```

注意事项：

- 不要在@PostConstruct中使用EurekaCLient
- 不要在@Scheduled方法中使用EurekaCLient
- 不要在任何ApplicationContext可能尚未初始化的地方使用
- 因为在phase=0的SmartLifecycle中被初始化，所以最早能使用的时机是在另一个phase大于0的SmartLifecycle中

```jav
// 说明什么是SmartLifececle接口
org.springframework.context.SmartLifecycle extands Lifecycle,Phase{
booelan isAutoStartUp()；
void start();
int getPhase();
}
Lifecycle在容器启动和停止时，可以动态感知。
SmartLifecycle只不过是在此基础上增加了先后顺序Phase，以及是否自动启动功能而已。
其中phase越小越最先执行。

```

#### 通过Jersey使用Eureka客户端

默认情况下使用Spring RestTemplate进行HTTP的通信。如果希望使用Jersey，需要在classpath中添加相应依赖，如下所示：

```text
// Jersey框架说明
为了开放RESTfull 服务及客户端，提出了轻量级的标准JAX-RS APi。
首先Jerysey是其参考实现，并且进行了扩充，进一步简化了RESTfull服务和客户端的开发
```

```
<dependency>
    <groupId>com.sun.jersey</groupId>
    <artifactId>jersey-client</artifactId>
</dependency>
<dependency>
    <groupId>com.sun.jersey</groupId>
    <artifactId>jersey-core</artifactId>
</dependency>
<dependency>
    <groupId>com.sun.jersey.contribs</groupId>
    <artifactId>jersey-apache-client4</artifactId>
</dependency>
```

### 原生的Netflix EurekaClient的替代选择

通常不需要使用原生的Netflix EurekaClient，而是使用更便利的在其之上的Wapper。Spring Cloud已经支持Feign以及spring template使用逻辑服务描述符而非物理服务URL来进行访问。

同样可以使用org.springframework.cloud.client.discovery.DiscoveryClient，提供通用的抽象API(不仅仅特定于Netflix)，如下application.yam所示：

```yaml
@Autowired
private DiscoveryClient discoveryClient;

public String serviceUrl() {
    List<ServiceInstance> list = discoveryClient.getInstances("STORES");//serviceID
    if (list != null && list.size() > 0 ) {
        return list.get(0).getUri();
    }
    return null;
}
```



### 为什么注册服务如此之慢

Being an instance also involves a periodic heartbeat to the registry (through the client’s `serviceUrl`) with a default duration of 30 seconds. A service is not available for discovery by clients until the instance, the server, and the client all have the same metadata in their local cache (so it could take 3 heartbeats). You can change the period by setting `eureka.instance.leaseRenewalIntervalInSeconds`. Setting it to a value of less than 30 speeds up the process of getting clients connected to other services. In production, it is probably better to stick with the default, because of internal computations in the server that make assumptions about the lease renewal period.

作为一个实例需要维持和注册中心的周期心跳，默认周期是30s。可以通过配置```eureka.instance.leaseRenewalIntervalSeconds```进行改变。设置小于30s的值可以加快获取连接到其他服务的客户端列表的时间。在生产环境，建议使用默认配置，因为server内部会对续租周期做出假设。

服务能够被客户端发现的条件：

1. instance，server，client同时都含有相同的本地Cache(通常需要花费3个心跳周期)

### 区Zones

如果你将Eureka客户端部署在多个Zone，你可能会倾向于客户端优先使用相同区的服务，然后再去尝试其他区的。实现此目的需要正确配置客户端。

- 确保部署到不同zone的Eureka服务器是同级的，可参照zones and regions这一节
- 告诉Eureka当前服务所在区

举例说明如何使用MetadataMap进行设置

如果一个service1想要发布到zone1和zone2两个区，如下配置：

service1的zone1

```yaml
eureka.instance.metadataMap.zone=zone1
eureka.client.preferSameZoneEureka=true
```

service1的zone2

```yaml
eureka.instance.metadataMap.zone=zone2
eureka.client.preferSameZoneEureka=true
```

### 刷新Eureka客户端@RefreshScope

By default, the `EurekaClient` bean is refreshable, meaning the Eureka client properties can be changed and refreshed. When a refresh occurs clients will be unregistered from the Eureka server and there might be a brief moment of time where all instance of a given service are not available. One way to eliminate this from happening is to disable the ability to refresh Eureka clients. To do this set `eureka.client.refresh.enable=false`.

默认情况下Eureka 客户端是可刷新的bean。这意味着Eureka客户端属性能够被改变和刷新，当刷新发生时客户端会先从server取消注册，这会导致短暂的某个服务的所有实例无法被使用。

强烈建议禁止refresh Eureka 客户端，通过```eureka.client.refresh.enable=false```来禁止。

### 搭配spring cloud LoadBanlance使用Eureka

我们提供spring cloud LoadBanlancer```ZonePreferenceServiceInstanceListSupplier```.

Eureka 实例metadata中配置的zone信息 eureka.instance.metadataMap.zone用来设置其属性值spring-cloud-loadbalancer-zone从而可以针对zone对服务进行过滤。

如果没有设置，将会选择hostname作为zone的代理，前提是属性spring.cloud.loadbalancer.eureka.approximateZoneFromHostname设置为true。

如果没有zone信息，会基于客户端配置进行guess（区别于实例配置）

使用eureka.client.availabilityZones，key是region名，value是zone列表，并且取出第一个zone作为实例的自己所属region

也就是eureka.client.region指定的值，默认是”us-east-1“为了兼容原生的Netflix版本。

## 服务发现 Eureka 服务端

这一节讲述如何搭建服务发现server

### 如何引入Eureka服务端



### 如何启动Eureka服务器

### 高可用，Zones和Regions

### 单机模式

### Eureka服务器相互感应

### 何时更倾向于使用IP地址

### 安全的Eureka服务端

### JDK11的支持

## 配置属性