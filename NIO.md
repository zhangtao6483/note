# NIO

## 1 缓冲区（Buffer）

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/nio/nio_1.png)

> MappedByteBuffer，是 ByteBuffer 专门用于内存映射文件的一种特例

### 1.1 属性
0 <= mark <= position <= limit <= capacity

#### 容量（Capacity） 
缓冲区能够容纳的数据元素的最大数量。这一容量在缓冲区创建时被设定，并且永远不能被改变。 
#### 上界（Limit） 
缓冲区的第一个不能被读或写的元素。或者说，缓冲区中现存元素的计数。 
#### 位置（Position） 
下一个要被读或写的元素的索引。位置会自动由相应的 get()和 put()函数更新。 
#### 标记（Mark） 
一个备忘位置。调用 mark()来设定 mark = postion。调用 reset()设定 position = mark。标记在设定前是未定义的(undefined)

### 1.2 缓冲区 API

**isReadOnly()**：所有的缓冲区都是可读的，但并非所都可写。每个具体的缓冲区类都通过执行 isReadOnly() 来标示其是否允许该缓存区的内容被修改。一些类型的缓冲区类可能未使其数据元素存储在一个数组中 。

**Flip()**：将一个能够继续添加数据元素的填充状态的缓冲区翻转成一个准备读出元素的释放状态。

**clear()** 函数将清空缓冲区，而 **reset()** 位置返回到一个先前设定 **mark()** 的标记。reset()函数将位置设为当前的标记值。如果标记值未定义，调用reset()将导致 InvalidMarkException 异常

**compareTo()** 两个缓冲区被认为相等的充要条件是:

- 两个对象类型相同。包含不同数据类型的buffer永远不会相等，而且buffer绝不会等于非buffer 对象。- 两个对象都剩余同样数量的元素。Buffer的容量不需要相同，而且缓冲区中剩 余数据的索引也不必相同。但每个缓冲区中剩余元素的数目(从位置到上界)必须相同。- 在每个缓冲区中应被Get()函数返回的剩余数据元素序列必须一致。

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/nio/nio_2.png)

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/nio/nio_3.png)

> 缓冲区并不是多线程安全的。如果您想以多线程同时存取特定的缓冲区，您需要在存取缓冲区之前进行同步(例如对缓冲区对象进行跟踪)

### 1.3 批量移动

```java
char [] smallArray = new char [10];while (buffer.hasRemaining( )) {	int length = Math.min (buffer.remaining( ), smallArray.length);	buffer.get (smallArray, 0, length);	processData (smallArray, length);}
```

### 1.4 创建缓冲区

**allocate()**: 方法分配非直接缓冲区，将缓冲区建立在JVM内存中

**allocateDirect()**: 方法分配直接缓冲区，将缓冲区建立在物理内存中，可以提高效率

**wrap()**：使用自己提供的数组用做缓冲区的备份存储器

```java
char [] myArray = new char [100];CharBuffer charbuffer = CharBuffer.wrap (myArray);
```

## 2 通道（Channel）

