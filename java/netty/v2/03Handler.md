

# Handler

## 常见Handler

### 1.粘包半包的那些Handler

### 2.LoggingHandler

当添加`.addLast("logging", new LoggingHandler(LogLevel.INFO))`这行代码时

Netty就会以给定的日志级别打印出`LoggingHandler`中的日志。

可以对入站\出站事件进行日志记录，从而方便我们进行问题排查。

假如现在添加这行代码访问http://127.0.0.1:8007/Action?name=1234510

```java
19:10:52.089 [nioEventLoopGroup-2-6] INFO io.netty.handler.logging.LoggingHandler - [id: 0x4a9db561, L:/127.0.0.1:8007 - R:/127.0.0.1:53151] REGISTERED
19:10:52.089 [nioEventLoopGroup-2-6] INFO io.netty.handler.logging.LoggingHandler - [id: 0x4a9db561, L:/127.0.0.1:8007 - R:/127.0.0.1:53151] ACTIVE
19:10:52.090 [nioEventLoopGroup-2-6] DEBUG com.bihang.seaya.server.handler.SeayaHandler - io.netty.handler.codec.http.DefaultHttpRequest
19:10:52.090 [nioEventLoopGroup-2-6] DEBUG com.bihang.seaya.server.handler.SeayaHandler - uri/Action?name=1234510
19:10:52.090 [nioEventLoopGroup-2-6] INFO io.netty.handler.logging.LoggingHandler - [id: 0x4a9db561, L:/127.0.0.1:8007 - R:/127.0.0.1:53151] CLOSE
19:10:52.090 [nioEventLoopGroup-2-6] INFO io.netty.handler.logging.LoggingHandler - [id: 0x4a9db561, L:/127.0.0.1:8007 ! R:/127.0.0.1:53151] INACTIVE
19:10:52.090 [nioEventLoopGroup-2-6] INFO io.netty.handler.logging.LoggingHandler - [id: 0x4a9db561, L:/127.0.0.1:8007 ! R:/127.0.0.1:53151] UNREGISTERED
```

如果没有这行代码的打印信息

```java
19:15:02.292 [nioEventLoopGroup-2-2] DEBUG com.bihang.seaya.server.handler.SeayaHandler - io.netty.handler.codec.http.DefaultHttpRequest
19:15:02.292 [nioEventLoopGroup-2-2] DEBUG com.bihang.seaya.server.handler.SeayaHandler - uri/Action?name=1234510
```

### 3.IdleStatHandler

检测空闲时间的Handler

如果超过读空闲时间 会触发`IdleStat#READER_IDLE`事件

如果超过写空闲时间 会触发`IdleStat#READ_IDLE`事件

如果超过读写空闲时间 会触发`IdleStat#ALL_IDLE`事件

```java
//第一个参数 读空闲时间 seconds
//第二个参数 写空闲时间 seconds
//第三个参数 读写空闲时间 seconds
pipeline.addLast(new IdleStateHandler(5,5,5));
pipeline.addLast(new ChannelDuplexHandler(){
    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
      IdleState evt1 = (IdleState) evt;

      if(evt1.equals(IdleState.READER_IDLE)){
      	//todo
      }else{
      	//todo...
      }
    }
});
```

### 4.ByteToMessageCodec

ByteBuf和自定义Message的编解码器

不能是@Shareable

不用自己管理Bytebuf

### 5.MessageToMessageCodec

不同Message之间的编解码器

可以使@Shareable

不用自己管理ByteBuf

### 6.SimpleChannelInboundHandler

处理某种特定消息(泛型)的入栈处理器

可以使用@Shareable



## @Shareable

**标注一个channel handler可以被多个channel安全地共享。**

ChannelHandlerAdapter还提供了实用方法isSharable()。如果其对应的实现被标注为Sharable，那么这个方法将返回true，表示它可以被添加到多个ChannelPipeline中。

因为一个ChannelHandler可以从属于多个ChannelPipeline，所以它也可以绑定到多个ChannelHandlerContext实例。用于这种用法的ChannelHandler必须要使用@Sharable注解标注；否则，试图将它添加到多个ChannelPipeline时将会触发异常。显而易见，为了安全地被用于多个并发的Channel（即连接），这样的ChannelHandler必须是线程安全的。

**共享的Handler可以在pipeLine.addLast的时候提成公共变量，每个pipeLine中都是一个实例**



## ChannelOutboundHandler会声明一个read方法

## 异常处理

入站异常处理：

1、ChannelHandler.exceptionCaught()的默认实现是简单地将当前异常转发给ChannelPipeline 中的下一个 ChannelHandler；
2、如果异常到达了 ChannelPipeline 的尾端，它将会被记录为未被处理；
3、要想定义自定义的处理逻辑，你需要重写 exceptionCaught()方法。一般将这个Channelhandler放在 ChannelPipeline 的最后，确保所有的入站异常都总会被处理。

```java
@Override
public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    cause.printStackTrace();
    ctx.channel().close();
}
```

出站异常处理：

```
@Override
public void channelActive(ChannelHandlerContext ctx) {
    ChannelFuture channelFuture = ctx.writeAndFlush(Unpooled.copiedBuffer("Netty rock!", CharsetUtil.UTF_8));
    //出站异常处理
    channelFuture.addListener(new ChannelFutureListener() {
        @Override
        public void operationComplete(ChannelFuture future) throws Exception {
            if (!future.isSuccess()) {
                future.cause().printStackTrace();
                future.channel().close();
            }
        }
    });
}
```