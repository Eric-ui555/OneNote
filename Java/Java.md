# JavaSE

## IO流

根据冯.诺依曼结构，计算机结构分为 5 大部分：**运算器、控制器、存储器、输入设备、输出设备**。

从计算机结构的视角来看： **I/O 描述了计算机系统与外部设备之间通信的过程。**

IO流定义：**存储和读取数据的解决方案**

作用：用于读写数据（本地文件和网络）

分类：

- 根据数据流向分类：
  - 输入流（文件->程序）
  - 输出流（程序->文件）
- 根据操作文件类型分类：
  - 字节流：可以操作所有类型的文件
  - 字符流：只能操作纯文本文件

### 体系结构

继承四个超类：`InputStream`、`OutputStream`、`Reader`、`Writer`

#### 1、文件IO流

- `FileIutputStream`：操作本地文件的字节输入流

```java
// 1. 创建字节输入流对象
// 如果文件不存在，就直接报错
FileInputStream inputStream = new FileInputStream(new File("a.txt"));
// 2. 读数据
// 一次都一个字节，读出来的是数据在ASCII上对应的数字
// 读到文件末尾了，read方法返回-1
int read = inputStream.read();
System.out.println(read);
// 3。释放资源
// 注意：每次使用完流之后都要释放资源
inputStream.close();
```

- `FileOutputStream`：操作本地文件的字节输出流


```java
// 1. 创建字节输出流对象
// 注意1：参数是字符串表示的路径或者File对象都是可以的
// 注意2：如果文件不存在会创建一个新的文件，但是要保证父级路径是存在的
// 注意3：如果文件已经存在，则会清空文件
FileOutputStream outputStream = new FileOutputStream("a.txt");
// 2. 写数据
// 注意：write方法的参数是证书，但是实际上写到本地文件中的是证书在ASCII上对应的字符
outputStream.write(97);
// 3。释放资源
// 注意：每次使用完流之后都要释放资源
outputStream.close();
```

- 换行符：windows：`\r\n`，linux：`\n`，mac：`\r`

- 续写：打开续写开关：`FileOutputStream(String name, boolean append)`

#### 2、管道IO流

- `PipedInputStream`（字节输入流）
- `PipedOutputStream`（字节输出流）
- `PipedReader`（字符输入流）
- `PipedWriter`（字符输出流）

#### 3、字节/字符数组

- 字节数组输入流：`ByteArrayInputStream`

- 字节数组输出流：`ByteArrayOutputStream`

- 字符数组输入流：`CharArrayReader`

- 字符数组输出流：`CharArrayWriter`

#### 4、**Buffered 缓冲流**

**字节缓冲流**：IO 操作是很消耗性能的，缓冲流将数据加载至缓冲区，一次性读取/写入多个字节，从而避免频繁的 IO 操作，提高流的传输效率。字节缓冲流这里采用了**装饰器模式**来增强 `InputStream` 和`OutputStream`子类对象的功能。

- `BufferedInputStream`：从源头（通常是文件）读取数据（字节信息）到内存的过程中不会一个字节一个字节的读取，而是会先将读取到的字节存放在缓存区，并从内部缓冲区中单独读取字节。这样大幅减少了 IO 次数，提高了读取效率。

  ```java
  // 新建一个 BufferedInputStream 对象
  BufferedInputStream bufferedInputStream = new BufferedInputStream(new FileInputStream("input.txt"));
  // 读取文件的内容并复制到 String 对象中
  String result = new String(bufferedInputStream.readAllBytes());
  System.out.println(result);
  ```

  字节流和字节缓冲流的性能差别主要体现在我们使用两者的时候都是调用 `write(int b)` 和 `read()` 这两个一次只读取一个字节的方法的时候。由于字节缓冲流内部有缓冲区（字节数组），因此，字节缓冲流会先将读取到的字节存放在缓存区，大幅减少 IO 次数，提高读取效率。

  缓冲区源码：

  ```java
  public class BufferedInputStream extends FilterInputStream {
      // 内部缓冲区数组
      protected volatile byte buf[];
      // 缓冲区的默认大小
      private static int DEFAULT_BUFFER_SIZE = 8192;
      // 使用默认的缓冲区大小
      public BufferedInputStream(InputStream in) {
          this(in, DEFAULT_BUFFER_SIZE);
      }
      // 自定义缓冲区大小
      public BufferedInputStream(InputStream in, int size) {
          super(in);
          if (size <= 0) {
              throw new IllegalArgumentException("Buffer size <= 0");
          }
          buf = new byte[size];
      }
  }
  ```
  
- `BufferedOutputStream`：将数据（字节信息）写入到目的地（通常是文件）的过程中不会一个字节一个字节的写入，而是会先将要写入的字节存放在缓存区，并从内部缓冲区中单独写入字节。这样大幅减少了 IO 次数，提高了读取效率

  ```java
  FileOutputStream fileOutputStream = new FileOutputStream("output.txt");
  BufferedOutputStream bos = new BufferedOutputStream(fileOutputStream)
  ```

**字符缓冲流**

- `BufferedReader`
- `BufferedWriter`

#### 5、**字节转化成字符流**

- `InputStreamReader`：是字节流转换为字符流的桥梁，其子类 `FileReader` 是基于该基础上的封装，可以直接操作字符文件。

  ```java
  // 字节流转换为字符流的桥梁
  public class InputStreamReader extends Reader {
  }
  // 用于读取字符文件
  public class FileReader extends InputStreamReader {
  }
  ```

  `FileReader` 代码示例：

  ```java
  try (FileReader fileReader = new FileReader("input.txt");) {
      int content;
      long skip = fileReader.skip(3);
      System.out.println("The actual number of bytes skipped:" + skip);
      System.out.print("The content read from file:");
      while ((content = fileReader.read()) != -1) {
          System.out.print((char) content);
      }
  } catch (IOException e) {
      e.printStackTrace();
  }
  ```

- `OutputStreamWriter`：是字符流转换为字节流的桥梁，其子类 `FileWriter` 是基于该基础上的封装，可以直接将字符写入到文件。

  ```java
  // 字符流转换为字节流的桥梁
  public class OutputStreamWriter extends Writer {
  }
  // 用于写入字符到文件
  public class FileWriter extends OutputStreamWriter {
  }
  ```

  ```java
  // 字符流转换为字节流的桥梁
  public class OutputStreamWriter extends Writer {
  }
  // 用于写入字符到文件
  public class FileWriter extends OutputStreamWriter {
  }
  ```

  `FileWriter` 代码示例：

  ```java
  try (Writer output = new FileWriter("output.txt")) {
      output.write("你好，我是Guide。");
  } catch (IOException e) {
      e.printStackTrace();
  }
  ```

#### 6、**数据流**

- `DataInputStream：`： 用于读取指定类型数据，不能单独使用，必须结合其它流，比如 `FileInputStream` 

  ```java
  FileInputStream fileInputStream = new FileInputStream("input.txt");
  // 必须将fileInputStream作为构造参数才能使用
  DataInputStream dataInputStream = new DataInputStream(fileInputStream);
  // 可以读取任意具体的类型数据
  dataInputStream.readBoolean();
  dataInputStream.readInt();
  dataInputStream.readUTF();
  ```

- `DataOutputStream`：用于写入指定类型数据，不能单独使用，必须结合其它流，比如 `FileOutputStream` 

  ```java
  // 输出流
  FileOutputStream fileOutputStream = new FileOutputStream("out.txt");
  DataOutputStream dataOutputStream = new DataOutputStream(fileOutputStream);
  // 输出任意数据类型
  dataOutputStream.writeBoolean(true);
  dataOutputStream.writeByte(1);
  ```

#### 7、**打印流**

- `PrintStream`
  - `System.out` 实际是用于获取一个 `PrintStream` 对象，`print`方法实际调用的是 `PrintStream` 对象的 `write` 方法。
  - `PrintStream` 是 `OutputStream` 的子类
- `PrintWriter`
- - `PrintWriter` 是 `Writer` 的子类。

```java
public class PrintStream extends FilterOutputStream
    implements Appendable, Closeable {
}
public class PrintWriter extends Writer {
}
```

#### 8、**对象流**

- `ObjectInputStream`：用于从输入流中读取 Java 对象（反序列化）

```java
ObjectInputStream input = new ObjectInputStream(new FileInputStream("object.data"));
MyClass object = (MyClass) input.readObject();
input.close();
```

- `ObjectOutputStream`：用于将对象写入到输出流(序列化)

```java
ObjectOutputStream output = new ObjectOutputStream(new FileOutputStream("file.txt")
Person person = new Person("Guide哥", "JavaGuide作者");
output.writeObject(person);
```

#### 9、**序列化流**

- `SequenceInputStream`

> **那为什么 I/O 流操作要分为字节流操作和字符流操作呢？**
>
> - 字符流是由 Java 虚拟机将字节转换得到的，这个过程还算是比较耗时。
> - 如果我们不知道编码类型就很容易出现乱码问题。

### 字符集

在计算机中，任意数据都是以二进制的形式组织的

**计算机存储规则**

`ASCII`：一个英文占一个字节

`GB2312字符集`：1980年发布，1981年5月1日实施的简体中文汉字编码国家标准

- 收录7447个图形字符，其中包括6763个简体汉字

`BIG5字符集`：台湾地区繁体中文标准字符集，共收录13053个中文字，1984年实施

`GBK字符集`：2000年3月17日发布，收录21003个汉字

- 包含国际标准`GB13000-1`中的全部中日韩汉字，和`BIG5`字符集的所有汉字
- windows系统默认使用的`GBK`，系统显示`ANSI`
- 英文：一个字节存储，完全兼容ASCII，二进制前面补0
- 汉字：汉字两个字节存储，高位字节二进制一定以1开头，转成十进制之后是一个负数

`Unicode字符集`：国际标准字符集，它将世界各种语言的每个字符定义一个唯一的编码，以满足跨语言、跨平台的文本信息转换

- **UTF-16编码规则**：用2-4个字节保存
- **UTF-32编码规则**：固定使用四个字节保存
- **UTF-8编码规则**：用1-4个字节保存
  - **英文字母：1个字节**，二进制的第一位是0，转成十进制是正数
  - **中文汉字：3个字节**，二进制的第一位是1，第一个字节转成十进制是负数

> 为什么会有乱码？

- 读取数字时未读完整个汉字
- 编码和解码的规则不一致

### IO 设计模式

#### 装饰器（Decorator）模式

**装饰器（Decorator）模式** 可以在不改变原有对象的情况下拓展其功能。

装饰器模式通过组合替代继承来扩展原始类的功能，在一些继承关系比较复杂的场景（IO 这一场景各种类的继承关系就比较复杂）更加实用。

对于字节流来说， `FilterInputStream` （对应输入流）和`FilterOutputStream`（对应输出流）是装饰器模式的核心，分别用于增强 `InputStream` 和`OutputStream`子类对象的功能。

我们常见的`BufferedInputStream`(字节缓冲输入流)、`DataInputStream` 等等都是`FilterInputStream` 的子类，`BufferedOutputStream`（字节缓冲输出流）、`DataOutputStream`等等都是`FilterOutputStream`的子类。

**装饰器类需要跟原始类继承相同的抽象类或者实现相同的接口**。上面介绍到的这些 IO 相关的装饰类和原始类共同的父类是 `InputStream` 和`OutputStream`。

#### 适配器模式

**适配器（Adapter Pattern）模式** 主要用于**接口互不兼容的类的协调工作**，你可以将其联想到我们日常经常使用的电源适配器。

适配器模式中存在被适配的对象或者类称为 **适配者(Adaptee)** ，作用于适配者的对象或者类称为**适配器(Adapter)** 。适配器分为对象适配器和类适配器。**类适配器使用继承关系来实现，对象适配器使用组合关系来实现。**

IO 流中的字符流和字节流的接口不同，它们之间可以协调工作就是基于适配器模式来做的，更准确点来说是对象适配器。通过适配器，我们可以将字节流对象适配成一个字符流对象，这样我们可以直接通过字节流对象来读取或者写入字符数据。

`InputStreamReader` 和 `OutputStreamWriter` 就是两个适配器(Adapter)， 同时，它们两个也是字节流和字符流之间的桥梁。`InputStreamReader` 使用 `StreamDecoder` （流解码器）对字节进行解码，**实现字节流到字符流的转换，** `OutputStreamWriter` 使用`StreamEncoder`（流编码器）对字符进行编码，实现字符流到字节流的转换。

`InputStream` 和 `OutputStream` 的子类是被适配者， `InputStreamReader` 和 `OutputStreamWriter`是适配器。

```java
// InputStreamReader 是适配器，FileInputStream 是被适配的类
InputStreamReader isr = new InputStreamReader(new FileInputStream(fileName), "UTF-8");
// BufferedReader 增强 InputStreamReader 的功能（装饰器模式）
BufferedReader bufferedReader = new BufferedReader(isr);
```

`java.io.InputStreamReader` 部分源码：

```java
public class InputStreamReader extends Reader {
    //用于解码的对象
    private final StreamDecoder sd;
    public InputStreamReader(InputStream in) {
        super(in);
        try {
            // 获取 StreamDecoder 对象
            sd = StreamDecoder.forInputStreamReader(in, this, (String)null);
        } catch (UnsupportedEncodingException e) {
            throw new Error(e);
        }
    }
    // 使用 StreamDecoder 对象做具体的读取工作
    public int read() throws IOException {
        return sd.read();
    }
}
```

`java.io.OutputStreamWriter` 部分源码：

```java
public class OutputStreamWriter extends Writer {
    // 用于编码的对象
    private final StreamEncoder se;
    public OutputStreamWriter(OutputStream out) {
        super(out);
        try {
           // 获取 StreamEncoder 对象
            se = StreamEncoder.forOutputStreamWriter(out, this, (String)null);
        } catch (UnsupportedEncodingException e) {
            throw new Error(e);
        }
    }
    // 使用 StreamEncoder 对象做具体的写入工作
    public void write(int c) throws IOException {
        se.write(c);
    }
}
```

**适配器模式和装饰器模式有什么区别呢？**

**装饰器模式** 更侧重于**动态地增强原始类的功能**，装饰器类需要跟原始类继承相同的抽象类或者实现相同的接口。并且，装饰器模式支持对原始类嵌套使用多个装饰器。

**适配器模式** 更侧重于**让接口不兼容而不能交互的类可以一起工作**，当我们调用适配器对应的方法时，适配器内部会调用适配者类或者和适配类相关的类的方法，这个过程透明的。就比如说 `StreamDecoder` （流解码器）和`StreamEncoder`（流编码器）就是分别基于 `InputStream` 和 `OutputStream` 来获取 `FileChannel`对象并调用对应的 `read` 方法和 `write` 方法进行字节数据的读取和写入。

```java
StreamDecoder(InputStream in, Object lock, CharsetDecoder dec) {
    // 省略大部分代码
    // 根据 InputStream 对象获取 FileChannel 对象
    ch = getChannel((FileInputStream)in);
}
```

适配器和适配者两者不需要继承相同的抽象类或者实现相同的接口。

另外，`FutureTask` 类使用了适配器模式，`Executors` 的内部类 `RunnableAdapter` 实现属于适配器，用于将 `Runnable` 适配成 `Callable`。

`FutureTask`参数包含 `Runnable` 的一个构造方法。

```java
public FutureTask(Runnable runnable, V result) {
    // 调用 Executors 类的 callable 方法
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;
}
```

`Executors`中对应的方法和适配器：

```java
// 实际调用的是 Executors 的内部类 RunnableAdapter 的构造方法
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}
// 适配器
static final class RunnableAdapter<T> implements Callable<T> {
    final Runnable task;
    final T result;
    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}
```

#### 工厂模式

工厂模式用于创建对象，NIO 中大量用到了工厂模式，比如 `Files` 类的 `newInputStream` 方法用于创建 `InputStream` 对象（静态工厂）、 `Paths` 类的 `get` 方法创建 `Path` 对象（静态工厂）、`ZipFileSystem` 类（`sun.nio`包下的类，属于 `java.nio` 相关的一些内部实现）的 `getPath` 的方法创建 `Path` 对象（简单工厂）

```java
InputStream is = Files.newInputStream(Paths.get(generatorLogoPath))
```

#### 观测者模式

NIO 中的文件目录监听服务使用到了观察者模式。

NIO 中的文件目录监听服务基于 `WatchService` 接口和 `Watchable` 接口。`WatchService` 属于观察者，`Watchable` 属于被观察者。

`Watchable` 接口定义了一个用于将对象注册到 `WatchService`（监控服务） 并绑定监听事件的方法 `register` 。

