# RPC框架

本文将由浅入深介绍有关框架的内部原理，重点讲解 RPC 框架的基本概念，并从编码层、传输协议层和网络通信层分析其分层设计

## 1. RPC的概念模型

### 1.1 一次RPC 的完整过程

**IDL (Interface description language)文件**

IDL通过一种中立的方式来描述接口，使得在不同平台上运行的对象和用不同语言编写的程序可以相互通信

生成代码

通过编译器工具把 IDL文件转换成语言对应的静态库

编解码

从内存中表示到字节序列的转换称为编码，反之为解码，也常叫做序列化和反序列化

通信协议

规范了数据在网络中的传输内容和格式。除必须的请求/响应数据外，通常还会包含额外的元数据

网络传输

通常基于成熟的网络库走 TCP/UDP传输

### 1.2 RPC 的好处

1.单一职责，有利于分工协作和运维开发
2.可扩展性强，资源使用率更优
3.故障隔离，服务的整体可靠性更高

## 2. THeader 协议

SOCKET API\
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9648d0f10e584391bb6e09b433d7e01d~tplv-k3u1fbpfcp-watermark.image?)

协议层-协议构造:

LENGTH:数据包大小,不包含自身

HEADER MAGIC:标识版本信息,协议解析时候快速校验

SEQUENCE NUMBER:表示数据包的seqID,可用于多路复用，单连接内递增

HEADER SIZE:头部长度，从第14个字节开始计算一直到 PAYLOAD前

PROTOCOL ID:编解码方式,有Binary和Compact两种

TRANSFORM ID:压缩方式，如 zlib和snappy

INFO ID:传递一些定制的meta 信息

PAYLOAD:消息体

## 3. 小结

1.框架通过中间件来注入各种服务治理策略，保障服务的稳定性

2.通过提供合理的默认配置和方便的命令行工具可以提升框架的易用性

3.框架应当提供丰富的扩展点，例如核心的传输层和协议层

4.观测性除了传统的Log、Metric和Tracing之外，内置状态暴露服务也很有必要

5.性能可以从多个层面去优化，例如选择高性能的编解码协议和网络库