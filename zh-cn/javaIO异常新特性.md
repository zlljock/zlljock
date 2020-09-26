### IO

1.3、IO流的分类
              1.3.1、按数据流的方向分为 输入流、输出流

　　　　　　　　此输入、输出是相对于我们写的代码程序而言，

　　　　　　　　输入流：从别的地方(本地文件，网络上的资源等)获取资源 输入到 我们的程序中

　　　　　　　　输出流：从我们的程序中 输出到 别的地方(本地文件)， 将一个字符串保存到本地文件中，就需要使用输出流。

　　　　1.3.2、按处理数据单位不同分为 字节流、字符流  

　　　　　　　　　1字符 = 2字节 、 1字节(byte) = 8位(bit)  、 一个汉字占两个字节长度

　　　　　　　字节流：每次读取(写出)一个字节，当传输的资源文件有中文时，就会出现乱码，

　　　　　　　　字符流：每次读取(写出)两个字节，有中文时，使用该流就可以正确传输显示中文。

　　　　1.3.3、按功能不同分为 节点流、处理流

　　　　　　　   节点流：以从或向一个特定的地方（节点）读写数据。如FileInputStream　

　　　　　　　   处理流：是对一个已存在的流的连接和封装，通过所封装的流的功能调用实现数据读写。如BufferedReader。处理流的构造方法总是要带一个其他的流对象做参数。一个流对象经过其他流的多次包装，

　　　　1.3.4、4个基本的抽象流类型，所有的流都继承这四个。

　　　　　　　　　　　输入流　　　　　　输出流

　　　　　　字节流　　InputStream　　outputStream

　　　　　　字符流　　Reader　　　　　　Writer

　　　　　　inputStream：字节输入流

　　　　　　outputStream：字节输出流

　　　　　　Reader：字符输入流

　　　　　　Writer：字符输出

　　　　1.5、总结流的分类

　　　　　　　　看上面的几个分类，可能对于初次学io的同学会感觉到有些混乱，那什么时候用字节流，什么时候该用输出流呢？其实非常简单，举一个例子就学会了，

　　　　　　　　1、首先自己要知道是选择输入流还是输出流，这就要根据自己的情况而定，如果你想从程序写东西到别的地方，那么就选择输出流，反之用输入流

　　　　　　　　2、然后考虑你传输数据时，是选择使用字节流传输还是字符流，也就是每次传1个字节还是2个字节，有中文肯定就选择字符流了。（详情见1.8）

　　　　　　　　3、前面两步就可以选出一个合适的节点流了，比如字节输入流inputStream，如果要在此基础上增强功能，那么就在处理流中选择一个合适的即可。

字符流的由来： Java中字符是采用Unicode标准，一个字符是16位，即一个字符使用两个字节来表示。为此，JAVA中引入了处理字符的流。因为数据编码的不同，而有了对字符进行高效操作的流对象。本质其实就是基于字节流读取时，去查了指定的码表。

1.4、IO流特性
1、先进先出，最先写入输出流的数据最先被输入流读取到。

2、顺序存取，可以一个接一个地往流中写入一串字节，读出时也将按写入顺序读取一串字节，不能随机访问中间的数据。（RandomAccessFile可以从文件的任意位置进行存取（输入输出）操作）

3、只读或只写，每个流只能是输入流或输出流的一种，不能同时具备两个功能，输入流只能进行读操作，对输出流只能进行写操作。在一个数据传输通道中，如果既要写入数据，又要读取数据，则要分别提供两个流。

1.5、IO流常用到的五类一接口
在整个Java.io包中最重要的就是5个类和一个接口。5个类指的是File、OutputStream、InputStream、Writer、Reader；一个接口指的是Serializable.掌握了这些IO的核心操作那么对于Java中的IO体系也就有了一个初步的认识了。

主要的类如下：

     1. File（文件特征与管理）：File类是对文件系统中文件以及文件夹进行封装的对象，可以通过对象的思想来操作文件和文件夹。 File类保存文件或目录的各种元数据信息，包括文件名、文件长度、最后修改时间、是否可读、获取当前文件的路径名，判断指定文件是否存在、获得当前目录中的文件列表，创建、删除文件和目录等方法。  
    
     2. InputStream（二进制格式操作）：抽象类，基于字节的输入操作，是所有输入流的父类。定义了所有输入流都具有的共同特征。
    
     3. OutputStream（二进制格式操作）：抽象类。基于字节的输出操作。是所有输出流的父类。定义了所有输出流都具有的共同特征。
    
     4.Reader（文件格式操作）：抽象类，基于字符的输入操作。
    
     5. Writer（文件格式操作）：抽象类，基于字符的输出操作。
    
     6. RandomAccessFile（随机文件操作）：一个独立的类，直接继承至Object.它的功能丰富，可以从文件的任意位置进行存取（输入输出）操作。

