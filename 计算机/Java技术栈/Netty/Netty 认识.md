# 一、概念

### Netty的稳定版本：3.x、4.x（推荐），3.x到4.x是不兼容的升级。
#### Netty4.x的新特性：
模块化程度更高，包名由org.jboss.netty更新为io.netty;
大多API支持流式风格;
Buffer相关优化;
![[图床/1716802336540.png]]
io.netty.util.concurrent包中提供了较多异步编程的数据结构;
# 二、Netty和JDK NIO、其他网络框架的比较
## 1、Netty相比JDK NIO的突出优势
### （1）易用性
屏蔽了NIO的复杂性；
封装更加人性化的API；
降低开发者的上手难度；
### （2）稳定性
修复和完善较多问题；
### （3）可扩展性
可定制化的线程模型；
可扩展的事件驱动模型；
### （4）更低的资源消耗
对象池复用技术；
零拷贝技术；
## 2、Netty和Tomcat的最大区别：对通信协议的支持
#### Tomcat
一个HTTP Server，主要解决HTTP协议层的传输；
Tomcat需要遵循Servlet规范。Tomcat3.0之前采用BIO。6.x版本之后支持NIO，性能得到较大提升。
#### Netty
支持多种应用层的协议，比如HTTP、SSH、TLS/SSL等协议；
能够自定义应用层协议；
Netty由于不受到Servlet规范的约束，可以最大发挥NIO的特性。
#### 总结
由于Tomcat在作为HTTP服务器的成熟度和稳定性更好，所以假如你仅仅只需要一个HTTP服务器，推荐使用Tomcat。但是，假如你需要做面向TCP的网络应用开发，那么推荐使用Netty。
## 3、Netty和Mina、Grizzly的对比
Mina是Apache Directory服务底层的NIO框架，它和Netty都是Trustin Lee的作品，所以两者在设计理念上基本一致，但是Netty是Mina的升级，解决了Mina一些设计上的问题，比如Netty提供可扩展的编解码接口、优化了ByteBuffer的分配方式，让用户使用起来更便捷、安全。而Grizzly出身于Sun公司，从设计理念上看，Grizzly没有Netty优雅，几乎是对JDK NIO的初级封装，目前业界使用的范围也比较小。
# 三、优势
[[#1、Netty相比JDK NIO的突出优势]]
2、社区成熟
# 四、应用
![[1716802878593.png]]
# 五、原理
## 1、Netty整体结构
### （1）图示
![[6e4c29ea3c46fd074a57bb80e10742b.png]]
### （2）主要组成
核心层
协议支持层
传输服务层
## 2、Netty逻辑架构
### （1）图示
![[4fc29e69d3360483b4d59e6cce9f985.png]]
### （2）主要组成
网络通信层
事件调度层

