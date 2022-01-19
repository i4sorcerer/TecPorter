## 深入理解不同的I/O模型及初识NIO框架-Netty

### 什么是I/O

在计算机系统中，指计算机内存和外界的信息交换(输入与输出)

外界包括：

- 其他计算机
- 外部设备转换后的人的动作等

外部设备作用是将人的动作等信息转换为计算机能识别的信号或数据的设备

iput：计算机内存接收外部信号或数据：如：鼠标，键盘灯外部设备产生的数据

output：计算机内存将数据输出到外部设备：如：显示器展示的数据

### 什么是DMA

全称是Direct Memory Access，直接内存访问

是为了提高在I/O传输过程中，CPU和内存的访问效率，增加的一种硬件实现。来满足外部设备和内存直接进行数据的交换，而不需要经过CPU的控制，与此同时CPU可以继续正常执行程序。

涉及到的相关硬件

- 系统总线(总线控制权的转移)
- CPU
- DMA控制器
- DMA

DMA传送数据方式

- 停止CPU访问内存：DMA访问内存期间，停止CPU访问内存
- 挪用几个内存周期：没有DMA请求时，CPU正常执行CPU执行完当前时间周期后，会被DMA抢占，
- DMA和CPU交替访问内存（效率最高，透明DMA）



### 什么是缓存IO

缓存IO也就是标准IO，现在操作系统默认都是带缓存的IO。比如在读取文件到内存时，系统会先把数据拷贝到操作系统的内核缓冲区（PageCache）中，然后再从内核缓冲区拷贝到应用程序地址空间。

因此这种拷贝会涉及到内核和应用程序空间的多次拷贝，这些操作的CPU以及内存开销都是很大的。

因此引出了``` 零拷贝```技术，来减少内核空间和用户空间的数据拷贝，降低CPU和内存的开销。



### 彻底理解IO场景下的同步/异步 阻塞/非阻塞

同步与异步的区别：用户进程是否需要主动查看IO状态或者主动去拷贝数据

- 同步synchronous IO：需要主动拷贝数据或查看状态

- 异步asynchronous IO：被动接收完成通知

阻塞IO与非阻塞IO的区别：

- 阻塞 blocking IO: 调用阻塞IO操作会一致将进程阻塞知道操作完成
- 非阻塞non-blocking IO: 调用非阻塞IO操作，当数据未准备完成时，会立即返回不会阻塞。但当数据准备完成后，还是会阻塞拷贝数据(系统调用recvfrom来拷贝来拷贝数据)



### 几种I/O模型

同步模型

#### BIO

Blocking IO同步阻塞IO

传统的IO模型，使用ServerSocket.accept()方法，接收客户端的连接请求，如果没有客户端发送请求，则服务端会一致阻塞在accept方法。

#### NIO

Non-Blocking IO同步非阻塞IO

最大特点是用户进程需要不断主动询问内核，数据准备好了没有。

主动查询当前IO状态，因为是主动所以也是同步模型

非阻塞主要是Selector轮询+状态判断来实现的

三要素

- 通道Channel
- 缓冲区Buffer
- 轮询器Selector

#### 多路复用IO (mulitiplexing IO)

- 多路复用其实就是select，poll和epoll的系统调用，通过把多个IO的阻塞复用到同一个select的阻塞上，使得单进程可以处理更多连接

- select具体监视的是不同类型的文件描述符FD(文件描述符就是每个进程创建或操作的文件记录表的非负整数索引值)
- 可以等待多个socket，同时对多个IO端口进行监听。当启动任何一个socket准备好数据，就能返回可读。
- 用户进程再发起recvfrom系统调用将数据从内核拷贝到用户进程，这个过程是阻塞的。
- 当用户进程调用select/poll时，进程会阻塞在此方法上。当某个socket数据准备完成就可返回

优点：

- 单进程可以处理更多的网络连接
- 系统开销小，不必新建进程或线程，避免不必要的开销

缺点：

- 需要发起两次系统调用，在连接数并不大的场景下，新能反而会降低

