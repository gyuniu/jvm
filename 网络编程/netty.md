# netty

Netty是一款异步的事件驱动的网络应用程序框架，支持快速地开发可维护的**高性能**的面向协议的服务器和客户端。



| Netty的主要构建块    | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| Channel              | 可以把Channel看作是传入或者传出数据的载体。可以被打开或者被关闭，连接或者断开连接。 |
| 回调                 | 是一个方法的引用。这个引用提供给了另外的方法。另外的方法可以在适当的时候调用这个方法。netty在内部使用了回调来处理事件：当一个回调被触发时，相关的事件可以被一个interface  ChannelHandler的实现处理。 |
| Future               | ◾   其对象可以看作是一个异步操作的结果的占位符；它将在未来的某个时候完成，并提供对其结果的访问。<br>◾   JDK预置了接口java.util.concurrent.Future，但其提供的实现，只允许手动检查对应的操作是否已经完成，或者一直阻塞直到它完成。<br/>◾   Netty提供了自己的实现——ChannelFuture，用于在执行异步操作的时候使用。它提供了几种额外的方法，这些方法使得我们能够注册一个或者多个ChannelFutureListener实例。监听器的回调方法operationComplete()，将会在对应的操作完成时被调用（注：如果在ChannelFutureListener添加到ChannelFuture的时候，ChannelFuture已经完成，那么该listener将会被直接地通知）。然后监听器可以判断该操作是成功完成了还是出错了，如果是后者，我们可以检索产生地Throwable。<br/>◾   简而言之，由ChannelFutureListener提供地通知机制消除了手动检查对应的操作是否完成的必要。 |
| 事件和ChannelHandler |                                                              |



回调的案例：被回调触发的ChannelHandler

``` java
// 当一个新的连接已经被建立时，ChannelHandler的channelActive()回调方法将会被调用。并打印出一条信息
public class ConnectHandler extends ChannelInboundHandlerAdapter{
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("Client "+ctx.channel().remoteAddress() + " connected");
    }
}
```

Future案例：异步建立连接。

```java
/*
每个Netty的出战IO操作都将返回一个ChannelFuture；也就是说，它们都不会阻塞。Netty完全是异步和事件驱动的。
*/

```









