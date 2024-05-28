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
# 五、原理
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
#### 网络通信层
##### 1、概念
网络通信层的职责是执行网络I/O的操作，支持多种网络协议和I/O模型的连接操作。
##### 2、核心组件
###### Bootstrap
用于<mark style="background: #FFF3A3A6;">连接远端服务器</mark>，只绑定一个EventLoopGroup(Boss)
###### ServerBootstrap
用于<mark style="background: #FFF3A3A6;">服务端启动</mark>绑定本地端口，绑定两个EventLoopGroup(Worker)
###### Channel
1. **概念**
Channel是<mark style="background: #FFF3A3A6;">网络通信的载体</mark>，提供了基本的API用于网络 I/O 操作，如register、bind、connect、read、write、flush 等。Netty是以JDK NIO Channel为基础实现的 Channel。
2. **常见实现类**：
	- <mark style="background: #FFF3A3A6;">NioServerSocketChannel异步TCP服务端</mark>
	- <mark style="background: #FFF3A3A6;">NioSocketChannel异步TCP客户端</mark>
	- OioServerSocketChannel同步TCP 服务端
	- OioSocketChannel同步TCP客户端
	- NioDatagramChannel异步UDP连接
	- OioDatagramChannel同步UDP 连接
1. **存在多种状态，如连接建立、连接注册、数据读写、连接销毁等，如下表：**

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
#### 事件调度层

#### 服务编排层


