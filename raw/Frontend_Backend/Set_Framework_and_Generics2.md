## 1. Map 接口
- 专门处理键值映射数据存储，根据键对值操作。
- 最常用实现类：`HashMap`。
- 常用方法：
  - `Object put(Object key, Object val)`：键值对存储。
  - `Object get(Object key)`：根据键返回值，不存在返回 null。
  - `Object remove(Object key)`：删除键值对。
  - `int size()`：元素个数。
  - `Set keySet()`：返回键集合。
  - `Collection values()`：返回值集合。
  - `boolean containsKey(Object key)`：是否存在指定键。
- 遍历方式：
  - Iterator。
  - 增强 for。
  - 键值对。

## 2. 泛型
- 解决强制类型转换异常问题。
- JDK5.0 使用泛型改写集合框架所有接口和类。
- 将对象类型作为参数，指定到其他类或方法上，保证类型转换安全性和稳定性。
- 本质：参数化类型。
- 泛型集合：
  - `ArrayList<E>`、`HashMap<K,V>`
  - 约束集合内元素类型。
  - 数据不再转换为 Object。
  - 除指定元素类型外，用法与普通集合完全一样。
示例：
```java
ArrayList<Student> students = new ArrayList<Student>();
students.add(student);
// students.add(poem); 编译报错
```

## 3. Collections 算法类
- Java 集合框架将不同数据结构算法实现保存在工具类中。
- `Collections` 类定义一系列操作集合的静态方法。
- 与 `Collection` 不同：前者是集合操作类，后者是集合接口。
- 常用静态方法：
  - `sort()`：排序
  - `binarySearch()`：查找
  - `max()`：查找最大/最小值
- 排序：实现 `Comparable` 接口，重写 `compareTo()` 方法。

## 4. 小结
- Collection、List、Set、Map 联系和区别。
- ArrayList 和 LinkedList 异同及应用场合。
- 遍历集合的方法。
- 泛型集合用法及好处。
- Collections 工具类使用。