```
1、exists();判断文件（目录）是否存在
2、mkdir();创建一级目录；mkdirs()创建多级目录
3、delete();删除文件（目录）
4、isDirectory();判断是否是一个目录
5、isFile();判断是否是一个文件
6、createNewFile();创建一个文件
7、getAbsolutePath();获取绝对路径
8、getName()获取目录（文件）名称
9、getParent();获取父目录路径
10、getParentFile().getAbsolutePath();获取父目录文件的绝对路径
```

```

void close() 
关闭此输出流并释放与此流相关联的任何系统资源。  
void flush() 
刷新此输出流并强制任何缓冲的输出字节被写出。  
void write(byte[] b) 
将 b.length字节从指定的字节数组写入此输出流。  
void write(byte[] b, int off, int len) 
从指定的字节数组写入 len个字节，从偏移 off开始输出到此输出流。  
abstract void write(int b) 
将指定的字节写入此输出流。  



 inputStream
int available() 
返回从该输入流中可以读取（或跳过）的字节数的估计值，而不会被下一次调用此输入流的方法阻塞。  
void close() 
关闭此输入流并释放与流相关联的任何系统资源。  
void mark(int readlimit) 
标记此输入流中的当前位置。  
boolean markSupported() 
测试这个输入流是否支持 mark和 reset方法。  
abstract int read() 
从输入流读取数据的下一个字节。  返回-1
int read(byte[] b)     返回-1
从输入流读取一些字节数，并将它们存储到缓冲区 b 。   返回-1
int read(byte[] b, int off, int len) 
从输入流读取最多 len字节的数据到一个字节数组。  
void reset() 
将此流重新定位到上次在此输入流上调用 mark方法时的位置。  
long skip(long n) 
跳过并丢弃来自此输入流的 n字节数据。 

```



1.6、Java IO流对象


1.6.1、输入字节流InputStream


认识每个类的功能即作用

　　　　　ByteArrayInputStream：字节数组输入流，该类的功能就是从字节数组(byte[])中进行以字节为单位的读取，也就是将资源文件都以字节的形式存入到该类中的字节数组中去，我们拿也是从这个字节数组中拿

　　　　　PipedInputStream：管道字节输入流，它和PipedOutputStream一起使用，能实现多线程间的管道通信

　　　　　FilterInputStream ：装饰者模式中处于装饰者，具体的装饰者都要继承它，所以在该类的子类下都是用来装饰别的流的，也就是处理类。具体装饰者模式在下面会讲解到，到时就明白了

　　　　　BufferedInputStream：缓冲流，对处理流进行装饰，增强，内部会有一个缓存区，用来存放字节，每次都是将缓存区存满然后发送，而不是一个字节或两个字节这样发送。效率更高

　　　　   DataInputStream：数据输入流，它是用来装饰其它输入流，它“允许应用程序以与机器无关方式从底层输入流中读取基本 Java 数据类型”

　　　　　FileInputSream：文件输入流。它通常用于对文件进行读取操作

　　　　　File：对指定目录的文件进行操作，具体可以查看讲解File的博文。注意，该类虽然是在IO包下，但是并不继承自四大基础类。

 　　　　  ObjectInputStream：对象输入流，用来提供对“基本数据或对象”的持久存储。通俗点讲，也就是能直接传输对象（反序列化中使用），

1.6.2、输出字节流OutputStream

OutputStream 是所有的输出字节流的父类，它是一个抽象类。
ByteArrayOutputStream、FileOutputStream 是两种基本的介质流，它们分别向Byte 数组、和本地文件中写入数据。PipedOutputStream 是向与其它线程共用的管道中写入数据，
ObjectOutputStream 和所有FilterOutputStream 的子类都是装饰流(序列化中使用)。

1.6.3、字符输入流Reader