![Alt text](https://raw.githubusercontent.com/zhangtao6483/note/master/img/nio/nio_4.jpg)

### 2.1 获取通道

获取通道的一种方式是对支持通道的对象调用getChannel()方法。支持通道的类：

- FileInputStream
- FileOutputStream
- RandomAccessFile
- DatagramSocket
- Socket
- ServerSocket

获取通道的其他方式是使用Files类的静态方法newByteChannel()获取字节通道。或者通过通道的静态方法open() 打开并返回指定通道


```java
SocketChannel sc = SocketChannel.open();sc.connect (new InetSocketAddress ("somehost", someport));
ServerSocketChannel ssc = ServerSocketChannel.open();ssc.socket().bind (new InetSocketAddress (somelocalport));
DatagramChannel dc = DatagramChannel.open();
RandomAccessFile raf = new RandomAccessFile ("somefile", "r");FileChannel fc = raf.getChannel();

FileInputStream fis = new FileInputStream();
FileChannel fs = fis.getChannel();

```

### 2.2 通道数据传输

- 将Buffer中数据写入Channel<br> int bytesWritten = inChannel.write(buf);
- 从Channel读取数据到Buffer<br> int bytesRead = inChannel.read(buf);

### 2.3 分散（Scatter）和聚集（Gather）
- 分散读取（Scattering Reads）是指从Channel中读取的数据“分散”到多个Buffer中

```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = { header, body };

channel.read(bufferArray);
```

按照缓冲区的顺序，从Channel中读取的数据依次将Buffer填满

- 聚集写入（Gathering Writes）是指将多个Buffer中的数据“聚集”到Channel

```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

//write data into buffers

ByteBuffer[] bufferArray = { header, body };

channel.write(bufferArray);
```

按照缓冲区的顺序，写入position和limit之间的数据到Channel

### 2.4 Channel to Channel

**transferFrom()**

FileChannel.transferFrom方法把数据从通道源传输到FileChannel：

```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count    = fromChannel.size();

toChannel.transferFrom(fromChannel, position, count);
```

**transferTo()**

transferTo方法把FileChannel数据传输到另一个channel

```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count    = fromChannel.size();

fromChannel.transferTo(position, count, toChannel);
```

### 2.5 常用方法

方法     | 描述
------- | ------int read(ByteBuffer dst) | 从 Channel 中读取数据到 ByteBufferlong read(ByteBuffer[] dsts) | 将 Channel 中的数据“分散”到 ByteBuffer[]int write(ByteBuffer src)     | 将 ByteBuffer 中的数据写入到 Channellong write(ByteBuffer[] srcs) | 将 ByteBuffer[] 中的数据“聚集”到 Channellong position()              | 返回此通道的文件位置FileChannel position(long p) | 设置此通道的文件位置long size()                  | 返回此通道的文件的当前大小FileChannel truncate(long s) | 将此通道的文件截取为给定大小void force(boolean metaData) | 强制将所有对此通道的文件更新写入到存储设备中

## 3 选择器（Selector）

选择器(Selector) 是 SelectableChannle 对象的多路复用器，Selector 可 以同时监控多个 SelectableChannel 的 IO 状况，也就是说，利用 Selector 可使一个单独的线程管理多个 Channel。Selector 是非阻塞 IO 的核心。

### 3.1 Selector应用

- 创建 Selector :通过调用 Selector.open() 方法创建一个 Selector 

```java
// 创建Selector
Selector selector = Selector.open();
```

- 向选择器注册通道:SelectableChannel.register(Selector sel, int ops)

```java
// 创建一个Socket套接字
Socket socket = new Socket(InetAddress.getByName("127.0.0.1"), 9898);

// 获取SocketChannel
SocketChannel channel = socket.getChannel();

// 创建选择器
Selector selector = Selector.open();

// 将SocketChannel 切换到非阻塞模式
channel.configureBlocking(false);

// 向Selector注册Channel
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);

```

- 可以监听的事件类型(可使用 SelectionKey 的四个常量表示): 
  - 读 : SelectionKey.OP_READ (1)  - 写 : SelectionKey.OP_WRITE (4) 
  - 连接 : SelectionKey.OP_CONNECT (8) 
  - 接收 : SelectionKey.OP_ACCEPT (16)

- 若注册时不止监听一个事件，则可以使用“位或”操作符连接

### 3.2 SelectionKey

SelectionKey:表示 SelectableChannel 和 Selector 之间的注册关系。每次向选择器注册通道时就会选择一个事件(选择键)。选择键包含两个表示为整数值的操作集。操作集的每一位都表示该键的通道所支持的一类可选择操作。<br>
**SelectionKey常用方法**
<br>

方法 | 描述
--- | ---int interestOps() | 获取感兴趣事件集合int readyOps() | 获取通道已经准备就绪的操作的集合SelectableChannel channel() | 获取注册通道Selector selector() | 返回选择器boolean isReadable() | 检测 Channal 中读事件是否就绪boolean isWritable() | 检测 Channal 中写事件是否就绪boolean isConnectable() | 检测 Channel 中连接是否就绪boolean isAcceptable() | 检测 Channel 中接收是否就绪

<br>
**Selector常用方法**
<br>

方法  |  描述
---  | ---Set<SelectionKey> keys() | 所有的 SelectionKey 集合。代表注册在该Selector上的ChannelselectedKeys() | 被选择的 SelectionKey 集合。返回此Selector的已选择键集int select() | 监控所有注册的Channel，当它们中间有需要处理的 IO 操作时， 该方法返回，并将对应得的 SelectionKey 加入被选择的 SelectionKey 集合中，该方法返回这些 Channel 的数量。int select(long timeout) | 可以设置超时时长的 select() 操作int selectNow() | 执行一个立即返回的 select() 操作，该方法不会阻塞线程Selector wakeup() | 使一个还未返回的 select() 方法立即返回void close() | 关闭该选择器

<br>
参考 ： http://wiki.jikexueyuan.com/project/java-nio-zh/java-nio-selector.html