# 深入浅出REST设计风格

首先需要声明，REST是一种软件架构风格而不是标准！风格，风格，风格

符合REST设计风格的网络服务被称为RESTfull

## 定义及特点

- REST是Representational State Transfer ：具象的状态转换(表现的)
- 根植于HTTP协议之上的约束和属性
- 客户端以统一资源访问符URI来访问和操作网络请求
  - 对资源的操作包括获取，创建，修改，删除，刚好对应HTTP协议提供的GET，POST，PUT，DELETE方法
  - 
- 相比其他网络服务更加简洁
  - soap协议
  - xml-rpc方式

## 约束

- 客户端服务端架构
- 无状态的
- 缓存能力
- 分层
- code on demand
- Uniform Interface统一接口(是RESTful架构设计的基础约束)：包含的4种约束
  - Resource identification in requests ：请求包含资源id
  - 通过资源表现操作资源：不理解？？？？？什么叫做资源表现？？？？
  - 自描述信息
  - HATEOAS：Hypermedia as the engine of application state
    - 通过访问初始的uri，服务端返回以hyperlinks的形式(超媒体)表示的当前应用中提供的所有的资源

举例说明什么是HATEOAS

```http
GET /accounts/12345 HTTP/1.1
Host: bank.example.com

HTTP/1.1 200 OK

{
    "account": {
        "account_number": 12345,
        "balance": {
            "currency": "usd",
            "value": 100.00
        },
        "links": {
            "deposits": "/accounts/12345/deposits",
            "withdrawals": "/accounts/12345/withdrawals",
            "transfers": "/accounts/12345/transfers",
            "close-requests": "/accounts/12345/close-requests"
        }
    }
}
```





## 优点

- 可更高效利用缓存来提高响应速度
- 通讯本身的无状态性可以让不同的服务器的处理一系列请求中的不同请求，提高服务器的扩展性
- 浏览器即可作为客户端，简化软件需求
- 相对于其他叠加在[HTTP协议](https://zh.wikipedia.org/wiki/超文本传输协议)之上的机制，REST的软件依赖性更小
- 不需要额外的资源发现机制
- 在软件技术演进中的长期的兼容性更好