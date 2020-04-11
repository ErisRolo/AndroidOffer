## Java IO

### 一. 字符与字节

在Java中有输入、输出两种IO流，每种输入、输出流又分为字节流和字符流两大类。关于字节，每个字节(byte)由8bit组成，每种数据类型由几个字节组成等。关于字符，代表一个汉字或者英文字母。 

**Java采用unicode编码，2个字节来表示一个字符**，这点与C语言中不同，**C语言采用ASCII编码，在大多数系统中，一个字符通常占1个字节**，但是在0~127整数之间的字符映射，unicode向下兼容ASCII。而Java采用unicode来表示字符，一个中文或英文字符的unicode编码都占2个字节。如果采用其他编码方式，一个字符占用的字节数则各不相同。

简单来讲，一个字符表示一个汉字或英文字母，具体字符与字节之间的大小比例视编码情况而定。有时候读取的数据是乱码，就是因为编码方式不一致，需要进行转换，然后再按照unicode进行编码。



### 二. File类

File类是java.io包下代表与平台无关的文件和目录，在程序中操作文件和目录，都可以通过File类来完成。

**构造函数**示例如下

```java
//构造函数File(String pathname)
File f1 =new File("c:\\abc\\1.txt");
//File(String parent,String child)
File f2 =new File("c:\\abc","2.txt");
//File(File parent,String child)
File f3 =new File("c:"+File.separator+"abc");//separator 跨平台分隔符
File f4 =new File(f3,"3.txt"); System.out.println(f1);//c:\abc\3.txt
```

**创建与删除方法**示例如下

```java
//如果文件存在返回false，否则返回true并且创建文件
boolean createNewFile();
//创建一个File对象所对应的目录，成功返回true，否则false。
//且File对象必须为路径而不是文件。只会创建最后一级目录，如果上级目录不存在就抛异常。
boolean mkdir();
//创建一个File对象所对应的目录，成功返回true，否则false。
//且File对象必须为路径而不是文件。创建多级目录，创建路径中所有不存在的目录。
boolean mkdirs() ;
//如果文件存在返回true并且删除文件，否则返回false
boolean delete();
//在虚拟机终止时，删除File对象所表示的文件或目录。 
void deleteOnExit();
```

**判断方法**示例如下

```java
boolean canExecute() ;//判断文件是否可执行
boolean canRead();//判断文件是否可读
boolean canWrite();//判断文件是否可写
boolean exists();//判断文件是否存在
boolean isDirectory();//判断是否是目录
boolean isFile();//判断是否是文件
boolean isHidden();//判断是否是隐藏文件或隐藏目录
boolean isAbsolute();//判断是否是绝对路径 文件不存在也能判断
```

**获取方法**示例如下

```java
String getName();//返回文件或者是目录的名称
String getPath();//返回路径
String getAbsolutePath();//返回绝对路径
String getParent();//返回父目录，如果没有父目录则返回null
long lastModified();//返回最后一次修改的时间
long length();//返回文件的长度
File[] listRoots();//列出所有的根目录（Window中就是所有系统的盘符）
String[] list() ;//返回一个字符串数组，给定路径下的文件或目录名称字符串
String[] list(FilenameFilter filter);//返回满足过滤器要求的一个字符串数组
File[] listFiles();//返回一个文件对象数组，给定路径下文件或目录
File[] listFiles(FilenameFilter filter);//返回满足过滤器要求的一个文件对象数组
```

FileNameFilter是一个重要接口，该接口是个文件过滤器，包含了一个`accept(File dir,String name)`方法，该方法依次对指定File的所有子目录或者文件进行迭代，按照指定条件，进行过滤，过滤出满足条件的所有文件，示例如下

```java
//文件过滤，过滤出所有后缀为.mp3的文件
File[] files = file.listFiles(new FilenameFilter() {
    @Override
    public boolean accept(File file, String filename) {
        return filename.endsWith(".mp3"); 
    } 
});
```



### 三. IO流

#### 1. 概念

Java的IO流是实现输入/输出的基础，它可以方便地实现数据的输入/输出操作，在Java中把不同的输入/输出源抽象表述为"流"。**流是一组有顺序的，有起点和终点的字节集合，是对数据传输的总称或抽象。**即数据在两设备间的传输称为流，流的本质是数据传输，根据数据传输特性将流抽象为各种类，方便更直观的进行数据操作。流有输入和输出，输入时是流从数据源流向程序，输出时是流从程序传向数据源，而数据源可以是内存，文件，网络或程序等。

#### 2. 分类

① 根据数据流向分为：**输入流**：只能从中读取数据，而不能向其写入数据

​                                        **输出流**：只能向其写入数据，而不能从中读取数据

② 根据操作的数据单元分为：**字节流**和**字符流**

字节流和字符流的区别：1> 读写单位不同：字节流以字节（8bit）为单位，字符流以字符为单位

​                                           2> 处理对象不同：字节流能处理所有类型的数据，字符流只能处理字符类型的数据

处理纯文本数据优先考虑字符流，除此之外都使用字节流。

