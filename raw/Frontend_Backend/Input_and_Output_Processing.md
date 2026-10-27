# 一 .输入和输出处理（一）.md

## 1. 输入输出概念
- 文件、内存、键盘 → 程序 → 文件、内存、控制台。
- Java 程序访问文件属性：`java.io.File` 类。

## 2. File 类
```java
File file = new File(String pathname);
```
- 创建文件对象，指向物理文件或目录。
- 通过文件对象方法操作文件或目录属性（路径、权限、日期时间等）。
- 常用方法：
  - `boolean exists()`：判断文件或目录是否存在。
  - `boolean isFile()`：判断是否是文件。
  - `boolean isDirectory()`：判断是否是目录。
  - `String getPath()`：返回相关路径名。
  - `String getAbsolutePath()`：返回绝对路径名。
  - `String getName()`：返回文件或目录名称。
  - `boolean delete()`：删除指定的文件或目录。
  - `boolean createNewFile()`：创建名称的空文件，不创建文件夹。
  - `long length()`：返回文件长度（字节），不存在返回 0L。
- 练习：查看文件属性、创建和删除文件。

## 3. 流
- 流：一组有序的数据序列，以先进先出方式发送信息的通道。
- 输入流：来自数据源的数据流流向程序。
- 输出流：程序流向目的地的数据流。
- Java 流分类：
  - 按流向：输入流（InputStream/Reader 基类）、输出流（OutputStream/Writer 基类）。
  - 按处理数据单元：字节流（8位通用字节流，InputStream/OutputStream）、字符流（16位 Unicode 字符流，Reader/Writer）。
- 输入输出流是相对于计算机内存来说的。

## 4. 字节流读文本文件
### 4.1 FileInputStream
常用方法：
- `int read()`
- `int read(byte[] b)`
- `int read(byte[] b, int off, int len)`
- `void close()`
- `int available()`：可从输入流读取的字节数目。

构造方法：
```java
FileInputStream(File file)
FileInputStream(String name)
```
步骤：
1. 引入相关类。
2. 构造文件输入流 FileInputStream 对象。
3. 读取文本文件数据。
4. 关闭文件流对象。
```java
import java.io.IOException;
import java.io.FileInputStream;

FileInputStream fis = new FileInputStream("c:\\test.txt");
fis.available();
fis.read();
fis.close();
```

## 5. 字节流写文本文件
### 5.1 FileOutputStream
常用方法：
- `void write(int c)`
- `void write(byte[] buf)`
- `void write(byte[] b, int off, int len)`
- `void close()`
- `void flush()`：强制把缓冲区数据写到输出流。

构造方法：
```java
FileOutputStream(File file)
FileOutputStream(String name)
FileOutputStream(String name, boolean append)
```
- 前两种覆盖文件原有内容。
- 创建实例时文件不存在会自动创建空文件。
- 步骤：
  1. 引入相关类。
  2. 构造文件输出流 FileOutputStream 对象。
  3. 把数据写入文本文件。
  4. 关闭文件流对象。
```java
import java.io.IOException;
import java.io.FileOutputStream;

FileOutputStream fos = new FileOutputStream("text.txt");
String str = "好好学习 Java";
byte[] words = str.getBytes();
fos.write(words, 0, words.length);
fos.close();
```

## 6. 复制文本文件
- 文件“我的青春谁做主.txt”位于 D 盘根目录，复制到 `C:\myFile\myPrime.txt`。
- 常见错误：
  - 开始读了两次。
  - 不是按实际长度写，可能写入多余空格。
- 正确思路：循环读取实际长度，写入实际长度。

## 7. 小结
- File 类常用属性/方法。
- 流按流向和处理单元分类。
- 字节流读写文本文件步骤。
- FileInputStream/FileOutputStream 使用。
- 注意关闭流、按实际读取长度写入。

---

# 二 .输入和输出处理（二）.md

## 1. 字符流
### 1.1 Reader 类
常用方法：
- `int read()`
- `int read(char[] c)`
- `read(char[] c, int off, int len)`
- `void close()`

子类 InputStreamReader 构造方法：
```java
InputStreamReader(InputStream in)
InputStreamReader(InputStream in, String charsetName)
```

### 1.2 FileReader
- FileReader 是 InputStreamReader 的子类。
- 构造方法：
```java
FileReader(File file)
FileReader(String name)
```
- 只能按照本地平台字符编码读取数据，用户不能指定其他编码。
- 获取本地平台字符编码：`System.getProperty("file.encoding")`。
- 读取步骤类似 FileInputStream：
```java
import java.io.Reader;
import java.io.FileReader;
import java.io.IOException;

Reader fr = new FileReader("D:\\myDoc\\简介.txt");
fr.read();
fr.close();
```

