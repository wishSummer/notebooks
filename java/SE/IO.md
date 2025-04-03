# Java IO

## IO流特性

1. 先进先出，最先写入输出流的数据最先被输入流读取到。
2. 顺序存取，可以一个接一个地往流中写入一串字节，读出时也将按写入顺序读取一串字节，不能随机访问中间的数据。（RandomAccessFile可以从文件的任意位置进行存取（输入输出）操作）
3. 只读或只写，每个流只能是输入流或输出流的一种，不能同时具备两个功能，输入流只能进行读操作，对输出流只能进行写操作。在一个数据传输通道中，如果既要写入数据，又要读取数据，则要分别提供两个流。

## IO流的分类

* 缓冲流
  * 字符流
  * BufferedInputStream、BufferedOutputStream、BufferedReader、BufferedWriter

* 非缓冲流
  * 字节流

![image-20240411203401567](/assets/IO流.jpg "IO流总览")

## 字节流和字符流的区别（重点）

1. 节流没有缓冲区，是直接输出的，而字符流是输出到缓冲区，再由缓冲区输出到文件。因此字节流还未调用close()方法时，数据已经输出了，而字符流只有在调用close()方法关闭缓冲区时，缓冲区数据才会刷新输出，或手动调用flush()方法。
2. 字节流数据以字节（8bit）为传输单位。字符流以字符为单位，根据不同编码表映射字符，一次读多个字节。
3. 字节流根据配置的字符集能处理所有类型的数据（如图片、avi、文本等），而字符流只能处理字符类型的数据。
4. 字节流是直接操作文件，字符流是操作缓冲区，再由缓冲区操作文件。

## IO流常用到的五类一接口

* 五类：File、OutputStream、InputStream、Writer、Reader
  * File（文件特征与管理）：File类是对文件系统中文件以及文件夹进行封装的对象，File类保存文件或目录的各种元数据信息，包括文件名、文件长度、最后修改时间、是否可读、获取当前文件的路径名，判断指定文件是否存在、获得当前目录中的文件列表，创建、删除文件和目录等方法。
  * InputStream（二进制格式操作）：抽象类，基于字节的输入操作，是所有输入流的父类。定义了所有输入流都具有的共同特征。
  * OutputStream（二进制格式操作）：抽象类。基于字节的输出操作。是所有输出流的父类。定义了所有输出流都具有的共同特征。
  * Reader（文件格式操作）：抽象类，基于字符的输入操作。
  * Writer（文件格式操作）：抽象类，基于字符的输出操作。
  * RandomAccessFile（随机文件操作）：一个独立的类，直接继承至Object.它的功能丰富，可以从文件的任意位置进行存取（输入输出）操作。
* 接口：Serializable
  * Serializable：该接口没有任何方法，只是用来标识一个类可以序列化,可以将数据以对象的形式写入文件中，也可以从文件中读取对象数据。transient关键字用于标记类中某些属性不需要序列化。

## java IO流对象

> 字节流与字符流使用选择：

### 输入流

#### 字节输入流

```java
java.lang.Object
  └── java.io.InputStream
      ├── java.io.ByteArrayInputStream
      ├── java.io.FileInputStream
      ├── java.io.FilterInputStream
      │   ├── java.io.BufferedInputStream
      │   ├── java.io.DataInputStream
      │   └── java.io.PushbackInputStream
      ├── java.io.ObjectInputStream
      ├── java.io.PipedInputStream
      └── java.io.SequenceInputStream
```

1. ByteArrayInputStream：字节数组输入流，从内存中的字节数组读取数据。

   ```java
   byte[] data = {65, 66, 67, 68};
   InputStream input = new ByteArrayInputStream(data);
   ```

2. FilterInputStream ：装饰者模式中处于装饰者，具体的装饰者都要继承它，所以在该类的子类下都是用来装饰别的流的，也就是处理类。
3. BufferedInputStream：缓冲流，对处理流进行装饰，增强，内部会有一个缓存区，用来存放字节，每次都是将缓存区存满然后发送，而不是一个字节或两个字节这样发送,效率更高。

    ```java
   InputStream input = new FileInputStream("file.txt");
    ```