异步模型

#### AIO

Async IO异步非阻塞IO

- IO操作是由操作系统进行的，完成之后通过回调方法进行通知









### 多路复用几种实现方式

#### select(POSIX规定的)

- 能监听的fd个数有上限，可配置，FD_SETSIZE
- 线性遍历fd存储结构，如果有大量fd，无谓查询效率低

#### poll

- 链表存储fd结构，fd的size没有上限
- 线性遍历fd存储结构，如果有大量fd，无谓查询效率低

#### epoll(linux2.6内核以上中特有的)

- 采用事件就绪通知类似回调，仅仅只有就绪状态的FD才会调用回调函数，效率高
- 如果连接数较少，或者所有连接都处于就绪状态，则效率也是低下



### 什么是堆外内存？



### 什么是Reactor线程模型

- 单线程Reactor模型
- 多线程Reactor模型
- 主从Reaactor模型





### Java中提供的的NIO API

#### Selector

什么是Selector？？？？？

#### SelectionKey



### 什么是java直接缓冲区DirectBuffer

NIO中三要素之一就是Buffer，定义了多种类型的Buffer

ByteBuffer

DirectByteBuffer

......



### NIO框架-Netty

Netty一般是其他开源框架的底层通信框架，主要解决通信问题。是对java原生NIO的封装。

比如

zk

dubbo：多协议支持

spring5中的嵌入容器

。。。。

#### 高性能Netty通信的几大原因

- 异步非阻塞通信：自定义的ChannelFuture
- 零拷贝：使用的DirectBuffer等缓冲区
- 内存池：
- 线程模型Reactor：①单线程模型②多线程模型③主从线程模型
- 无锁化串行设计：主要指的是ChannelPipeline中的责任链机制
- 高效的并发编程：
- 高性能的序列化框架：
- 灵活的TCP参数配置（事件定义丰富）

### Netty-核心之Channel

 是一种与网络socket或者其他能够进行读/写/连接/绑定等IO操作的组件的连接通道

- 一个 channel对象能够提供给用户：
   1. 当前channnel的状态(关闭或打开)
   1. 通过ChannelConfig获取到配置参数，如：接收缓冲区大小
   1. 具体的IO操作，如读，写，连接，绑定等
   1. 通过ChannelPipeline来处理所有和当前Channel相关的IO事件及请求

- 所有的IO操作都是异步的
  1. Netty中所有IO操作都是异步的，意味着任意的IO调用都会立即返回，而不保证请求的IO操作已经结束。
  2. 异步是通过返回ChannelFuture对象来实现的，并可以获取到IO操作的结果
- Channel是具有分级的结构
  1. 每个Channel都有一个parent，基于此进行创建。例如：SocketChannel通过ServerSocketChannel的accept方法返回，同时其parent()返回ServerSocketChannel对象
  2. 分级结构的具体语义取决于它输入哪一种传输实现。
- Channel使用完后必须调用close进行资源释放

