# Java网络编程

## 常见网络协议

* TCP: 英文全名(Transmission Control Protocol,传输控制协议)。是一种面向连接的、可靠的、基于字节流的传输层通信协议,TCP 层是位于 IP 层之上,应用层之下的中间层。TCP 保障了两个应用程序之间的可靠通信。通常用于互联网协议,被称 TCP / IP。
* UDP: 英文全名(User Datagram Protocol,用户数据报协议)。位于 OSI 模型的传输层。一个无连接的协议。提供了应用程序之间要发送数据的数据报。由于UDP缺乏可靠性且属于无连接协议,所以应用程序通常必须容许一些丢失、错误或重复的数据包。

## InetAddress 对IP地址的获取和操作

```Java
public class InetAddressDemo {
    public static void main(String[] args) throws UnknownHostException {
    //    InetAddress address = InetAddress.getByName("DreamDragon");
    //    InetAddress address = InetAddress.getByName("10.19.65.91");
        InetAddress address = InetAddress.getByName("www.baidu.com");

        String name = address.getHostName();
        System.out.println("主机名：" + name);
        
        String ip = address.getHostAddress();
        System.out.println("IP地址: " + ip);

    }
}

```

## Java Socket

### ServerSocket、Socket

> TCP Socket

* 服务端

```java
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

import jakarta.annotation.PostConstruct;
import lombok.Setter;
import lombok.extern.slf4j.Slf4j;

/**
 * TCP Socket 服务器
 */
@Slf4j
public class TCPSocketService {

    private static final int THREADS = 3;
    private static final int PORT = 13000;

    @Setter
    private volatile boolean shutdown = false;

    private ExecutorService threadPool;
    private ServerSocket serverSocket;

    /**
     * 初始化、启动服务
     */
    @PostConstruct
    public void start() {
        try {
            // 线程池
            threadPool = Executors.newFixedThreadPool(THREADS);
            // 绑定端口
            serverSocket = new ServerSocket(PORT);
            log.info("TCP Server started on port {}", PORT);

            // 端口监听
            new Thread(new AcceptThread()).start();
        } catch (IOException e) {
            throw new RuntimeException("Failed to start TCP Server on port " + PORT, e);
        }
    }

    /**
     * 监听端口数据
     */
    private class AcceptThread implements Runnable {
        @Override
        public void run() {
            while (!shutdown) {
                try {
                    // 阻塞等待数据
                    Socket accept = serverSocket.accept();
                    // 数据交由线程池处理，不阻塞端口数据接收
                    threadPool.execute(new SocketHandler(accept));
                } catch (IOException e) {
                    if (shutdown) break;
                    log.error("TCP Server error", e);
                }
            }
        }
    }

    /**
     * 数据处理模块
     */
    private class SocketHandler implements Runnable {
        private final Socket accept;

        /**
         * 构造函数，初始化数据
         */
        public SocketHandler(Socket accept) {
            this.accept = accept;
        }

        /**
         * 具体数据处理流程
         */
        @Override
        public void run() {
            try (BufferedReader reader = new BufferedReader(new InputStreamReader(accept.getInputStream()));
                 OutputStream outputStream = accept.getOutputStream()) {

                String receivedData = reader.readLine();
                log.info("Received: {}", receivedData);

                response(outputStream, "Server received: " + receivedData);
            } catch (IOException e) {
                log.error("Error handling socket request", e);
            }
        }
    }

    /**
     * 响应数据
     */
    public void response(OutputStream outputStream, String response) throws IOException {
        outputStream.write(response.getBytes(StandardCharsets.UTF_8));
        outputStream.flush();
    }

    /**
     * 停止服务
     */
    public void stop() {
        shutdown = true;
        try {
            serverSocket.close();
            threadPool.shutdown();
            log.info("TCP Server stopped.");
        } catch (IOException e) {
            log.error("Error stopping TCP Server", e);
        }
    }
}

```

* 客户端

