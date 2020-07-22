原文链接：https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/BIO-NIO-AIO.md

# BIO, NIO, AIO 总结

Java 中的 BIO、NIO和 AIO 理解为是 Java 语言对操作系统的各种 IO 模型的封装。程序员在使用这些 API 的时候，不需要关心操作系统层面的知识，也不需要根据不同操作系统编写不同的代码。只需要使用 Java 的 API 就可以了。

## 一、背景知识

**- 同步和异步：**

- **同步** ：两个同步任务相互依赖，并且一个任务必须以依赖于另一任务的某种方式执行。 比如在`A->B`事件模型中，你需要先完成 A 才能执行B。 再换句话说，同步调用种被调用者未处理完请求之前，调用不返回，调用者会一直等待结果的返回。
- **异步**： 两个异步的任务完全独立的，一方的执行不需要等待另外一方的执行。再换句话说，异步调用种一调用就返回结果不需要等待结果返回，当结果返回的时候通过回调函数或者其他方式拿着结果再做相关事情。



**- 阻塞和非阻塞：**

- **阻塞：** 阻塞就是发起一个请求，调用者一直等待请求结果返回，也就是当前线程会被挂起，无法从事其他任务，只有当条件就绪才能继续。
- **非阻塞：** 非阻塞就是发起一个请求，调用者不用一直等着结果返回，可以先去干其他事情。

**那么，如何区分 “同步/异步 ”和 “阻塞/非阻塞” 呢 ？**

同步/异步是从行为角度描述事物的，而阻塞和非阻塞描述的当前事物的状态（等待调用结果时的状态）。



## 二、BIO (Blocking I/O)

同步阻塞I/O模式，数据的读取写入必须阻塞在一个线程内等待其完成。

### 1. 传统 BIO

采用 **BIO 通信模型** 的服务端，通常由一个独立的 Acceptor 线程负责监听客户端的连接。我们一般是通过在`while(true)` 循环中服务端会调用 `accept()` 方法等待接收客户端的连接的方式监听请求，请求一旦接收到一个连接请求，就可以建立通信套接字在这个通信套接字上进行读写操作，此时不能再接收其他客户端连接请求，只能等待同当前连接的客户端的操作执行完成， 不过可以通过多线程来支持多个客户端的连接，如下图所示。

