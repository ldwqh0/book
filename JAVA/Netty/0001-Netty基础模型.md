# Netty主要模型

Netty 的核心架构主要基于 Reactor 线程模型，并结合了 事件驱动 (Event-driven) 的设计理念。其主要模型可以从线程模型和核心组件两个维度来理解：

## 线程模型：改进的主从 Reactor 模型

Netty 默认采用主从 Reactor 多线程模型，这是其高性能的关键

* BossGroup (主 Reactor)：专门负责接收客户端的连接请求 (Accept 事件)。它将建立好的连接 (SocketChannel) 注册到 WorkerGroup
  中。
* WorkerGroup (从 Reactor)：专门负责网络的读写、业务处理及事件分发。
* NioEventLoop：线程模型中的核心执行单元，每个 EventLoop 包含一个线程、一个 Selector 和一个任务队列。它循环执行三个任务：  
  轮询注册在其上的 Channel 的 I/O 事件。  
  处理已就绪的 I/O 事件。  
  运行任务队列中的非 I/O 任务。

## 核心组件模型

Netty 通过一套精简的组件实现了数据的流转与处理：

* Channel: 对网络连接的抽象，代表了一个开放的连接（如 TCP 端口），具备读写能力。
* ChannelPipeline: 拦截流经 Channel 的入站和出站事件的处理器链。
* ChannelHandler: 实际处理业务逻辑的组件。入站处理器 (ChannelInboundHandler) 处理接收到的数据，出站处理器 (
  ChannelOutboundHandler) 处理发送出去的数据。
* ByteBuff: Netty 自定义的字节容器，相比 JDK 的 ByteBuffer 提供了更灵活的 API 和性能优化（如零拷贝支持）。

## 其他关键特性

* 异步模型：基于 Future 和 Promise 机制，所有的 I/O 操作都是异步非阻塞的，调用后立即返回，通过监听器 (Listener) 获取结果。
* 零拷贝 (Zero-Copy)：通过直接内存引用、CompositeByteBuf 组合视图以及 FileRegion 文件传输等技术，最大限度减少内存复制开销。