```java
import java.io.*;
import java.net.Socket;
import java.nio.charset.StandardCharsets;

import lombok.extern.slf4j.Slf4j;

/**
 * TCP 客户端
 */
@Slf4j
public class ClintSocketService {

    private static final int SERVER_PORT = 13000;
    private static final String SERVER_HOST = "localhost";

    public void send() throws IOException {
        try (Socket socket = new Socket(SERVER_HOST, SERVER_PORT);
            // 发送数据
             PrintWriter printWriter = new PrintWriter(socket.getOutputStream(), true);
             // 接收服务器返回的数据
             BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(socket.getInputStream()))) {

            // 发送数据
            printWriter.println("Hello Server!");
            log.info("消息已发送: Hello Server!");

            // 接收服务器返回的数据
            String response = bufferedReader.readLine();
            log.info("服务器响应: {}", response);
        }
    }
}

```

### DatagramSocket、DatagramPacket

> UDP Socket

* 服务端

```java
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

import jakarta.annotation.PostConstruct;
import lombok.Setter;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class UDPSocketService {

    private static final int BUFFER_SIZE = 1024;
    private static final int THREADS = 3;
    private static final int PORT = 13000;

    @Setter
    private volatile boolean shutdown = false;

    private ExecutorService threadPool;
    private DatagramSocket serverSocket;

    /**
     * 初始化、启动服务
     */
    @PostConstruct
    public void start() {
        try {
            // 绑定端口
            serverSocket = new DatagramSocket(PORT);
            // 线程池
            threadPool = Executors.newFixedThreadPool(THREADS);
            log.info("UDP Server started on port {}", PORT);

            // 端口监听
            new Thread(new Acceptor()).start();
        } catch (IOException e) {
            throw new RuntimeException("Failed to start UDP Server on port " + PORT, e);
        }
    }

    /**
     * 监听端口数据
     */
    private class Acceptor implements Runnable {
        @Override
        public void run() {
            while (!shutdown) {
                try {
                    // 初始化数据包
                    DatagramPacket packet = new DatagramPacket(new byte[BUFFER_SIZE], BUFFER_SIZE);
                    // 阻塞接收数据
                    serverSocket.receive(packet);
                    // 数据交由线程池处理，不阻塞端口数据接收
                    threadPool.execute(new PacketHandler(packet));
                } catch (IOException e) {
                    if (shutdown) break;
                    log.error("UDP Server error", e);
                }
            }
        }
    }

    /**
     * 数据处理模块
     */
    private class PacketHandler implements Runnable {
        private final DatagramPacket packet;

        public PacketHandler(DatagramPacket packet) {
            this.packet = packet;
        }

        @Override
        public void run() {
            String receivedData = new String(packet.getData(), packet.getOffset(), packet.getLength(), StandardCharsets.UTF_8);
            log.info("Received: {}", receivedData);

            response(packet, "Server received: " + receivedData);
        }
    }

    /**
     * 响应数据
     */
    private void response(DatagramPacket requestPacket, String responseData) {
        try {
            byte[] responseBytes = responseData.getBytes(StandardCharsets.UTF_8);
            DatagramPacket responsePacket = new DatagramPacket(
                responseBytes, responseBytes.length, 
                requestPacket.getAddress(), requestPacket.getPort()
            );
            serverSocket.send(responsePacket);
        } catch (IOException e) {
            log.error("Failed to send UDP response", e);
        }
    }

    /**
     * 停止服务
     */
    public void stop() {
        shutdown = true;
        serverSocket.close();
        threadPool.shutdown();
        log.info("UDP Server stopped.");
    }
}

```

* 客户端

```java
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.nio.charset.StandardCharsets;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class ClintDatagramSocketService {
    
    private static final String HOST = "localhost";
    private static final int SERVER_PORT = 13000;

    public void send() throws IOException {
        DatagramSocket socket = new DatagramSocket();
        InetAddress serverAddress = InetAddress.getByName(HOST);
        byte[] data = "你好，服务器！".getBytes(StandardCharsets.UTF_8);

        DatagramPacket packet = new DatagramPacket(data, data.length, serverAddress, SERVER_PORT);
        socket.send(packet);
        log.info("消息已发送: {}", new String(data, StandardCharsets.UTF_8));

        // 接收服务器返回的数据
        byte[] buffer = new byte[1024];
        DatagramPacket responsePacket = new DatagramPacket(buffer, buffer.length);
        socket.receive(responsePacket);

        String response = new String(responsePacket.getData(), 0, responsePacket.getLength(), StandardCharsets.UTF_8);
        log.info("服务器响应: {}", response);

        socket.close();
    }
}

```

