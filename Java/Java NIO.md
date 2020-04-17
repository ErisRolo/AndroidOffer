## Java NIO

### 一. NIO的概念

**Java NIO(New IO)是一个可以替代标准Java IO API的IO API(从Java1.4开始)**，Java NIO提供了与标准IO不同的IO工作方式。所以Java NIO是一种新式的IO标准，与之间的普通IO的工作方式不同，标准的IO基于字节流和字符流进行操作的，而**NIO是基于通道(Channel)和缓冲区(Buffer)进行操作**，数据总是从通道读取到缓冲区中，或者从缓冲区写入通道也类似。 

由上面的定义就说明NIO是一种新型的IO，但NIO不仅仅就是等于Non-blocking  IO（非阻塞IO），NIO中有实现非阻塞IO的具体类，但不代表NIO就是Non-blocking IO（非阻塞IO）。

**Java NIO由Buffer、Channel和Selector三个核心部分组成**，传统的IO操作面向数据流，意味着每次从流中读一个或多个字节，直至完成，数据没有被缓存在任何地方。NIO操作面向缓冲区，数据从Channel读取到Buffer缓冲 区，随后在Buffer中处理数据。



### 二. Buffer的使用

**利用Buffer读写数据通常遵循四个步骤**：① 把数据写入buffer；

​                                                                     ② 调用flip；

​                                                                     ③ 从buffer中读取数据；

​                                                                     ④ 调用buffer.clear()

当写入数据到buffer中时，buffer会记录已经写入的数据大小。当需要读数据时，通过`flip()`方法把buffer从写模式调整为读模式；在读模式下，可以读取所有已经写入的数据。当读取完数据后，需要清空buffer，以满足后续写入操作。清空buffer， 调用`clear()`，一旦读完buffer中的数据，需要让buffer准备好再次被写入，clear会恢复状态值，但不会擦除数据。 

**buffer缓冲区实质上就是一块内存，用于写入数据，也供后续再次读取数据。**这块内存被NIO Buffer管理，并提供一系列的方法用于更简单的操作这块内存。一个buffer有三个重要属性，如下

**容量(capacity)**：作为一块内存，buffer有一个固定的大小，叫做capacity容量。也就是最多只能写入容量值的字节，整形等数据。一旦buffer写满了就需要清空已读数据以便下次继续写入新的数据。 

**位置(position)**：当写入数据到buffer的时候需要中一个确定的位置开始，默认初始化时这个位置position为0，一旦写入了数据比如一个字节，整形数据，那么position的值就会指向数据之后的一个单元，position最大可以到capacity-1。 当从buffer读取数据时，也需要从一个确定的位置开始。buffer从写入模式变为读取模式时，position会归零，每次读取后，position向后移动。 

**上限(limit)**：在写模式，limit的含义是我们所能写入的最大数据量，它等同于buffer的容量；一旦切换到读模式，limit则代表我们所能读取的最大数据量，它的值等同于写模式下position的位置，数据读取的上限是buffer中已有的数据，也就是limit的位置（原position所指的位置）。

为了获取一个Buffer对象，必须先分配。每个Buffer实现类都有一个`allocate()`方法用于分配内存，示例如下

```java
ByteBuffer buf = ByteBuffer.allocate(48); //开辟一个48字节大小的buffer
CharBuffer buf = CharBuffer.allocate(1024); //开辟一个1024个字符的CharBuffer
```

Buffer有ByteBuffer、CharBuffer、DoubleBuffer等七种基本数据类型相关的实现类，其中ByteBuffer又有一个MappedByteBuffer实现类。Java类库中的NIO包相对于IO包来说有一个新功能是内存映射文件，日常编程中并不是经常用到，但是在处理大文件时是比较理想的提高效率的手段。MappedByteBuffer实现的就是内存映射文件，可以实现大文件的高效读写。



### 三. Channel的使用

**Java NIO Channel通道和流非常相似，主要有以下几点区别**：

① 通道可以读也可以写，流一般来说是单向的（只能读或者写）；

② 通道可以异步读写；

③ 通道总是基于缓冲区Buffer来读写；

④ 可以从通道中读取数据，写入到buffer；也可以中buffer内读数据，写入到通道中

**Channel有四个实现类**，如下：

**FileChannel**：用于文件的数据读写

**DatagramChannel**：用于UDP的数据读写

**SocketChannel**：用于TCP的数据读写

**ServerSocketChannel**：允许监听TCP链接请求，每个请求会创建会一个SocketChannel

Channel使用实例如下

```java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt" , "rw");
FileChannel inChannel = aFile.getChannel();
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);
while (bytesRead != -1) {
    
    System.out.println("Read " + bytesRead);
    buf.flip();
    
    while(buf.hasRemaining()){
        System.out.print((char) buf.get()); 
    }
    
    buf.clear();
    bytesRead = inChannel.read(buf);
}
aFile.close();
```



### 四. 阻塞/非阻塞/同步/非同步的关系

参考Richard Stevens的**《UNIX® Network Programming Volume 1, Third Edition: The Sockets Networking 》**的第6.2节“**I/O Models**”



### 五. NIO中的blocking IO/nonblocking IO/IO multiplexing/asynchronous IO

首先，标准的IO属于blocking IO。

其次，NIO中的实现了SelectableChannel类的对象，可以通过`SelectableChannel configureBlocking(boolean block)`方法调整通道的阻塞模式，如果为true，则通道将被置于阻塞模式；如果为false，则通道将被置于非阻塞模式。设置为false的NIO类为nonblocking IO。

再其次，通过Selector监听实现多个NIO对象的读写操作，属于IO multiplexing。关于Selector，其负责调度多个非阻塞式IO，当有其感兴趣的读写操作到来时，再执行相应的操作。Selector执行select()方法来进行轮询查找是否到来了读写操作，这个过程是阻塞的。 

最后，Java 7中增加了asynchronous IO。



### 六. Selector使用

Selector是Java NIO中的一个组件，用于检查一个或多个NIO Channel的状态是否处于可读、可写。如此可以实现单线程管理多个channels,也就是可以管理多个网络链接。Selector是一种IO multiplexing的情况。 

一个完整的示例如下

```java
//创建Selector
Selector selector = Selector.open();

//注册Channel到Selector上
//Channel必须是非阻塞的，因此FileChannel不适用，FileChannel不能切换非阻塞模式，而SocketChannel可以
channel.configureBlocking(false);
//第二个参数为“关注集合”，代表关注的channel状态，有四种基础类型可供监听：Connect、Accept、Read、Write
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);

while(true) {
    int readyChannels = selector.select();
    if(readyChannels == 0)
        continue;
    //遍历SelectionKey
    Set<SelectionKey> selectedKeys = selector.selectedKeys();
    Iterator<SelectionKey> keyIterator = selectedKeys.iterator(); 
    while(keyIterator.hasNext()) {
        SelectionKey key = keyIterator.next();
        if(key.isAcceptable()) {
            // a connection was accepted by a ServerSocketChannel.
        } else if (key.isConnectable()) {
            // a connection was established with a remote server. 
        } else if (key.isReadable()) {
            // a channel is ready for reading 
        } else if (key.isWritable()) {
            // a channel is ready for writing 
        }
        keyIterator.remove(); 
    } 
}
```