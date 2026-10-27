## 1. 为什么使用集合框架
- 数组长度固定，存储数量不确定时浪费空间或空间不足。
- 需要更复杂方式存储对象时使用 Java 集合框架。
- 集合位于 `java.util` 包。

## 2. 集合框架体系
- `Collection`：存储一组不唯一、无序的对象。
  - `List`：不唯一、有序（插入顺序）。
    - `ArrayList`
    - `LinkedList`
  - `Set`：唯一、无序。
    - `HashSet`
    - `TreeSet`
- `Map`：键值对象，key 到 value 映射。
  - `HashMap`
  - `TreeMap`
- 工具类：`Collections`、`Arrays`，提供排序、遍历等算法。

## 3. List 接口实现类
### 3.1 ArrayList
- 实现长度可变数组，内存连续空间。
- 遍历元素和随机访问效率高。
- 常用方法：
  - `boolean add(Object o)`
  - `void add(int index, Object o)`
  - `int size()`
  - `Object get(int index)`
  - `boolean contains(Object o)`
  - `boolean remove(Object o)`
  - `Object remove(int index)`
- Collection 通用方法：`clear()`、`isEmpty()`、`iterator()`、`toArray()`。

### 3.2 LinkedList
- 链表存储方式，插入、删除效率高。
- 常用方法：
  - `void addFirst(Object o)`
  - `void addLast(Object o)`
  - `Object getFirst()`
  - `Object getLast()`
  - `Object removeFirst()`
  - `Object removeLast()`

### 3.3 ArrayList 与 LinkedList 区别
- ArrayList：连续空间，随机访问/遍历快；插入删除可能移动元素。
- LinkedList：链表，插入删除快；随机访问较慢。
- 选择：查多用 ArrayList，增删多用 LinkedList。

## 4. Set 接口
- 存储唯一、无序对象。
- `HashSet` 是常用实现类。
- Set 存放对象引用。
- 判断对象是否已存在：采用对象 `equals()` 方法比较。
- Set 不存在 `get()` 方法。
- 遍历方式：
  - Iterator：
    - `iterator()`
    - `boolean hasNext()`
    - `Object next()`
  - 增强型 for 循环。

## 5. 新闻管理系统案例
- 需求：存储新闻标题（ID、名称、创建者），获取总数，逐条打印名称。
- 元素个数不确定 → 使用集合类。
- 需要遍历元素，类型为新闻标题。
- 需要头/尾添加删除 → LinkedList。
- 随机访问和遍历 → ArrayList。

## 6. 小结
- 集合框架接口：Collection、List、Set、Map。
- ArrayList 与 LinkedList 异同及应用场合。
- 遍历集合的方法：Iterator、增强 for。
- 泛型集合用法及好处。