4. DataInputStream：数据输入流，它是用来装饰其它输入流，它“允许应用程序以与机器无关方式从底层输入流中读取基本 Java 数据类型”

    ```java
   DataInputStream input = new DataInputStream(new FileInputStream("data.bin"));
   int i = input.readInt();
   double d = input.readDouble();
    ```

5. FileInputSream：文件输入流。它通常用于对文件进行读取操作。

    ```java
    InputStream input = new FileInputStream("file.txt");
    ```

6. File：对指定目录的文件进行操作，注意，该类虽然是在IO包下，但是并不继承自四大基础类。
7. ObjectInputStream：对象输入流，用于将读取到的数据反序列化为对象。

    ```java
    ObjectInputStream input = new ObjectInputStream(new FileInputStream("object.dat"));
    MyClass obj = (MyClass) input.readObject();
    ```

8. SequenceInputStream：将多个输入流逻辑串联起来，形成一个连续的输入流。

    ```java
    InputStream input1 = new FileInputStream("file1.txt");
    InputStream input2 = new FileInputStream("file2.txt");
    SequenceInputStream sequence = new SequenceInputStream(input1, input2);
    ```

9. PushbackInputStream：允许将字节推回流中。

    ```java
    PushbackInputStream input = new PushbackInputStream(new FileInputStream("file.txt"));
    int data = input.read();
    input.unread(data); // 将字节推回流中
    ```

10. PipedInputStream：管道字节输入流，是 Java I/O 系统中的一种特殊输入流，它必须与 PipedOutputStream 配合使用，用于在两个线程之间建立管道通信。PipedInputStream 和 PipedOutputStream 共同创建了一个管道，数据从一个线程的 PipedOutputStream 写入，可以从另一个线程的 PipedInputStream 读取，实现线程间的通信。

    ```java
    import java.io.*;

    public class PipeExample {
        public static void main(String[] args) throws IOException {
            final PipedOutputStream output = new PipedOutputStream();
            final PipedInputStream input = new PipedInputStream(output);
            
            // 也可以使用 connect 方法连接
            // PipedInputStream input = new PipedInputStream();
            // input.connect(output);

            Thread writerThread = new Thread(() -> {
                try {
                    output.write("Hello from PipedOutputStream!".getBytes());
                    output.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            });

            Thread readerThread = new Thread(() -> {
                try {
                    int data;
                    while ((data = input.read()) != -1) {
                        System.out.print((char) data);
                    }
                    input.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            });

            writerThread.start();
            readerThread.start();
        }
    }
    ```

#### 字符输入流

```java
java.lang.Object
  └── java.io.Reader
      ├── java.io.BufferedReader
      ├── java.io.CharArrayReader
      ├── java.io.FilterReader
      │   └── java.io.PushbackReader
      ├── java.io.InputStreamReader
      │   └── java.io.FileReader
      ├── java.io.PipedReader
      ├── java.io.StringReader
      └── java.io.LineNumberReader
```

1. Reader 是所有的输入字符流的父类，它是一个抽象类。
2. InputStreamReader 字节流到字符流的桥梁，可以指定字符编码将字节流转变为字符流。

    ```java
    // 使用默认字符编码
    Reader reader = new InputStreamReader(new FileInputStream("file.txt"));

    // 指定字符编码
    Reader reader = new InputStreamReader(
        new FileInputStream("file.txt"), "UTF-8");
    ```

3. CharReader：从内存中的字符数组读取数据。

    ```java
    char[] data = {'H', 'e', 'l', 'l', 'o'};
    Reader reader = new CharArrayReader(data);
    ```

4. BufferedReader 很明显就是一个装饰器，它和其子类负责装饰其它Reader 对象。

    ```java
    BufferedReader reader = new BufferedReader(
        new FileReader("file.txt"));

    // 读取一行文本
    String line = reader.readLine();
    ```

