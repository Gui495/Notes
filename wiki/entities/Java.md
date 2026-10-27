---
type: entity
status: evergreen
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Frontend_Backend/Java_Basics.md
  - raw/Frontend_Backend/Classes_and_Objects.md
  - raw/Frontend_Backend/Methods_and_Method_Overloading.md
  - raw/Frontend_Backend/Input_and_Output_Processing.md
  - raw/Frontend_Backend/Set_Framework_and_Generics1.md
  - raw/Frontend_Backend/Set_Framework_and_Generics2.md
  - raw/Frontend_Backend/Multithreading.md
  - raw/Frontend_Backend/Reflex_Mechanism.md
  - raw/Frontend_Backend/Practical_Introduction.md
tags:
  - java
---

Java 是一门面向对象的通用编程语言，以「一次编写、到处运行」的虚拟机方案和庞大的标准类库为特征；本知识库覆盖的 Java 素材集中在基础语法、面向对象、集合与泛型、多线程、反射、实用类与 IO 九篇，构成一条从语法到框架基础设施的入门主线。

## 素材组织（9 篇的顺序逻辑）

| 篇目 | 主题 | 对应概念页 |
|---|---|---|
| Java_Basics | 变量、8 种基本类型、运算符、流程控制、数组 | — |
| Classes_and_Objects | 类、对象、封装边界、类图 | [[面向对象]]、[[类与对象]] |
| Methods_and_Method_Overloading | 带参方法、值/引用传递、构造方法、`this`、重载 | [[方法重载]] |
| Input_and_Output_Processing | `File`、字节流/字符流、转换流、序列化 | [[IO流]] |
| Set_Framework_and_Generics1 | `Collection` 体系、`List`/`Set`、迭代器 | [[集合框架]] |
| Set_Framework_and_Generics2 | `Map`、泛型、`Collections` 与 `Comparable` | [[泛型]] |
| Multithreading | 进程/线程、两种建线程、五状态、`synchronized`、并发工具 | [[多线程]] |
| Reflex_Mechanism | 运行期观察与修改类、`java.lang.reflect` | [[反射机制]] |
| Practical_Introduction | 枚举、包装类、`Math`/`Random`、`String` | [[枚举]] |

## 语言要点

**类型系统**：8 种基本类型分四组——整数 `byte`(1 字节 8 位，-128~127) / `short`(2) / `int`(4) / `long`(8)，浮点 `float`(4) / `double`(8)，字符 `char`，布尔 `boolean`；`char` 与 `boolean` 素材未标位数。`String` 不是基本类型而是引用类型。基本类型有对应的包装类（`Integer` 等），二者之间自动装箱拆箱。

**常量**用 `final` 修饰，赋值后不可改；`final` 与 `static` 合用时得到类级常量。

**类型转换**：小范围到大范围自动转换，反之必须强制转换（可能丢精度）。

**对象语义**：Java 的方法参数一律**值传递**——传基本类型传的是值的副本，改不到外部变量；传对象传的是引用的副本，两个引用指向同一个堆对象，所以改对象的属性会生效，但让形参指向新对象不影响外部。这个「传引用也是传值」的细节是初学者最容易误解的地方。

**字符串比较**必须用 `equals` 而非 `==`：`==` 比的是引用（地址），`equals` 比的是内容。素材在基础篇与实用类篇各强调了一次。

**内存模型**：对象在堆、基本类型与引用变量在栈、类信息在方法区——素材里 `String` 的 `==` 行为正是依赖这一点解释的。

## 与其他页面的关系

[[MySQL]] 与 Java 的交集在 JDBC：反射的典型应用场景之一就是用 `Class.forName` 加载数据库驱动，见 [[反射机制]]。并发容器（`BlockingQueue` 等）与 [[集合框架]] 同源但面向并发场景，见 [[多线程]]。

## 素材边界（贯通 9 篇的共同缺口）

这 9 篇是一套课堂讲义式素材，共同缺口集中在语言后半段：**没有继承、接口、多态**（面向对象只讲到封装）、**没有异常处理**（`try-catch-finally` 仅在个别例子里出现，未系统讲）、**没有注解**、**没有 Lambda 与 Stream**、**没有 JVM 内存模型与 GC**、**没有项目构建工具（Maven/Gradle）**。用它入门语法与常用类库是完整的，用它覆盖「Java 工程师」的知识面则远远不够。

## 关联

- 概念：[[面向对象]]、[[类与对象]]、[[方法重载]]、[[集合框架]]、[[泛型]]、[[多线程]]、[[反射机制]]、[[枚举]]、[[IO流]]
- 对照实体：[[MySQL]]

来源：[[raw/Frontend_Backend/Java_Basics]]、[[raw/Frontend_Backend/Classes_and_Objects]]、[[raw/Frontend_Backend/Methods_and_Method_Overloading]]、[[raw/Frontend_Backend/Input_and_Output_Processing]]、[[raw/Frontend_Backend/Set_Framework_and_Generics1]]、[[raw/Frontend_Backend/Set_Framework_and_Generics2]]、[[raw/Frontend_Backend/Multithreading]]、[[raw/Frontend_Backend/Reflex_Mechanism]]、[[raw/Frontend_Backend/Practical_Introduction]]