Reader 是所有的输入字符流的父类，它是一个抽象类。
CharReader、StringReader 是两种基本的介质流，它们分别将Char 数组、String中读取数据。PipedReader 是从与其它线程共用的管道中读取数据。
BufferedReader 很明显就是一个装饰器，它和其子类负责装饰其它Reader 对象。
FilterReader 是所有自定义具体装饰流的父类，其子类PushbackReader 对Reader 对象进行装饰，会增加一个行号。
InputStreamReader 是一个连接字节流和字符流的桥梁，它将字节流转变为字符流。FileReader 可以说是一个达到此功能、常用的工具类，在其源代码中明显使用了将FileInputStream 转变为Reader 的方法。我们可以从这个类中得到一定的技巧。Reader 中各个类的用途和使用方法基本和InputStream 中的类使用一致。后面会有Reader 与InputStream 的对应关系。
1.6.4、字符输出流Writer

Writer 是所有的输出字符流的父类，它是一个抽象类。
CharArrayWriter、StringWriter 是两种基本的介质流，它们分别向Char 数组、String 中写入数据。PipedWriter 是向与其它线程共用的管道中写入数据，
BufferedWriter 是一个装饰器为Writer 提供缓冲功能。
PrintWriter 和PrintStream 极其类似，功能和使用也非常相似。
OutputStreamWriter 是OutputStream 到Writer 转换的桥梁，它的子类FileWriter 其实就是一个实现此功能的具体类（具体可以研究一SourceCode）。功能和使用和OutputStream 极其类似，后面会有它们的对应图。


1.6.5、字节流和字符流使用情况：（重要）
字符流和字节流的使用范围：字节流一般用来处理图像，视频，以及PPT，Word类型的文件。字符流一般用于处理纯文本类型的文件，如TXT文件等，字节流可以用来处理纯文本文件，但是字符流不能用于处理图像视频等非文本类型的文件。

1.7、字符流与字节流转换


转换流的作用，文本文件在硬盘中以字节流的形式存储时，通过InputStreamReader读取后转化为字符流给程序处理，程序处理的字符流通过OutputStreamWriter转换为字节流保存。

转换流的特点：

其是字符流和字节流之间的桥梁
可对读取到的字节数据经过指定编码转换成字符
可对读取到的字符数据经过指定编码转换成字节
何时使用转换流？

当字节和字符之间有转换动作时；
流操作的数据需要编码或解码时。
具体的对象体现：

InputStreamReader:字节到字符的桥梁
OutputStreamWriter:字符到字节的桥梁
这两个流对象是字符体系中的成员，它们有转换作用，本身又是字符流，所以在构造的时候需要传入字节流对象进来。

 

OutputStreamWriter(OutStream out):将字节流以字符流输出。

InputStreamReader(InputStream in)：将字节流以字符流输入。

1.8、字节流和字符流的区别（重点）
字节流和字符流的区别：

         节流没有缓冲区，是直接输出的，而字符流是输出到缓冲区的。因此在输出时，字节流不调用colse()方法时，信息已经输出了，而字符流只有在调用close()方法关闭缓冲区时，信息才输出。要想字符流在未关闭时输出信息，则需要手动调用flush()方法。

·        读写单位不同：字节流以字节（8bit）为单位，字符流以字符为单位，根据码表映射字符，一次可能读多个字节。

·        处理对象不同：字节流能处理所有类型的数据（如图片、avi等），而字符流只能处理字符类型的数据。

结论：只要是处理纯文本数据，就优先考虑使用字符流。除此之外都使用字节流。

1.9、System类对IO的支持

 针对一些频繁的设备交互，Java语言系统预定了3个可以直接使用的流对象，分别是：

·        System.in（标准输入），通常代表键盘输入。

·        System.out（标准输出）：通常写往显示器。

·        System.err（标准错误输出）：通常写往显示器。

 标准I/O
      Java程序可通过命令行参数与外界进行简短的信息交换，同时，也规定了与标准输入、输出设备，如键盘、显示器进行信息交换的方式。而通过文件可以与外界进行任意数据形式的信息交换。

2.0、处理流BufferedReader，BufferedWriter,BufferedInputStream
BufferedOutputsStream，都要包上一层节点流。也就是说处理流是在节点流的基础之上进行的，带有Buffered的流又称为缓冲流，缓冲流处理文件的输入输出的速度是最快的。所以一般缓冲流的使用比较多。