5. FilterReader 是所有自定义具体装饰流的父类，其子类PushbackReader 对Reader 对象进行装饰，会增加一个行号。

    ```java

    ```

6. FileReader 用于读取字符文件。可以直接声明为字符流，也可由字节流转换为字符流。

    ```java
    Reader reader = new FileReader("file.txt");
    // 等价于：
    new InputStreamReader(new FileInputStream("file.txt"))
    ```

7. CharArrayReader：从内存中的字符数组读取数据。

    ```java
    char[] data = {'H', 'e', 'l', 'l', 'o'};
    Reader reader = new CharArrayReader(data);
    ```

8. StringReader：从字符串中读取字符数据。

    ```java
    Reader reader = new StringReader("Hello World");
    ```

9. PipedReader：与 PipedWriter 配合使用，用于线程间通信。

    ```java
    PipedWriter writer = new PipedWriter();
    PipedReader reader = new PipedReader(writer);
    ```

10. PushbackReader：允许将字符推回流中。

    ```java
    PushbackReader reader = new PushbackReader(
        new FileReader("file.txt"), 10); // 缓冲区大小10字符

    int ch = reader.read();
    reader.unread(ch); // 将字符推回流
    ```

11. LineNumberReader：从字符流中读取数据，并记录行号。

    ```java
    LineNumberReader reader = new LineNumberReader(
        new FileReader("file.txt"));

    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(reader.getLineNumber() + ": " + line);
    }
    ```

### 输出流

#### 字节输出流

```java
java.lang.Object
  └── java.io.OutputStream
      ├── java.io.ByteArrayOutputStream
      ├── java.io.FileOutputStream
      ├── java.io.FilterOutputStream
      │   ├── java.io.BufferedOutputStream
      │   ├── java.io.DataOutputStream
      │   └── java.io.PrintStream
      ├── java.io.ObjectOutputStream
      └── java.io.PipedOutputStream
```

1. OutputStream 是所有的输出字节流的父类，它是一个抽象类。
2. FileOutputStream：用于写入文件。

    ```java
    try (OutputStream out = new FileOutputStream("output.dat")) {
        out.write("Hello".getBytes());
    }
    ```

3. BufferedOutputStream：提供缓冲功能。

    ```java
    try (OutputStream out = new BufferedOutputStream(
            new FileOutputStream("output.dat"))) {
        out.write("Buffered data".getBytes());
    }
    ```

4. DataOutputStream：写入基本数据类型。

    ```java
    try (DataOutputStream out = new DataOutputStream(
            new FileOutputStream("data.bin"))) {
        out.writeInt(123);
        out.writeDouble(3.14);
        out.writeBoolean(true);
    }
    ```

5. ByteArrayOutputStream、FileOutputStream 是两种基本的介质流，它们分别向Byte 数组、和本地文件中写入数据。

    ```java
    ByteArrayOutputStream out = new ByteArrayOutputStream();
    out.write("Test".getBytes());
    byte[] result = out.toByteArray();
    ```

6. PipedOutputStream 是向与其它线程共用的管道中写入数据。
7. ObjectOutputStream 用于序列化对象的核心类，它可以将 Java 对象转换为字节流以便存储或传输。

    ```java
    import java.io.*;

    public class SerializationExample {
        public static void main(String[] args) {
            // 要序列化的对象
            Employee emp = new Employee("John Doe", "IT", 35);
            // objectOutputStream包装bufferedOutputStream包装fileOutputStream
            // 根据需求可以去掉bufferedOutputStream缓冲区。
            try (ObjectOutputStream out = new ObjectOutputStream(
                    new BufferedOutputStream(
                        new FileOutputStream("employee.ser")))) {
                
                // 序列化对象
                out.writeObject(emp);
                System.out.println("对象序列化完成");
                
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    ```

