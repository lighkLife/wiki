# Java NIO

## nio Support

NIO 就是非阻塞 IO，Java NIO 相关的类都在`java.nio`包下。主要包含：

![alt text](image-8.png)

### Channels

Channels 非阻塞读写的通道，实际上是文件或 socket 连接（文件描述符）。

以 SocketChannelImpl 为例

![alt text](image-10.png)

### Buffers
Buffers 缓冲区数组，可以直接被 Channels 读写

### Selectors
Selectors IO 多路复用的查询器，可以查询出哪些 Channel 有 IO 事件

### SelectionKeys
SelectionKeys 维护 IO 事件的状态（连接建立、可读、可写），绑定相关的 Selector 和 Channel

## Java NIO 实现 Reactor 

以下使用 [Doug Lea](https://www.cs.oswego.edu/~dl/) 老爷子的 Scalable IO in Java [1^] 演讲搞中的例子，来使用 JAVA NIO 实现单线程的 reactor 模型。

![alt text](image-9.png)

### Set up

{lineno-start=1 emphasize-lines="17"}
```Java 
public class Reactor {
    final Selector selector;
    final ServerSocketChannel serverSocket;

    /**
     * 1. Set up
     */
    public Reactor(int port) throws IOException {
        // SelectorProvider provider = SelectorProvider.provider();
        // AbstractSelector selector = provider.openSelector();
        // ServerSocketChannel serverSocket = provider.openServerSocketChannel();

        selector = Selector.open();
        serverSocket = ServerSocketChannel.open();
        serverSocket.configureBlocking(false);
        SelectionKey selectionKey = serverSocket.register(selector, SelectionKey.OP_ACCEPT);
        selectionKey.attach(new Acceptor());
    }
}
```

初始化主要是创建查询器和服务端 Socket，主要包含以下操作：

- 打开一个事件查询器 Selector
- 打开一个服务端 Socket 通道，用来监听客户端的 TCP 连接
- 在服务端 Socket 通道上注册 Selector，查询`OP_ACCEPT`事件
- 在对应 SelectionKey 上绑定 Acceptor， 处理建立的连接

### Dispatch Loop

{lineno-start=1 emphasize-lines="16,25"}
```Java
    // class Reactor continued

    /**
     * 2. Dispatch Loop
     */
    @Override
    public void run() {
        // normally in a new Thread
        try {
            while (!Thread.interrupted()) {
                // blocking, util at least one channel is selected
                selector.select();
                Set<SelectionKey> selected = selector.selectedKeys();
                Iterator<SelectionKey> it = selected.iterator();
                while (it.hasNext()) {
                    dispatch(it.next());
                    selected.clear();
                }
            }
        } catch (IOException e) {
            // handle ex
        }
    }

    private void dispatch(SelectionKey key) {
        Runnable runnable = (Runnable) key.attachment();
        if (runnable != null) {
            runnable.run();
        }
    }
```

这一部主要是持续监听客户端的连接，即有新的 `OP_ACCEPT` 事件后，就运行对应 SelectionKey 中绑定的处理逻辑。

### Acceptor

{lineno-start=1 emphasize-lines="12"}
```Java
    // class Reactor continued
    
    /**
     * 3. Acceptor
     */
    public class Acceptor implements Runnable {
        @Override
        public void run() {
            try {
                SocketChannel channel = serverSocket.accept();
                if (channel != null) {
                    new Handler(selector, channel);
                }
            } catch (IOException e) {
                // handle ex
            }
        }
    }
```

这一步就是服务端同意客户端发起的连接建立请求，并产生一个服务端和客户端之间的 SocketChannel，然后分配一个 Handler 去专门处理这个连接上读写操作。

### Handler Setup

{lineno-start=1 emphasize-lines="17,18,19,20"}
```Java
public class Handler implements Runnable {
    private static final int MAX_IN = 5 * 1024;
    private static final int MAX_OUT = 5 * 1024;

    final SocketChannel socket;
    final SelectionKey selectionKey;
    ByteBuffer input = ByteBuffer.allocate(MAX_IN);
    ByteBuffer output = ByteBuffer.allocate(MAX_OUT);
    static final int READING = 0, SENDING = 1;
    int state = READING;


    public Handler(Selector selector, SocketChannel socketChannel) throws IOException {
        socket = socketChannel;
        socket.configureBlocking(false);
        // Optionally try first read now
        selectionKey = socket.register(selector, 0);
        selectionKey.attach(this);
        selectionKey.interestOps(SelectionKey.OP_READ);
        selector.wakeup();
    }

    boolean inputIsComplete() { /* ... */ }
    boolean outputIsComplete() { /* ... */ }
    void process() { /* ... */ }
}
```

### Request handling

{lineno-start=1 emphasize-lines="5,6,8,9,16,26"}
```Java
// class Handler continued 
    @Override
    public void run() {
        try {
            if (state == READING) {
                read();
            }
            else if (state == SENDING) {
                send();
            }
        } catch (IOException ex) {
            /* ... */
        }
    }

    void read() throws IOException {
        socket.read(input);
        if (inputIsComplete()) {
            process();
            state = SENDING;
            // Normally also do first write now
            selectionKey.interestOps(SelectionKey.OP_WRITE);
        }
    }

    void send() throws IOException {
        socket.write(output);
        if (outputIsComplete()) sk.cancel();
    }
```

[1^]: [Scalable IO in Java. Doug Lea. State University of New York at Oswego](https://gee.cs.oswego.edu/dl/cpjslides/nio.pdf)
