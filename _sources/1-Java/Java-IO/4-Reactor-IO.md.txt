# Reactor 模型

## 传统服务设计[^1]

![alt text](image-12.png)

![alt text](image-14.png)

1. Each thread synchronously blocks in the accept
socket call waiting for a client connection request;[1^]
2. A client connects to the server, and the connection is
accepted;
3. The new client’s HTTP request is synchronously read
from the network connection;
4. The request is parsed;
5. The requested file is synchronously read;
6. The file is synchronously sent to the client.

这种设计的缺点：

- 可伸缩性： 为每个请求分配一个线程，高负载下容易导致资源耗尽（可以使用线程池技术缓解此问题）。
- 资源利用： 线程在等待 IO 时会完全阻塞，导致资源利用率低。
- 通吐量低： 频繁的线程切换，使其处理大量小型请求时吞吐量受限。

## Reactor[^1]

### 连接

![alt text](image-19.png)

1. The Web Server registers an Acceptor with the
Initiation Dispatcher to accept new connections;
2. The
Web Server invokes event loop of the Initiation
Dispatcher;
3. A client connects to the Web Server;
4. The Acceptor is notified by the Initiation
Dispatcher of the new connection request and the
Acceptor accepts the new connection;
5. The Acceptor creates an HTTP Handler to service
the new client;
6. HTTP Handler registers the connection with the
Initiation Dispatcher for reading client request data (that is, when the connection becomes “ready
for reading”);
7. The HTTP Handler services the request from the
new client.

### 读写

![alt text](image-20.png)

1. The client sends an HTTP GET request;
2. The Initiation Dispatcher notifies the HTTP
Handler when client request data arrives at the server;
3. The request is read in a non-blocking manner such that
the read operation returns EWOULDBLOCK if the operation would cause the calling thread to block (steps 2
and 3 repeat until the request has been completely read);
4. The HTTP Handler parses the HTTP request;
5. The requested file is synchronously read from the file
system;
6. The HTTP Handler registers the connection with
the Initiation Dispatcher for sending file data
(that is, when the connection becomes “ready for writing”);
7. The Initiation Dispatcher notifies the HTTP
Handler when the TCP connection is ready for writing;
8. The HTTP Handler sends the requested file to the
client in a non-blocking manner such that the write
operation returns EWOULDBLOCK if the operation
would cause the calling thread to block (steps 7 and
8 will repeat until the data has been delivered completely).

## Reactor 单线程[^3]

![alt text](image-15.png)

一般不会这样子使用。

## Reactor 使用线程池[^3]

![alt text](image-16.png)

这就是 netty 中  `bossGroup` 和 `workerGroup` 使用同一个 EventLoopGroup 的模式。

{lineno-start=1 emphasize-lines="3"}
```Java
    EventLoopGroup eventLoopGroup = new NioEventLoopGroup(); // (1)
    ServerBootstrap server = new ServerBootstrap(); // (2)
    server.group(eventLoopGroup)
        .channel(NioServerSocketChannel.class)
        //...
```

## 多个 Reactor 

![alt text](image-17.png)

这其是就是 netty 中 `bossGroup` 和 `workerGroup`，一个专门用来查询客户端的连接事件，一个专门用来查询连接的读写事件。

{lineno-start=1 emphasize-lines="4"}
```Java
    EventLoopGroup bossGroup = new NioEventLoopGroup(); // (1)
    EventLoopGroup workerGroup = new NioEventLoopGroup();
    ServerBootstrap server = new ServerBootstrap(); // (2)
    server.group(bossGroup, workerGroup)
        .channel(NioServerSocketChannel.class) // (3)
        .childHandler(new ChannelInitializer<SocketChannel>() { // (4)
            @Override
            public void initChannel(SocketChannel ch) throws Exception {
            
            }
        })
        .option(ChannelOption.SO_BACKLOG, 128)          // (5)
        .childOption(ChannelOption.SO_KEEPALIVE, true); // (6)
```

## Proactor[^2]

经典服务设计使用的是阻塞IO，不能有效的利用资源，Reactor 服务设计使用 IO 多路复用，是目前使用最广泛的模式，也有进一步的优化设计——Proactor，但目前使用并不广泛，主要是因为它依赖于操作系统提供的异步 IO，而且不太符合人类直觉，基于它设计的程序理解起来比较困难。（Linux 中 的 io_uring 异步 io 估计会流性起来 ）。

[^1]: [Reactor. An Object Behavioral Pattern for De-multiplexing and Dispatching Handles for Synchronous Events. Douglas C. Schmidt](https://www.dre.vanderbilt.edu/~schmidt/PDF/Reactor.pdf)
[^2]: [Proactor An Object Behavioral Pattern for De-multiplexing and Dispatching Handlers for Asynchronous Events. Irfan Pyarali, Tim Harrison, and Douglas C. Schmidt Thomas D. Jordan](https://www.dre.vanderbilt.edu/~schmidt/PDF/proactor.pdf)
[^3]: [Scalable IO in Java. Doug Lea. State University of New York at Oswego](https://gee.cs.oswego.edu/dl/cpjslides/nio.pdf)