③ 根据流的角色分为：**节点流**：可以从/向一个特定的IO设备（如磁盘、网络）读/写数据的流，也称低级流

​                                        **处理流**：对一个已存在的流进行连接或封装，通过封装后的流进行数据读写，也称高级流

```java
//节点流，直接传入的参数是IO设备
FileInputStream fis = new FileInputStream("test.txt");
//处理流，直接传入的参数是流对象
BufferedInputStream bis = new BufferedInputStream(fis);
```

当使用处理流进行输入/输出时，程序并不会直接连接到实际的数据源，没有和实际的输入/输出节点连接。使用处理流的一个明显好处是，只要使用相同的处理流，程序就可以采用完全相同的输入/输出代码来访问不同的数据源，随着处理流所包装节点流的变化，程序实际所访问的数据源也相应地发生变化。 

实际上，Java使用处理流来包装节点流是一种典型的装饰器设计模式，通过使用处理流来包装不同的节点流，既可以消除不同节点流的实现差异，也可以提供更方便的方法来完成输入/输出功能。

#### 3. 四大基类

根据流的流向以及操作的数据单元不同，将流分为了四种类型，每种类型对应一种 抽象基类。这四种抽象基类分别为：**Reader**,**Writer**,**InputStream**以及**OutputStream**。四种基类下，对应不同的实现类，具有不同的特性。在这些实现类中，又可以分为节点流和处理流。这四大抽象基类，本身并不能创建实例来执行输入/输出，但它们将成为所有输入/输出流的模版，所以它们的方法是所有输入/输出流都可以使用的方法。类似于集合中的Collection接口。 

##### ① InputStream

InputStream是所有的输入字节流的父类，它是一个抽象类，主要包含三个方法，如下

```java
//读取一个字节并以整数的形式返回(0~255)，如果返回-1已到输入流的末尾。
int read();
//读取一系列字节并存储到一个数组buffer，返回实际读取的字节数，如果读取前已到输入流的末尾返回-1。
int read(byte[] buffer);
//读取len个字节并存储到一个字节数组buffer，从off位置开始存，最多读取len，返回实际读取的字节数，如果读取前已到输入流的末尾返回-1。
int read(byte[] buffer, int off, int len);
```

##### ② Reader

Reader是所有的输入字符流的父类，它是一个抽象类，主要包含三个方法，如下

```java
//读取一个字符并以整数的形式返回(0~255)，如果返回-1已到输入流的末尾。
int read();
//读取一系列字符并存储到一个数组buffer，返回实际读取的字符数，如果读取前已到输入流的末尾返回-1。
int read(char[] cbuf);
//读取len个字符并存储到一个字符数组buffer，从off位置开始存，最多读取len，返回实际读取的字符数，如果读取前已到输入流的末尾返回-1。
int read(char[] cbuf, int off, int len);
```

对比InputStream和Reader所提供的方法，不难发现两个基类的功能基本一样的，只不过读取的数据单元不同。

在执行完流操作后，要调用`close()`方法来关系输入流，因为程序里打开的IO资源不属于内存资源，垃圾回收机制无法回收该资源，所以应该显式关闭文件IO资源。

除此之外，InputStream和Reader还支持移动流中的指针位置，如下

```java
//在此输入流中标记当前的位置。
//readlimit-在标记位置失效前可以读取字节的最大限制。
void mark(int readlimit);
//测试此输入流是否支持mark方法。
boolean markSupported();
//跳过和丢弃此输入流中数据的n个字节/字符。
long skip(long n);
//将此流重新定位到最后一次对此输入流调用mark方法时的位置。
void reset();
```

##### ③ OutputStream

OutputStream是所有的输出字节流的父类，它是一个抽象类，主要包含四个方法，如下

```java
//向输出流中写入一个字节数据,该字节数据为参数b的低8位。
void write(int b) ;
//将一个字节类型的数组中的数据写入输出流。
void write(byte[] b);
//将一个字节类型的数组中的从指定位置（off）开始的len个字节写入到输出流。
void write(byte[] b, int off, int len); 
//将输出流中缓冲的数据全部写出到目的地。 
void flush();
```

##### ④ Writer

Writer是所有的输出字符流的父类，它是一个抽象类，主要包含六个方法，如下

```java
//向输出流中写入一个字符数据，该字节数据为参数b的低16位。
void write(int c);
//将一个字符类型的数组中的数据写入输出流。
void write(char[] cbuf)
//将一个字符类型的数组中的从指定位置（offset）开始的length个字符写入到输出流。
void write(char[] cbuf, int offset, int length);
//将一个字符串中的字符写入到输出流。
void write(String string);
//将一个字符串从offset开始的length个字符写入到输出流。
void write(String string, int offset, int length);
//将输出流中缓冲的数据全部写出到目的地。
void flush();
```

可以看出，Writer比OutputStream多出两个方法，主要是支持写入字符和字符串类型的数据。

使用Java的IO流执行输出时，不要忘记关闭输出流，关闭输出流除了可以保证流的物理资源被回收之外，还能将输出流缓冲区的数据flush到物理节点里（因为在执行`close()`方法之前，自动执行输出流的`flush()`方法）。