```java
public interface Channel extends AttributeMap, ChannelOutboundInvoker, Comparable<Channel> {
	  // 全局唯一ID
    ChannelId id();
	  // 返回Channel被注册到哪一个EventLoop
    EventLoop eventLoop();
  	// 父级Channel
    Channel parent();
  	// 返回Channel的配置参数
    ChannelConfig config();
  	// 返回一个ChannelFuture对象，当Channel关闭的时候会获得一个通知
    ChannelFuture closeFuture();
  	// 返回仅供内部使用的unsafe操作类
    Unsafe unsafe();
  	// 返回负责分配字节缓冲区内存的Allocator
    ByteBufAllocator alloc();
  	// 返回Channel上绑定的pipeline
    ChannelPipeline pipeline();
    
  	@Override
    Channel read();

    @Override
    Channel flush();
  
    boolean isOpen();
    boolean isRegistered();
    boolean isActive();
  	// 只有当IO线程能立即执行写操作时才会返回true，否则都是false，
  	// 并且写请求会在队列中进行排队，直到IO线程准备好了来处理排队的写请求。
    boolean isWritable();

    /**
     * Return the {@link ChannelMetadata} of the {@link Channel} which describe the nature of the {@link Channel}.
     */
    ChannelMetadata metadata();

    /**
     * Returns the local address where this channel is bound to.  The returned
     * {@link SocketAddress} is supposed to be down-cast into more concrete
     * type such as {@link InetSocketAddress} to retrieve the detailed
     * information.
     *
     * @return the local address of this channel.
     *         {@code null} if this channel is not bound.
     */
    SocketAddress localAddress();

    /**
     * Returns the remote address where this channel is connected to.  The
     * returned {@link SocketAddress} is supposed to be down-cast into more
     * concrete type such as {@link InetSocketAddress} to retrieve the detailed
     * information.
     *
     * @return the remote address of this channel.
     *         {@code null} if this channel is not connected.
     *         If this channel is not connected but it can receive messages
     *         from arbitrary remote addresses (e.g. {@link DatagramChannel},
     *         use {@link DatagramPacket#recipient()} to determine
     *         the origination of the received message as this method will
     *         return {@code null}.
     */
    SocketAddress remoteAddress();
    /**
     * Get how many bytes can be written until {@link #isWritable()} returns {@code false}.
     * This quantity will always be non-negative. If {@link #isWritable()} is {@code false} then 0.
     */
    long bytesBeforeUnwritable();

    /**
     * Get how many bytes must be drained from underlying buffers until {@link #isWritable()} returns {@code true}.
     * This quantity will always be non-negative. If {@link #isWritable()} is {@code true} then 0.
     */
    long bytesBeforeWritable();


    /**
     * <em>Unsafe</em> operations that should <em>never</em> be called from user-code. These methods
     * are only provided to implement the actual transport, and must be invoked from an I/O thread except for the
     * following methods:
     * <ul>
     *   <li>{@link #localAddress()}</li>
     *   <li>{@link #remoteAddress()}</li>
     *   <li>{@link #closeForcibly()}</li>
     *   <li>{@link #register(EventLoop, ChannelPromise)}</li>
     *   <li>{@link #deregister(ChannelPromise)}</li>
     *   <li>{@link #voidPromise()}</li>
     * </ul>
     */
    interface Unsafe {

        /**
         * Return the assigned {@link RecvByteBufAllocator.Handle} which will be used to allocate {@link ByteBuf}'s when
         * receiving data.
         */
        RecvByteBufAllocator.Handle recvBufAllocHandle();

        /**
         * Return the {@link SocketAddress} to which is bound local or
         * {@code null} if none.
         */
        SocketAddress localAddress();

        /**
         * Return the {@link SocketAddress} to which is bound remote or
         * {@code null} if none is bound yet.
         */
        SocketAddress remoteAddress();

        /**
         * Register the {@link Channel} of the {@link ChannelPromise} and notify
         * the {@link ChannelFuture} once the registration was complete.
         */
        void register(EventLoop eventLoop, ChannelPromise promise);

        /**
         * Bind the {@link SocketAddress} to the {@link Channel} of the {@link ChannelPromise} and notify
         * it once its done.
         */
        void bind(SocketAddress localAddress, ChannelPromise promise);

        /**
         * Connect the {@link Channel} of the given {@link ChannelFuture} with the given remote {@link SocketAddress}.
         * If a specific local {@link SocketAddress} should be used it need to be given as argument. Otherwise just
         * pass {@code null} to it.
         *
         * The {@link ChannelPromise} will get notified once the connect operation was complete.
         */
        void connect(SocketAddress remoteAddress, SocketAddress localAddress, ChannelPromise promise);

        /**
         * Disconnect the {@link Channel} of the {@link ChannelFuture} and notify the {@link ChannelPromise} once the
         * operation was complete.
         */
        void disconnect(ChannelPromise promise);

        /**
         * Close the {@link Channel} of the {@link ChannelPromise} and notify the {@link ChannelPromise} once the
         * operation was complete.
         */
        void close(ChannelPromise promise);

        /**
         * Closes the {@link Channel} immediately without firing any events.  Probably only useful
         * when registration attempt failed.
         */
        void closeForcibly();

        /**
         * Deregister the {@link Channel} of the {@link ChannelPromise} from {@link EventLoop} and notify the
         * {@link ChannelPromise} once the operation was complete.
         */
        void deregister(ChannelPromise promise);

        /**
         * Schedules a read operation that fills the inbound buffer of the first {@link ChannelInboundHandler} in the
         * {@link ChannelPipeline}.  If there's already a pending read operation, this method does nothing.
         */
        void beginRead();

        /**
         * Schedules a write operation.
         */
        void write(Object msg, ChannelPromise promise);

        /**
         * Flush out all write operations scheduled via {@link #write(Object, ChannelPromise)}.
         */
        void flush();

        /**
         * Return a special ChannelPromise which can be reused and passed to the operations in {@link Unsafe}.
         * It will never be notified of a success or error and so is only a placeholder for operations
         * that take a {@link ChannelPromise} as argument but for which you not want to get notified.
         */
        ChannelPromise voidPromise();

        /**
         * Returns the {@link ChannelOutboundBuffer} of the {@link Channel} where the pending write requests are stored.
         */
        ChannelOutboundBuffer outboundBuffer();
    }
}
  
}
```

