# ares 框架中的 RPC 

## 代码结构

首先先看 RPC 模块的类结构：

1. ares 中的协议类都继承自 ares.io.Msg 类，这个类本身提供了 send()、dispatch()、process() 等方法。
2. ares.io.Msg 类继承了 java.lang.Runnable 接口和 ares.io.Coder 接口。
   1. java.lang.Runable 提供了 run() 方法。
   2. ares.io.Coder 提供了 decode() 和 encode() 两个方法，同时又继承了 ares.Checker 接口。
   3. ares.Checker 提供了 check() 方法。
3. ares.io.Msg 类中同时又包含一个内部类 MsgHeader，MsgHeader 中包含 pvid 和 type 两个属性。

整体结构图如下图：

![rpc_in_ares](https://raw.githubusercontent.com/3rdyeah/3rdpics/master/picbed/rpc_in_ares.png)

具体的 RPC 类在使用的时候，在 process() 方法中实现自己的逻辑，当 ares 框架接收到这条协议的时候，就会通过 run() 方法，调用到 process()，继而实现对我们自己实现的逻辑的调用。

## 代码分析

### 消息基类

首先看以下消息的基类 Msg：

```java
package ares.io;

import ares.AUtil;
import ares.CodeUtil;
import io.netty.buffer.Unpooled;

public abstract class Msg implements Runnable, Coder {
	protected Session session;
	protected Object context;
	protected MsgHeader header;
	public static final int MSG_LEN_LEN = 4;
	
	public Msg() {
		this(new MsgHeader());
	}
	
	public Msg(MsgHeader header) {
		this.header = header;
	}
	
	@Override
	public BuffStream encode(BuffStream _dst) {
		return encode(_dst, null);
	}
	
	/**
	 * 在逻辑线程执行，Msg执行顺序不做保证
	 * @throws Exception
	 */
	public void process() throws Exception {};
	
	public abstract BuffStream encode(BuffStream _dst, Session session);
	
	public boolean send(Session session) {
		if (null == session) {
			IO.logger.error(this, new NullPointerException("Session is Null"));
			return false;
		}
		return session.send(this);
	}

	@Override
	public void run() {
		try {
			process();
		} catch (Throwable e) {
			IO.logger.error("msg process", e);
		}
	}
	
	/**
	 * 在网络线程执行，保证同一session的Msg执行顺序
	 * @throws Exception
	 */
	public void dispatch() throws Exception {
		session
		.getPort()
		.getIO()
		.getExecutor()
		.execute(this);
	}
	
	@SuppressWarnings("unchecked")
	public <T extends Session> T getSession() {
		return (T) session;
	}

	public void setSession(Session session) {
		this.session = session;
	}
	
	public MsgHeader getHeader() {
		return header;
	}
	
	public void setHeader(MsgHeader header) {
		this.header = header;
	}
	
	@SuppressWarnings("unchecked")
	public <T> T getContext() {
		return (T) context;
	}

	public void setContext(Object context) {
		this.context = context;
	}
	
	public static class MsgHeader implements Coder {
		private static final MsgHeader NULL = new MsgHeader((short)0, (short)0);
		public static final int HEADER_LENGTH = NULL.encode(new BuffStream(Unpooled.buffer())).readableBytes();
		
		private short pvid;
		private short type;
		
		public MsgHeader() {
		}
		
		public MsgHeader(MsgHeader other) {
			this.pvid = other.pvid;
			this.type = other.type;
		}
		
		public MsgHeader(short pvid, short type) {
			this.pvid = pvid;
			this.type = type;
		}
		
		@Override
		public final void decode(BuffStream _src) {
			pvid = CodeUtil.decodeShort(_src);
			type = CodeUtil.decodeShort(_src);
		}

		@Override
		public final BuffStream encode(BuffStream _dst) {
			CodeUtil.encodeShort(pvid, _dst);
			CodeUtil.encodeShort(type, _dst);
			return _dst;
		}
		
		public short getType() {
			return type;
		}

		public short getPvid() {
			return pvid;
		}
		
		@Override
		public String toString() {
			return AUtil.getString("pvid:", pvid, ",type:", type);
		}

		public void setPvid(short pvid) {
			this.pvid = pvid;
		}

		public void setType(short type) {
			this.type = type;
		}

		@Override
		public void check() {
		}
	}
}

```

主要关注几个方法：

```java
public void process() throws Exception {}
```

这个方法是收到消息后具体要实现什么逻辑，在继承自 Msg 的子类中各自实现，在逻辑线程执行，不保证消息的执行顺序。

```java
public void dispatch() throws Exception {
	session
	.getPort()
	.getIO()
	.getExecutor()
	.execute(this);
}
```

这个方法同样是收到消息后去执行逻辑，和上面不同的是这个方法在网络线程执行，保证同一个 session 的消息执行顺序。

```java
public abstract BuffStream encode(BuffStream _dst, Session session);
```

这个方法是发送消息前进行编码，在子类中各自实现。

对应 encode() 方法，应该有一个 decode() 方法，这在 Msg 类中没有体现，其实是因为 Msg 继承了 Coder 接口，decode() 方法也同样在子类中各自实现。

```java
public boolean send(Session session) {
	if (null == session) {
		IO.logger.error(this, new NullPointerException("Session is Null"));
		return false;
	}
	return session.send(this);
}
```

上面这个是发送消息的方法，具体是现在 session.send(this) 这一步种，其实就是把消息通过 netty 的 ChannelHandlerContext 实例发送出去，如下：

```java
public boolean send(Msg msg) {
	if (!beforeSend(msg)) {
		return false;
	}
	ctx.writeAndFlush(msg).addListener(ChannelFutureListener.FIRE_EXCEPTION_ON_FAILURE);
	return true;
}
```

### 几个 handler

在 ares 框架中，消息的传递时通过 netty 来实现的，我们需要实现 RPC 消息的编解码 handler，编码器 handler 继承了 netty 的 MessageToByteEncoder 类，解码器 handler 继承了 netty 的 ByteToMessageDecoder 类。

这里我们先看编码 handler：MsgEncoder 类。

```java
package ares.io.handler;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandler.Sharable;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.MessageToByteEncoder;
import io.netty.util.Attribute;
import ares.debug.MsgDebug;
import ares.io.BuffStream;
import ares.io.Const;
import ares.io.Msg;
import ares.io.Msg.MsgHeader;
import ares.io.Port;
import ares.io.Session;

@Sharable
public class MsgEncoder extends MessageToByteEncoder<Msg> {
	private final Port port;

	public MsgEncoder(Port port) {
		this.port = port;
	}

	@Override
	protected void encode(ChannelHandlerContext ctx, Msg msg, ByteBuf out) throws Exception {
		Attribute<Session> sessionAtr = ctx.channel().attr(Const.CHANNEL_ATTR_SESSION);
		Session session = sessionAtr.get();
		encode(msg, out, session);
	}

	private void encode(Msg msg, ByteBuf out, Session session) throws Exception {
		BuffStream bs = new BuffStream(out);
		bs.ensureWritable(Msg.MSG_LEN_LEN);
		bs.writerIndex(Msg.MSG_LEN_LEN);
		bs.readerIndex(Msg.MSG_LEN_LEN);
		MsgHeader header = msg.getHeader();
		header.encode(bs);
		msg.encode(bs, session);
		int msgSize = bs.readableBytes() - MsgHeader.HEADER_LENGTH;
		port.getNode().checkSend(msg, msgSize);
		
		session.doEncodeFilter(bs);
		
		bs.setInt(0, bs.readableBytes());
		bs.readerIndex(0);
		port.getStatistics().onSend(header.getType(), msgSize);
		MsgDebug.onSendMsg(msg, session);
	}
}
```

这个类包含一个属性——Port 类型的 port 属性，这个 Port 类型需要解释一下，并不是简单的端口的概念，它同时包含了 ip 和端口还有其他多个属性，现在我们把它看作一个包含 ip 和端口的地址类型即可。

在重写的的 encode() 方法中，从上下文 ChannelHanlderContext 中获取到消息的接收者的 Session，将这个 Session 传给私有的 encode() 方法，在这个方法中我们可以看到：

1. 首先是构造了一个 BuffStream（继承自 netty 的 ByteBuf，拥有一个 ByteBuf 类型的属性来存放具体数据）类型的变量 bs。
2. 检查是否可写，并且将 bs 的写索引和读索引都设置为 Msg.MSG_LEN_LEN 的位置，这个值在 ares 中被设置为 4 字节。
3. 通过 MsgHeader 的 encode() 方法向 bs 中写入消息头（写入 pvid 和 type）。
4. 通过 Msg 的 encode() 方法将 msg 消息内容写入 bs，这里 Msg 的 encode() 方法是个 override 方法，所以是在具体的继承自 Msg 的协议子类中具体各自实现的，具体逻辑就是把 msg 消息中的每个属性依次写道 bs 中。
5. port.getNode().checkSend(msg, msgSize) 这里实在检查这个消息是否可以发送，检查了消息的类型和大小。
6. session.doEncodeFilter(bs) 这里是对 bs 又进行了一次编码。
7. 向 bs 中在所以 0 的位置写入 bs 可读字节数的大小，这里就呼应了第二步中为何将写索引置为 4，其实就是预留了写消息大小的空间，并将读索引置为 0。
8. port.getStatistics().onSend(header.getType(), msgSize) 和编码其实不相关，这里是记录了一些统计数据。
9. MsgDebug.onSendMsg(msg, session) 故名思意，就是打印调试信息。

这样，编码 handler 就完成了对消息的编码处理。

接下来看解码 handler：MsgDecoder。

```java
package ares.io.handler;

import java.util.List;

import ares.io.BuffStream;
import ares.io.Const;
import ares.io.Msg;
import ares.io.Port;
import ares.io.Session;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.handler.codec.ByteToMessageDecoder;
import io.netty.util.Attribute;

public class MsgDecoder extends ByteToMessageDecoder {
	private final Port port;
	
	public MsgDecoder(Port port) {
		this.port = port;
	}

	@Override
	protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
		BuffStream bs = new BuffStream(in);
		Attribute<Session> sessionAtr = ctx.channel().attr(Const.CHANNEL_ATTR_SESSION);
		Session session = sessionAtr.get();
		session.doDecodeFilter(bs);
		
		Msg.MsgHeader header = new Msg.MsgHeader();
		header.decode(bs);
		
		Msg msg = port.getNode().createMsg(header, session, bs);
		if (null != msg) {
			out.add(msg);
		}
	}
}
```

这个类其实也不难理解，就是和 MsgEncoder 反着来，只是一些解码和检查的逻辑在 port.getNode().createMsg(header, session, bs) 中实现了。

### 消息的收发

其实当我们知道 ares 封装了 netty 之后，就可以很容易理解 ares 的消息收发是如何实现的了，其实就是使用了 netty 来收发消息。

消息的接收我们需要从 IO 类开始，层层深入到 Port 中，比如我们从 IO 类的 start() 方法开始：

```java
// IO 类
public Builder start() {
	List<ChannelFuture> futures = new LinkedList<ChannelFuture>();
	for (Service ser : sers.values()) {
		futures.addAll(ser.start());
	}
	return new Builder(futures);
}

// Service 类
public final List<ChannelFuture> start() {
	List<ChannelFuture> futures = new LinkedList<ChannelFuture>();
	for (Node node : nodes.values()) {
		futures.addAll(node.start());
	}
	IO.logger.info("Service:", this, " Start");
	return futures;
}

// Node 类
public List<ChannelFuture> start() {
    List<ChannelFuture> futures = new LinkedList<ChannelFuture>();
    for (Port port : ports.values()) {
        ChannelFuture future = port.start();
        futures.add(future);
    }
    IO.logger.info("Node:", this, " Start");
    return futures;
}

// Port 类，这里选取了 Port 的一个子类 Connector 类的 start 方法
// private final Bootstrap b = new Bootstrap();
@Override
public final ChannelFuture start() {
    final ConnectorNodeConf nodeConf = node.getConf();
    b.group(getIO().getWorkerGroup())
        .channel(NioSocketChannel.class)
        .option(ChannelOption.TCP_NODELAY, nodeConf.tcpNodelay)
        .option(ChannelOption.SO_KEEPALIVE, nodeConf.tcpKeepalive)
        .option(ChannelOption.RCVBUF_ALLOCATOR, new AdaptiveRecvByteBufAllocator())
        .option(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT)
        .option(ChannelOption.WRITE_BUFFER_WATER_MARK, new WriteBufferWaterMark(nodeConf.outputBuff, nodeConf.outputBuff))
        .handler(new ChannelInitializer<SocketChannel>() {
            @Override
            public void initChannel(SocketChannel ch) throws Exception {
                ChannelPipeline p = ch.pipeline();
                addHandler(p);
            }
        });

    int revBuff = nodeConf.revBuff;
    if (revBuff != Conf.NULL && revBuff > 0) {
        b.option(ChannelOption.SO_RCVBUF, revBuff);
    }
    int sndBuff = nodeConf.sndBuff;
    if (sndBuff != Conf.NULL && sndBuff > 0) {
        b.option(ChannelOption.SO_SNDBUF, sndBuff);
    }

    IO.logger.info("Port:", this, " Start");
    return doConnect();
}
```

其实从 Connector 类的 start() 方法就可以看出，最终还是通过一个 Bootstrap 对象绑定了一个 NioSocketChannel 来传递消息。

[^注]: 这里解释一下，每个通过 ares 启动的 IO 终端（这里不一定是客户端 Connector 还是服务器 Accepter）都不一定绑定了多少个其他终端，具体要看服务器的配置，每一个 Port 绑定一个终端。

发送消息我们在上面说 Msg 类的末尾处已经提到过，通过 Msg 类的 session 的 send() 方法，将消息内容写入 ChannelHandlerContext 中来实现发送消息。