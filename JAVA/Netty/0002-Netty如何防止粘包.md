# Netty 如何防止粘包

在 Netty 中处理“粘包”和“拆包”的本质是定义消息边界。由于 TCP 是面向字节流的协议，没有消息界限，Netty 提供了多种内置的
Decoder（解码器） 来解决这个问题。

## 常见的解决方案有以下四种：

1. 固定长度（FixedLengthFrameDecoder）  
   不管发送方发多快，解码器每次只读取固定字节数。  
   适用场景：消息长度恒定的简单协议。  
   缺点：浪费带宽（短消息需补位）。
2. 特定分隔符（DelimiterBasedFrameDecoder）  
   通过消息末尾的特殊符号（如 \n 或 \$_\$）来划分消息边界。  
   典型实现：LineBasedFrameDecoder（专门处理 \n 或 \r\n）。  
   适用场景：文本协议（如 FTP、SMTP）。
3. 消息头指定长度（LengthFieldBasedFrameDecoder） —— 最通用
   在消息体前面加一个长度字段。解码器先读长度，再根据长度读取后面的数据。  
   原理：这是最强大的解码器，支持复杂的协议头配置（如跳过偏移量、长度修正等）。  
   适用场景：自定义私有协议、大多数高性能 RPC 框架。
4. 封装自定义协议
   你可以继承 ByteToMessageDecoder 手动编写逻辑：

```java
public class MyDecoder extends ByteToMessageDecoder {

    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
        if (in.readableBytes() < 4) return; // 假设长度字段占 4 字节
        in.markReaderIndex();
        int length = in.readInt();
        if (in.readableBytes() < length) {
            in.resetReaderIndex(); // 剩余字节不够，回滚指针等下次数据
            return;
        }
        out.add(in.readBytes(length)); // 提取完整消息
    }

}
```

## 总结与建议

如果是 HTTP 协议：直接用 HttpServerCodec，它内置了处理逻辑。  
如果是 自定义 TCP 协议：首选 LengthFieldBasedFrameDecoder，它性能最好且配置灵活。