8. PrintStream：提供格式化的打印输出。

    ```java
    try (PrintStream out = new PrintStream(
            new FileOutputStream("print.txt"))) {
        out.println("Hello");
        out.printf("PI: %.2f", Math.PI);
    }
    ```

#### 字符输出流

```java
java.lang.Object
  └── java.io.Writer
      ├── java.io.BufferedWriter
      ├── java.io.CharArrayWriter
      ├── java.io.FilterWriter
      ├── java.io.OutputStreamWriter
      │   └── java.io.FileWriter
      ├── java.io.PipedWriter
      ├── java.io.PrintWriter
      └── java.io.StringWriter
```

1. Writer 是所有的输出字符流的父类，它是一个抽象类。
2. FileWriter：写入字符文件。

    ```java
    try (Writer writer = new FileWriter("text.txt")) {
        writer.write("Hello World");
    }
    ```

3. CharArrayWriter：输出到内存中的字符数组。

    ```java
    char[] data = {'H', 'e', 'l', 'l', 'o'};
    try (CharArrayWriter writer = new CharArrayWriter()) {
        writer.write(data);
        System.out.println(writer.toString());
    }
    ```

4. BufferedWriter 是一个装饰器为Writer 提供缓冲功能。

    ```java
    try (BufferedWriter writer = new BufferedWriter(
            new FileWriter("text.txt"))) {
        writer.write("Line 1");
        writer.newLine();  // 写入换行符
        writer.write("Line 2");
    }
    ```

5. PrintWriter 和PrintStream 极其类似，功能和使用也非常相似。

    ```java
    try (PrintWriter writer = new PrintWriter("print.txt")) {
        writer.println("Hello");
        writer.printf("Today is %tF", new Date());
    }
    ```

6. OutputStreamWriter 是OutputStream 到Writer 转换的桥梁，它的子类FileWriter 其实就是一个实现此功能的具体类（具体可以研究一SourceCode）。

    ```java
    try (Writer writer = new OutputStreamWriter(
            new FileOutputStream("text.txt"), "UTF-8")) {
        writer.write("UTF-8编码文本");
    }
    ```

7. StringWriter：将输出写入字符串。

    ```java
    StringWriter writer = new StringWriter();
    writer.write("Test");
    String result = writer.toString();
    ```

## 字节流与字符流转换

* InputStreamReader:字节到字符的桥梁，将字节流以字符流输入。
* OutputStreamWriter:字符到字节的桥梁，将字节流以字符流输出。

```java
    // 指定字符编码
    Reader reader = new InputStreamReader(
        new FileInputStream("file.txt"), "UTF-8");

    try (Writer writer = new OutputStreamWriter(
            new FileOutputStream("text.txt"), "UTF-8")) {
        writer.write("UTF-8编码文本");
    }
```

## Scanner类 单独搜索补充

> Java 5添加了Scanner 类，是 Java 中一个非常实用的工具类，位于 java.util 包中，主要用于解析原始类型和字符串的简单文本扫描器。它能够将输入分解为标记（tokens），然后使用各种 next 方法将这些标记转换为不同类型的值。

### 基本介绍

1. 可以解析基本类型和字符串
2. 使用正则表达式进行分词
3. 支持多种输入源（文件、字符串、输入流等）
4. 提供丰富的 API 进行不同类型数据的读取

### 构造方法

> Scanner 有多种构造方法，可以接受不同类型的输入源

1. 从标准输入流读取

    ```java
    Scanner scanner = new Scanner(System.in);
    ```

2. 从字符串读取

    ```java
    Scanner scanner = new Scanner("Hello World 123");
    ```

3. 从文件读取

    ```java
    Scanner scanner = new Scanner(new File("input.txt"));
    ```

4. 从输入流读取

    ```java
    Scanner scanner = new Scanner(new FileInputStream("data.bin"));
    ```

5. 指定字符编码

    ```java
    Scanner scanner = new Scanner(new File("data.txt"), "UTF-8");
    ```