![image-20200722102522392](https://github.com/OnlyThePiano/Notes/blob/master/images/image-20200722102522392.png)

可以看出，如果要让 **BIO 通信模型** 能够同时处理多个客户端请求，就必须使用多线程（主要原因是`socket.accept()`、`socket.read()`、`socket.write()` 涉及的三个主要函数都是同步阻塞的），也就是说它在接收到客户端连接请求之后为每个客户端创建一个新的线程进行链路处理，处理完成之后，通过输出流返回应答给客户端，线程销毁。这就是典型的 **一请求一应答通信模型** 。但是如果这个连接不做任何事情的话就会造成不必要的线程开销，不过可以通过 **线程池机制** 改善，线程池还可以让线程的创建和回收成本相对较低。

**我们再设想一下当客户端并发访问量增加后这种模型会出现什么问题 ？**

在 Java 虚拟机中，线程是宝贵的资源，线程的创建和销毁成本很高，除此之外，线程的切换成本也是很高的。尤其在 Linux 这样的操作系统中，线程本质上就是一个进程，创建和销毁线程都是重量级的 `系统函数` 。如果并发访问量增加会导致线程数急剧膨胀可能会导致线程堆栈溢出、创建新线程失败等问题，最终导致进程宕机或者僵死，不能对外提供服务。

**- 线程池机制：**

* [线程池机制]([https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/Multithread/java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93.md#432-execute-vs-submit](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/Multithread/java线程池学习总结.md#432-execute-vs-submit))

### 2. 伪异步 IO

上面提到了传统 IO 会造成不必要的线程开销，所以后来有人对它的线程模型进行了优化一一一后端通过一个线程池来处理多个客户端的请求接入，形成 `客户端个数M` ：`线程池最大线程数N` 的比例关系，其中 M 可以远远大于 N 。通过线程池可以灵活地调配线程资源，设置线程的最大值，防止由于海量并发接入导致线程耗尽。

使用`FixedThreadPool` 可以有效的控制了线程的最大数量，保证了系统有限的资源的控制，实现了伪异步I/O模型如下：

![image-20200722103317071](https://github.com/OnlyThePiano/Notes/blob/master/images/image-20200722103317071.png)

当有新的客户端接入时，将客户端的 Socket 封装成一个Task（该任务实现java.lang.Runnable接口）投递到后端的线程池中进行处理，JDK 的线程池维护一个消息队列和 N 个活跃线程，对消息队列中的任务进行处理。 **由于线程池可以设置消息队列的大小和最大线程数，因此，它的资源占用是可控的，无论多少个客户端并发访问，都不会导致资源的耗尽和宕机。** 

 



## 三、NIO (Non-Blocking/New-Blocking)

BIO 传统输入流、输出流都是通过字节的移动来处理的 (即使不直接去处理字节流，但底层的实现还是依赖于字节处理)，也就是说，面向流的输入/输出系统一次只能处理一个字节， 因此面向流的输入/输出系统通常效率不高。

NIO 和传统的 IO 有相同的目的，都是用于进行输入/输出；但新 IO 使用了不同的方式来处理输入/输出，新 IO 采用**内存映射文件的方式**来处理输入/输出，新 IO 将文件或文件的一段区域映射到内存中，这样就可以像访问内存一样来访问文件了(这种方式模拟了操作系统上的虛拟内存的概念)，通过这种方式来进行输入/输出比传统的输入/输出要快得多。

**- NIO 的两个核心对象：**

* Channel (管道)
* Buffer (缓冲)



### 1. Buffer

Buffer 可以被理解成一个 容器，它的本质是一个数组，发送到 Channel 中的所有对象都必须首先放到 Buffer 中，而从 Channel 中读取的数据也必须先放到 Buffer 中。此处的 Buffer 有点类似于 “竹简” ，但该 Buffer 既可以像 “竹简” 那样一次次去 Channel 中取水，也允许使用 Channel 直接将文件的某块数据映射成 Buffer 。

Buffer 是一个抽象类，其最常用的子类是 `ByteBuffer` 和 `CharBuffer` 。其中 `ByteBuffer` 还有一个子类 `MappedByteBuffer` ：表示 Channel 将磁盘文件的部分或全部内容映射到内存中后得到的结果。

**- 怎么创建 Buffer对象呢 ？**

Buffer 类都没有构造器，所以创建方式适合一般对象的创建不一样，它的创建要使用如下方法：

~~~
: static XxxBuffer allocate(int capacity)
#
CharBuffer buff = CharBuffer.allocate(8);
~~~



**- Buffer 中的三个重要概念：**

* **容量 (capacity)：** 最多可以存储的数据
* **界限 (limit)：** limit 后的数据不可读不可写
* **位置 (position)：** 下一个可以被读/写的位置索引

![image-20200722123934371](https://github.com/OnlyThePiano/Notes/blob/master/images/image-20200722123934371.png)

**- Buffer 读写的过程：**

1. 装入数据
2. 为输出数据做准备 (`flip()` 方法)
3. 为装入数据做准备 (`clear()` 方法)

详细过程如下：

（1）开始时 Buffer 的 position为 0 , limit 为capacity ，程序可通过 `put()`方法向 Buffer 中放入一些数据 (或者从 Channel 中获取一些数据)，每放入一些数据， Buffer position 相应地向后移动一些位置。

（2）当 Buffer 装入数据结束后，调用 Buffer的 `flip()` 方法，该方法将 limit 设置为 position 所在位置，并将 position 设为 0 ，这就使得 Buffer 的读写指针又移到了开始位置。也就是说，Buffer 调用  `flip()` 方法之后，从而避免了读取 Buffer 数据时读到 `null值` ，此时 Buffer 为输出数据做好准备；

（3）当Buffer输出数据结束后，Buffer 调用  `clear()` 方法，`clear()` 方法不是清空Buffer的数据，它仅仅将 position 置为 0，将 limit 置为 capacity；这样为再次向Buffer 中装入数据做好准备。



**- Buffer 常用方法：**

1. 创建 Buffer 对象：  static XxxBuffer allocate(int capacity)
2. 返回 Buffer 的大小：int capacity()



### 2. Channel

Channel (通道) 是对传统的输入/输出系统的模拟；在 NIO 系统中所有的数据都需要通过通道传输；但是存在的区别如下。

**- Channel 和传统流的两个主要区别：**

* Channel 可以直接将指定文件的部分或全部直接映射成Buffer。它提供了一个 `map()` 方法，通过该 `map()` 方法可以直接将 “一块数据” 映射到内存中。
* 程序不能直接访问 Channel 中的数据，包括读取、写入都不行，Channel 只能与Buffer 进行交互。



Channel 接口提供的实现类有 `FileChannel` 、`SocketChannel` 、 `ServerSocketChannel` 等。和 `Buffer` 一样，Channel 也不该通过构造器直接创建，而是通过传统的节点 `InputStream`、`OutputStream` 的 `getChannel()` 方法来返回对应的 Channel，不同的节点流获得的 Channel 不一样 。



**- Channel 常用方法：**

* map()：用于将 Channel 对应的部分或者全部数据映射成 `ByteBuffer`
* read()：从 `Buffer` 读取数据
* write()：向 `Buffer` 写入数据

**注意：** map() 方法的方法签名为 `MappedByteBuffer map(FileChannel.MapMode mode, long position, long
size)` ，第一个参数执行映射时的模式，分别有只读、读写等模式；第二个、第三个参数用于控制将 Channel 的哪些数据映射成 ByteBuffer。

**下面例子示范了直接将 `FileChannel` 的全部数据映射成 `ByteBuffer` ：**

~~~java
public class FileChannelTest {
    public static void main(String[] args)
    {
        File f = new File("FileChannelTest.Java");
        
        try {
            //(1) Channel 的创建方式：通过 getChannel() 方法
            FileChannel inChannel = new FileInputStream(f).getChannel();
            FileChannel outChannel = new FileOutStream(a.txt).getChannel();
            
            //(2) 将 Channel 数据映射成 ByteBuffer
            MappedBytedBuffer buffer = inChannel.map(FileChannel.MapMode.READ_ONLY, 0, f.length());
            Charset charset = Charset.forName("GBK");	//使用 GBK 的字符集来创建解码器
            
            //(3) 向 Buffer 写入数据
            out.Channel.write(Buffer);
            
            buffer.clear();		//注意
            
            //(4) 创建解码器对象, 使用解码器完成转换
            CharsetDecoder decoder = decoder.decoder.decode(buffer);
            CharBuffer charBuffer = decoder.decode(buffer);
            
            System.out.println(charBuffer);
                
        }
    }
}
~~~

到 (3) 时，完成了将整个 `ByteBuffer` 的全部数据写入一个输出的 `FileChannel` ，这就完成了文件复制。 

之后 (4) 的部分是将文件的内容打印出来，使用了 `Charset 类` 和 `CharsetDecoder 类` 将  `ByteBuffer` 转换成 `CharBuffer` 。



### 4. 字符集 和 Charset

计算机里的文件、数据、图片文件只是一种表面现象，所有文件在底层都是二进制文件，即全部都是字节码。图片、音乐文件暂时先不说，对于文本文件而言，之所以可以看到一个个的字符，这完全是因为系统将底层的二进制序列转换成字符的缘故。在这个过程中涉及两个概念：`编码(Encode)`和 `解码(Decode)`  。

![image-20200722222640142](https://github.com/OnlyThePiano/Notes/blob/master/images/image-20200722222640142.png)

~~~java
public class CharsetTest {
    public static void main(String[] args)
    {
        //(1) 创建 Charset 对象
        Charset cn = Charset.forName("GBK");
        //(2) 获取 Charset 对应的编码器和解码器
        CharsetEncoder encoder = cn.newEncoder();
        CharsetDecoder decoder = cn.newDecoder();
        
        CharBuffer cbuff = CharBuffer.allocate(8);
        cbuff.put("A");
        cbuff.put("B");
        cbuff.put("C");
        cbuff.flip();
        //(3) 通过编码器将 CharBuffer 中的字符序列转换成字节序列
        ByteBuffer bbuff = encoder.encode(cbuff);
        for (int i=0; i < bbuff.capacity(); i++)
        {
            System.out.println(bbuff.get(i) + "");
        }
        
        //(4) 通过解码器又将 ByteBuffer 转换为字符序列
        System.out.println(decoder.decode(bbuff));
    }
}


运行结果:
65 
66 
67 

ABC
    
~~~



## 四、AIO (Asynchronous I/O)

AIO 也就是 NIO 2。AIO 是异步 IO 的缩写，虽然 NIO 在网络操作中，提供了非阻塞的方法，但是 NIO 的 IO 行为还是同步的。对于 NIO 来说，我们的业务线程是在 IO 操作准备好时，得到通知，接着就由这个线程自行进行 IO 操作，IO 操作本身是同步的。

NIO 2 对原来的 NIO 进行了改进主要包括如下两方面：

* 提供了全面的文件 IO 和文件系统访问支持。
* 基于异步 Channel 的 IO。