## NIO

* Selector（选择器）
  * Selector 是 Java NIO 的核心，它可以同时管理多个 Channel，用于非阻塞 IO 操作。
  * select() 监听所有注册的通道，阻塞直到至少有一个事件触发。
  * selectedKeys() 获取触发的事件集合。
  * wakeup() 立即唤醒 select()，避免阻塞。
* ServerSocketChannel（服务器通道）
  * 负责监听客户端的 TCP 连接，类似于传统的 ServerSocket。
  * 需要调用 accept() 方法获取 SocketChannel，但 accept() 是非阻塞的。
* SocketChannel（客户端通道）
  * 代表与客户端的 TCP 连接，用于数据读写，类似 Socket。
  * 客户端通过 SocketChannel.open() 连接服务器。
  * 服务器通过 accept() 获取客户端的 SocketChannel。
* SelectionKey（选择键）
  * SelectionKey 记录 Channel 和事件的绑定关系。
  * OP_ACCEPT（连接事件）：服务器端 ServerSocketChannel 接受新连接。
  * OP_READ（读事件）：某个 SocketChannel 有数据可读。
  * OP_WRITE（写事件）：某个 SocketChannel 可以写入数据。
* ByteBuffer（缓冲区）
  * allocate(size) 创建缓冲区。
  * put() 写入数据。
  * flip() 切换模式（写 → 读）。
  * get() 读取数据。
* 服务端