### 核心方法

1. 基本类型读取
    nextInt() - 读取下一个整数

    nextDouble() - 读取下一个双精度浮点数

    nextBoolean() - 读取下一个布尔值

    nextByte() - 读取下一个字节

    nextShort() - 读取下一个短整数

    nextLong() - 读取下一个长整数

    nextFloat() - 读取下一个单精度浮点数

2. 字符串读取
    next() - 读取下一个字符串（默认以空白字符为分隔符）

    nextLine() - 读取下一行文本（包括换行符之前的所有内容）

    next(Pattern pattern) - 读取下一个匹配指定模式的字符串

    next(String pattern) - 读取下一个匹配指定正则表达式的字符串

3. 判断方法
    hasNext() - 检查是否还有下一个标记

    hasNextInt() - 检查下一个标记是否是整数

    hasNextDouble() - 检查下一个标记是否是双精度数

    hasNextLine() - 检查是否还有下一行

    hasNext(Pattern pattern) - 检查下一个标记是否匹配指定模式

4. 其他方法
    useDelimiter(String pattern) - 设置分隔符模式

    useLocale(Locale locale) - 设置区域设置（影响数字解析）

    close() - 关闭扫描器

    skip(Pattern pattern) - 跳过匹配指定模式的输入

    findInLine(String pattern) - 在当前行中查找指定模式

### 示例

```java
import java.io.File;
import java.io.FileNotFoundException;
import java.util.Scanner;

public class FileScannerExample {
    public static void main(String[] args) {
        try {
            Scanner fileScanner = new Scanner(new File("data.txt"));
            
            // 设置分隔符为逗号或换行符
            fileScanner.useDelimiter(",|\\n");
            
            while (fileScanner.hasNext()) {
                if (fileScanner.hasNextInt()) {
                    System.out.println("整数: " + fileScanner.nextInt());
                } else if (fileScanner.hasNextDouble()) {
                    System.out.println("浮点数: " + fileScanner.nextDouble());
                } else {
                    System.out.println("字符串: " + fileScanner.next());
                }
            }
            
            fileScanner.close();
        } catch (FileNotFoundException e) {
            System.out.println("文件未找到: " + e.getMessage());
        }
    }
}
```

```java
// 使用正则表达式
Scanner scanner = new Scanner("abc123def456");
scanner.useDelimiter("\\d+");  // 以数字作为分隔符
while (scanner.hasNext()) {
    System.out.println(scanner.next());  // 输出 abc, def
}
scanner.close();

// 多语言数字解析
String number = "1.234,56";  // 某些地区使用逗号作为小数点
Scanner scanner = new Scanner(number);
scanner.useLocale(Locale.GERMANY);  // 使用德国地区设置
double value = scanner.nextDouble();  // 正确解析为 1234.56
System.out.println(value);
scanner.close();

// 查找模式
Scanner scanner = new Scanner("The price is $19.99");
String price = scanner.findInLine("\\$\\d+\\.\\d\\d");  // 查找价格模式
System.out.println(price);  // 输出 $19.99
scanner.close();

```

## 序列化

1. 将保存在内存中的对象数据转化为二进制数据流进行传输，任何对象都可以序列化
2. 如果父类已经实现了Serializable序列化接口的话，其子类就不用再实现这个接口了，但是反过来就不成立了。
3. 只有非静态的数据成员才会通过序列化操作被序列化。
4. 静态（Static）数据成员和被标记了transient的数据成员在序列化的过程中将不会被序列化，因此，如果你不想要保存一个非静态的数据成员你就可以给标记上transient。
5. 当一个对象被反序列化时，这个对象的构造函数将不会在被调用的。
6. 需要实现序列化的对象如果内部引用了一个没有实现序列化接口的对象，在其序列化过程中将会发生错误。

transient关键字（一个类某些属性不需要序列化）

## 装饰者模式

[装饰器模式](./设计模式.md#装饰器模式 "装饰器模式")
