## 
# 一、概念

### Netty的稳定版本：3.x、4.x（推荐），3.x到4.x是不兼容的升级。
#### Netty4.x的新特性：
- 模块化程度更高，包名由org.jboss.netty更新为io.netty;
- 大多API支持流式风格;
- Buffer相关优化;
![](https://raw.githubusercontent.com/liuxiaofeii/BC4A0327-E9BF-B504-C6AE-24BEC8348190/main/1716802336540.png)
- io.netty.util.concurrent包中提供了较多异步编程的数据结构;
# 二、Netty和JDK NIO、其他网络框架的比较
## 1、Netty相比JDK NIO的突出优势
### （1）易用性
- 屏蔽了NIO的复杂性；
- 封装更加人性化的API；
- 降低开发者的上手难度；
### （2）稳定性
- 修复和完善较多问题；
### （3）可扩展性
- 可定制化的线程模型；
- 可扩展的事件驱动模型；
### （4）更低的资源消耗
- 对象池复用技术；
- 零拷贝技术；
## 2、Netty和Tomcat的最大区别：对通信协议的支持
#### Tomcat
- 一个HTTP Server，主要解决HTTP协议层的传输；
- Tomcat需要遵循Servlet规范。Tomcat3.0之前采用BIO。6.x版本之后支持NIO，性能得到较大提升。
#### Netty
- 支持多种应用层的协议，比如HTTP、SSH、TLS/SSL等协议；
- 能够自定义应用层协议；
- Netty由于不受到Servlet规范的约束，可以最大发挥NIO的特性。
#### 总结
由于Tomcat在作为HTTP服务器的成熟度和稳定性更好，所以假如你仅仅只需要一个HTTP服务器，推荐使用Tomcat。但是，假如你需要做面向TCP的网络应用开发，那么推荐使用Netty。
## 3、Netty和Mina、Grizzly的对比
Mina是Apache Directory服务底层的NIO框架，它和Netty都是Trustin Lee的作品，所以两者在设计理念上基本一致，但是Netty是Mina的升级，解决了Mina一些设计上的问题，比如Netty提供可扩展的编解码接口、优化了ByteBuffer的分配方式，让用户使用起来更便捷、安全。而Grizzly出身于Sun公司，从设计理念上看，Grizzly没有Netty优雅，几乎是对JDK NIO的初级封装，目前业界使用的范围也比较小。
# 三、优势
[[#1、Netty相比JDK NIO的突出优势]]
2、社区成熟
# 四、应用
![](https://raw.githubusercontent.com/liuxiaofeii/BC4A0327-E9BF-B504-C6AE-24BEC8348190/main/1716802878593.png)
# 五、使用
## 1、一个简单的HTTP服务器的实现

# 六、原理
## 1、Netty整体结构
### （1）图示
![](https://raw.githubusercontent.com/liuxiaofeii/BC4A0327-E9BF-B504-C6AE-24BEC8348190/main/20240528102659.png)
### （2）主要组成
1. 核心层
	- 提供<mark style="background: #FFF3A3A6;">底层网络通信的通用抽象和实现</mark>，包括可扩展的事件模型、通用的通信 API、支持零拷贝的 ByteBuf等。
2. 协议支持层（Protocol Support）
	- <mark style="background: #FFF3A3A6;">覆盖了主流协议的编解码实现</mark>，如 HTTP、SSL、Protobuf、压缩、大文件传输、WebSocket、文本、二进制等。
3. 传输服务层（Transport Service）
	- 传输服务层提供了网络传输能力的定义和实现方法<mark style="background: #FFF3A3A6;">支持 Socket、HTTP 隧道</mark>、虚拟机管道等传输方式
	- Netty 的模块设计具备较高的通用性和可扩展性
## 2、Netty逻辑架构
### （1）图示
![](https://raw.githubusercontent.com/liuxiaofeii/BC4A0327-E9BF-B504-C6AE-24BEC8348190/main/20240528102522.png)
### （2）主要组成
[[#2、网络通信层]]
[[#3、事件调度层]]
[[#4、服务编排层]]
### （3）组件关系图
<mark style="background: #FFF3A3A6;">服务端启动初始化时</mark>，有Boss EvenLoopGroup和Worker EventLoopGroup两个组件。其中Boss负责监听网络连接事件，当有新的网络连接事件到达时，则将Channel注册到Worker EventLoopGroup。
Worker EventLoopGroup会分配一个EventLoop来负责处理该Channel的读写事件，每个EventLoop都是单线程的。
<mark style="background: #FFF3A3A6;">通过Selector进行事件循环</mark>，<mark style="background: #FFF3A3A6;">当客户端发起IO读写事件时</mark>，服务端EventLoop会进行数据的读取。然后通过ChannelPipeline触发各种监听器进行数据的加工处理，客户端数据会被传递到ChannelPipeline的第一个channelInboundHandler中。<mark style="background: #FFF3A3A6;">数据处理完成后</mark>，将加工完成的数据传递给下一个ChannelInboundHandler。当数据写回客户端时，会将处理结果在ChannelPipeline的ChannelOutboundHandler中传播，<mark style="background: #FFF3A3A6;">最后到达客户端</mark>。
![](https://raw.githubusercontent.com/liuxiaofeii/BC4A0327-E9BF-B504-C6AE-24BEC8348190/main/20240528160740.png)
## 3、网络通信层
### （1）概念
网络通信层的职责是执行网络I/O的操作，支持多种网络协议和I/O模型的连接操作。
### （2）核心组件
#### Netty的引导器
1. 概念
	Netty的引导器是作为整个Netty服务端和客户端的程序入口，引导器种类有：Bootstrap、ServerBootstrap。
	- Bootstrap：用于<mark style="background: #FFF3A3A6;">连接远端服务器</mark>，只绑定一个EventLoopGroup(Boss)
	- ServerBootstrap：用于<mark style="background: #FFF3A3A6;">服务端启动</mark>绑定本地端口，绑定两个EventLoopGroup(Worker)
2. 引导器的使用
[[#1、一个简单的HTTP服务器的实现]]
#### Channel
1. **概念**
Channel是<mark style="background: #FFF3A3A6;">网络通信的载体</mark>，提供了基本的API用于网络 I/O 操作，如register、bind、connect、read、write、flush 等。Netty是以JDK NIO Channel为基础实现的 Channel。
2. **常见实现类**：
	- <mark style="background: #FFF3A3A6;">NioServerSocketChannel异步TCP服务端</mark>
	- <mark style="background: #FFF3A3A6;">NioSocketChannel异步TCP客户端</mark>
	- OioServerSocketChannel同步TCP 服务端
	- OioSocketChannel同步TCP客户端
	- NioDatagramChannel异步UDP连接
	- OioDatagramChannel同步UDP 连接
3. **存在多种状态，如连接建立、连接注册、数据读写、连接销毁等，如下表：**

	| 事件                   | 说明                              |
	| -------------------- | ------------------------------- |
	| channelRegistered\|  | Channel创建后被注册到 EventLoop 上      |
	| channelUnregistered  | Channel创建后未注册或者从 EventLoop 取消注册 |
	| **channelActive**    | **Channel处于就绪状态，可以被读写**         |
	| channellnactive      | Channel处于非就绪状态                  |
	| **channelRead**      | **Channel可以从远端读取到数据**           |
	| channelReadCompletel | Channel读取数据完成                   |

**TIP**:
- Boss和Worker的区别：
	每个服务器中都会有一个 Boss，会有一群做事情的 Worker。<mark style="background: #FFF3A3A6;">Boss 会不停地接收新的连接，将连接分配给一个个Worker处理连接</mark>。
## 4、事件调度层
### （1）概念
事件调度层负责通过<mark style="background: #FFF3A3A6;">Reactor 线程模型</mark>对各类事件进行聚合处理、通过 Selector主循环线程集成多种事件。
### （2）Reactor线程模型
#### 1. 概念
线程模型的优劣直接决定了系统的吞吐量、可扩展性、安全性等。Reactor模式负责将读写事件分发给对应的读写事件处理者。
#### 2. 分类
##### **单线程模型**
在Reactor单线程模型当中，所有IO操作包括连接建立，数据读写，事件分发等，都是由一个线程完成的。对应Netty的配置是：EventLoopGroup只包含<mark style="background: #FFF3A3A6;">一个EventLoop</mark>，Boss和Worker使用同一个EventLoopGroup。
![image.png](https://raw.githubusercontent.com/liuxiaofeii/BC4A0327-E9BF-B504-C6AE-24BEC8348190/main/20240528162839.png)
**优点**：逻辑简单
**缺点**：
- 一个线程支持处理的连接数有限，CPU很容易打满，性能方面有明显瓶颈;
- 当多个事件被同时触发，只要有一个事件没有处理完，其他后面的事件就无法执行;
- 线程在处理I/O事件时，Select无法同时处理连接建立、事件分发等操作;
- 如果I/O线程一直处于满负荷状态，可能造成服务端节点不可用;
##### **多线程模型**
Reactor多线程模型把业务逻辑交给多个线程进行处理。除此之外，多线程模型其他的操作与单线程模型是类似的，例如读取数据依然保留了串行化的设计。当客户端有数据发送给服务端时，select会监听到可读事件数据，读取完毕后提交到业务线程池中并发处理。对应Netty的配置是：EventLoopGroup包含<mark style="background: #FFF3A3A6;">多个EventLoop</mark>，Boss和Worker使用同一个EventLoopGroup。
![image.png](https://raw.githubusercontent.com/liuxiaofeii/BC4A0327-E9BF-B504-C6AE-24BEC8348190/main/20240528163138.png)

##### **主从线程模型**
主从多线程模型由多个Reactor线程组成，每个Reactor线程都有独立的selector对象。mainReactor仅负责处理客户端连接的Access的事件，<mark style="background: #ADCCFFA6;">连接建立成功后将新创建的连接对象注册至subReactor</mark>，再由subReactor分配线程池中的IO线程与其连接绑定，该IO线程会负责连接生命周期内所有的IO事件。<mark style="background: #FFF3A3A6;">Netty推荐使用主从多线程模型，这样可以达到成千上万规模的客户端连接</mark>。在海量客户端并发请求的场景下，主从多线程模式可以适当增加subReactor线程的数量，从而利用多核能力提升系统的吞吐量。
对应Netty的配置是：EventLoopGroup包含<mark style="background: #FFF3A3A6;">多个EventLoop</mark>，<mark style="background: #FFF3A3A6;">Boss是主Reactor</mark>，<mark style="background: #FFF3A3A6;">Worker是从 Reactor</mark>。主 Reactor负责新的网络连接Channel创建，然后把Channel注册到从Reactor。
![image.png](https://raw.githubusercontent.com/liuxiaofeii/BC4A0327-E9BF-B504-C6AE-24BEC8348190/main/20240528163241.png)
#### 3. 运行机制
	- 连接注册：Channel建立后，注册至 Reactor线程中的Selector选择器；
	- 事件轮询：轮询 Selector选择器中已注册的所有Channel的I/O 事件；
	- 事件分发：为准备就绪的 I/O事件分配相应的处理线程；
	- 任务处理：每个Worker 线程从各自维护的任务队列中取出任务异步执行；
### （3）核心组件
#### EventLoopGroup
EventLoopGroup本质上是一个线程池，主要负责IO请求，并分配线程来执行处理请求。一个EventLoopGroup可以包含一个或多个EventLoop。
EventLoopGroup是Netty Reactor线程模型的具体实现，也是Netty Reactor线程模型的核心处理引擎。
常见实现类：NioEventLoopGroup（Netty中最被推荐使用的线程模型）。
#### EventLoop
每个EventLoop同一时间会与一个线程绑定，每个EventLoop负责处理多个Channel。每新建一个Channel，EventLoopGroup会选择一个EventLoop与其绑定。该Channel在生命周期内都可以对EventLoop进行多次绑定和解绑。
## 5、服务编排层
### （1）概念
服务编排层负责组装各类服务，用以实现网络事件的动态编排和有序传播。
### （2）核心组件
#### ChannelPipeline
1. 概念
ChannelPipeline 负责组装各种 ChannelHandler。
当I/O读写事件触发时，ChannelPipeline会依次调用 ChannelHandler列表对 Channel的数据进行拦裁和处理。
由于每一个新的Channel都会绑定一个新的ChannelPipeline，所以ChannelPipeline是线程安全的。
一个ChannelPipeline关联一个EventLoop；一个EventLoop仅会绑定一个线程。
2. 客户端和服务端一次完整的请求应答过程
![](https://raw.githubusercontent.com/liuxiaofeii/BC4A0327-E9BF-B504-C6AE-24BEC8348190/main/20240528160201.png)
#### ChannelHandler
ChannelHandler负责实际数据的编解码以及加工处理操作。
#### ChannelHandlerContext
1. 概念
	每个ChannelHandler绑定一个ChannelHandlerContext。
1. 作用
	- 保存 ChannelHandler上下文；
	- 实现 ChannelHandler 之间的交互；
	- 包含 ChannelHandler生命周期的所有事件如connect、bind、read、flush、write、close等；