### 1.3 BufferedReader
- 提高字符流读取文本文件效率。
- BufferedReader 是 Reader 子类，带缓冲区。
- 特有方法：`readLine()` 按行读取。
- 构造方法：`BufferedReader(Reader in)`。
```java
import java.io.FileReader;
import java.io.BufferedReader;
import java.io.IOException;

Reader fr = new FileReader("C:\\myTest.txt");
BufferedReader br = new BufferedReader(fr);
br.readLine();
br.close();
fr.close();
```
- 中文乱码解决：使用 InputStreamReader 并设置编码。
```java
FileInputStream fis = new FileInputStream("c:\\myDoc\\hello.txt");
InputStreamReader fr = new InputStreamReader(fis, "UTF-8");
BufferedReader br = new BufferedReader(fr);
```

## 2. Writer 类
常用方法：
- `write(String str)`
- `write(String str, int off, int len)`
- `void close()`
- `void flush()`

子类 OutputStreamWriter 构造方法：
```java
OutputStreamWriter(OutputStream out)
OutputStreamWriter(OutputStream out, String charsetName)
```

### 2.1 FileWriter
- FileWriter 是 OutputStreamWriter 的子类。
- 构造方法：
```java
FileWriter(File file)
FileWriter(String name)
```
- 只能按照本地平台字符编码写数据，不能指定其他编码。
- 步骤类似 FileOutputStream：
```java
import java.io.Writer;
import java.io.FileWriter;
import java.io.IOException;

Writer fw = new FileWriter("D:\\myDoc\\简介.txt");
fw.write("内容");
fw.close();
```

### 2.2 BufferedWriter
- 提高字符流写文本文件效率。
- BufferedWriter 是 Writer 子类，带缓冲区。
- 构造方法：`BufferedWriter(Writer out)`。
```java
FileWriter fw = new FileWriter("C:\\myTest.txt");
BufferedWriter bw = new BufferedWriter(fw);
bw.write("hello");
bw.flush();
fw.close();
```
- 练习：替换文本文件内容，将模板 `{name}`、`{type}`、`{master}` 替换为具体宠物信息，写入 `C:\myDoc\pet.txt`。

## 3. 读写二进制文件
- `DataInputStream`：FileInputStream 子类，与 FileInputStream 结合读取二进制文件。
- `DataOutputStream`：FileOutputStream 子类，与 FileOutputStream 结合写二进制文件。
```java
import java.io.FileOutputStream;
import java.io.DataOutputStream;

FileOutputStream outFile = new FileOutputStream("C:\\temp.class");
DataOutputStream out = new DataOutputStream(outFile);
out.write(...);
out.close();
```
- 练习：复制图片 `star.jpg` 从 C 盘根目录到 `D:\myDoc\myPicture.jpg`。

## 4. 序列化和反序列化
### 4.1 场景
- 内存对象需要在其他环境下使用：网络通信需二进制序列传送。
- 内存对象需要在将来某个时间使用：持久化到磁盘。

### 4.2 常用序列化方案
- Java 内置：实现 Serializable 接口。
- XML：文档标记描述语言，跨平台跨语言。
- JSON：属性-值方式描述对象，协议简单，解析快。
- Thrift：RPC 框架，空间开销和解析性能提升。
- Avro：序列化文件自我描述，广泛应用于 Hive、Pig、MapReduce。

### 4.3 序列化过程
- 序列化：将对象状态写入特定流。
- 反序列化：从特定流获取数据重新构建对象。

### 4.4 实现序列化步骤
1. 实现 `Serializable` 接口。
2. 创建对象输出流。
3. 调用 `writeObject()` 方法将对象写入文件。
4. 关闭对象输出流。
- 注意：使用集合保存对象，可以将集合中所有对象序列化。

### 4.5 实现反序列化步骤
1. 实现 `Serializable` 接口。
2. 创建对象输入流。
3. 调用 `readObject()` 方法读取对象。
4. 关闭对象输入流。
- 注意：如果向文件中使用序列化机制写入多个对象，反序列化恢复时必须按照写入顺序读取。

## 5. 小结
- BufferedReader/BufferedWriter 读写文本文件优势：缓冲、按行读取/写入。
- 复制二进制文件使用字节流：FileInputStream/FileOutputStream。
- 序列化/反序列化：Serializable、ObjectOutputStream/ObjectInputStream。