```java
public interface Path
    extends Comparable<Path>, Iterable<Path>, Watchable{
}

public interface Watchable {
    WatchKey register(WatchService watcher,
                      WatchEvent.Kind<?>[] events,
                      WatchEvent.Modifier... modifiers)
        throws IOException;
}
```

`WatchService` 用于监听文件目录的变化，同一个 `WatchService` 对象能够监听多个文件目录。

```java
// 创建 WatchService 对象
WatchService watchService = FileSystems.getDefault().newWatchService();

// 初始化一个被监控文件夹的 Path 类:
Path path = Paths.get("workingDirectory");
// 将这个 path 对象注册到 WatchService（监控服务） 中去
WatchKey watchKey = path.register(
watchService, StandardWatchEventKinds...);
```

`Path` 类 `register` 方法的第二个参数 `events` （需要监听的事件）为可变长参数，也就是说我们可以同时监听多种事件。

```java
WatchKey register(WatchService watcher,
                  WatchEvent.Kind<?>... events)
    throws IOException;
```

常用的监听事件有 3 种：

- `StandardWatchEventKinds.ENTRY_CREATE`：文件创建。
- `StandardWatchEventKinds.ENTRY_DELETE` : 文件删除。
- `StandardWatchEventKinds.ENTRY_MODIFY` : 文件修改。

`register` 方法返回 `WatchKey` 对象，通过`WatchKey` 对象可以获取事件的具体信息比如文件目录下是创建、删除还是修改了文件、创建、删除或者修改的文件的具体名称是什么

```java
WatchKey key;
while ((key = watchService.take()) != null) {
    for (WatchEvent<?> event : key.pollEvents()) {
      // 可以调用 WatchEvent 对象的方法做一些事情比如输出事件的具体上下文信息
    }
    key.reset();
}
```

`WatchService` 内部是通过一个 daemon thread（守护线程）采用定期轮询的方式来检测文件的变化，简化后的源码如下所示。

```java
class PollingWatchService
    extends AbstractWatchService
{
    // 定义一个 daemon thread（守护线程）轮询检测文件变化
    private final ScheduledExecutorService scheduledExecutor;

    PollingWatchService() {
        scheduledExecutor = Executors
            .newSingleThreadScheduledExecutor(new ThreadFactory() {
                 @Override
                 public Thread newThread(Runnable r) {
                     Thread t = new Thread(r);
                     t.setDaemon(true);
                     return t;
                 }});
    }

  void enable(Set<? extends WatchEvent.Kind<?>> events, long period) {
    synchronized (this) {
      // 更新监听事件
      this.events = events;

        // 开启定期轮询
      Runnable thunk = new Runnable() { public void run() { poll(); }};
      this.poller = scheduledExecutor
        .scheduleAtFixedRate(thunk, period, period, TimeUnit.SECONDS);
    }
  }
}
```

### IO 设计模型

**同步和异步**

- 同步：发出一个调用时，在没有得到结果之前，该调用就不返回
- 异步：在调用发出后，被调用者返回结果之后会通知调用者，或通过回调函数处理这个调用

**阻塞和非阻塞**

- 阻塞和非阻塞关注的是**线程的状态**。
- 阻塞调用是指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会恢复运行。
- 非阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程。

UNIX 系统下， IO 模型一共有 5 种：**同步阻塞 I/O**、**同步非阻塞 I/O**、**I/O 多路复用**、**信号驱动 I/O** 和**异步 I/O**。

Java中3种常见的IO模型：`BIO（Blocking I/O）、NIO、AIO`

#### BIO(Blocking I/O)

**`BIO` 属于同步阻塞 IO 模型** 。

同步阻塞 IO 模型中，应用程序发起 read 调用后，会一直阻塞，直到内核把数据拷贝到用户空间。

<img src="Java.assets/6a9e704af49b4380bb686f0c96d33b81tplv-k3u1fbpfcp-watermark.png" alt="图源：《深入拆解Tomcat & Jetty》" style="zoom: 50%;" />

在**客户端连接数量不高**的情况下，是没问题的。但是，当面对十万甚至百万级连接的时候，传统的 BIO 模型是无能为力的。因此，我们需要一种更高效的 I/O 处理模型来应对更高的并发量。

#### NIO(Non-blocking/New I/O)

Java 中的 `NIO` 于 Java 1.4 中引入，对应 `java.nio` 包，提供了 `Channel` , `Selector`，`Buffer` 等抽象。

`NIO` 中的 N 可以理解为 `Non-blocking`，不单纯是 New。它是支持面向缓冲的，基于通道的 I/O 操作方法。 对于高负载、高并发的（网络）应用，应使用 `NIO` 。

Java 中的 `NIO` 可以看作是 **I/O 多路复用模型**。

##### **1、同步非阻塞模型**

<img src="Java.assets/bb174e22dbe04bb79fe3fc126aed0c61tplv-k3u1fbpfcp-watermark.png" alt="图源：《深入拆解Tomcat & Jetty》" style="zoom:50%;" />

同步非阻塞 IO 模型中，应用程序会一直发起 read 调用，等待数据从内核空间拷贝到用户空间的这段时间里，线程依然是阻塞的，直到在内核把数据拷贝到用户空间。

> 相较于BIO的优点：通过**轮询操作**，避免了一直阻塞
>
> 缺点：**应用程序不断进行 I/O 系统调用轮询数据是否已经准备好的过程是十分消耗 CPU 资源的。**

##### **2、I/O多路复用模型**

IO 多路复用模型中，线程首先发起 **select 调用**，询问内核数据是否准备就绪，等内核把数据准备好了，用户线程再发起 read 调用。read 调用的过程（数据从内核空间 -> 用户空间）还是阻塞的。

**IO 多路复用模型，通过减少无效的系统调用，减少了对 CPU 资源的消耗。**

<img src="Java.assets/88ff862764024c3b8567367df11df6abtplv-k3u1fbpfcp-watermark.png" alt="img" style="zoom:50%;" />

#### AIO(Asynchronous I/O)

`AIO` 也就是 `NIO 2`。Java 7 中引入了 `NIO` 的改进版 `NIO 2`，它是异步 IO 模型。

异步 IO 是基于**事件和回调机制**实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

<img src="Java.assets/3077e72a1af049559e81d18205b56fd7tplv-k3u1fbpfcp-watermark.png" alt="img" style="zoom:50%;" />

------

## 集合

总的区分：

`List`(对付顺序的好帮手): 存储的元素是有序的、可重复的。

`Set`(注重独一无二的性质): 存储的元素不可重复的。

`Queue`(实现排队功能的叫号机): 按特定的排队规则来确定先后顺序，存储的元素是有序的、可重复的。

`Map`(用 key 来搜索的专家): 使用键值对（key-value）存储，key 是无序的、不可重复的，value 是无序的、可重复的，每个键最多映射到一个值

![Java 集合框架概览](Java.assets/java-collection-hierarchy.png)

### Collections

#### Set

#### List

#### Queue

### Map

------

## 多线程

