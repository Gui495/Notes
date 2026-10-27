# 一.实用类介绍（一）

## 1. 枚举
- 枚举指由一组固定常量组成的类型。
```java
[Modifier] enum enumName {
    enumConstantName1 [,enumConstantName2...[:]]
    // [field, method]
}
```
- 示例：
```java
public enum Genders { Male, Female }
public class Student { public Genders sex; }
Student stu = new Student();
stu.sex = Genders.Male;
// stu.sex = "你好"; 错误
```
- 好处：类型安全、易于输入、代码清晰。
- 示例：大数据岗位课程三个阶段 L1、L2、L3，输出学习目标。

## 2. 常用 Java API
- `java.lang`：Enum、包装类、Math、String、StringBuffer、System 等。
- `java.util`、`java.io`、`java.sql` 等。
- 学习方法：查阅 Java API 文档。

## 3. 包装类
- 把基本类型数据转换为对象。
- 每个基本类型在 `java.lang` 包中都有相应包装类。
- 作用：
  - 提供一系列实用方法。
  - 集合不允许存放基本数据类型，存放数字时要用包装类型。
- 继承关系：Object → Boolean/Number/Character；Number → Byte/Short/Integer/Long/Float/Double。

### 3.1 构造方法
- 所有包装类都可将对应基本数据类型作为参数构造实例：
```java
Integer i = new Integer(1);
```
- 除 Character 外，其他包装类可将字符串作为参数：
```java
Integer i = new Integer("123");
```
- 注意事项：
  - Boolean 构造参数为 String 时，内容为 true（忽略大小写）则为 true，否则 false。
  - Number 包装类构造参数为 String 时，字符串不能为 null，且必须可解析为相应基本类型，否则运行时 NumberFormatException。

### 3.2 常用方法
- `XXXValue()`：包装类转基本类型。
```java
Integer integerId = new Integer(25);
int intId = integerId.intValue();
```
- `toString()`：基本类型转字符串。
```java
String sex = Character.toString('男');
String id = Integer.toString(25);
```
- `parseXXX()`：字符串转基本类型（Character 除外）。
```java
int num = Integer.parseInt("36");
boolean bool = Boolean.parseBoolean("false");
```
- `valueOf()`：基本类型转包装类；除 Character 外字符串转包装类。
```java
Integer intValue = Integer.valueOf(21);
Integer intValue2 = Integer.valueOf("21");
```

### 3.3 装箱与拆箱
```java
Integer intObject = 5; // 装箱：基本类型 -> 包装类对象
int intValue = intObject; // 拆箱：包装类对象 -> 基本类型
```
- JDK1.5 后允许基本类型和包装类型混合数学运算。
- 包装类不是取代基本数据类型，在基本类型需要用对象表示时使用。

## 4. Math 类
- `java.lang.Math` 提供常用数学运算方法和两个静态常量 E、PI。
```java
Math.abs(-3.5);       // 3.5
Math.max(2.5, 90.5);  // 90.5
int random = (int)(Math.random()*10); // 0-9 随机数
```

## 5. 小结
- 枚举作用：固定常量、类型安全。
- 装箱/拆箱：基本类型与包装类转换。
- Math 类获取随机数、绝对值、最大值等。

---

# 二 .实用类介绍（二）.md

## 1. Random 类
```java
Random rand = new Random();
for(int i=0;i<20;i++){
    int num = rand.nextInt(10); // 0-9
    System.out.println("第"+(i+1)+"个随机数是："+num);
}
```
- 用同一个种子值初始化两个 Random 对象，用相同方法得到相同随机数。

## 2. String 类
- 位于 `java.lang` 包，具有丰富方法。
- 创建：
```java
String s = "Hello World";
String s2 = new String();
```
- 常用操作：计算长度、比较字符串、连接字符串、提取字符串。

### 2.1 length()
- 返回字符串中的字符数。
```java
String password = "1234567";
password.length();
```
- 案例：注册新用户，密码长度不能小于6位。

### 2.2 equals() 与 ==
- `equals()`：检查组成字符串内容的字符是否完全一致。
- `==`：判断两个字符串在内存中的地址，即是否是同一个字符串对象。
- 示例：登录验证用户名 TOM、密码 1234567。
- 忽略大小写：`equalsIgnoreCase()`。
- 转小写：`toLowerCase()`；转大写：`toUpperCase()`。

### 2.3 字符串连接
- 方法1：`+`
- 方法2：`concat()`
```java
String s = "Hello, ".concat("World");
```

### 2.4 字符串提取
- `indexOf(int ch)` / `indexOf(String value)`：搜索第一个出现的位置，未找到返回 -1。
- `lastIndexOf(int ch)` / `lastIndexOf(String value)`：搜索最后一个出现的位置。
- `substring(int index)`：从位置索引开始提取。
- `substring(int beginindex, int endindex)`：提取 beginindex 和 endindex 之间的字符串部分。
- `trim()`：返回前后不含任何空格的调用字符串副本。
- 示例：
```java
String word = "Hello, ";
word = word.trim();
String s = word.concat("小鱼儿!");
int index1 = s.indexOf(',');
int index2 = s.indexOf('!');
System.out.println(s.substring(index1+1, index2));
```

## 3. 练习
- 会员注册：
  - 用户名长度不小于3。
  - 密码长度不小于6。
  - 两次输入密码必须相同。
- 查找特定字符出现次数：
  - 输入字符串，再输入要查找的字符，判断出现次数。
- 判断 `.java` 文件名、邮箱格式是否正确。

## 4. 小结
- Random 获取随机数。
- String 长度、比较、连接、提取、拆分。
- `==` 与 `equals()` 区别。