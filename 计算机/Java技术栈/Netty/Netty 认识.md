# 一、概念


# 二、核心知识


# 三、Netty和JDK NIO、Tomcat的比较
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
#### Netty
支持多种应用层的协议，比如HTTP、SSH、TLS/SSL等协议；
能够自定义应用层协议；
总结：
#### 总结
Tomcat需要遵循Servlet规范。Tomcat3.0之前采用BIO。6.x版本之后支持NIO，性能得到较大提升。
Netty由于不受到Servlet规范的约束，可以最大发挥NIO的特性。
由于Tomcat在作为HTTP服务器的成熟度和稳定性更好，所以假如你仅仅只需要一个HTTP服务器，推荐使用Tomcat。
# 四、原理
