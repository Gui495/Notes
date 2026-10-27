---
type: concept
status: budding
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Frontend_Backend/Input_and_Output_Processing.md
tags:
  - java
  - io
---

IO 流是 Java 中用于在程序与外部数据源（文件、内存、网络）之间搬运字节或字符的通道抽象，数据像水流一样从一端流向另一端，按方向分输入输出、按处理单元分字节字符。

**四个分类维度**共同决定了该挑哪个类：

| 维度 | 两类 | 说明 |
|---|---|---|
| 流向 | 输入流 / 输出流 | 相对**程序**而言，读进来是输入 |
| 处理单元 | 字节流 / 字符流 | 字节流以 `byte` 为单位，字符流以 `char` 为单位 |
| 功能 | 节点流 / 处理流 | 节点流直接连数据源，处理流套在别的流上做加工 |
| 数据源 | 文件 / 数组 / 管道 / 网络 | 决定头一层用哪个节点流 |

**字节流的基类**是 `InputStream` / `OutputStream`，文件版是 `FileInputStream` / `FileOutputStream`；**字符流的基类**是 `Reader` / `Writer`，文件版是 `FileReader` / `FileWriter`。字符流只能使用**本机默认编码**处理，一旦文件编码与平台编码不一致就出现乱码——这是用 `FileReader` 读 UTF-8 文件在 GBK 平台上翻车的根源。

**乱码的正解**是在字节流与字符流之间架一层转换流：`InputStreamReader` / `OutputStreamWriter`，在构造时显式指定编码。再套 `BufferedReader` 就能用 `readLine()` 按行读，这几乎是读取文本文件的标准姿势。

**复制文件的两个经典坑**（素材用 `FileInputStream`/`FileOutputStream` 复制演示）：一是把 `read()` 调用了两次（`while (in.read() != -1) { out.write(in.read()); }` 这种写法），导致隔一个字节丢一个；二是按缓冲区**容量**写出而不是按本次实际读到的**长度**写出，导致末尾补上一堆脏数据。正确写法是每次 `read` 只调一次、把返回值（实际长度）传给 `write`。

**处理流的其他成员**：`DataInputStream` / `DataOutputStream` 让读写基本类型（`readInt` / `writeInt`）变方便，且读写顺序必须一致。

**序列化**把对象转成字节序列以便存盘或传输，反序列化是逆过程。要求对象所属类实现 `Serializable`，并用 `ObjectOutputStream` / `ObjectInputStream` 配合文件流完成。素材另列了 XML、JSON、Thrift、Avro 等替代方案，但只给了名字、没有代码。

**素材边界**：没有 `try-with-resources`（8 篇素材里所有流都是手动 `close()`）、没有 NIO / `Files`、没有 `RandomAccessFile`。

它在 [[Java]] 平台里承担「一切与外部世界交换数据」的职责，与 [[反射机制]] 一起构成框架层的两个基础设施（一个管数据进出，一个管类结构）。

## 关联

- [[Java]]
- 素材提炼：[[2026-10-27-输入输出处理]]