[Java多线程最佳实践](https://juejin.cn/post/7284221961621848119#heading-8)

[Java并发常见面试题](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#%E5%A6%82%E4%BD%95%E4%BF%9D%E8%AF%81%E5%8F%98%E9%87%8F%E7%9A%84%E5%8F%AF%E8%A7%81%E6%80%A7)

### 线程池

#### **线程池创建核心参数**

<img src="Java.assets/640.webp" alt="图片" style="zoom: 67%;" />

> 解释：
>
> **`corePoolSize`**：定义了线程池中的核心线程数量。即使这些线程处于空闲状态，它们也不会被回收。这是线程池保持在等待状态下的线程数。
>
> `maximumPoolSize`：线程池允许的最大线程数量。当工作队列满了之后，线程池会创建新线程来处理任务，直到线程数达到这个最大值。
>
> **`keepAliveTime`**：非核心线程的空闲存活时间。如果线程池中的线程数量超过了 corePoolSize，那么这些多余的线程在空闲时间超过 keepAliveTime 时会被终止。
>
> **`unit`**：keepAliveTime 参数的时间单位：
>
> **`workQueue`**：用于存放待处理任务的阻塞队列。当所有核心线程都忙时，新任务会被放在这个队列里等待执行。
>
> **`threadFactory`**：一个创建新线程的工厂。它用于创建线程池中的线程。可以通过自定义 `ThreadFactory` 来给线程池中的线程设置有意义的名字，或设置优先级等。
>
> `handler`：拒绝策略 `RejectedExecutionHandler`，定义了当线程池和工作队列都满了之后对新提交的任务的处理策略。常见的拒绝策略包括抛出异常、直接丢弃、丢弃队列中最老的任务、由提交任务的线程来直接执行任务等。

#### 线程池的工作流程

> 当应用程序提交一个任务时，线程池会根据**当前线程的状态和参数**决定如何处理这个任务。
>
> - 如果线程池中的核心线程都在忙，并且线程池未达到最大线程数，新提交的任务会被放入队列中进行等待。
> - 如果任务队列已满，且当前线程数量小于最大线程数，线程池会创建新的线程来处理任务。
>
> 空闲的线程会从任务队列中取出任务来执行，当任务执行完毕后，线程并不会立即销毁，而是继续保持在池中等待下一个任务。
>
> 当线程空闲时间超出指定时间，且当前线程数量大于核心线程数时，线程会被回收。



------

## 反射

概念：

- 反射赋予了我们**在运行时分析类以及执行类中方法**的能力
- 通过反射获取任意一个类的所有属性和方法，还可以调用这些方法和属性

应用场景

- 动态代理

```java
public class DebugInvocationHandler implements InvocationHandler {
    /**
     * 代理类中的真实对象
     */
    private final Object target;

    public DebugInvocationHandler(Object target) {
        this.target = target;
    }


    public Object invoke(Object proxy, Method method, Object[] args) throws InvocationTargetException, IllegalAccessException {
        System.out.println("before method " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("after method " + method.getName());
        return result;
    }
}
```

- 注解：

  - @Component 声明一个类为Spring Bean 

  - @Value 读取配置文件中的值

优缺点：

- 优点：可以让咱们的代码更加灵活、为各种框架提供开箱即用的功能提供了便利
- 缺点：增加了安全问题，比如可以无视泛型参数的安全检查（泛型参数的安全检查发生在编译时）

获取Class的四种方式

- 1，知道具体类的情况下：`Class alunbarClass = TargetObject.class;`
- 2，通过 `Class.forName()`传入类的全路径获取：`Class alunbarClass1 = Class.forName("cn.javaguide.TargetObject");`
- 3，通过对象实例获取`instance.getClass()`获取：`TargetObject o = new TargetObject();  Class alunbarClass2 = o.getClass();`
- 4，通过类加载器`xxxClassLoader.loadClass()`传入类路径获取:`ClassLoader.getSystemClassLoader().loadClass("cn.javaguide.TargetObject");`

反射的一些基本操作

1. 创建一个我们要使用反射操作的类 `TargetObject`。

```java
package cn.javaguide;

public class TargetObject {
    private String value;

    public TargetObject() {
        value = "JavaGuide";
    }

    public void publicMethod(String s) {
        System.out.println("I love " + s);
    }

    private void privateMethod() {
        System.out.println("value is " + value);
    }
}
```

2. 使用反射操作这个类的方法以及参数

```java
package cn.javaguide;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InstantiationException, InvocationTargetException, NoSuchFieldException {
        /**
         * 获取 TargetObject 类的 Class 对象并且创建 TargetObject 类实例
         */
        Class<?> targetClass = Class.forName("cn.javaguide.TargetObject");
        TargetObject targetObject = (TargetObject) targetClass.newInstance();
        /**
         * 获取 TargetObject 类中定义的所有方法
         */
        Method[] methods = targetClass.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println(method.getName());
        }

        /**
         * 获取指定方法并调用
         */
        Method publicMethod = targetClass.getDeclaredMethod("publicMethod",
                String.class);

        publicMethod.invoke(targetObject, "JavaGuide");

        /**
         * 获取指定参数并对参数进行修改
         */
        Field field = targetClass.getDeclaredField("value");
        //为了对类中的参数进行修改我们取消安全检查
        field.setAccessible(true);
        field.set(targetObject, "JavaGuide");

        /**
         * 调用 private 方法
         */
        Method privateMethod = targetClass.getDeclaredMethod("privateMethod");
        //为了调用private方法我们取消安全检查
        privateMethod.setAccessible(true);
        privateMethod.invoke(targetObject);
    }
}
```

输出内容

```java
publicMethod
privateMethod
I love JavaGuide
value is JavaGuide
```

------

# JVM

------

## JVM基础

### 初识JVM

JVM 本质上是一个运行在计算机上的程序，他的职责是**运行Java字节码文件**。

![image-20240306164940469](Java.assets/image-20240306164940469.png)

JVM的功能

- **解释和运行**：对字节码文件中的指令， 实时的解释成机器码， 让计算机执行
- **内存管理**：自动为对象、方法等分配内存；自动的垃圾回收机制， 回收不再使用的对象
- **即时编译**（**Just-In-Time 简称JIT**）：对热点代码进行优化， 提升执行效率

常见的JVM

![image-20240306165144288](Java.assets/image-20240306165144288.png)

常见的JVM - HotSpot的发展历程

![image-20240306165218960](Java.assets/image-20240306165218960.png)

**JVM的组成**

![image-20240306165328030](Java.assets/image-20240306165328030.png)

### 字节码文件

**1、以正确的姿势打开文件**

字节码文件中保存了源代码编译之后的内容，以二进制的方式存储，无法直接用记事本打开阅读。 通过NotePad++使用十六进制插件查看class文件：

![image-20240306165451700](Java.assets/image-20240306165451700.png)

推荐使用 `jclasslib工具`查看字节码文件。

玩转字节码常用工具: 阿里arthas：[简介 | arthas (aliyun.com)](https://arthas.aliyun.com/doc/)

**2、字节码文件的组成**

![image-20240306165548662](Java.assets/image-20240306165548662.png)

------

### 类的生命周期

定义：类的生命周期描述了一个类加载、使用、卸载的整个过程

应用场景：

- 运行时常量池
- 多态的原理
- 类的加密和解密

类的生命周期全部过程

![image-20240306170002295](Java.assets/image-20240306170002295.png)

**1、加载阶段**

加载(Loading)阶段第一步是类加载器根据类的全限定名通过不同的渠道以二进制流的方式获取字节码信息。 程序员可以使用Java代码拓展的不同的渠道，类加载器在加载完类之后，Java虚拟机会将字节码中的信息保存到内存的方法区中。同时，Java虚拟机还会在堆中生成一份与方法区中数据类似的java.lang.Class对象。

**2、连接阶段**

连接（Linking）阶段的第一个环节是**验证**，验证的主要目的是检测Java字节码文件是否遵守了《Java虚拟机规 范》中的约束。**这个阶段一般不需要程序员参与**。

主要包含如下四部分，具体详见《Java虚拟机规范》： 

- 1、文件格式验证，比如文件是否以0xCAFEBABE开头，主次版本号是否满足当前Java虚拟机版本要求。
- 2、元信息验证，例如类必须有父类（super不能为空）
- 3、验证程序执行指令的语义，比如方法内的指令执行中跳转到不正确的位置。
- 4、符号引用验证，例如是否访问了其他类中private的方法等

第二个环节是准备。**准备阶段为静态变量（static）分配内存并设置初始值。**而每一种基本数据类型和引用数据类型都有其初始值。

![image-20240306170507239](Java.assets/image-20240306170507239.png)

final修饰的基本数据类型的静态变量，准备阶段直接会将代码中的值进行赋值。

第三个环节：**解析阶段**，主要是将常量池中的符号引用替换为直接引用。

- **符号引用就是在字节码文件中使用编号来访问常量池中的内容。**
- **直接引用不在使用编号，而是使用内存中地址进行访问具体的数据。**

**3、初始化阶段**

初始化阶段会执行**静态代码块中的代码**，**并为静态变量赋值**。执行流程与代码流程一致。

初始化阶段会执行**字节码文件中clinit部分的字节码指令**。

![image-20240306170807910](Java.assets/image-20240306170807910.png)

![image-20240306170820019](Java.assets/image-20240306170820019.png)

![image-20240306170843715](Java.assets/image-20240306170843715.png)

**以下几种方式会导致类的初始化**：添加`-XX:+TraceClassLoading` 参数可以打印出加载并初始化的类

- 1.访问一个类的静态变量或者静态方法，注意变量是final修饰的并且等号右边是常量不会触发初始化。 
- 2.调用Class.forName(String className)。 
- 3.new一个该类的对象时。 
- 4.执行Main方法的当前类。

**clinit指令在特定情况下不会出现**，比如：如下几种情况是不会进行初始化指令执行的。 

- 1.无静态代码块且无静态变量赋值语句。 
- 2.有静态变量的声明，但是没有赋值语句。 
- 3.静态变量的定义使用final关键字，这类变量会在准备阶段直接进行初始化。

> 注意：
>
> **直接访问父类的静态变量，不会触发子类的初始化。**
>
> **子类的初始化clinit调用之前，会先调用父类的clinit初始化方法。**
>
> **数组的创建不会导致数组中元素的类进行初始化。**
>
> **final修饰的变量如果赋值的内容需要执行指令才能得出结果，会执行clinit方法进行初始化**

> 解决大厂经典笔试题

<img src="Java.assets/image-20240306171103349.png" alt="image-20240306171103349" style="zoom: 67%;" />



2019执行流程

- 执行main方法先初始化Test1的初始化方法，输出结果DA
- 执行两次Test1的构造方法，输出结果DACBCB

### 总结

几个要点： 

- 1.静态变量的定义使用final关键字，这类变量会在准备阶段直接进行初始化 （除非要执行方法）。 
- 2.直接访问父类的静态变量，不会触发子类的初始化。子类的初始化cinit调用之前， 会先调用父类的cinit初始化方法。

![image-20240306171455358](Java.assets/image-20240306171455358.png)

## 类加载器

### 核心问题

> **类加载过程**
>
> 类加载过程有：加载、验证、准备、解析、初始化。
>
> 1、加载阶段
>
> - 1）通过一个类的全限定名来获取定义此类的二进制字节流。
> - 2）将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
> - 3）在内存中生成一个代表这个类的 `java.lang.Class` 对象，作为方法区这个类的各种数据的访问入口。
>
> 2、验证阶段：对二进制字节流进行校验，只有符合 JVM 字节码规范的才能被 JVM 正确执行。
>
> 3、准备阶段：对类变量（也称为静态变量，static 关键字修饰的变量）分配内存并初始化，初始化为数据类型的默认值，如 0、0L、null、false 等。
>
> 4、解析阶段：是虚拟机将常量池内的符号引用替换为直接引用的过程。解析动作主要针对类或接口、字段、类方法、接口方法、成员方法等。
>
> 5、初始化阶段：类加载过程的最后一步。在准备阶段，类变量已经被赋过默认初始值了，而在初始化阶段，类变量将被赋值为代码期望赋的值。

**1、类加载器的作用是什么？**

答：类加载器（ClassLoader）负责在类加载过程中的字节码获取并加载到内存这一部分。通过加载字节码数据放入内存转化成byte[]，接下来调用虚拟机底层方法将byte[]转化成方法区和堆区中的数据。

**2、有几种类加载器？**

1. 启动类加载器（`Bootstrap ClassLoader`）加载核心类 

2. 扩展类加载器（`Extension ClassLoader`）加载扩展类 

3. 应用程序类加载器（`Application ClassLoader`）加载应用`classpath`中的类 

4. 自定义类加载器，重写`findClass`方法。 

JDK9及之后扩展类加载器（`Extension ClassLoader`）变成了平台类加载器（`Platform  ClassLoader`）

![image-20240304195644715](Java.assets/image-20240304195644715.png)

> **3、类的双亲委派机制是什么？**
>
> 答：双亲委派模型（Parent Delegation Model）是 Java 类加载机制中的一个重要概念。这种模型指的是一个类加载器在尝试加载某个类时，首先会将加载任务委托给其父类加载器去完成。只有当父类加载器无法完成这个加载请求（即它找不到指定的类）时，子类加载器才会尝试自己去加载这个类。
>
> - 每个Java实现的类加载器中保存了一个成员变量叫**“父”（Parent）类加载器**。 **自底向上查找是否加载过，再由顶向下进行加载。避免了核心类被应用程序重写并覆盖的问题，提升了安全性**。
>
> - 当一个类加载器去加载某个类的时候，会自底向上查找是否加载过，如果加载过就直接返回，如果一直到最顶层的类加载器都没有加载，再自顶向下进行加载
>
> - 应用程序类加载器的父类是扩展类加载器，扩展类加载器的父类是启动类加载器；
>
> **双亲委派机制的好处有两点：**
>
> - **避免恶意代码替换`JDK`中的核心类库**，比如`java.lang.String`，确保核心类库的安全性和完整性；
> - **避免一个类重复地被加载**；

**4、这怎么打破类的双亲委派机制？**

1. 重写loadClass方法，不再实现双亲委派机制。 
2. JNDI、JDBC、JCE、JAXB和JBI等框架使用了SPI机制+线程上下文类加载器。 
3. OSGi实现了一整套类加载机制，允许同级类加载器之间互相调用

### 类加载器

**类加载器的作用是什么？**

答：类加载器（ClassLoader）负责在类加载过程中的字节码获取并加载到内存这一部分。通过加载字节码数据放入内存转化成byte[]，接下来调用虚拟机底层方法将byte[]转化成方法区和堆区中的数据。

1、虚拟机底层代码实现的类加载器

- 这里的虚拟机一般指`Hotspot`虚拟机
- **启动类加载器（Bootstrap）**：加载程序运行时的基础类，比如`java.lang.String`

2、Java中提供的默认或者自定义的类加载器

- 继承自抽象类`ClassLoader`
- **扩展类加载器(Extension)**：允许扩展Java中比较常用的类
- **应用程序加载器Application**：加载应用使用的类

![image-20240304161412305](Java.assets/image-20240304161412305.png)

### 类的双亲委派机制

由于Java虚拟机中有多个类加载器，双亲委派机制的核心是**解决一个类到底由谁加载的问题**

作用：

- 保证类加载的安全性，避免恶意代码替换JDK中的核心类库，确保核心类库的完整性和安全性
- 避免重复加载

到底类的双亲委派机制是什么？

双亲委派机制指的是：当一个类加载器接收到加载类的任务时，会**自底向上查找是否加载过**该类，**再由顶向下进行加载**

![image-20240304161942592](Java.assets/image-20240304161942592.png)

问题：

> 1、重复的类：如果一个类重复出现在三个类加载器的加载位置，应该由谁来加载？

- 启动类加载器加载，根据双亲委派机制，它的优先级是最高的

> 2、String类能覆盖吗？在自己的项目中去创建一个`java.lang.String`类，会被加载吗？

- 不能，会返回启动类加载器加载在`rt.jar`包中的String类

> 3、类加载器的关系？

- 应用程序类加载器的parent父类加载器是扩展类加载器，而扩展类加载器的parent是null，但是在代码逻辑上，扩展类加载器依然会把启动类加载器当成父类加载器处理；

- 启动类加载器是使用C++编写，没有父类加载器

- 可以使用`arthas`来查看类加载器的父子关系

![image-20240304164015113](Java.assets/image-20240304164015113.png)

在Java中如何使用代码的方式去主动加载一个类？

1. 使用`Class.forName`方法，使用当前类的类加载器去加载指定的类
2. 获取到类加载器，通过类加载器的`loadClass`方法指定某个类加载器加载

```java
public class Main {
    public static void main(String[] args) throws ClassNotFoundException {
        ClassLoader classLoader = Main.class.getClassLoader();
        System.out.println(classLoader);

        Class<?> stringClass = classLoader.loadClass("java.lang.String");
        System.out.println(stringClass.getClassLoader());
    }
}

// result: 
// sun.misc.Launcher$AppClassLoader@18b4aac2
// null
```

------

### 打破双亲委派机制

三种方式

- 自定义类加载器
  - 自定义类加载器并且重写 `loadClass`方法，就可以将双亲委派机制的代码去除
  - Tomcat通过这种方式实现应用之间类隔离
- 线程上下文类加载器
  - 利用上下文类加载器加载类，比如 `JDBC和JNDI`等
- `Osgi`框架的类加载器
  - 历史上`Osgi`框架实现了一套新的类加载器机制，允许同级之间委托进行类的加载

#### 自定义类加载器

`ClassLoader`的原理，包含了四个核心方法：

![image-20240304165942277](Java.assets/image-20240304165942277.png)

```java
// parent等于null说明父类加载器是启动类加载器，直接调用findBootstrapClassOrNull
// 否则调用父类加载器的加载方法
if (parent != null) {
    c = parent.loadClass(name, false);
} else {
    c = findBootstrapClassOrNull(name);
}
// 父类加载器爱莫能助，我来加载！
if (c == null) {
    c = findClass(name);
}
```

问题：

> 1、自定义类加载器父类怎么是`AppClassLoader`呢？

![image-20240304171316001](Java.assets/image-20240304171316001.png)

以JDK8为例，ClassLoader类中提供构造方法设置parent的内容:

这个构造方法由另外一个构造方法调用，其中父类加载器由`getSystemClassLoader`方法设置，该方法返回的是`AppClassLoader`

```java
private ClassLoader(Void unused, ClassLoader parent) {
    this.parent = parent;
    if (ParallelLoaders.isRegistered(this.getClass())) {
        parallelLockMap = new ConcurrentHashMap<>();
        package2certs = new ConcurrentHashMap<>();
        assertionLock = new Object();
    } else {
        // no finer-grained lock; lock on the classloader instance
        parallelLockMap = null;
        package2certs = new Hashtable<>();
        assertionLock = this;
    }
}

protected ClassLoader() {
    this(checkCreateClassLoader(), getSystemClassLoader());
}
```

> 2、两个自定义类加载器加载相同限定名的类，不会冲突吗？

- **不会冲突**，在同一个Java虚拟机中，只有**相同类加载器+相同的类限定名**才会被认为是同一个类。

- 在`Arthas`中使用`sc –d 类名`的方式查看具体的情况

------

#### 线程上下文类加载器

`JDBC`案例：

- `JDBC`中使用了`DriverManager`来管理项目中引入的不同数据库的驱动，比如mysql驱动、oracle驱动。
- `DriverManager`类位于`rt.jar`包中，由启动类加载器加载。
- 依赖中的`mysql`驱动对应的类，由应用程序类加载器来加载。
- `DriverManager`属于`rt.jar`是启动类加载器加载的。而用户jar包中的驱动需要由应用类加载器加载，这就违反了双亲委派机制。

解答：

- 启动类加载器加载`DriverManager`。
- 在初始化`DriverManager`时，通过`SPI`机制加载jar包中的`myql`驱动。
- `SPI`中利用了线程上下文类加载器（应用程序类加载器）去加载类并创建对象。
- **这种由启动类加载器加载的类，委派应用程序类加载器去加载类的方式，打破了双亲委派机制。**

![image-20240304174338133](Java.assets/image-20240304174338133.png)

> 1、`DriverManager`怎么知道jar包中要加载的驱动在哪儿？

用到了SPI机制：

SPI全称为（Service Provider Interface）,是JDK内置的一种服务提供发现机制

SPI的工作原理

1、在ClassPath路径下META-INF/services文件夹中，以接口的全限定名来命名文件名，对应的文件里面写该接口的实现

2、使用ServiceLoader加载实现类

![image-20240304172931713](Java.assets/image-20240304172931713.png)

> 2、SPI中是如何获取应用类加载器的

SPI中使用了线程上下文中保存的类加载器进行类的加载，这个类加载器一般是应用程序类加载器。

![image-20240304174253199](Java.assets/image-20240304174253199.png)

> 2、`JDBC`案例真的打破了双亲委派机制吗

`DriverManager`：JDK8中位于rt.jar，由启动类加载器加载

Jar包中的`mysql`驱动： 位于`classpath`，由应用程序类加载器加载

因此**没有打破双亲委派机制**：`JDBC`只是在`DriverManager`加载完之后，通过初始化阶段触发了驱动类的加载，类的加载依然遵循双亲委派机制

**打破双亲委派机制的唯一方法是重写loadClass方法**

#### `OSGi`模块化

历史上，`OSGi`模块化框架。它存在同级之间的类加载器的委托加载。`OSGi`还使用类加载器实现了热部署的 功能。 

**热部署指的是在服务不停止的情况下，动态地更新字节码文件到内存中。**

------

### JDK9之后的类加载器

JDK8及之前的版本中，扩展类加载器和应用程序类加载器的源码位于`rt.jar`包中的`sun.misc.Launcher.java`。

![image-20240304194834852](Java.assets/image-20240304194834852.png)

由于`JDK9`引入了module的概念，类加载器在设计上发生了很多变化。

1、**启动类加载器使用Java编写，位于`jdk.internal.loader.ClassLoaders`类中**。

-  Java中的`BootClassLoader`继承自`BuiltinClassLoader`实现从模块中找到要加载的字节码资源文件。 

- 启动类加载器依然无法通过java代码获取到，返回的仍然是null，保持了统一。

2、**扩展类加载器被替换成了平台类加载器（Platform Class Loader）**。

- 平台类加载器遵循模块化方式加载字节码文件，所以继承关系从`URLClassLoader`变成了 `BuiltinClassLoader`，`BuiltinClassLoader`实现了从模块中加载字节码文件。平台类加载器的存在更多的是 为了与老版本的设计方案兼容，自身没有特殊的逻辑。

------

## 运行时数据区

 Java虚拟机在运行Java程序过程中管理的内存区域，称之为**运行时内存区**，其中：

- **线程共享的区域：方法区、堆；**
- **线程不共享的区域：程序计数器，Java虚拟机栈，本地方法栈；**

------

### 核心问题

> 1、Java的内存分成哪几个部分，详细介绍一下

- **程序计数器**：每个线程会通过程序计数器记录当前要执行的字节码指令的地址，程序计数器可以控制程序指令的进行，实现分支、跳转、异常等逻辑；
- **Java虚拟机栈和本地方法栈**：采用栈的数据结构来管理方法调用中的基本数据（局部变量、操作数等），每个方法的调用使用一个栈帧来保存。
- **方法区**：方法区中主要存放的是类的元信息，同时还保存了常量池
- **堆**：堆中存放的是创建出来的对象， 这也是最容易产生内存溢出的位置

> 2、Java内存中哪些部分会内存溢出（OOM，out of Memory）？

程序计数器是唯一不会出现OOM的。Java虚拟机栈和本地方法栈、堆和方法区都会出现OOM

> 3、JDK7和8在内存结构上的区别是什么？

方法区：

- **JDK7及之前的版本**将方法区存放在**堆区域中的永久代空间**

- **JDK8及之后的版本**将方法区存放在**元空间**中，**元空间位于操作系统维护的直接内存中**

字符串常量池

- JDK6及之前的版本将字符串常量池存放在方法区（永久代）中
- JDK7及之后的版本将字符串常量池存放在堆中

------

### 程序计数器

程序计数器（Program Counter Register）也叫PC寄存器，每个线程会通过程序计数器记录当前要执行的的字节码指令的地址。

作用

- 程序计数器可以**控制程序指令的进行**，实现分支、跳转、异常等逻辑。

- 在多线程执行情况下，Java虚拟机需要通过程序计数器记录CPU切换前解释执行到那一句指令并继续解释运行。

> 程序计数器在运行中会出现内存溢出吗？

**内存溢出**：指的是程序在使用某一块内存区域时，存放的数据需要占用的内存大小超过了虚拟机能提供的内存上限。

答，不会，因为每个线程**只存储一个固定长度的内存地址**，**程序计数器是不会发生内存溢出的。程序员无需对程序计数器做任何处理**。

------

### Java虚拟机栈

Java虚拟机栈（Java Virtual Machine Stack）采用栈的数据结构来管理方法调用中的基本数据，先进后出（First In Last Out），每一个方法的调用使用**一个栈帧（Stack Frame）**来保存。

Java虚拟机栈随着线程的创建而创建，而回收则会在线程的销毁时进行。由于方法可能会在不同线程中执行，**每个线程都会包含一个自己的虚拟机栈**。

栈帧的组成：

- 局部变量表：在运行过程 中存放所有的局部变量
- 操作数栈：操作数栈是栈帧中虚拟机在执行指令过程中用来存放中间数据的一块区域。
- 帧数据：主要包含动态链接、方法出口、异常表的引用

> 1、局部变量表

- **局部变量表**：在运行过程中存放所有的局部变量；

  - 编译成字节码文件时就可以确定局部变量表的内容。

  - 栈帧中的局部变量表是一个**数组**，数组中每一个位置称之为槽(slot) ，**long和double类型占用两个槽，其他类型占用一个槽**。
  - 局部变量表保存的内容有：**实例方法的this对象，方法的参数，方法体中声明的局部变量**。
    -  **实例方法中的序号为0的位置存放的是this**，指的是当前调用方法的对象，运行时会在内存中存放实例对象的地址。
    - 方法参数也会保存在局部变量表中，其顺序与方法中参数定义的顺序一致。
    - 为了节省空间，**局部变量表中的槽是可以复用的**，一旦某个局部变量不再生效，当前槽就可以再次被使用。

- 操作数栈是栈帧中虚拟机在执行指令过程中用来存放临时数据的一块区域；

- 帧数据主要包含动态链接、方法出口、异常表的引用；

> 2、操作数栈

操作数栈是栈帧中虚拟机在执行指令过程中用来存放中间数据的一块区域。它是一种栈式的数据结构，如果一条指令将一个值压入操作数栈，则后面的指令可以弹出并使用该值。在编译期就可以确定操作数栈的最大深度，从而在执行时正确的分配内存大小。

> 3、**帧数据**

- **动态链接**

  - 当前类的字节码指令引用了其他类的属性或者方法时，需要将符号引用（编号）转换成对应的运行时常量池中的内存地址。

  - 动态链接就保存了编号到运行时常量池的内存地址的映射关系。

- **方法出口**
  - 方法出口指的是方法在正确或者异常结束时，当前栈帧会被弹出，同时程序计数器应该指向上一个栈帧中的下一条指令的地址。所以在当前栈帧中，需要存储此方法出口的地址。

- **异常表**
  - 异常表存放的是代码中异常的处理信息，包含了异常捕获的生效范围以及异常发生后跳转到的字节码指令位置。

![image-20240304210245572](Java.assets/image-20240304210245572.png)

> Java虚拟机栈会出现内存溢出吗？

Java虚拟机栈如果栈帧过多，占用内存超过栈内存可以分配的最大大小就会出现内存溢出。 

Java虚拟机栈内存溢出时会出现`StackOverflowError`的错误

如果我们不指定栈的大小，JVM 将创建一个 具有默认大小的栈。大小取决于操作系统和 计算机的体系结构。

要修改Java虚拟机栈的大小，可以使用虚拟机参数 -Xss 。

- 语法：-Xss栈大小

- 单位：字节（默认，必须是 1024 的倍数）、k或者K(KB)、m或者M(MB)、g或者G(GB)，比如-Xss1024K 

注意事项：

- 与-Xss类似，也可以使用 -XX:ThreadStackSize 调整标志来配置堆栈大小。 格式为： -XX:ThreadStackSize=1024

- HotSpot JVM对栈大小的最大值和最小值有要求：Windows（64位）下的JDK8测试最小值为180k，最大值为1024m。
- 局部变量过多、操作数栈深度过大也会影响栈内存的大小。

一般情况下，工作中即便使用了递归进行操作，栈的深度最多也只能到几百,不会出现栈的溢出。所以此参数 可以手动指定为-Xss256k节省内存。

------

### 本地方法栈

Java虚拟机栈存储了Java方法调用时的栈帧，而本地方法栈存储的是native本地方法的栈帧。

 在`Hotspot`虚拟机中，**Java虚拟机栈和本地方法栈实现上使用了同一个栈空间**。本地方法栈会在栈内存上生成一个栈帧，临时保存方法的参数同时方便出现异常时也把本地方法的栈信息打印出来。

------

### 堆

一般Java程序中堆内存是空间最大的一块内存区域。创建出来的对象都存在于堆上。

栈上的局部变量表中，可以存放堆上对象的引用。静态变量也可以存放堆对象的引用，通过静态变量就可以实现对象在线程之间共享。

> 堆内存会溢出吗？
>
> 堆内存大小是有上限的，当对象一直向堆中放入对象达到上限之后，就会抛出`OutOfMemory` 错误

堆空间有三个需要关注的值，used total max

- used指的是当前已使用的堆内存
- total是java虚拟机已经分配的可用堆内存
- max是java虚拟机可以分配的最大堆内存。

堆内存used total max三个值可以通过dashboard命令看到；

手动指定刷新频率（不指定默认5秒一次）：dashboard –i 刷新频率(毫秒)

如果不设置任何的虚拟机参数，max默认是系统内存的1/4，total默认是系统内存的1/64。在实际应用中一般都需要设置 total和max的值。

> **堆 – 设置大小**

- 要修改堆的大小，可以使用虚拟机参数 –Xmx（max最大值）和-Xms (初始的total)。
-  语法：**-Xmx值 -Xms值**
- 单位：字节（默认，必须是 1024 的倍数）、k或者K(KB)、m或者M(MB)、g或者G(GB) 
- 限制：**Xmx必须大于 2 MB，Xms必须大于1MB**

> 问题：为什么arthas中显示的heap堆大小与设置的值不一样呢？

arthas中的heap堆内存使用了JMX技术中内存获取方式，这种方式与垃圾回收器有关，计算的是**可以分配对象的内存**，而不是整个内存

<img src="Java.assets/image-20240304214906703.png" alt="image-20240304214906703" style="zoom:80%;" />

Java服务端程序开发时，**建议将-Xmx和-Xms设置为相同的值**，这样在程序启动之后可使用的总内存就是最大内存，而无 需向java虚拟机再次申请，减少了申请并分配内存时间上的开销，同时也不会出现内存过剩之后堆收缩的情况。

------

### 方法区

方法区是存放基础信息的位置，**线程共享**，主要包含三部分内容：

- **类的元信息**：保存了所有类的基本信息
  - 一般称之为InstanceKlass对象，包含类的基本信息，常量池、字段、方法（存放引用）和虚方法表（实现多态的基础）。
  - 在类的加载阶段完成。
- **运行时常量池**：保存了字节码文件中的常量池内容
  - 字节码文件中通过编号查表的方式找到常量，这种常量池称为**静态常量池**。
  - 当常量池加载到内存中之后，可以通过内存地址快速的定位到常量池中的内容，这种常量池称为**运行时常量池**。
- **字符串常量池**：保存了字符串常量 

1、Hotspot方法区设计：

- **JDK7及之前的版本**将方法区存放在**堆区域中的永久代空间**，堆的大小由虚拟机参数`-XX:MaxPermSize=值`来控制。
- **JDK8及之后的版本**将方法区存放在**元空间**中，**元空间位于操作系统维护的直接内存中**，默认情况下只要不超过操作系统承受的上限，可以一直分配。可以使用`-XX:MaxMetaspaceSize=值`将元空间最大大小进行限制。

2、arthas中查看方法区

- 使用`memory`打印出内存情况，JDK7及之前的版本查看`ps_perm_gen`属性
- JDK8及之后的版本查看`metaspace`属性。

![image-20240305103716635](Java.assets/image-20240305103716635.png)

3、字符串常量池的变更

![image-20240305105347671](Java.assets/image-20240305105347671.png)

> 练习1：

通过字节码指令分析如下代码的运行结果？

```java
public static void main(String[] args) {
    String a = "1";
    String b = "2";
    String c = "12";
    String d = a + b;
    System.out.println(c == d);   // false
}
```

其字节码指令：

- 从常量池中获取字符串1的地址放入操作数栈，然后将操作数栈中的值放入局部变量表，重复三次，局部变量表中分别存储“1”，“2”，“12”
- 两个变量的相加调用了StringBuilder对象，在堆中生成了一个新的值，使用append方法实现字符串相加，并将其放入局部变量表中
- 因为c和d所指向的内容不一样，故输出为false;

```java
 0 ldc #2 <1>
 2 astore_1
 3 ldc #3 <2>
 5 astore_2
 6 ldc #4 <12>
 8 astore_3
 9 new #5 <java/lang/StringBuilder>
12 dup
13 invokespecial #6 <java/lang/StringBuilder.<init> : ()V>
16 aload_1
17 invokevirtual #7 <java/lang/StringBuilder.append : (Ljava/lang/String;)Ljava/lang/StringBuilder;>
20 aload_2
21 invokevirtual #7 <java/lang/StringBuilder.append : (Ljava/lang/String;)Ljava/lang/StringBuilder;>
24 invokevirtual #8 <java/lang/StringBuilder.toString : ()Ljava/lang/String;>
27 astore 4
29 getstatic #9 <java/lang/System.out : Ljava/io/PrintStream;>
32 aload_3
33 aload 4
35 if_acmpne 42 (+7)
38 iconst_1
39 goto 43 (+4)
42 iconst_0
43 invokevirtual #10 <java/io/PrintStream.println : (Z)V>
46 return
```

> 练习2：

通过字节码指令分析如下代码的运行结果？

```java
public static void main(String[] args) {
    String a = "1";
    String b = "2";
    String c = "12";
    String d = "1" + "2";
    System.out.println(c == d);  // true
}
```

其字节码指令：

- 从常量池中获取字符串1的地址放入操作数栈，然后将操作数栈中的值放入局部变量表，重复三次，局部变量表中分别存储“1”，“2”，“12”
- 两个常量相加，编译阶段直接连接；

```java
 0 ldc #2 <1>
 2 astore_1
 3 ldc #3 <2>
 5 astore_2
 6 ldc #4 <12>
 8 astore_3
 9 ldc #4 <12>
11 astore 4
13 getstatic #5 <java/lang/System.out : Ljava/io/PrintStream;>
16 aload_3
17 aload 4
19 if_acmpne 26 (+7)
22 iconst_1
23 goto 27 (+4)
26 iconst_0
27 invokevirtual #6 <java/io/PrintStream.println : (Z)V>
30 return
```

> 神奇的intern

- JDK6版本中intern () 方法会把第一次遇到的字符串实例复制到永久代的字符串常量池中，返回 的也是永久代里面这个字符串实例的引用。JVM启动时就会把java加入到常量池中。
- JDK7及之后版本中**由于字符串常量池在堆上**，所以intern () 方法会把第一次遇到的字符串的**引用**放入字符串常量池。

>  静态变量的存储

- JDK6及之前的版本中，静态变量是存放在方法区中的，也就是永久代。
- JDK7及之后的版本中，静态变量是存放在**堆中的Class对象**中，脱离了永久代。

------

### 直接内存

直接内存（Direct Memory）并不在《Java虚拟机规范》中存在，所以并不属于Java运行时的内存区域。

在 JDK 1.4 中引入了 NIO 机制，使用了直接内存，主要为了解决以下两个问题:

1、Java堆中的对象如果不再使用要回收，回收时会影响对象的创建和使用。 

2、IO操作比如读文件，需要先把文件读入直接内存（缓冲区）再把数据复制到Java堆中。 现在直接放入直接内存即可，同时Java堆上维护直接内存的引用，减少了数据复制的开销。写文件也是类似的思路。

直接内存

- 要创建直接内存上的数据，可以使用`ByteBuffer`。

- 语法： `ByteBuffer directBuffer = ByteBuffer.allocateDirect(size);`

- 注意事项： arthas的memory命令可以查看直接内存大小，属性名direct。

![image-20240305145054247](Java.assets/image-20240305145054247.png)

- 如果需要手动调整直接内存的大小，可以使用`-XX:MaxDirectMemorySize=大小` 

  - 单位k或K表示千字节，m或M表示兆字节，g或G表示千兆字节。默认不设置该参数情况下，JVM 自动选择最大分配的大小。

  - 以下示例以不同的单位说明如何将直接内存大小设置为 `1024 KB`：
    - `-XX:MaxDirectMemorySize=1m`
    - `-XX:MaxDirectMemorySize=1024k`
    - `-XX:MaxDirectMemorySize=1048576`

------

## 自动垃圾回收

**C/C++的内存管理**

- 在C/C++这类没有自动垃圾回收机制的语言中，一个对象如果不再使用，需要手动释放，否则就会出现**内存泄漏**。我们称这种释放对象的过程为垃圾回收，而需要程序员编写代码进行回收的方式为**手动回收**。
- **内存泄漏**指的是不再使用的对象在系统中未被回收，内存泄漏的积累可能会导致**内存溢出**。

**Java的内存管理**

- 执行引擎的一部分

- Java中为了简化对象的释放，引入了**自动的垃圾回收（Garbage Collection简称GC）机制**。通过垃圾回收器来对不再使用的对象完成自动的回收，垃圾回收器主要负责对**堆**上的内存进行回收。其他很多现代语言比如C#、Python、Go都拥有自己的垃圾回收器。

**垃圾回收的对比**

- 自动垃圾回收：自动根据对象是否使用由虚拟机来回收对象
  - 优点：降低程序员实现难度、降低对象回收bug的可能性
  - 缺点：程序员无法控制内存回收的及时性

- 手动垃圾回收：由程序员编程实现对象的删除
  - 优点：回收及时性高，由程序员把控回收的时机
  - 缺点：编写不当容易出现悬空指针、重复释放、内存泄漏等问题

**应用场景**

- 解决系统僵死的问题：大厂的系统出现的许多系统僵死问题，都与频繁的垃圾回收有关

- 性能优化：对垃圾回收器进行合理的设置可以有效地提升程序的执行性能

- 高频面试题：常见的垃圾回收器、常见的垃圾回收算法、四种引用、项目中用了哪一种垃圾回收器

> 线程不共享的部分，都是**伴随着线程的创建而创建，线程的销毁而销毁**。而方法的栈帧在执行完方法之后就会自动弹出栈并释放掉对应的内存。

------

### 方法区的回收

方法区中能回收的内容主要就是不再使用的类。 

判定一个类可以被卸载。需要同时满足下面三个条件： 

1、**此类所有实例对象都已经被回收，在堆中不存在任何该类的实例对象以及子类对象。** 

2、**加载该类的类加载器已经被回收**。 

3、**该类对应的 `java.lang.Class` 对象没有在任何地方被引用**

方法区的回收 – 手动触发回收

如果需要手动触发垃圾回收，可以调用`System.gc()`方法。

- 语法： `System.gc()` 
- 注意事项： 调用`System.gc()`方法并不一定会立即回收垃圾，仅仅是向Java虚拟机发送一个垃圾回收的请求，具体是否需要执行垃圾回收Java虚拟机会自行判断。

------

### 堆的回收

> 如何判断堆上的对象可以回收？

Java中的对象是否能被回收，是根据对象是否被引用来决定的。如果对象被引用了，说明该对象还在使用，不允许被回收。只有无法通过引用获取到对象时，该对象才能被回收。

------

#### 引用计数法和可达性分析法

**引用计数法**

- 引用计数法会为每个对象维护一个引用计数器，当对象被引用时加1，取消引用时减1。
- 优点是实现简单，C++中的智能指针就采用了引用计数法
- 缺点主要有两点：
  - 1.每次引用和取消引用都需要维护计数器，对系统性能会有一定的影响
  - 2.存在循环引用问题，所谓循环引用就是当A引用B，B同时引用A时会出现对象无法回收的问题。
- 如果想要查看垃圾回收的信息，可以使用`-verbose:gc`参数。
  - 语法： `-verbose:gc`

**可达性分析算法**

Java使用的是**可达性分析算法**来判断对象是否可以被回收。

可达性分析将对象分为两类：**垃圾回收的根对象（`GC  Root`）和普通对象**，对象与对象之间存在引用关系。

可达性分析算法指的是如果从某个对象到`GC Root`对象是可达的，对象就不可被回收。

- `GC Root`对象常规情况下是不会被回收的

> 哪些对象被称之为`GC Root`对象呢？ 

- **线程Thread对象**，引用线程栈帧中的方法参数、局部变量等。

- **系统类加载器加载的`java.lang.Class`对象**，引用类中的静态变量。

- **监视器对象**，用来保存同步锁synchronized关键字持有的对象。 

- **本地方法调用时使用的全局对象**

------

#### 五种对象引用

可达性算法中描述的对象引用，一般指的是强引用，即是GCRoot对象对普通对象有引用关系，只要这层关系存在， 普通对象就不会被回收。除了**强引用**之外，Java中还设计了几种其他引用方式：

- **软引用**
- **弱引用**
- **虚引用**
- **终结器引用**

**1、软引用**

- 软引用相对于强引用是一种比较弱的引用关系，如果一个对象只有软引用关联到它，当程序内存不足时，就会将软引用中的数据进行回收。
- 软引用的执行过程如下： 
  - 1.将对象使用软引用包装起来，`new SoftReference<对象类型>(对象)`。 
  - 2.内存不足时，虚拟机尝试进行垃圾回收。 
  - 3.如果垃圾回收仍不能解决内存不足的问题，回收软引用中的对象。 
  - 4.如果依然内存不足，抛出OutOfMemory异常。

![image-20240305171450173](Java.assets/image-20240305171450173.png)

2、**弱引用**

弱引用的整体机制和软引用基本一致，区别在于弱引用包含的对象在垃圾回收时，**不管内存够不够都会直接被回收**。

在`JDK 1.2`版之后提供了`WeakReference`类来实现弱引用，弱引用主要在`ThreadLocal`中使用。

弱引用对象本身也可以使用引用队列进行回收。

```java
byte[] bytes = new byte[1024 * 1024 * 100];
WeakReference<byte[]> weakReference = new WeakReference<byte[]>(bytes);
bytes = null;
System.out.println(weakReference.get());

System.gc();

System.out.println(weakReference.get());

// 输出
// [B@5cad8086
// null3、虚引用
```

**3、虚引用和终结器引用**

- **这两种引用在常规开发中是不会使用的**。

- 虚引用也叫幽灵引用/幻影引用，不能通过虚引用对象获取到包含的对象。虚引用唯一的用途是当对象被垃圾回收器回收时可以接收到对应的通知。Java中使用`PhantomReference`实现了虚引用，直接内存中为了及时知道直接内存对象不再使用，从而回收内存，使用了虚引用来实现。
- 终结器引用指的是在对象需要被回收时，终结器引用会关联对象并放置在`Finalizer`类中的引用队列中，在稍后由一条由`FinalizerThread`线程从队列中获取对象，然后执行对象的finalize方法，在对象第二次被回收时，该对象才真正的被回收。在这个过程中可以在finalize方法中再将自身对象使用强引用关联上，但是不建议这样做

------

#### 垃圾回收算法

核心思想：Java是如何实现垃圾回收的呢？简单来说，垃圾回收要做的有两件事： 

1. 找到内存中存活的对象

2. 释放不再存活对象的内存，使得程序能再次利用这部分空间

四种垃圾回收算法

- **标记-清除算法**
- **复制算法**

- **标记-整理算法**

- **分代GC**

> 垃圾回收算法的评价标准

Java垃圾回收过程会通过单独的`GC线程`来完成，但是不管使用哪一种`GC算法`，都会有部分阶段需要停止所有的用户线程。这个过程被称之为`Stop The World`简称`STW`，如果`STW`时间过长则会影响用户的使用。所以判断`GC`算法是否优秀，可以从三个方面来考虑：

**1、吞吐量**

- 吞吐量指的是 CPU 用于执行用户代码的时间与 CPU 总执行时间的比值，即**`吞吐量 = 执行用户代码时间 /（执行用户代码时间 + GC时间）`**。吞吐量数值越高，垃圾回收的效率就越高。

**2、最大暂停时间**

- 最大暂停时间指的是所有在垃圾回收过程中的`STW`时间最大值。
- 比如如下的图中，黄色部分的STW就是最大暂停时间，显而易见上面的图比下面的图拥有更少的最大暂停时间。最大暂停时间越短，用户使用系统时受到的影响就越短。

<img src="Java.assets/image-20240306112551599.png" alt="image-20240306112551599" style="zoom:50%;" />

**3、堆使用效率**

- 不同垃圾回收算法，对堆内存的使用方式是不同的。比如标记清除算法，可以使用完整的堆内存。而复制算法会将堆内存一分为二，每次只能使用一半内存。从堆使用效率上来说，标记清除算法要优于复制算法。

上述三种评价标准：**堆使用效率、吞吐量，以及最大暂停时间不可兼得**。 一般来说，堆内存越大，最大暂停时间就越长。想要减少最大暂停时间，就会降低吞吐量。不同的垃圾回收算法，适用于不同的场景。

> **标记-清除算法**

标记清除算法的核心思想分为两个阶段： 

- 1、标记阶段，将所有存活的对象进行标记。Java中使用**可达性分析算法**，从`GC Root`开始通过引用链遍历出所有存活对象。

- 2、清除阶段，从内存中删除没有被标记也就是非存活对象。

优点：实现简单，只需要在第一阶段给每个对象维护标志位，第二阶段删除对象即可。

缺点：

- **碎片化问题**：由于内存是连续的，所以在对象被删除之后，内存中会出现很多细小的可用内存单元。如果我们需要的是一个比较大的空间，很有可能这些内存单元的大小过小无法进行分配。
- **分配速度慢**。由于内存碎片的存在，需要维护一个空闲链表，极有可能发生每次需要遍历到链表的最后才能获得合适的内存空间。

> **复制算法**

核心思想：

- 1、准备两块空间From空间和To空间，每次在对象分配阶段，只能使用其中一块空间（From空间）

- 2、在垃圾回收`GC`阶段，将From中存活对象复制到To空间。

- 3、将两块空间的From和To名字互换。

优点：

- 吞吐量高：复制算法只需要遍历一次存活对象复制到To空间即可，比标记-整理算法少了一次遍历的过程，因而性能较好，但是不如标记-清除算法， 因为标记清除算法不需要进行对象的移动

- **不会发生碎片化**：复制算法在复制之后就会将对象按顺序放入To空间中，所以对象以外的区域都是可用空间，不存在碎片化内存空间。

缺点：内存使用效率低，每次只能让一半的内存空间来为创建对象使用

> **标记-整理算法**

标记整理算法也叫标记压缩算法，是对标记清理算法中容易产生内存碎片问题的一种解决方案。 

核心思想分为两个阶段：

1. 标记阶段，将所有存活的对象进行标记。Java中使用可达性分析算法，从`GC Root`开始通过引用链遍历出所有存活对象。
2. 整理阶段，将存活对象移动到堆的一端。清理掉存活对象的内存空间。

优点

- 内存使用效率高：整个堆内存都可以使用，不会像复制算法只能使用半个堆内存
- **不会发生碎片化**：在整理阶段可以将对象往内存的一侧进行移动，剩下的空间都是可以分配对象的有效空间

缺点

- 整理阶段的效率不高：整理算法有很多种，比如`Lisp2`整理算法需要对整个堆中的对象搜索3次，整体性能不佳。可以通过`TwoFinger`、表格算法、`ImmixGC`等高效的整理算法优化此阶段的性能

> **分代GC**

现代优秀的垃圾回收算法，会将上述描述的垃圾回收算法组合进行使用，其中应用最广的就是**分代垃圾回收算法(Generational GC)**。
分代垃圾回收将整个内存区域划分为年轻代和老年代：

- 年轻代：用于存放存活时间比较短的对象
- 老年代：用于存放存活时间比较长的对象

<img src="Java.assets/image-20240306151646207.png" alt="image-20240306151646207" style="zoom:50%;" />

**arthas查看分代之后的内存情况** 

- 在`JDK8`中，添加`-XX:+UseSerialGC`参数使用分代回收的垃圾回收器，运行程序。

- 在arthas中使用`memory`命令查看内存，显示出三个区域的内存情况。

![image-20240306150156291](Java.assets/image-20240306150156291.png)

**调整内存区域的大小**

<img src="Java.assets/image-20240306150219531.png" alt="image-20240306150219531" style="zoom: 80%;" />

分代垃圾回收算法执行流程

- 1、分代回收时，创建出来的对象，首先会被放入`Eden`伊甸园区。 
- 2、随着对象在Eden区越来越多，如果Eden区满，新创建的对象已经无法放入，就会触发年轻代的`GC`，称为 `Minor GC或者Young GC`。`Minor GC`会把需要eden中和From需要回收的对象回收，把没有回收的对象放入To区。 

- 3、接下来，`S0`会变成To区，`S1`变成From区。当eden区满时再往里放入对象，依然会发生`Minor GC`。 此时会回收eden区和`S1(from)`中的对象，并把eden和from区中剩余的对象放入`S0`。注意：每次`Minor GC`中都会为对象记录他的年龄，初始值为0，每次GC完加1。
- 4、如果`Minor GC`后对象的年龄达到阈值（最大15，默认值和垃圾回收器有关），对象就会被晋升至老年代。
- 5、当老年代中空间不足，无法放入新的对象时，先尝试`minor gc`如果还是不足，就会触发`Full GC`，`Full GC`会对整个堆进行垃圾回收。 如果`Full GC`依然无法回收掉老年代的对象，那么当对象继续放入老年代时，就会抛出`Out Of Memory`异常。

#### 垃圾回收器

> 为什么分代`GC`算法要把堆分成年轻代和老年代？

系统中的大部分对象，都是创建出来之后很快就不再使用可以被回收，比如用户获取订单数据，订单数据返回给用户之后就可以释放了。

老年代中会存放长期存活的对象，比如Spring的大部分bean对象，在程序启动之后就不会被回收了。

在虚拟机的默认设置中，新生代大小要远小于老年代的大小。

**分代`GC`算法将堆分成年轻代和老年代主要原因有：**

- 1、可以通过调整年轻代和老年代的比例来适应不同类型的应用程序，提高内存的利用率和性能。 
- 2、新生代和老年代使用不同的垃圾回收算法，**新生代一般选择复制算法**，**老年代可以选择标记-清除和标记-整理算法**，由程序员来选择灵活度较高。 
- 3、分代的设计中允许只回收新生代（`minor gc`），如果能满足对象分配的要求就不需要对整个堆进行回收`(full gc`)，`STW`时间就会减少。

**垃圾回收器的组合关系**

垃圾回收器是垃圾回收算法的具体实现。 由于垃圾回收器分为年轻代和老年代，**除了`G1`之外其他垃圾回收器必须成对组合进行使用**。

![image-20240306152423530](Java.assets/image-20240306152423530.png)

**1、`Serial`和`SerialOld`组合关系**

|          | **年轻代-`Serial`垃圾回收器**                                | 老年代-`SerialOld`垃圾回收器                                 |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 介绍     | Serial是一种**单线程串行**回收年轻代的垃圾回收器             | `SerialOld`是`Serial`垃圾回收器的老年代版本，采用**单线程串行**回收 <br/>**`-XX:+UseSerialGC` 新生代、老年代都使用串行回收器**。 |
| 回收年代 | 年轻代                                                       | 老年代                                                       |
| 回收算法 | 复制算法                                                     | 标记-整理算法                                                |
| 优点     | 单CPU处理器下吞吐量非常出色                                  | 单CPU处理器下吞吐量非常出色                                  |
| 缺点     | 多CPU下吞吐量不如其他垃圾回收器，堆如果偏大会让用户线程处于长期的等待 | 多CPU下吞吐量不如其他垃圾回收器，堆如果偏大会让用户线程处于长期的等待 |
| 适用场景 | Java编写的客户端程序或者硬件配置有限的场景                   | 与`Seriel`垃圾回收器搭配使用或者在`CMS`特殊情况下适用        |

**2、`ParNew`和`CMS`组合关系**

|          | **年轻代-`ParNew`垃圾回收器**                                | 老年代- `CMS`(Concurrent Mark Sweep)垃圾回收器               |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 介绍     | `ParNew`垃圾回收器本质上是对Serial在多 CPU下的优化，使用**多线程**进行垃圾回收 `-XX:+UseParNewGC` 新生代使用`ParNew`回收器， 老年代使用串行回收器 | `CMS`垃圾回收器关注的是**系统的暂停时间**， 允许用户线程和垃圾回收线程在某些步骤中同时执行，减少了用户线程的等待时间。<br/>参数：`XX:+UseConcMarkSweepGC` |
| 回收年代 | 年轻代                                                       | 老年代                                                       |
| 回收算法 | **复制算法**                                                 | **标记-清除算法**                                            |
| 优点     | 多CPU处理器下停顿时间较短                                    | 系统由于垃圾回收出现的停顿时间较短，用户体验好               |
| 缺点     | 吞吐量和停顿时间不如`G1`， 所以在`JDK9`之后不建议使用        | 内存碎片问题；退化问题；浮动垃圾问题                         |
| 适用场景 | `JDK8`及之前的版本中，与`CMS`老年代垃圾回收器搭配使用        | 大型的互联网系统中用户请求数据量大、频率高的场景，比如订单接口、商品接口等 |

`CMS`执行步骤： 

- 1.初始标记，用极短的时间标记出GC Roots能直接关联到的对象。 

- 2.并发标记，标记所有的对象，用户线程不需要暂停。

- 3.重新标记，由于并发标记阶段有些对象会发生了变化，存在错标、漏标等情况，需要重新标记。

- 4.并发清理，清理死亡的对象，用户线程不需要暂停。

![image-20240306154500654](Java.assets/image-20240306154500654.png)

`CMS` 缺点

- `CMS`使用了标记-清除算法，在垃圾收集结束之后会出现大量的内存碎片，`CMS`会在`Full GC`时进行碎片的整理。 这样会导致用户线程暂停，可以使用`-XX:CMSFullGCsBeforeCompaction=N 参数（默认0）`调整N次`Full GC`之后再整理。 
- 无法处理在并发清理过程中产生的“浮动垃圾”，不能做到完全的垃圾回收。
- 如果老年代内存不足无法分配对象，`CMS`就会退化成Serial Old单线程回收老年代。

**3、Parallel Scavenge和Parallel Old组合关系**

|          | 年轻代-Parallel Scavenge垃圾回收器                           | 老年代-Parallel Old垃圾回收器                                |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 介绍     | Parallel Scavenge是**`JDK8`默认的年轻代垃圾回收器**， **多线程并行回收**，关注的是**系统的吞吐量**。具备**自动调整堆内存大小**的特点。 | Parallel Old是为Parallel Scavenge收集器 设计的老年代版本，利用多线程并发收集。<br />参数： `-XX:+UseParallelGC 或 -XX:+UseParallelOldGC`可以使用`Parallel Scavenge + Parallel Old`这种组合 |
| 回收年代 | 年轻代                                                       | 老年代                                                       |
| 回收算法 | **复制算法**                                                 | **标记-整理算法**                                            |
| 优点     | 吞吐量高，而且手动可控。为了提高吞吐量，虚拟机会动态调整堆的参数 | 并发收集，在多核CPU下效率较高                                |
| 缺点     | 不能保证单次的停顿时间                                       | 暂停时间会比较长                                             |
| 适用场景 | 后台任务，不需要与用户交互，并 且容易产生大量的对象，比如：大数据的处理，大文件导出 | 与`Parallel Scavenge`配套使用                                |

**Parallel Scavenge垃圾回收器**

- Parallel Scavenge允许手动设置最大暂停时间和吞吐量。

- Oracle官方建议在使用这个组合时，不要设置堆内存的最大值，垃圾回收器会根据最大暂停时间和吞吐量自动调整内存大小。

最大暂停时间：`-XX:MaxGCPauseMillis=n` 设置每次垃圾回收时的最大停顿毫秒数

吞吐量：`-XX:GCTimeRatio=n` 设置吞吐量为n（用户线程执行时间 = n/n + 1）

自动调整内存大小：`-XX:+UseAdaptiveSizePolicy`设置可以让垃圾回收器根据吞吐量和最大停顿的毫秒数自动调整内存大小

**4、`G1`垃圾回收器**

**`JDK9`之后默认的垃圾回收器是`G1`（Garbage First）垃圾回收器。** 

- Parallel Scavenge关注吞吐量，允许用户设置最大暂停时间 ，但是会减少年轻代可用空间的大小。 `CMS`关注暂停时间，但是吞吐量方面会下降。 

- 而G1设计目标就是将上述两种垃圾回收器的优点融合： 
  - 1.支持巨大的堆空间回收，并有较高的吞吐量。 
  - 2.支持多CPU并行垃圾回收。 
  - 3.允许用户设置最大暂停时间。
  

> **`JDK9`之后强烈建议使用`G1`垃圾回收器。**

**`G1`垃圾回收器 – 内存结构**

`G1`出现之前的垃圾回收器，内存结构一般是连续的，如下图：

![image-20240306161418800](Java.assets/image-20240306161418800.png)

`G1`的整个堆会被划分成**多个大小相等的区域**，称之为`区Region`，区域不要求是连续的。分为Eden、Survivor、 Old区。Region的大小通过`堆空间大小/2048`计算得到，也可以通过参数`-XX:G1HeapRegionSize=32m`指定(其中`32m`指定region大小为`32M`)，Region size必须是2的指数幂，取值范围从`1M到32M`。

<img src="Java.assets/image-20240306161436973.png" alt="image-20240306161436973" style="zoom: 50%;" />

`G1`垃圾回收有两种方式：

1、年轻代回收（`Young GC`）

- 年轻代回收（`Young GC`），回收Eden区和Survivor区中不用的对象。会导致`STW`，`G1`中可以通过参数 `-XX:MaxGCPauseMillis=n`（默认200） 设置每次垃圾回收时的最大暂停时间毫秒数，`G1`垃圾回收器会尽可能地保证暂停时间。
- 执行流程
  - 1、新创建的对象会存放在Eden区。当`G1`判断年轻代区不足（max默认60%），无法分配对象时需要回收时会执行 Young GC。
  - 2、标记出Eden和Survivor区域中的存活对象。
  - 3、根据配置的最大暂停时间选择某些区域将存活对象复制到一个新的Survivor区中（年龄+1），清空这些区域。
    - `G1`在进行`Young GC`的过程中会去记录每次垃圾回收时每个Eden区和Survivor区的平均耗时，以作为下次回收时的 参考依据。这样就可以根据配置的最大暂停时间计算出本次回收时最多能回收多少个Region区域了。比如 `-XX:MaxGCPauseMillis=n`（默认200），每个Region回收耗时`40ms`，那么这次回收最多只能回收4个Region。
  - 4、后续`Young GC`时与之前相同，只不过`Survivor`区中存活对象会被搬运到另一个`Survivor`区。 
  - 5、当某个存活对象的年龄到达阈值（默认15），将被放入老年代。
  - 6、部分对象如果大小超过Region的一半，会直接放入老年代，这类老年代被称为`Humongous区`。比如堆内存是 4G，每个Region是`2M`，只要一个大对象超过了`1M`就被放入Humongous区，如果对象过大会横跨多个Region。
  - 7、多次回收之后，会出现很多Old老年代区，此时总堆占有率达到阈值时 （`-XX:InitiatingHeapOccupancyPercent默认45%`）会触发混合回收`MixedGC`。回收所有年轻代和部分老年代的对象以及大对象区。采用复制算法来完成。

2、混合回收（`Mixed GC`）

- 混合回收分为：

  - 初始标记（initial mark）

  - 并发标记（concurrent mark）

  - 最终标记（remark或者Finalize Marking）

  - 并发清理（cleanup）


`G1`对老年代的清理会选择存活度最低的区域来进行回收，这样可以保证回收效率最高，这也是`G1（Garbage first）`名称的由来。

![image-20240306162536173](Java.assets/image-20240306162536173.png)

最后清理阶段使用复制算法，不会产生内存碎片。

<img src="Java.assets/image-20240306162610653.png" alt="image-20240306162610653" style="zoom: 50%;" />

**`FULL GC`**：注意：如果清理过程中发现没有足够的空Region存放转移的对象，会出现`Full GC`。单线程执行标记-整理算法， 此时会导致用户线程的暂停。所以尽量保证应该用的堆内存有一定多余的空间。

<img src="Java.assets/image-20240306162641789.png" alt="image-20240306162641789" style="zoom:50%;" />

|          | G1 – Garbage First 垃圾回收器                                |
| -------- | ------------------------------------------------------------ |
| 介绍     | 参数1： `-XX:+UseG1GC` 打开G1的开关， JDK9之后默认不需要打开 <br />参数2：`-XX:MaxGCPauseMillis=毫秒值`  最大暂停的时间 |
| 回收年代 | 年轻代+老年代                                                |
| 回收算法 | **复制算法**                                                 |
| 优点     | 对比较大的堆如超过6G的堆回收时，延迟可控不会产生内存碎片，并发标记的SATB算法效率高 |
| 缺点     | JDK8之前还不够成熟                                           |
| 适用场景 | JDK8最新版本、JDK9之后建 议默认使用                          |

#### 总结

> Java中有哪几块内存需要进行垃圾回收？

线程不共享的区域：Java虚拟机栈和本地方法栈会随着线程的生命周期随着线程回收而回收

线程共享的区域：

- 方法区：一般不需要回收，JSP等技术会通过回收类加载器去回收方法区中的类

- 堆：由垃圾回收器负责进行回收

> 有哪几种常见的引用类型？

强引用，最常见的引用方式，由可达性分析算法来判断

软引用，对象在没有强引用情况下，内存不足时会回收

弱引用，对象在没有强引用情况下，会直接回收

虚引用，通过虚引用知道对象被回收了

终结器引用，对象回收时可以自救，不建议使用

> 有哪几种常见的垃圾回收算法？

| 垃圾回收算法  | 定义                                                         |
| ------------- | ------------------------------------------------------------ |
| 标记-清除算法 | 标记之后再清除，容易产生内存 碎片                            |
| 复制算法      | 从一块区域复制到另一块区域，容易造成只能使用一部分内存；     |
| 标记-整理算法 | 标记之后将存活的对象推到一边 对象会移动，效率不高            |
| 分代GC        | 将内存区域划分成年轻代、幸存者区、老年代进行回收，可以使用多种回收算法 |

> 常见的垃圾回收器有哪些？

| 垃圾回收器                                  | 应用                                                  |
| ------------------------------------------- | ----------------------------------------------------- |
| `Serial`和`SerialOld`组合关系               | 单线程，主要适用于单核CPU场景                         |
| `parNew`和`CMS`组合关系                     | 暂停时间较短，适用于大型互联网应用中与用户交互的部分  |
| `Parallel Scavenge`和`Parallel Old`组合关系 | 吞吐量高，适用于后台进行大量数据操作                  |
| `G1`                                        | 暂停时间较短，适用于大型互联网应用中与用户交互的部分8 |

> 什么时候触发`Young GC`

 当Eden区满了的时候，会触发`Young GC`

> 什么情况下进行`Full GC`

1. 在发生`Young GC`的时候，虚拟机会检测之前每次晋升到老年代的平均大小是否大于年老代的剩余空间，如果大于，则直接进行Full GC；如果小于，但设置了`Handle PromotionFailure`，那么也会执行Full GC。
2. 永久代空间不足，会触发`Full GC`
3. `System.gc()`也会触发`Full GC`
4. 堆中分配很大的对象



## 内存调优

**内存泄漏（memory leak）**：在Java中如果不再使用一个对象，但是该对象依然在GC ROOT的引用链上， 这个对象就不会被垃圾回收器回收，这种情况就称之为内存泄漏。

- 内存泄漏绝大多数情况都是由**堆内存**泄漏引起的，所以后续没有特别说明则讨论的都是堆内存泄漏。

- 少量的内存泄漏可以容忍，但是如果发生持续的内存泄漏，就像滚雪球雪球越滚越大，不管有多大的内存迟早会被消耗完，最终导致的结果就是**内存溢出**。**但是产生内存溢出并不是只有内存泄漏这一种原因**。

内存泄漏的常见场景

- 大型Java后端应用中在处理用户的请求之后，没有及时将用户的数据删除。随着用户请求数量越来越多，内存泄漏的对象占满了堆内存最终导致内存溢出。
- 分布式任务调度系统，如Elastic-job、Quartz等进行任务调度时，被调度的Java应用在 调度任务结束中出现了内存泄漏，最终导致多次调度之后内存溢出。

**解决内存溢出的方法**

- 1、发现问题：通过监控工具尽可能早地发 现内存慢慢变大的现象
- 2、诊断原因：通过分析工具，诊断问题的产生原因，定位到出现问题的源代码
- 3、修复问题：修复源代码中的问题
- 4、测试验证：在测试化境验证问题解决，最后发布上线

发现问题：**Top命令**

- top命令是linux下用来查看系统信息的一个命令，它提供给我们去实时地 去查看系统的资源，比如执行时的进程、线程和系统参数等信息。
- 进程使用的内存为RES（常驻内存）- SHR（共享内存）

- 优点：操作简单，无需额外的软件安装
- 缺点：只能查看最基础的进程信息，无 法查看到每个部分的内存占用 （堆、方法区、堆外）

发现问题：**Visual VM**

- VisualVM是多功能合一的Java故障排除工具并且他是一款可视化工具，整合了 命令行 JDK 工具和轻量级分析功能，功能非常强大。 
- 这款软件在Oracle JDK 6~8 中发布，但是在 Oracle JDK 9 之后不在 JDK安装目录下需要单独下载。下载地址：https://visualvm.github.io/
- 优点：功能丰富，实时监控CPU、 内存、线程等详细信息；支持Idea插件，开发过程中也可以使用
- 缺点：对大量集群化部署的Java进程需要手动进行管理

发现问题：**Arthas**

- Arthas 是一款线上监控诊断产品，通过全局视角实时查看应用 load、内存、 gc、线程的状态信息，并能在不修改应用代码的情况下，对业务问题进行诊断， 包括查看方法调用的出入参、异常，监测方法执行耗时，类加载信息等，大大提升 线上问题排查效率。
- 优点：功能强大，不止于监控基 础的信息，还能监控单个 方法的执行耗时等细节内容；支持应用的集群管理；
- 缺点：部分高级功能使用门槛较高

------

# Spring

## 基础

**Spring 是一个轻量级、非入侵式的控制反转 (IoC) 和面向切面 (AOP) 的框架**

优点：

- **`IoC` 和 DI 的支持**：Spring 的核心就是一个大的工厂容器，可以维护所有对象的创建和依赖关系，Spring 工厂用于生成 Bean，并且管理 Bean 的生命周期，实现**高内聚低耦合**的设计理念。
- **`AOP` 编程的支持**：Spring 提供了**面向切面编程**，可以方便的实现对程序进行权限拦截、运行监控等切面功能。
- **声明式事务的支持**：支持通过配置就来完成对事务的管理，而不需要通过硬编码的方式，以前重复的一些事务提交、回滚的 JDBC 代码，都可以不用自己写了。
- **快捷测试的支持**：Spring 对 `Junit` 提供支持，可以通过**注解**快捷地测试 Spring 程序。
- **快速集成功能**：方便集成各种优秀框架，Spring 不排斥各种优秀的开源框架，其内部提供了对各种优秀框架（如：Struts、Hibernate、`MyBatis`、Quartz 等）的直接支持。
- **复杂 `API` 模板封装**：Spring 对 `JavaEE` 开发中非常难用的一些 `API`（`JDBC`、`JavaMail`、远程调用等）都提供了模板化的封装，这些封装 `API` 的提供使得应用难度大大降低。

> **Spring的常用注解**

<img src="Java.assets/spring-8d0a1518-a425-4887-9735-45321095d927.png" alt="Spring常用注解" style="zoom: 67%;" />

> **Spring包含哪些模块？**
>
> 最主要的七大模块：
>
> 1. **Spring Core**：Spring 核心，它是框架最基础的部分，提供 IoC 和依赖注入 DI 特性。
> 2. **Spring Context**：Spring 上下文容器，它是 BeanFactory 功能加强的一个子接口。
> 3. **Spring Web**：它提供 Web 应用开发的支持。
> 4. **Spring MVC**：它针对 Web 应用中 MVC 思想的实现。
> 5. **Spring DAO**：提供对 JDBC 抽象层，简化了 JDBC 编码，同时，编码更具有健壮性。
> 6. **Spring ORM**：它支持用于流行的 ORM 框架的整合，比如：Spring + Hibernate、Spring + iBatis、Spring + JDO 的整合等。
> 7. **Spring AOP**：即面向切面编程，它提供了与 AOP 联盟兼容的编程实现。

> **Spring中的设计模式**
>
> 1. **工厂模式** : Spring 容器本质是一个大工厂，使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
> 2. **代理模式** : Spring AOP 功能就是通过代理模式来实现的，分为动态代理和静态代理。
> 3. **单例模式** : Spring 中的 Bean 默认都是单例的，这样有利于容器对 Bean 的管理。
> 4. **模板模式** : Spring 中 `JdbcTemplate`、`RestTemplate` 等以 Template 结尾的对数据库、网络等等进行操作的模板类，就使用到了模板模式。
> 5. **观察者模式**: **Spring 事件驱动模型**就是观察者模式很经典的一个应用。
> 6. **适配器模式** :Spring AOP 的增强或通知 (Advice) 使用到了适配器模式、Spring MVC 中也是用到了适配器模式适配 Controller。
> 7. **策略模式**：Spring 中有一个 Resource 接口，它的不同实现类，会根据不同的策略去访问资源。

------

## 事务管理

概念：事务是一组操作的集合，它是一个不可分割的工作单位，这些操作要么同时成功，要么同时失败

操作：

- 开启事务（一组操作开始前，开启事务）：start transaction / begin
- 提交事务(这组操作全部成功后，提交事务)：commit
- 回滚事务(中间任何一个操作出现失误，回滚事务）：rollback

**Spring事务管理**

事务就是一系列的操作原子执行。Spring事务机制主要包括**声明式事务和编程式事务**。

- 编程式事务：通过编程的方式管理事务，这种方式带来了很大的灵活性，但很难维护。 
- 声明式事务：将事务管理代码从业务方法中分离出来，通过`aop`进行封装。Spring声明式事务使得我们无需要去处理获得连接、关闭连接、事务提交和回滚等这些操作。使用 `@Transactional` 注解开启声明式事务。

`@transactional`注解使用：

- 注解：`@transactional`
- 位置：业务service层的方法上，类上，接口上
- 作用：将当前方法交给spring进行事务管理，方法执行前，开启事务；成功执行完毕，提交事务；出现异常，回滚事务

**事务属性**

- 1、回滚`rollbackFor`

  - 默认情况下，只有出现`RuntimeException`才回滚异常。`rollbackFor`属性用于控制出现何种异常类型，回滚事务
  - `@Transactional(rollbackFor = Exception.class)`

- 2、传播行为`propagation`

  - 事务传播行为：指的是当一个事务方法被另一个事务方法调用时，这个事务方法应该如何进行事务控制。
  - `REQUIRED`：大部分情况下都是用该传播行为即可。
  - `REQUIRES_NEW`：当我们不希望事务之间相互影响时，可以使用该传播行为。比如：下订单前需要记录日志，不论订单保存成功与否，都需要日志记录能够记录成功。

  | 属性值        | 含义                                                         |
  | ------------- | ------------------------------------------------------------ |
  | **REQUIRED**  | 【默认值】需要事务，有则加入当前事务，无则创建新事务         |
  | REQUIRES_NEW  | 需要新事务，无论有无，总是创建新事务                         |
  | SUPPORTS      | 支持事务，有则加入，无则在无事务状态中运行                   |
  | NOT_SUPPORTED | 不支持事务，在无事务状态下运行，如果当前存在已有事务，则挂起当前事务 |
  | MANDATORY     | 必须有事务，否则抛异常                                       |
  | NEVER         | 必须没事务，否则抛异常                                       |
  | ...           |                                                              |

> 使用`PROPAGATION_REQUIRES_NEW`时，内层事务与外层事务是两个独立的事务。一旦内层事务进行了提交后，外层事务不能对其进行回滚。两个事务互不影响。

> Spring事务在什么情况下会失效？
>
> - **访问权限问题**：即事务方法的访问权限必须定义成`public`
> - **方法用final修饰**：因为spring事务底层使用了`aop`，也就是通过`jdk`动态代理或者`cglib`，帮我们生成了代理类，在代理类中实现的事务功能。但如果某个方法用final修饰了，那么在它的代理类中，就无法重写该方法，而添加事务功能。
> - 对象没有被Spring管理：使用spring事务的前提是：对象要被spring管理，需要创建bean实例。
> - 表不支持事务：`MyISAM`存储引擎不支持事务
> - 方法内部调用：
> - 未开启事务：
> - 吞了异常：

**Spring 事务隔离级别**

Spring 的接口 `TransactionDefinition` 中定义了表示隔离级别的常量，当然其实主要还是对应数据库的事务隔离级别：

1. `ISOLATION_DEFAULT`：使用后端数据库默认的隔离界别，`MySQL` 默认可重复读，Oracle 默认读已提交。
2. `ISOLATION_READ_UNCOMMITTED`：读未提交
3. `ISOLATION_READ_COMMITTED`：读已提交
4. `ISOLATION_REPEATABLE_READ`：可重复读
5. `ISOLATION_SERIALIZABLE`：串行化

**方法内部调用详解：**

- update方法上面没有加 `@Transactional` 注解，调用有 **`@Transactional`** 注解的 `updateOrder` 方法，`updateOrder` 方法上的事务会失效。因为发生了自身调用，调用该类自己的方法，而没有经过 Spring 的代理类，只有在外部调用事务才会生效。

```java
@Service 
public class OrderServiceImpl implements OrderService {
    public void update(Order order) { 
        this.updateOrder(order);
    }
    
    @Transactional 
    public void updateOrder(Order order) { // update order
    }
}
```

解决方法：

1、再声明一个service，将内部调用改为外部调用；

2、使用编程式事务；

3、使用`AopContext.currentProxy()`获取代理对象

```java
@Servcie 
public class OrderServiceImpl implements OrderService {
    public void update(Order order) { 
        ((OrderService)AopContext.currentProxy()).updateOrder(order);
    }
    @Transactional
    public void updateOrder(Order order) { // update order
    }
}
```

> **Spring如何解决循环依赖的问题？**
>
> 首先，有两种Bean注入的方式：构造器注入和属性注入。
>
> - 对于构造器注入的循环依赖，Spring处理不了，会直接抛出`BeanCurrentlylnCreationException`异常。
>
> - 对于属性注入的循环依赖（**单例模式下**），是通过**三级缓存处理**来循环依赖的。而非单例对象的循环依赖，则无法处理。

> **单例模式下属性注入的循环依赖是怎么处理的？**
>
> 首先，Spring单例对象的初始化大略分为三步：
>
> 1. `createBeanInstance`：实例化bean，使用构造方法创建对象，为对象分配内存。 
> 2. `populateBean`：进行依赖注入。 
> 3. `initializeBean`：初始化bean。
>
> Spring为了解决**单例的循环依赖**问题，使用了三级缓存： 
>
> 1. `singletonObjects`：完成了初始化的单例对象map，`bean name --> bean instance` 
> 2. `earlySingletonObjects`：完成实例化未初始化的单例对象map，`bean name --> bean instance` 
> 3. `singletonFactories`： 单例对象工厂map，`bean name --> ObjectFactory`，单例对象实例化完成之后会加入`singletonFactories`。 
>
> 在调用`createBeanInstance`进行实例化之后，会调用`addSingletonFactory`，将单例对象放到`singletonFactories`中。
>
> 举个例子：
>
> 假如A依赖了B的实例对象，同时B也依赖A的实例对象。
>
> 1. A首先完成了实例化，并且将自己添加到`singletonFactories`中 。
> 2. 接着进行依赖注入，发现自己依赖对象B，此时就尝试去get(B) 。
> 3. 发现B还没有被实例化，对B进行实例化。
> 4. 然后B在初始化的时候发现自己依赖了对象A，于是尝试get(A)，尝试一级缓存`singletonObjects`和二级缓存`earlySingletonObjects`没找到，尝试三级缓存`singletonFactories`，由于A初始化时将自己添加到了`singletonFactories`，所以B可以拿到A对象，然后将A从三级缓存中移到二级缓存中。
>
> 5. B拿到A对象后顺利完成了初始化，然后将自己放入到一级缓存`singletonObjects`中。
> 6. 此时返回A中，A此时能拿到B的对象顺利完成自己的初始化。

> **Spring中实例化和初始化的区别？**
>
> 1. 实例化（Instantiation）：实例化是指创建 Bean 对象的过程，即根据配置信息或注解等方式，Spring 容器通过反射或其他手段创建 Bean 的实例。实例化阶段是指对象通过 new 关键字或反射等方式从类定义创建出来的阶段。
> 2. 初始化（Initialization）：初始化是指创建的 Bean 对象完成实例化后，进行一系列初始化操作的过程。在 Spring 中初始化一般会涉及到依赖注入、属性设置、调用初始化方法等步骤。可以通过配置 init-method 属性、`@PostConstruct` 注解等方式指定初始化方法。
>
> 实例化是对象创建的阶段，而初始化是对象完成创建后进行初始化操作的阶段。

## IOC

[IOC源码](https://javadoop.com/post/spring-ioc)

含义：**IoC（Inversion of Control:控制反转）** 是一种**设计思想**，而不是一个具体的技术实现。`IoC` 的思想就是将**原本在程序中手动创建对象的控制权，交由 Spring 框架来管理**。不过， `IoC` 并非 Spring 特有，在其他语言中也有应用。

- **控制**：指的是对象创建（实例化、管理）的权力
- **反转**：控制权交给外部环境（Spring 框架、IoC 容器）

核心思想：**资源不由使用资源者管理，而是由统一的第三方管理。**

解决问题：

- 对象之间的耦合度或者说依赖程度降低；
- 资源变的容易管理；比如你用 Spring 容器提供的话很容易就可以实现一个单例。

好处：

- 资源集中管理，实现资源的可配置和易管理
- 降低了使用资源双方的依赖程度（耦合度）

> `BeanFactory`和`ApplicationContext`?
>
> 简而言之：`BeanFactory` 是 Spring 的“心脏”，`ApplicantContext` 是完整的“身躯”。
>
> - `BeanFactory`（Bean 工厂）是 Spring 框架的基础设施，面向 Spring 本身。
> - `ApplicantContext`（应用上下文）建立在 `BeanFactory` 基础上，面向使用 Spring 框架的开发者。

> Spring容器启动阶段的动作？
>
> Spring 的 IoC 容器工作的过程，其实可以划分为两个阶段：**容器启动阶段**和**Bean 实例化阶段**。
>
> - 容器启动阶段：主要做的工作是加载和解析配置文件，保存到对应的 Bean 定义中。
> - Bean实例化阶段：实例化对象、装配依赖、生命周期回调，对象其它处理

### 依赖注入DI

**依赖注入DI：是IoC 最常见以及最合理的实现方式**

> spring-context 会自动将 spring-core、spring-beans、spring-aop、spring-expression 这几个基础 jar 包带进来。

在Spring创建对象的过程中，把对象依赖的属性注入到对象中。

依赖注入主要有两种方式：**构造器注入和属性注入**。

`IOC`**容器初始化过程**

1. 从XML中读取配置文件。 
2. 将bean标签解析成 `BeanDefinition`，如解析 `property` 元素， 并注入到 `BeanDefinition` 实例中。 、
3. 将 `BeanDefinition` 注册到容器 `BeanDefinitionMap` 中。
4. `BeanFactory` 根据 `BeanDefinition` 的定义信息创建实例化和初始化 bean。

> 单例bean的初始化以及依赖注入一般都在**容器初始化**阶段进行，只有懒加载（`lazy-init=true`）的单例 bean是在应用第一次调用`getBean()`时进行初始化和依赖注入。
>
> 多例bean 在容器启动时不实例化，即使设置 `lazy-init 为 false` 也没用，只有调用了`getBean()`才进行实例化。

### Spring Bean的生命周期

Spring 中 Bean 的生命周期大致分为四个阶段：

- **实例化**：Spring 容器根据 Bean 的定义创建 Bean 的实例，相当于执行构造方法，也就是 new 一个对象。
- **属性赋值**：相当于执行 setter 方法为字段赋值。
- **初始化**：初始化阶段允许执行自定义的逻辑，比如设置某些必要的属性值、开启资源、执行预加载操作等，以确保 Bean 在使用之前是完全配置好的。
- **销毁**：相当于执行 `= null`，释放资源。

对应的完整步骤如下图所示：

![三分恶面渣逆袭：Spring Bean生命周期](Java.assets/spring-942a927a-86e4-4a01-8f52-9addd89642ff.png)

> `BeanFactory`和`FactoryBean`的区别？

## AOP

**概述**

- AOP：Aspect Oriented Programming(面向切面编程，面向方法编程)，其实就是**面向特定方法编程**，作为面向对象的一种补充，将公共逻辑（事务管理、日志、缓存等）封装成切面，跟业务代码分离，可以减少系统的重复代码和降低模块之间的耦合度
- 场景：统计每一个业务方法的执行耗时
- 实现：**动态代理**，SpringAOP是Spring框架的高级技术，只在管理bean对象的过程中，主要通过**底层的动态代理机制**，对特定的方法进行编程

> **`JDK`动态代理和`CGLIB`动态代理的区别？**
>
> `JDK`动态代理
>
> - 如果目标类实现了接口，`Spring AOP`会选择使用`JDK`动态代理目标类。代理类根据目标类实现的接口动态生成，不需要自己编写，生成的动态代理类和目标类都实现相同的接口。`JDK`动态代理的核心是`InvocationHandler`接口和Proxy类。
> - 缺点：**目标类必须有实现的接口**。如果某个类没有实现接口，那么这个类就不能用JDK动态代理。
>
> `CGLIB`动态代理
>
> - 通过继承实现。如果目标类没有实现接口，那么`Spring AOP`会选择使用`CGLIB`来动态代理目标类。 `CGLIB（Code Generation Library）`可以在运行时动态生成类的字节码，动态创建目标类的子类对象，在子类对象中增强目标类。
> - 缺点：`CGLIB`是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用`CGLIB`做动态代理的。
> - 优点：**目标类不需要实现特定的接口，更加灵活。**
>
> 什么时候采用哪种动态代理？
>
> - 如果目标对象实现了接口，默认情况下会采用`JDK`的动态代理实现`AOP` 
> - 如果目标对象实现了接口，可以强制使用`CGLIB`实现`AOP`
> - 如果目标对象没有实现了接口，必须采用`CGLIB`库
>
> 二者区别
>
> - **实现方式不同**：`jdk`动态代理使用`jdk`中的类`Proxy`来创建代理对象，它使用反射技术来实现，不需要导入其他依赖。`cglib`需要引入相关依赖：`asm.jar`，它使用字节码增强技术来实现。
> - **默认使用方式不同**：当目标类实现了接口的时候`Spring Aop`默认使用`jdk`动态代理方式来增强方法，没有实现接口的时候使用`cglib`动态代理方式增强方法。

**核心概念**

- **连接点**：`JoinPoint`，可以被AOP控制的方法（暗含方法执行时的相关信息）
- **通知**：`Advice`，指哪些重复的逻辑，也就是共性功能（最终体现为一个方法）
- **切入点**：`PointCut`，匹配连接点的条件，通知仅会在切入点方法执行时被调用
- **切面**：`Aspect`，描述通知与切入点的对应关系（通知+切入点）
- **目标对象**：`Target`，通知所应用的对象
- ![aop](Java.assets/aop.png)

**执行流程**：运行的不再是原始对象，而是基于目标对象所生成的代理对象

**通知类型及顺序**

通知类型

- `@Around`：环绕通知，此注解标注的通知方法在目标方法前、后都被执行
- `@Before`：前置通知，此注解标注的通知方法在目标方法前都被执行
- `@After`：后置通知，此注解标注的通知方法在目标方法后都被执行
- `@AfterReturning`：返回后通知，，此注解标注的通知方法在目标方法后被执行，有异常不会执行
- `@AfterThrowing`：异常后通知，，此注解标注的通知方法发生异常后执行

通知顺序：

- 当有多个切面的切入点都匹配到了目标方法，目标方法运行时，多个通知方法都会被执行
- 执行顺序
  - 不同切面类中，默认按照**切面类的类名字母**排序
    - 目标方法前的通知方法：字母排序靠前的先执行
    - 目标方法后的通知方法：字母排名靠前的后执行
  - 用`@Order(数字)`加在切面类上来控制顺序
    - 目标方法前的通知方法：数字小的先执行
    - 目标方法后的通知方法：数字小的后执行

**切入点表达式**

- 概念：描述切入点方法一种表达式

- 作用：主要用来决定项目中的哪些方法需要加入通知

- 常见形式

  - `execution(...)`：根据方法的签名来匹配

    - ```java
      @Around("execution(* com.cug.service.impl.*.*(..))")
      public Object recordTime(ProceedingJoinPoint joinPoint) throws Throwable {}
      ```

    - `execution(访问修饰符？返回值 包名.类名.?方法名(方法参数) throws 异常？)`

    - 其中带？的表示可以省略的部分

      - 访问修饰符：可省略
      - 包名.类名：可省略
      - throws 异常：可省略

    - 可以使用通配符描述切入点

      - `*`：单个独立的任意符号，可以通配任意返回值、包名、类名、方法名、任意类型的一个参数，也可以通配包、类、方法名的一部分
      - `..` ：多个连续的任意符号，可以统配任意层级的包，或任意类型、任意个数的参数

    - 注意事项：根据业务需要，可以使用`且（&&）、或（||）、非（！）`来组合比较复杂的切入点表达式

    - 书写建议

      - 所有**业务方法名在命名时尽量规范**，方便切入点表达式快速匹配。如：查询类方法都是find开头，更新类方法都是update开头
      - 描述切入点方法通常基于**接口描述**，而不是直接描述实现类，增强拓展性
      - 在满足业务需要的前提下，**尽量缩小切入点的匹配范围**。如报名匹配尽量不使用.. ，使用*匹配单个包

  - `@annotation(...)`：根据注解匹配

    - ```java
      @Retention(RetentionPolicy.RUNTIME)
      @Target(ElementType.METHOD)
      public @interface Log {
      }
      ```

    - ```java
      @Around("@annotation(com.cug.anno.Log)")
      public Object recordLog(ProceedingJoinPoint joinPoint) throws Throwable {}
      ```

    - @annotation切入点表达式，用于匹配标识有特定注解的方法

    - 使用步骤：定义一个注解，将该注解放到对应的方法上

**快速入门**

- AOP:记录每个方法的耗时

  - 导入依赖：`spring-boot-starter-aop`

  - ```java
    @Slf4j
    @Component
    // @Aspect // AOP类
    public class TimeAspect {
    
        @Pointcut("execution(* com.cug.service.impl.*.*(..))")
        private void pt() {
        }
    
        @Around("pt()")
        public Object recordTime(ProceedingJoinPoint joinPoint) throws Throwable {
            // 1.记录开始时间
            long begin = System.currentTimeMillis();
            // 2.运行原始方法
            Object result = joinPoint.proceed();
            // 3.获取方法运行结束时间，计算执行耗时
            long end = System.currentTimeMillis();
            log.info(joinPoint.getSignature() + "方法执行耗时：{}ms", end - begin);
            return result;
        }
    }
    ```

- 事务管理

- 权限控制

- 记录操作日志

**连接点**

在Spring中用`JoinPoint`抽象了连接点，用它可以获得方法执行时的相关信息，如目标类名，方法名，方法参数等

对于@Around 通知，获取连接点信息只能使用`ProceedingJoinPoint`

对于其它四种通知，获取连接点信息只能使用`JoinPoint`，它是`ProceedingJoinPoint`的父类型

------

## SpringMVC

定义：`SpringMVC`是一种基于 Java 的实现`MVC`设计模型的请求驱动类型的轻量级Web框架，属于Spring框架 的一个模块。它通过一套注解，让一个简单的Java类成为处理请求的控制器，而无须实现任何接口。同时它还支持`RESTful`编程风格的请求。

优点：

1. 与 Spring 集成使用非常方便，生态好。 
2. 配置简单，快速上手。 
3. 支持 `RESTful` 风格。
4. 支持各种视图技术，支持各种请求资源映射策略。

工作原理

1. `DispatcherServlet` 接收用户的请求 
2. 找到用于处理request的 `handler` 和 `Interceptors`，构造成 `HandlerExecutionChain` 执行链
3. 找到 `handler` 相对应的 `HandlerAdapter` 
4. 执行所有注册拦截器的`preHandler`方法 
5. 调用 `HandlerAdapter` 的 `handle()` 方法处理请求，返回 `ModelAndView` 
6. 倒序执行所有注册拦截器的`postHandler`方法
7. 请求视图解析和视图渲染

![image-20240401112046286](Java.assets/image-20240401112046286.png)

SpringMVC的主要组件：

- 前端控制器（`DispatcherServlet`）：接收用户请求，给用户返回结果。 
- 处理器映射器（`HandlerMapping`）：根据请求的`url`路径，通过注解或者`xml`配置，寻找匹配的 Handler。
- 处理器适配器（`HandlerAdapter`）：Handler 的适配器，调用 handler 的方法处理请求。
- 处理器（Handler）：执行相关的请求处理逻辑，并返回相应的数据和视图信息，将其封装到 ModelAndView对象中。
- 视图解析器（ViewResolver）：将逻辑视图名解析成真正的视图View。
- 视图（View）：接口类，实现类可支持不同的View类型（JSP、`FreeMarker`、Excel等）。

Spring MVC的常用注解：

- `@Controller`：用于标识此类的实例是一个控制器。 
- `@RequestMapping`：映射Web请求（访问路径和参数）。
- `@ResponseBody`：注解返回数据而不是返回页面 
- `@RequestBody`：注解实现接收 `http` 请求的 `json` 数据，将 `json` 数据转换为 `java` 对象。 
- `@PathVariable`：获得URL中路径变量中的值 
- `@RestController`：`@Controller+@ResponseBody`
- `@ExceptionHandler`：标识一个方法为全局异常处理的方法。

### 过滤器Filter

概述：

- 概念：Filter过滤器，是JavaWeb三大组件（Servlet、Filter、Listener）之一
- 过滤器可以把对资源的请求拦截下来，从而实现一些特殊的功能
- 过滤器一般完成一些通用的操作，比如登录校验、统一编码处理、敏感字符处理等

快速入门

- 定义Filter：定义一个类，实现Filter接口，并重写其中所有方法
- 配置Filter：Filter类上加`@WebFilter(urlPatterns="/*")`注解，配置拦截的资源。引导类上加`@ServiceComponentScan`开启Servlet组件支持

执行流程

- 请求->放行前逻辑->放行->资源->放行后逻辑
- 放行后访问对应资源，资源访问完成后，还会回到Filter中吗？会
- 如果回到Filter中，是重新执行还是执行放行后的逻辑呢？执行放行后的逻辑

拦截路径

- ```java
  @WebFilter(urlPatterns="/*")
  public class DemoFilter implements Filter{}
  ```

- Filter可以根据需求，配置不同的拦截资源路径

- 拦截具体路径：`urlPatterns = "/login"`, 只有访问login路径时，才会被拦截

- 目录拦截：`urlPatterns = '/emps/*'`，访问/emps下的所有资源，都会被拦截

- 拦截所有：`urlPatterns = '/*'`，访问所有资源，都会被拦截

过滤器链

- 介绍：在一个web应用中，可以配置多个过滤器，这样多个过滤波器就形成了一个过滤器链
- 顺序：注解配置的Filter，优先级是按照**过滤器类名（字符串）的自然排序**

应用：登录校验Filter

- 所有的请求，拦截到了之后，都需要校验令牌吗？**有一个例外，登录请求**
- 拦截到请求后，什么情况下放行，执行业务操作？**有令牌，且令牌校验通过（合法）；否则都返回未登录错误结果**

- <img src="Java.assets/image-20240318093711284.png" alt="image-20240318093711284" style="zoom:50%;" />

### 拦截器Interceptor

概述

- 概念：是一种动态拦截方法调用的机制，类似于过滤器。spring框架中提供的，用来动态拦截控制器方法的执行

- 作用：拦截请求，在指定的方法调用前后，根据业务需要执行预先设定的代码

快速入门

- 定义拦截器，实现`HandlerInterceptor`接口，并重写其所有方法
- 注册拦截器：

```java
public class LoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        // 1.判断是否需要拦截（ThreadLocal中是否有用户）
        if (UserHolder.getUser() == null) {
            // 没有，需要拦截，设置状态码
            response.setStatus(401);
            // 拦截
            return false;
        }
        // 有用户，则放行
        return true;
    }
}


public class RefreshTokenInterceptor implements HandlerInterceptor {

    private final StringRedisTemplate stringRedisTemplate;

    public RefreshTokenInterceptor(StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        // 1.获取请求头中的token
        String token = request.getHeader("LOGIN_USER_KEY");
        if (StrUtil.isBlank(token)) {
            return true;
        }
        // 2.基于TOKEN获取redis中的用户
        String key = LOGIN_USER_KEY + token;
        Map<Object, Object> userMap = stringRedisTemplate.opsForHash().entries(key);
        // 3.判断用户是否存在
        if (userMap.isEmpty()) {
            return true;
        }
        // 5.将查询到的hash数据转为UserDTO
        UserDTO userDTO = BeanUtil.fillBeanWithMap(userMap, new UserDTO(), false);
        // 6.存在，保存用户信息到 ThreadLocal
        UserHolder.saveUser(userDTO);
        // 7.刷新token有效期
        stringRedisTemplate.expire(key, LOGIN_USER_TTL, TimeUnit.MINUTES);
        // 8.放行
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        // 移除用户
        UserHolder.removeUser();
    }
}
```

```java
/**
 * 注册拦截器
 */
@Configuration
public class MvcConfig implements WebMvcConfigurer {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 拦截所有请求
        registry.addInterceptor(new RefreshTokenInterceptor(stringRedisTemplate)).order(0);
        // 拦截部分请求
        registry.addInterceptor(new LoginInterceptor())
                .excludePathPatterns(
                        "/user/code",
                        "/user/login",
                        "/blog/hot",
                        "/shop/**",
                        "/shop-type/**",
                        "/voucher/**"
                ).order(1);
    }
}
```

拦截路径

- 拦截器可以根据需求，配置不同的拦截路径
- <img src="Java.assets/interceptor拦截路径.png" alt="image-20240318093711284"  />

- 执行流程
- ![image-20240318094603774](Java.assets/image-20240318094603774.png)

### 过滤器和拦截器区别

**实现原理不同**

- 过滤器基于函数回调的，一般自定义过滤器都会有一个doFilter()方法，这个方法有一个FilterClain参数，它是一个回调接口;
- 拦截器是基于Java的反射机制（动态代理）实现的

**使用范围不同**

- 过滤器需要实现`javax.serlvet.Filter`接口(`Serlvet`规范中定义的)，也就是说Filter的使用依赖于`Tomcat`容器，导致它只能再web程序中使用
- 拦截器是一个Spring组件，并由Spring容器管理，并不依赖于Tomcat等容器，可以单独使用，需要实现`HandlerInterceptor`接口

**使用场景不同**

- 拦截器主要用来实现项目中的业务判断，比如日志记录，权限判断等业务
- 过滤器通常是用来实现通用功能过滤，比如敏感词过滤、响应数据压缩等功能

**触发时机不同**

- 过滤器Filter是在请求进入容器后，但在进入servlet之前进行预处理，请求结束是在servlet处理完以后
- 拦截器 Interceptor 是在请求进入servlet后，在进入Controller之前进行预处理的，Controller 中渲染了 对应的视图之后请求结束

**拦截范围不同**

- 请求的执行顺序是：**请求进入容器 -> 进入过滤器 -> 进入 Servlet -> 进入拦截器 -> 执行控制器**。
- 过滤器`Filter`会拦截所有的资源，而`Interceptor`只会拦截`Spring`环境中的资源

------

### 异常处理

全局异常处理器

- 在类上添加注解：`@RestControllerAdvice = @ControllerAdvice + @ResponseBody` 

- 在方法上添加注解：`@ExceptionHandler`

```java
@Slf4j
@RestControllerAdvice
public class WebExceptionAdvice {

    @ExceptionHandler(RuntimeException.class)
    public Result handleRuntimeException(RuntimeException e) {
        log.error(e.toString(), e);
        return Result.fail("服务器异常");
    }
}
```

------

# Maven

## 分模块设计与开发

- 将项目按照功能分为若干个子模块，方便项目的管理维护、扩展，也方便模块间的相互调用，资源共享
- 注意事项：分模块设计需要先针对模块功能进行设计，在进行编码。不会湘江工程开发完毕，然后在进行拆分

## 继承和聚合

**继承**

- 继承描述的是两个工程间的关系，与java中的继承相似，子工程可以继承父工程中的配置信息，常见于依赖关系的继承
- 作用：简化依赖配置、统一管理依赖
- 实现：`<parent>...</parent>`
- 版本锁定
  - ![继承](Java.assets/继承.png)
  - 在maven中，可以在父工程中的pom文件中通过`<dependencyManagement>`来统一管理依赖版本
  - 注意事项：子工程引入依赖时，无需指定`<version>`版本号，父工程统一管理。变更依赖本本，只需在父工程中统一变更
- `<dependencyManagement>`和`<dependencies>`的区别是什么?
  - `<dependencies>`是直接依赖，在父工程配置了依赖，子工程会直接继承下来
  - `<dependencyManagement>`是统一管理依赖版本，不会直接依赖，还需要在子工程中引入所需依赖（无需指定版本）

**聚合**

- 将多个模块组织成一个整体，同时进行项目的构建

- 聚合工程：一个不具有业务功能的”空“，有且只有一个`pom`文件

**二者关系**

- 作用
  - 聚合用于快速构建项目
  - 继承用于简化依赖配置，统一管理依赖
- 相同点
  - 聚合和继承的pom.xml文件打包方式均为pom，可以将两种关系只做到同一个pom文件中
  - 聚合与继承均属于设计型模块，并无实际的模块内容
- 不同点
  - 聚合实在聚合工程中配置关系，聚合可以感知到参与聚合的模块有哪些
  - 继承是在子模块中配置关系，父模块无法感知哪些子模块继承了自己

## 私服

- 私服是一种特殊的远程仓库，它是架设在局域网内的仓库服务，用来代理位于外部的中央仓库，用于解决团队内部的资源共享与资源同步问题
- 私服在企业项目开发中，一个项目/公司，只需要一台即可（无需我们自己搭建，会使用即可）
- 依赖查找顺序：本地仓库->私服->中央仓库

Spring打包方式

- jar：普通模块打包，`springboot`项目基本都是jar包（内嵌tomcat运行）
- war：普通web程序打包，需要部署在外部的tomcat服务器中运行
- pom：父工程或聚合工程，该模块不写代码，仅进行依赖管理

------

# 设计模式

设计模式是软件工程中常用的解决特定问题的模版或者蓝图，可以帮助我们开发者以一种更加清晰、高效和可重用的方式来编写代码。通常分为三类：

1. **创建型模式**：涉及对象实例化，用于创建对象的模式，可以增加程序的灵活性和可重用性。常见的创建型模式有**工厂方法、抽象工厂、单例、建造者、原型**等。
2. **结构型模式**：涉及类和对象的组合，用于设计类和对象的结构，以便更好地实现程序的功能。常见的结构型模式有**适配器、桥接、组合、装饰、外观、享元、代理**等。
3. **行为型模式**：关注对象之间的通信，包括责任链、命令、解释器、迭代器、中介者、备忘录、观察者、状态、策略、模板方法、访问者等。

------

## 单例模式

单例模式（Singleton Pattern）是一种创建型设计模式，**它确保一个类只有一个实例**，**并提供一个全局访问点来获取该实例**。

单例模式主要用于控制对某些共享资源的访问，例如配置管理器、连接池、线程池、日志对象等。

**饿汉式单例**（Eager Initialization）在类加载时就急切地创建实例，不管你后续用不用得到，这也是饿汉式的来源，简单但不支持延迟加载实例。

```java
public class Singleton {
    private static final Singleton instance = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return instance;
    }
}
```

**懒汉式单例**（Lazy Initialization）在实际使用时才创建实例，“确实懒”（😂）。这种实现方式需要考虑线程安全问题，因此一般会带上 synchronized 关键字。

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

通常情况下，使用枚举（Enum）实现单例是最简单的方式，也能防止反射攻击和序列化问题。

```java
public enum Singleton {
    INSTANCE;
    // 可以添加实例方法
}
```

------

## 装饰器模式

**装饰器（Decorator）模式** 可以在不改变原有对象的情况下拓展其功能。

装饰器模式通过组合替代继承来扩展原始类的功能，在一些继承关系比较复杂的场景（IO 这一场景各种类的继承关系就比较复杂）更加实用。

------

## 代理模式

定义：为其他对象提供一种代理以控制对这个对象的访问

![img](Java.assets/v2-9ebf1cd2183e699d766ddf9b7296e1ed_1440w.webp)

代理模式的通用类图。上图中，Subject是一个抽象类或者接口，`RealSubject`是实现方法类，具体的业务执行，`Proxy`则是`RealSubject`的代理，直接和client接触的。代理模式可以在不修改被代理对象的基础上，通过扩展代理类，进行一些功能的附加与增强。值得注意的是，代理类和被代理类应该共同实现一个接口，或者是共同继承某个类。

代理模式优点

- 职责清晰
- 高扩展，只要实现了接口，都可以用代理。
- 智能化，动态代理。

### 静态代理

以租房为例，我们一般用租房软件、找中介或者找房东。这里的中介就是代理者。

首先定义一个提供了租房方法的接口。

```java
public interface IRentHouse {
    void rentHouse();
}
```

定义租房的实现类

```java
public class RentHouse implements IRentHouse {
    @Override
    public void rentHouse() {
        System.out.println("租了一间房子。。。");
    }
}
```

我要租房，房源都在中介手中，所以找中介

```java
public class IntermediaryProxy implements IRentHouse {

    private IRentHouse rentHouse;

    public IntermediaryProxy(IRentHouse irentHouse){
        rentHouse = irentHouse;
    }

    @Override
    public void rentHouse() {
        System.out.println("交中介费");
        rentHouse.rentHouse();
        System.out.println("中介负责维修管理");
    }
}
```

这里中介也实现了租房的接口。

再main方法中测试

```java
public class Main {

    public static void main(String[] args){
        //定义租房
        IRentHouse rentHouse = new RentHouse();
        //定义中介
        IRentHouse intermediary = new IntermediaryProxy(rentHouse);
        //中介租房
        intermediary.rentHouse();
    }
}
```

返回信息

```
交中介费
租了一间房子。。。
中介负责维修管理
```

这就是静态代理，因为中介这个代理类已经事先写好了，只负责代理租房业务

### 强制代理

如果我们直接找房东要租房，房东会说我把房子委托给中介了，你找中介去租吧。这样我们就又要交一部分中介费了，真坑。

来看代码如何实现，定义一个租房接口，增加一个方法。

```java
public interface IRentHouse {
    void rentHouse();
    IRentHouse getProxy();
}
```

这时中介的方法也稍微做一下修改

```java
public class IntermediaryProxy implements IRentHouse {

    private IRentHouse rentHouse;

    public IntermediaryProxy(IRentHouse irentHouse){
        rentHouse = irentHouse;
    }

    @Override
    public void rentHouse() {
        rentHouse.rentHouse();
    }

    @Override
    public IRentHouse getProxy() {
        return this;
    }
}
```

其中的`getProxy()`方法返回中介的代理类对象

我们再来看房东是如何实现租房：

```java
public class LandLord implements IRentHouse {

    private IRentHouse iRentHouse = null;

    @Override
    public void rentHouse() {
        if (isProxy()){
            System.out.println("租了一间房子。。。");
        }else {
            System.out.println("请找中介");
        }
    }

    @Override
    public IRentHouse getProxy() {
        iRentHouse = new IntermediaryProxy(this);
        return iRentHouse;
    }

    /**
     * 校验是否是代理访问
     * @return
     */
    private boolean isProxy(){
        if(this.iRentHouse == null){
            return false;
        }else{
            return true;
        }
    }
}
```

房东的`getProxy`方法返回的是代理类，然后判断租房方法的调用者是否是中介，不是中介就不租房。

main方法测试：

```java
public static void main(String[] args){
        IRentHouse iRentHosue = new LandLord();
        //租客找房东租房
        iRentHouse.rentHouse();
        //找中介租房
        IRentHouse rentHouse = iRentHouse.getProxy();
        rentHouse.rentHouse();
    }
}
```

```
请找中介
租了一间房子。。
```

看，这样就是强制你使用代理，如果不是代理就没法访问。

### 动态代理

我们知道现在的中介不仅仅是有租房业务，同时还有卖房、家政、维修等得业务，只是我们就不能对每一个业务都增加一个代理，就要提供通用的代理方法，这就要通过动态代理来实现了。

中介的代理方法做了一下修改

```java
public class IntermediaryProxy implements InvocationHandler {


    private Object obj;

    public IntermediaryProxy(Object object){
        obj = object;
    }

    /**
     * 调用被代理的方法
     * @param proxy
     * @param method
     * @param args
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result = method.invoke(this.obj, args);
        return result;
    }

}
```

在这里实现`InvocationHandler`接口，此接口是 `JDK` 提供的动态代理接口，对被代理的方法提供代理。其中`invoke`方法是接口`InvocationHandler`定义必须实现的， 它完成对真实方法的调用。动态代理是根据被代理的接口生成所有的方法，也就是说给定一个接口，动态代理就会实现接口下所有的方法。通过 `InvocationHandler`接口， 所有方法都由该`Handler`来进行处理， 即所有被代理的方法都由 `InvocationHandler`接管实际的处理任务。

这里增加一个卖房的业务，代码和租房代码类似。

main方法测试：

```java
public static void main(String[] args){

    IRentHouse rentHouse = new RentHouse();
    //定义一个handler
    InvocationHandler handler = new IntermediaryProxy(rentHouse);
    //获得类的class loader
    ClassLoader cl = rentHouse.getClass().getClassLoader();
    //动态产生一个代理者
    IRentHouse proxy = (IRentHouse) Proxy.newProxyInstance(cl, new Class[]{IRentHouse.class}, handler);
    proxy.rentHouse();

    ISellHouse sellHouse = new SellHouse();
    InvocationHandler handler1 = new IntermediaryProxy(sellHouse);
    ClassLoader classLoader = sellHouse.getClass().getClassLoader();
    ISellHouse proxy1 = (ISellHouse) Proxy.newProxyInstance(classLoader, new Class[]{ISellHouse.class}, handler1);
    proxy1.sellHouse();

}
```

```
租了一间房子。。。
买了一间房子。。。
```

在main方法中我们用到了Proxy这个类的方法，

```java
public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
```

`loader`：类加载器，interfaces：代码要用来代理的接口， h：一个 `InvocationHandler` 对象 。

`InvocationHandler` 是一个接口，每个代理的实例都有一个与之关联的 `InvocationHandler` 实现类，如果代理的方法被调用，那么代理便会通知和转发给内部的 `InvocationHandler` 实现类，由它决定处理。

```java
public interface InvocationHandler {

    public Object invoke(Object proxy, Method method, Object[] args)
        throws Throwable;
}
```

`InvocationHandler` 内部只是一个 invoke() 方法，正是这个方法决定了怎么样处理代理传递过来的方法调用。

因为，Proxy 动态产生的代理会调用 `InvocationHandler` 实现类，所以 `InvocationHandler` 是实际执行者。

### 总结

1. 静态代理，代理类需要自己编写代码写成。
2. 动态代理，代理类通过 `Proxy.newInstance()` 方法生成。
3. `JDK`实现的代理中不管是静态代理还是动态代理，代理与被代理者都要实现两样接口，它们的实质是面向接口编程。`CGLib`可以不需要接口。
4. 动态代理通过 Proxy 动态生成 proxy class，但是它也指定了一个 `InvocationHandler` 的实现类。

# 面试经典问题

> 1、动态代理的实现方式：`JDK`动态代理和`CGLIB`动态代理？