```java
package com.example.nio.server;

import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

@Slf4j
@Component
public class NioServer {
    private static final int PORT = 8080;
    private static final int BUFFER_SIZE = 1024;
    // 线程池大小 = cpu核心数 * 2
    private static final int THREAD_POOL_SIZE = Runtime.getRuntime().availableProcessors() * 2;

    private Selector selector;
    private ServerSocketChannel serverChannel;
    private final ExecutorService threadPool = Executors.newFixedThreadPool(THREAD_POOL_SIZE);
    private volatile boolean running = true;

    /**
     * 初始化、启动服务
     */
    @PostConstruct
    public void startServer() {
        try {
            // 初始化选择器
            selector = Selector.open();
            // 初始化服务器通道
            serverChannel = ServerSocketChannel.open();
            // 设置为非阻塞
            serverChannel.configureBlocking(false);
            // 绑定端口
            serverChannel.bind(new InetSocketAddress(PORT));
            // 将通道注册到选择器
            serverChannel.register(selector, SelectionKey.OP_ACCEPT);

            log.info("NIO Server started on port {}", PORT);

            // 开启服务端口监听
            new Thread(this::eventLoop).start();
        } catch (IOException e) {
            log.error("Failed to start NIO Server", e);
            shutdown();
        }
    }

    /**
     * 事件循环
     */
    private void eventLoop() {
        while (running) {
            try {
                // 阻塞直到有事件发生
                int readyChannels = selector.select();
                if (readyChannels == 0) continue;

                // 获取事件集合
                Set<SelectionKey> selectedKeys = selector.selectedKeys();
                // 遍历事件集合
                Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

                // 处理事件
                while (keyIterator.hasNext()) {
                    // 获取事件
                    SelectionKey key = keyIterator.next();
                    // 将当前事件从迭代器中移除，避免处理完成后依旧会触发事件。
                    keyIterator.remove();

                    // 处理连接事件
                    if (key.isAcceptable()) {
                        acceptConnection();
                        //  处理可读事件
                    } else if (key.isReadable()) {
                        handleRead(key);
                    }
                }
            } catch (IOException e) {
                log.error("Error in event loop", e);
            }
        }
    }
    
    /**
     * 处理连接事件
     */
    private void acceptConnection() throws IOException {
        // 获取新连接的客户端通道
        SocketChannel clientChannel = serverChannel.accept();
        // 设置为非阻塞
        clientChannel.configureBlocking(false);
        // 将通道注册到选择器，并监听可读事件
        clientChannel.register(selector, SelectionKey.OP_READ);
        log.info("New client connected: {}", clientChannel.getRemoteAddress());
        selector.wakeup(); // 立即唤醒 selector，用于立即响应当前注册的可读通道是否有可读事件触发
    }

    /**
     * 处理可读事件
     */
    private void handleRead(SelectionKey key) {
        // 获取客户端通道
        SocketChannel clientChannel = (SocketChannel) key.channel();
        // 创建缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(BUFFER_SIZE);

        try {
            // 读取数据
            int bytesRead = clientChannel.read(buffer);
            // 如果读取到 -1，表示客户端已经关闭连接
            if (bytesRead == -1) {
                // 关闭客户端通道
                closeClient(key);
                return;
            }

            // 切换模式 从写模式切换到读模式
            buffer.flip();
            // 读取数据
            byte[] data = new byte[buffer.remaining()];
            // 将数据读出
            buffer.get(data);

            // 处理数据逻辑
            String message = new String(data);
            log.info("Received: {}", message);

            // 交由线程池处理数据
            threadPool.submit(() -> processRequest(clientChannel, message));
        } catch (IOException e) {
            log.error("Error handling read", e);
            closeClient(key);
        }
    }

    /**
     * 数据处理逻辑
     */
    private void processRequest(SocketChannel clientChannel, String request) {
        try {
            String response = "Server received: " + request;
            ByteBuffer buffer = ByteBuffer.wrap(response.getBytes());
            while (buffer.hasRemaining()) {
                clientChannel.write(buffer);
            }
        } catch (IOException e) {
            log.error("Error sending response", e);
        }
    }

    /**
     * 关闭客户端
     */
    private void closeClient(SelectionKey key) {
        try {
            log.info("Closing connection: {}", key.channel());
            key.channel().close();
        } catch (IOException e) {
            log.error("Error closing client", e);
        }
        key.cancel();
    }

    /**
     * 停止服务
     */
    @PreDestroy
    public void shutdown() {
        try {
            log.info("Shutting down NIO Server...");
            running = false;
            selector.close();
            serverChannel.close();
            threadPool.shutdown();
        } catch (IOException e) {
            log.error("Error shutting down server", e);
        }
    }
}
```

* 客户端

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;

public class NioClient {
    private static final int BUFFER_SIZE = 1024;
    private static final String SERVER_HOST = "localhost";
    private static final int SERVER_PORT = 8080;