### Netty-核心之ChannelPipeline

每一个请求都要在线程中，全部走一遍pipeline，每个handler都会检查一遍，看看自己能不能处理。

其实就是一个处理每个请求的责任链，结构是双向列表，链表中存储的是各种Handler实例

- Next ：InboundHandler
- Pre ：OutboundHandler

#### 消息的传递

消息或者事件的传递是通过各种fireXXXX方法进行的。

比如：

- channelActive事件，当前Handler可以处理，处理完成后不再继续后续Handler处理，则无需执行fireXXXX操作，事件到此即中止。
- 当前Handler不可处理，则直接调用fireChannelActive()方法将事件往下继续传递

### Netty-核心之ChannelHandler

- InboundHandler：
- OutboundHandler：

### Netty-核心之BootStrap

需要了解BootStrap的启动过程，以及如何初始化ChannelPipeline的

### Netty-核心之EventLoopGroup/EventLoop

```java

/**
 * Will handle all the I/O operations for a {@link Channel} once registered.
 *
 * One {@link EventLoop} instance will usually handle more than one {@link Channel} but this may depend on
 * implementation details and internals.
 *
 */
public interface EventLoop extends OrderedEventExecutor, EventLoopGroup {
    @Override
    EventLoopGroup parent();
}
```



```java
// MultithreadEventExecutorGroup构造函数中，如果入参executor为空，则使用默认的ThreadPerTaskExecutor线程池
executor = new ThreadPerTaskExecutor(newDefaultThreadFactory());

```



### Netty中的粘包与拆包

内置了几种拆包的Handler供使用，只要添加进pipeline就可以了

- LineBasedFrameDecoder：基于消息结尾的换行符"\n","\r\n"
-  DelimiterBasedFrameDecoder（添加特殊分隔符报文来分包） 
- FixedLengthFrameDecoder（使用定长的报文来分包）
- LengthFieldBasedFrameDecoder







### Netty中设计模式

#### 灵魂-责任链模式

多个对象可以处理同一个请求，避免发送方和接收方的耦合

ChannelPipeline中典型的责任链模式





### 其他

#### Netty-解决JDK Nio的空轮询Bug



#### 参考文章

[聊聊IO多路复用之select/poll/epoll]: https://www.jianshu.com/p/dfd940e7fca2
[基于Netty实现服务端与客户端的通信]: https://juejin.cn/post/6844904122441809934
[Linux系统中的5种IO模型]: https://www.jianshu.com/p/486b0965c296
[TCP粘包/拆包与Netty解决方案]: https://blog.csdn.net/J080624/article/details/87209637