```
public static void main(String[] args) {
        String path = "D:/learning/a.txt";
        FileReader reader =null;
        try {
            reader = new FileReader(path);
            char[] b = new char[1024];
            int len;
            while ((len=reader.read(b))>0){
                System.out.println(new String(b, 0, len));
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }


        FileInputStream inputStream = null;

        try {
            inputStream = new FileInputStream(path);
            byte[] c = new byte[1024];
            int len;
            while ((len=inputStream.read(c))>0){
                System.out.println(new String(c, 0, len));
            }
            inputStream.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

```



```
 public static void main(String[] args) {
        String path = "D:/learning/a.txt";
        String path2 = "D:/learning/b.txt";
        FileInputStream inputStream = null;
        FileOutputStream outputStream = null;
        try {
            inputStream = new FileInputStream(path2);
            outputStream = new FileOutputStream(path,true);
            byte[] bytes = new byte[1024];
            int len=0;
            while ((len =inputStream.read(bytes))>0){
                outputStream.write(bytes,0,len);
//                System.out.println((new String(bytes,0,len)));
            }
            outputStream.flush();
            inputStream.close();
            outputStream.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

```
 public static void main(String[] args) {
        String path = "D:/learning/a.txt";
        String path2 = "D:/learning/b.txt";
        FileInputStream inputStream =null;
        FileOutputStream outputStream = null;
        BufferedInputStream bufferedInputStream = null;
        BufferedOutputStream bufferedOutputStream = null;

        try {
            inputStream=new FileInputStream("path");
            outputStream = new FileOutputStream("path2");
            bufferedInputStream = new BufferedInputStream(inputStream);
            bufferedOutputStream = new BufferedOutputStream(outputStream);

            byte[] bytes = new byte[1024];
            int len;
            while ((len=bufferedInputStream.read())>0){
                bufferedOutputStream.write(bytes,0,len);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```



### NIO

提高速度：通道和缓冲器：

IO操作往往在两个场景

文件IO

网络IO

NIO的魅力：在网络中使用





- 可简单认为：**IO是面向流的处理，NIO是面向块(缓冲区)的处理**

- - 面向流的I/O 系统**一次一个字节地处理数据**。
  - 一个面向块(缓冲区)的I/O系统**以块的形式处理数据**。



NIO主要有**三个核心部分组成**：

- **buffer缓冲区**
- **Channel管道**
- **Selector选择器**

## 2.1buffer缓冲区和Channel管道

在NIO中并不是以流的方式来处理数据的，而是以buffer缓冲区和Channel管道**配合使用**来处理数据。

简单理解一下：

- Channel管道比作成铁路，buffer缓冲区比作成火车(运载着货物)

而我们的NIO就是**通过Channel管道运输着存储数据的Buffer缓冲区的来实现数据的处理**！

- 要时刻记住：Channel不与数据打交道，它只负责运输数据。与数据打交道的是Buffer缓冲区

- - **Channel-->运输**
  - **Buffer-->数据**



相对于传统IO而言，**流是单向的**。对于NIO而言，有了Channel管道这个概念，我们的**读写都是双向**的(铁路上的火车能从广州去北京、自然就能从北京返还到广州)！

### 2.1.1buffer缓冲区核心要点

我们来看看Buffer缓冲区有什么值得我们注意的地方。

Buffer是缓冲区的抽象类：

```
//Buffer的源码
public abstract class Buffer {

    /**
     * The characteristics of Spliterators that traverse and split elements
     * maintained in Buffers.
     */
    static final int SPLITERATOR_CHARACTERISTICS =
        Spliterator.SIZED | Spliterator.SUBSIZED | Spliterator.ORDERED;

    // Invariants: mark <= position <= limit <= capacity
    private int mark = -1;
    private int position = 0;
    private int limit;
    private int capacity;

```

Buffer类维护了4个核心变量属性来提供**关于其所包含的数组的信息**。它们是：

- 容量Capacity

- - **缓冲区能够容纳的数据元素的最大数量**。容量在缓冲区创建时被设定，并且永远不能被改变。(不能被改变的原因也很简单，底层是数组嘛)

- 上界Limit

- - **缓冲区里的数据的总数**，代表了当前缓冲区中一共有多少数据。

- 位置Position

- - **下一个要被读或写的元素的位置**。Position会自动由相应的 `get( )`和 `put( )`函数更新。

- 标记Mark

- - 一个备忘位置。**用于记录上一次读写的位置**。
  - 

缓冲区（Buffer）在java NIO中负责数据得存储，缓冲区就是数组，用于存储不同数据类型得数据列：

```
ByteBuffer// 其中ByteBuffer是用的最多得实现类(在管道中读写字节数据)
CharBuffer
ShortBuffer
IntBuffer
LongBuffer
FloatBuffer
DoubleBuffer
//上述缓冲区的管道方式几乎一致，通过allocate()获取缓冲区
```

就是**读取缓冲区的数据/写数据到缓冲区中**。所以，缓冲区的核心方法就是:

- `put()`
- `get()`

![image-20200515225216654](C:\Users\drew\AppData\Roaming\Typora\typora-user-images\image-20200515225216654.png)



### FileChannel通道核心要点

Channel通道**只负责传输数据、不直接操作数据的**。操作数据都是通过Buffer缓冲区来进行操作！

```
//1.通过本地IO的方式来获取通道
FileInputStream fileInputStream = new FileInputStream("物理地址");
//得到文件的输入通道
FileChannel inchannel = fileInputStream.getChannel();
//2.通过静态方法.open()获取通道
FileChannel.open(Path.get("物理地址"))
```

通道之间通过`transfer()`实现数据的传输(直接操作缓冲区)：

### 2.1.4直接与非直接缓冲区

- 非直接缓冲区是**需要**经过一个：copy的阶段的(从内核空间copy到用户空间)
- 直接缓冲区![img](https://pic1.zhimg.com/80/v2-51ec191305139e98a3b43d82c8ba17b1_720w.jpg)**不需要**经过copy阶段，也可以理解成--->**内存映射文件**，(上面的图片也有过例子)。![img](https://pic3.zhimg.com/80/v2-b75abc108506e1f48abbba0f91b80fdc_720w.jpg)

NIO被叫为 `no-blocking io`，其实是在**网络这个层次中理解的**，对于**FileChannel来说一样是阻塞**。

所以说：我们**通常**使用NIO是在网络中使用的，网上大部分讨论NIO都是在**网络通信的基础之上**的！说NIO是非阻塞的NIO也是**网络中体现**的！

从上面的图我们可以发现还有一个`Selector`选择器这么一个东东。从一开始我们就说过了，nio的**核心要素**有：

- Buffer缓冲区
- Channel通道
- Selector选择器

我们在网络中使用NIO往往是I/O模型的**多路复用模型**！

- Selector选择器就可以比喻成麦当劳的**广播**。
- **一个线程能够管理多个Channel的状态**

```
 public static void main(String[] args) {
        //创建一个缓存区
            ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

            //看一下初始时4个核心变量的值
        System.out.println("初始值-->limit-->"+byteBuffer.limit());
        System.out.println("初始值-->position-->"+byteBuffer.position());
        System.out.println("初始值-->capacity-->"+byteBuffer.capacity());
        System.out.println("初始值-->mark-->"+byteBuffer.mark());
        System.out.println("=========================================");
        //添加一些数据到缓冲区
        String str= "java";
        byteBuffer.put(str.getBytes());

        //看下初始时4个核心变量的值
        System.out.println("put完之后--->limit---->"+byteBuffer.limit());
        System.out.println("put完之后--->position---->"+byteBuffer.position());
        System.out.println("put完之后--->capacity---->"+byteBuffer.capacity());
        System.out.println("put完之后--->mark---->"+byteBuffer.mark());
        //切换成读模式，我们就可以读取缓冲区的数据
        System.out.println( byteBuffer.flip());

        byte[] bytes = new byte[byteBuffer.limit()];
        byteBuffer.get(bytes);
        System.out.println(new String(bytes,0,bytes.length));
        System.out.println(new String(bytes));

        System.out.println("========================");
        System.out.println("get完之后--->limit---->"+byteBuffer.limit());
        System.out.println("get完之后--->position---->"+byteBuffer.position());
        System.out.println("get完之后--->capacity---->"+byteBuffer.capacity());
        System.out.println("get完之后--->mark---->"+byteBuffer.mark());

        byteBuffer.clear();
        //这个函数会清空缓冲区
        //数据没有真正的被清空，只是被遗忘掉了
        System.out.println("========================");
        System.out.println("clear完之后--->limit---->"+byteBuffer.limit());
        System.out.println("clear完之后--->position---->"+byteBuffer.position());
        System.out.println("clear完之后--->capacity---->"+byteBuffer.capacity());
        System.out.println("clear完之后--->mark---->"+byteBuffer.mark());
    }
    
    
    
   //结果 
初始值-->limit-->1024
初始值-->position-->0
初始值-->capacity-->1024
初始值-->mark-->java.nio.HeapByteBuffer[pos=0 lim=1024 cap=1024]
=========================================
put完之后--->limit---->1024
put完之后--->position---->4
put完之后--->capacity---->1024
put完之后--->mark---->java.nio.HeapByteBuffer[pos=4 lim=1024 cap=1024]
java.nio.HeapByteBuffer[pos=0 lim=4 cap=1024]
java
java
========================
get完之后--->limit---->4
get完之后--->position---->4
get完之后--->capacity---->1024
get完之后--->mark---->java.nio.HeapByteBuffer[pos=4 lim=4 cap=1024]
========================
clear完之后--->limit---->1024
clear完之后--->position---->0
clear完之后--->capacity---->1024
clear完之后--->mark---->java.nio.HeapByteBuffer[pos=0 lim=1024 cap=1024]
```



```
* 客户端
 */
public class Test2 {
    public static void main(String[] args) throws IOException {
       /* InetAddress address = InetAddress.getLocalHost();
        System.out.println(address);
        System.out.println(address.getHostAddress());*/
        SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("192.168.199.1",6667));

        FileChannel open = FileChannel.open(Paths.get("C:\\Users\\drew\\Desktop\\login.jpg"), StandardOpenOption.READ);

        ByteBuffer buffer = ByteBuffer.allocate(1024);

        while (open.read(buffer) !=-1){
            buffer.flip();
            socketChannel.write(buffer);
            buffer.clear();
        }


        //知道服务端要返回响应的数据给客户端，这边要接收下
       /* int len = 0;
        while ((len = socketChannel.read(buffer))!=-1){
            //切换读模式
            buffer.flip();
            System.out.println(new String(buffer.array(),0,len));
            //切换成写模式
            buffer.clear();
        }*/
        open.close();
        socketChannel.close();

    }
}




* 服务器
 */
public class Test2ToTest3 {

    public static void main(String[] args) throws IOException {

        // 1.获取通道
        ServerSocketChannel server = ServerSocketChannel.open();

        // 2.得到文件通道，将客户端传递过来的图片写到本地项目下(写模式、没有则创建)
        FileChannel outChannel = FileChannel.open(Paths.get("login.jpg"), StandardOpenOption.WRITE, StandardOpenOption.CREATE);

        // 3. 绑定链接
        server.bind(new InetSocketAddress(6667));

        // 4. 获取客户端的连接(阻塞的)
        SocketChannel client = server.accept();

        // 5. 要使用NIO，有了Channel，就必然要有Buffer，Buffer是与数据打交道的呢
        ByteBuffer buffer = ByteBuffer.allocate(1024);

        // 6.将客户端传递过来的图片保存在本地中
        while (client.read(buffer) != -1) {

            // 在读之前都要切换成读模式
            buffer.flip();

            outChannel.write(buffer);

            // 读完切换成写模式，能让管道继续读取文件的数据
            buffer.clear();

        }

        //这边服务端保存了图片之后，可以告诉客户端图片已经上传成果
//        buffer.put("img is success".getBytes());
//        buffer.flip();
//        client.write(buffer);
//        buffer.clear();

        // 7.关闭通道
        outChannel.close();
        client.close();
        server.close();
    }
}
```





### Serializable

什么是Serializable接口？

一个对象序列化的接口，一个类只有实现了Serializable接口，它的对象才能被序列化

什么是序列化？

将对象的状态信息转换为可以存储或传输的形式的过程，在序列化期间，对象将其当前状态写入到临时存储区或持久性存储区，之后，便可以通过从存储区中读取或反序列化对象的状态信息，来重新创建该对象

什么情况下需要序列化？

当我们需要把对象的状态信息通过网络进行传输，或者需要将对象的状态信息持久化，以便将来使用时都需要把对象进行序列

一、基本概念

1、序列化和反序列化的定义：

    (1)Java序列化就是指把Java对象转换为字节序列的过程
    
        Java反序列化就是指把字节序列恢复为Java对象的过程。

   (2)序列化最重要的作用：在传递和保存对象时.保证对象的完整性和可传递性。对象转换为有序字节流,以便在网络上传输或者保存在本地文件中。

       反序列化的最重要的作用：根据字节流中保存的对象状态及描述信息，通过反序列化重建对象。

   总结：核心作用就是对象状态的保存和重建。（整个过程核心点就是字节流中所保存的对象状态及描述信息）