    public static void main(String[] args) {
        // 创建客户端通道
        try (SocketChannel clientChannel = SocketChannel.open(new InetSocketAddress(SERVER_HOST, SERVER_PORT))) {
            clientChannel.configureBlocking(true); // 客户端使用阻塞模式

            // 发送数据
            String message = "Hello Server!";
            // 将数据写入通道
            ByteBuffer buffer = ByteBuffer.wrap(message.getBytes());
            while (buffer.hasRemaining()) {
                clientChannel.write(buffer);
            }

            // 清空缓冲区
            buffer.clear();
            // 接收服务器响应
            int bytesRead = clientChannel.read(buffer);
            // 如果读取到 -1，表示服务器已经关闭连接
            if (bytesRead != -1) {
                buffer.flip();
                byte[] data = new byte[buffer.remaining()];
                buffer.get(data);
                System.out.println("Server response: " + new String(data));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Netty

* 客户端
  * 目录结构

    ```java
    spring-boot-netty-server/
    ├── src/main/java/com/example/nettyserver/
    │   ├── NettyServerApplication.java      # Spring Boot 启动类
    │   ├── config/NettyServerConfig.java    # Netty 服务器配置
    │   ├── server/NettyServer.java          # Netty 服务器核心类
    │   ├── handler/NettyServerHandler.java  # 业务逻辑处理器
    |   ├── handler/MyServerHandler.java.java # 业务逻辑处理器
    │   ├── handler/NettyWebSocketHandler.java # WebSocket 处理器
    │   ├── utils/NettyUtils.java            # 工具类
    │   ├── model/Message.java               # 数据模型
    ├── pom.xml
    ```

  * 代码
    * pom.xml
  
    ```XML
        <dependencies>
        <!-- Spring Boot Starter -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <!-- Netty -->
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>4.1.100.Final</version>
        </dependency>

        <!-- Lombok (简化 Getter/Setter) -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>
    </dependencies>
    ```

    * NettyServerConfig.java

    ```java
    package com.example.nettyserver.config;

    import com.example.nettyserver.server.NettyServer;
    import jakarta.annotation.PostConstruct;
    import jakarta.annotation.PreDestroy;
    import lombok.RequiredArgsConstructor;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    @RequiredArgsConstructor
    public class NettyServerConfig {

        private final NettyServer nettyServer;

        @PostConstruct
        public void startServer() {
            nettyServer.start();
        }

        @PreDestroy
        public void stopServer() {
            nettyServer.stop();
        }
    }

    ```

    * NettyServer.java
      * bossGroup 用于监听客户端连接，专门负责与客户端创建连接，并把连接注册到workerGroup的Selector中。
      * workerGroup 用于处理每一个连接发生的读写事件。

    ```java
    package com.example.nettyserver.server;

    import com.example.nettyserver.handler.NettyServerHandler;
    import io.netty.bootstrap.ServerBootstrap;
    import io.netty.channel.*;
    import io.netty.channel.nio.NioEventLoopGroup;
    import io.netty.channel.socket.nio.NioServerSocketChannel;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.stereotype.Component;

    /**
    * Netty 服务器核心类，负责管理 Netty 的启动和关闭
    */
    @Component
    @Slf4j
    public class NettyServer {
        private static final int PORT = 8080; // 服务器监听端口
        private EventLoopGroup bossGroup;  // 处理连接请求的线程组
        private EventLoopGroup workerGroup; // 处理具体数据读写的线程组
        private Channel serverChannel; // 服务器通道

        /**
        * 启动 Netty 服务器
        */
        public void start() {
            bossGroup = new NioEventLoopGroup(1); // 负责处理客户端连接
            workerGroup = new NioEventLoopGroup(); // 负责处理具体的 IO 事件

            try {
                // 创建服务器启动引导类
                ServerBootstrap bootstrap = new ServerBootstrap();
                bootstrap.group(bossGroup, workerGroup) // 绑定线程组
                        .channel(NioServerSocketChannel.class) // 指定使用 NIO 传输
                        .option(ChannelOption.SO_BACKLOG, 1024) // 最大连接数
                        .childOption(ChannelOption.SO_KEEPALIVE, true) // 开启 TCP 长连接
                        .childHandler(new ChannelInitializer<>() {
                            @Override
                            protected void initChannel(io.netty.channel.socket.SocketChannel ch) {
                                ch.pipeline().addLast(new NettyServerHandler()); // 绑定业务处理器
                                ch.pipeline().addLast(new NettyWebSocketHandler()); // 添加 WebSocket 处理器
                            }
                        });

                // 绑定端口并启动服务器
                ChannelFuture future = bootstrap.bind(PORT).sync(); // Netty 所有的 I/O 操作（如 bind()、connect()）默认是异步，sync() 主要用于等待绑定完成，确保端口被占用,确保端口绑定完成后，再继续向下运行。
                serverChannel = future.channel(); // 获取 Netty 服务器的 主 Channel，即 NioServerSocketChannel。代表 当前 Netty 服务器的主通信通道，用于监听 客户端连接。
                log.info("Netty Server started on port: {}", PORT);
                future.channel().closeFuture().sync(); // 阻塞主线程，直到服务器 Channel 关闭, sync()让当前线程阻塞，直到 serverChannel.close() 被调用。
            } catch (Exception e) {
                log.error("Netty Server failed to start", e);
            } finally {
                stop();
            }
        }

        /**
        * 停止 Netty 服务器
        */
        public void stop() {
            if (bossGroup != null) {
                bossGroup.shutdownGracefully();
            }
            if (workerGroup != null) {
                workerGroup.shutdownGracefully();
            }
            if (serverChannel != null) {
                serverChannel.close();
            }
        }
    }

    ```

    * NettyServerHandler.java

      ```java
      package com.example.nettyserver.handler;

      import io.netty.channel.*;
      import io.netty.util.ReferenceCountUtil;
      import lombok.extern.slf4j.Slf4j;

      /**
       * 业务处理器，处理 TCP 连接的数据
       */
      @Slf4j
      @ChannelHandler.Sharable
      public class NettyServerHandler extends SimpleChannelInboundHandler<String> {

          /**
           * 客户端连接成功时调用
           */
          @Override
          public void channelActive(ChannelHandlerContext ctx) {
              log.info("客户端连接: {}", ctx.channel().remoteAddress());
          }

          /**
           * 处理客户端发送的消息
           */
          @Override
          protected void channelRead0(ChannelHandlerContext ctx, String msg) {
              log.info("收到消息: {}", msg);
              ctx.writeAndFlush("服务器收到: " + msg + "\n"); // 发送响应
          }

          /**
           * 发生异常时关闭连接
           */
          @Override
          public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
              log.error("发生异常", cause);
              ctx.close();
          }
      }
      ```

    * MyServerHandler.java

    ```java
    /**
     * 自定义的Handler需要继承Netty规定好的HandlerAdapter
     * 才能被Netty框架所关联，有点类似SpringMVC的适配器模式
     **/
    public class MyServerHandler extends ChannelInboundHandlerAdapter {

        @Override
        public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
            //获取客户端发送过来的消息
            ByteBuf byteBuf = (ByteBuf) msg;
            System.out.println("收到客户端" + ctx.channel().remoteAddress() + "发送的消息：" + byteBuf.toString(CharsetUtil.UTF_8));
        }

        @Override
        public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
            //发送消息给客户端
            ctx.writeAndFlush(Unpooled.copiedBuffer("服务端已收到消息，并给你发送一个问号?", CharsetUtil.UTF_8));
        }

        @Override
        public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
            //发生异常，关闭通道
            ctx.close();
        }
    }
    ```
    * NettyWebSocketHandler.java

    ```java
    package com.example.nettyserver.handler;

    import io.netty.channel.*;
    import io.netty.handler.codec.http.websocketx.*;
    import lombok.extern.slf4j.Slf4j;

    /**
     * WebSocket 处理器，处理 WebSocket 连接和消息
     */
    @Slf4j
    public class NettyWebSocketHandler extends SimpleChannelInboundHandler<WebSocketFrame> {

        /**
         * 处理 WebSocket 消息
         */
        @Override
        protected void channelRead0(ChannelHandlerContext ctx, WebSocketFrame frame) {
            if (frame instanceof TextWebSocketFrame textFrame) {
                log.info("收到 WebSocket 消息: {}", textFrame.text());
                ctx.channel().writeAndFlush(new TextWebSocketFrame("服务器响应: " + textFrame.text()));
            } else if (frame instanceof CloseWebSocketFrame) {
                ctx.channel().close();
            }
        }

        /**
         * 发生异常时关闭连接
         */
        @Override
        public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
            log.error("WebSocket 发生异常", cause);
            ctx.close();
        }
    }
    ```

    * NettyUtils.java

    ```java
    package com.example.nettyserver.utils;

    import io.netty.channel.Channel;
    import java.util.Map;
    import java.util.concurrent.ConcurrentHashMap;

    /**
     * Netty 连接管理工具类
     */
    public class NettyUtils {
        private static final Map<String, Channel> CLIENTS = new ConcurrentHashMap<>();

        public static void addClient(String clientId, Channel channel) {
            CLIENTS.put(clientId, channel);
        }

        public static Channel getClient(String clientId) {
            return CLIENTS.get(clientId);
        }

        public static void removeClient(String clientId) {
            CLIENTS.remove(clientId);
        }
    }

    ```
